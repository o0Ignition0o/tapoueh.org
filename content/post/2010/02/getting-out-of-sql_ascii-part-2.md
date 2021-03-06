+++
date = "2010-02-23T17:30:00.000000+01:00"
title = "Getting out of SQL_ASCII, part 2"
tags = ["PostgreSQL", "plpgsql", "Encoding"]
categories = ["PostgreSQL","Encoding"]
thumbnailImage = "/img/encodings.png"
thumbnailImagePosition = "left"
coverImage = "/img/encodings.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/02/23-getting-out-of-sql_ascii-part-2",
           "/blog/2010/02/23-getting-out-of-sql_ascii-part-2.html"]
+++

So, if you followed the previous blog entry, now you have a new database
containing all the 
*static* tables encoded in 
`UTF-8` rather than
`SQL_ASCII`. Because if it was not yet the case, you now severely distrust
this non-encoding.

Now is the time to have a look at properly encoding the 
*live* data, those
stored in tables that continue to receive write traffic. The idea is to use
the 
`UPDATE` facilities of PostgreSQL to tweak the data, and too fix the
applications so as not to continue inserting badly encoded strings in there.


## Finding non UTF-8 data

First you want to find out the badly encoded data. You can do that with this
helper function that 
[RhodiumToad](http://blog.rhodiumtoad.org.uk/) gave me on IRC. I had a version from the
archives before that, but the 
*regexp* was hard to maintain and quote into a
`PL` function. This is avoided by two means, first one is to have a separate
pure 
`SQL` function for the 
*regexp* checking (so that you can index it should
you need to) and the other one is to apply the regexp to 
`hex` encoded
data. Here we go:

~~~
create or replace function public.utf8hex_valid(str text) 
 returns boolean
 language sql immutable
as $f$
   select $1 ~ $r$(?x)
                  ^(?:(?:[0-7][0-9a-f])
                     |(?:(?:c[2-9a-f]|d[0-9a-f])
                        |e0[ab][0-9a-f]
                        |ed[89][0-9a-f]
                        |(?:(?:e[1-9abcef])
                           |f0[9ab][0-9a-f]
                           |f[1-3][89ab][0-9a-f]
                           |f48[0-9a-f]
                          )[89ab][0-9a-f]
                       )[89ab][0-9a-f]
                    )*$
                $r$;
$f$;
~~~


Now some little scripting around it in order to skip intense manual and
boring work (and see, some more catalog queries). Don't forget we will have
to work on a per-column basis here...

~~~
create or replace function public.check_encoding_utf8
 (
   IN schemaname text,
   IN tablename  text,
  OUT relname    text,
  OUT attname    text,
  OUT count      bigint
 )
 returns setof record
 language plpgsql
as $f$
DECLARE
  v_sql text;
BEGIN
  FOR relname, attname
   IN SELECT c.relname, a.attname 
        FROM pg_attribute a 
             JOIN pg_class c on a.attrelid = c.oid
             JOIN pg_namespace s on s.oid = c.relnamespace 
	     JOIN pg_roles r on r.oid = c.relowner
       WHERE s.nspname = schemaname
         AND atttypid IN (25, 1043) -- text, varchar
         AND relkind = 'r'          -- ordinary table
         AND r.rolname = 'some_specific_role'
	 AND CASE WHEN tablename IS NOT NULL
	     	  THEN c.relname ~ tablename
		  ELSE true
	      END
  LOOP
    v_sql := 'SELECT count(*) '
          || '  FROM ONLY '|| schemaname || '.' || relname 
          || ' WHERE NOT public.utf8hex_valid(encode(textsend(' 
          || attname
          || '), ''hex''))';

    -- RAISE NOTICE 'Checking: %.%', relname, attname;
    -- RAISE NOTICE 'SQL: %', v_sql;
    EXECUTE v_sql INTO count;
    RETURN NEXT;
  END LOOP;
END;
$f$; 
~~~


Note that the 
`tablename` is compared using the 
`~` operator, so that's 
*regexp*
matching there too. Also note that I wanted only to check those tables that
are owned by a specific role, your case may vary.

The way I used this function was like this:

~~~
create table leon.check_utf8 as
 select * 
   from public.check_encoding_utf8();
~~~


Then you need to take action on those lines in 
`leon.check_utf8` table which
have a 
`count > 0`. Rince and repeat, but you may soon realise building the
table over and over again is costly.


## Cleaning up the data

Up for some more helper tools? Unless you really want to manually fix this
huge amount of columns where some data ain't 
`UTF-8` compatible... here's some
more:

~~~
create or replace function leon.nettoyeur
 (
  IN  action      text,
  IN  encoding    text,
  IN  tablename   text,
  IN  columname   text,

  OUT orig        text,
  OUT utf8        text
 )
 returns setof record
 language plpgsql
as $f$
DECLARE
  p_convert text;
BEGIN
  IF encoding IS NULL
  THEN
    p_convert := 'translate(' 
              || columname || ', ' 
              || $$'\211\203\202'$$ 
              || ', '
              || $$'   '$$
	      || ') ';
  ELSE
    -- in 8.2, write convert using, in 8.3, the other expression
    -- p_convert := 'convert(' || columname || ' using ' || conversion || ') ';
    p_convert := 'convert(textsend(' || columname || '), '''|| encoding ||''', ''utf-8'' ) ';
  END IF;

  IF action = 'select'
  THEN
    FOR orig, utf8
     IN EXECUTE 'SELECT ' || columname || ', '
         || p_convert
         || '  FROM ONLY ' || tablename
         || ' WHERE not public.utf8hex_valid('
         || 'encode(textsend('|| columname ||'), ''hex''))'
    LOOP
      RETURN NEXT;
    END LOOP;

  ELSIF action = 'update'
  THEN
    EXECUTE 'UPDATE ONLY ' || tablename 
         || ' SET ' || columname || ' = ' || p_convert
         || ' WHERE not public.utf8hex_valid('
         || 'encode(textsend('|| columname ||'), ''hex''))';

    FOR orig, utf8 
     IN SELECT * 
          FROM leon.nettoyeur('select', encoding, tablename, columname)
    LOOP
      RETURN NEXT;
    END LOOP;

  ELSE
    RAISE EXCEPTION 'L&#xE9;on, Nettoyeur, veut de l''action.';

  END IF;
END;
$f$;
~~~


As you can see, this function allows to check the conversion process from a
given supposed encoding before to actually convert the data in place. This
is very useful as even when you're pretty sure the non-utf8 data is 
`latin1`,
sometime you find it's 
`windows-1252` or such. So double check before telling
`leon.nettoyeur()` to update your precious data!

Also, there's a facility to use 
`translate()` when none of the encoding match
your expectations. This is a skeleton just replacing invalid characters with
a 
`space`, tweak it at will!


## Conclusion

Enjoy your clean database now, even if it still accepts new data that will
probably not pass the checks, so we still have to be careful about that and
re-clean every day until the migration is effective. Or maybe add a 
`CHECK`
clause that will reject badly encoded data...

In fact here we're using 
[Londiste](http://wiki.postgresql.org/wiki/Londiste_Tutorial) to replicate the 
*live* data from the old to
the new server, and that means the replication will break each time there's
new data written in non-utf8, as the new server is running 
`8.4`, which by
design ain't very forgiving. Our plan is to clean-up as we go (remove table
from the 
*subscriber*, fix it, add it again) and migrate as soon as possible!

Bonus points to those of you getting the convoluted reference :)
