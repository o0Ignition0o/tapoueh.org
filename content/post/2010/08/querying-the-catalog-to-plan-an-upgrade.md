+++
date = "2010-08-05T11:00:00.000000+02:00"
title = "Querying the Catalog to plan an upgrade"
tags = ["PostgreSQL", "release", "catalogs"]
categories = ["PostgreSQL","Catalogs"]
thumbnailImage = "/img/old/library-card-catalogs.small.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/library-card-catalogs.small.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/08/05-querying-the-catalog-to-plan-an-upgrade",
           "/blog/2010/08/05-querying-the-catalog-to-plan-an-upgrade.html"]
+++

Some user on 
`IRC` was reading the releases notes in order to plan for a minor
upgrade of his 
`8.3.3` installation, and was puzzled about potential needs for
rebuilding 
`GIST` indexes. That's from the 
[8.3.5 release notes](http://www.postgresql.org/docs/8.3/static/release-8-3-5.html), and from the
[8.3.8 notes](http://www.postgresql.org/docs/8.3/static/release-8-3-8.html) you see that you need to consider 
*hash* indexes on 
*interval*
columns too. Now the question is, how to find out if any such beasts are in
use in your database?

It happens that 
[PostgreSQL](http://www.postgresql.org/) is letting you know those things by querying its
[system catalogs](http://www.postgresql.org/docs/8.4/static/catalogs.html). That might look hairy at first, but it's very worth getting
used to those system tables. You could compare that to introspection and
reflexive facilities of some programming languages, except much more useful,
because you're reaching all the system at once. But, well, here it goes:

~~~
SELECT schemaname, tablename, relname, amname, indexdef
  FROM pg_indexes i 
       JOIN pg_class c ON i.indexname = c.relname and c.relkind = 'i' 
       JOIN pg_am am ON c.relam = am.oid
 WHERE amname = 'gist';
~~~


Now you could replace the 
`WHERE` clause with 
`WHERE amname IN ('gist', 'hash')`
to check both conditions at once. What about pursuing the restriction on the
*hash* indexes rebuild to schedule, as they should only get done to indexes on
`interval` columns. Well let's try it:

~~~
SELECT schemaname, tablename, relname as indexname, amname, indclass
  FROM pg_indexes i 
       JOIN pg_class c on i.indexname = c.relname and c.relkind = 'i' 
       JOIN pg_am am on c.relam = am.oid 
       JOIN pg_index x on x.indexrelid = c.oid 
 WHERE amname in ('btree', 'gist') 
       and schemaname not in ('pg_catalog', 'information_schema');
~~~


We're not there yet, because as you notice, the catalogs are somewhat
optimized and not always in a normal form. That's good for the system's
performance, but it makes querying a bit uneasy. What we want is to get from
the 
`indclass` column if there's any of them (it's an 
`oidvector`) that applies
to an 
`interval` data type. There's a subtlety here as the index could store
`interval` data even if the column is not of an 
`interval` type itself, so we
have to find both cases.

Well the 
*subtlety* applies after you know what an 
[operator class](http://www.postgresql.org/docs/8.4/static/xindex.html) is: 
*“An
operator class defines how a particular data type can be used with an
index”* is what the 
[CREATE OPERATOR CLASS](http://www.postgresql.org/docs/8.4/static/sql-createopclass.html) manual page teaches us. What we
need to know here is that an index will talk to an operator class to get to
the data type, either the 
*column* data type or the index 
*storage* one.

~~~
SELECT schemaname, tablename, relname as indexname, amname, indclass, opcname, typname
  FROM pg_indexes i 
       JOIN pg_class c on i.indexname = c.relname and c.relkind = 'i' 
       JOIN pg_am am on c.relam = am.oid 
       JOIN pg_index x on x.indexrelid = c.oid 
       JOIN pg_opclass o 
         on string_to_array(x.indclass::text, ' ')::oid[] @> array[o.oid]::oid[]
       JOIN pg_type t on o.opckeytype = t.oid
WHERE amname = 'hash' and t.typname = 'interval'

UNION ALL

SELECT schemaname, tablename, relname as indexname, amname, indclass, opcname, typname
  FROM pg_indexes i 
       JOIN pg_class c on i.indexname = c.relname and c.relkind = 'i' 
       JOIN pg_am am on c.relam = am.oid 
       JOIN pg_index x on x.indexrelid = c.oid 
       JOIN pg_opclass o 
         on string_to_array(x.indclass::text, ' ')::oid[] @> array[o.oid]::oid[]
       JOIN pg_type t on o.opcintype = t.oid
WHERE amname = 'hash' and t.typname = 'interval';
~~~


Most certainly this query will return no row for you, as 
*hash* indexes are
not widely used, mainly because they are not crash tolerant. For seeing some
results you could remove the 
`amname` restriction of course, that would show
the query is working, but don't forget to add the restriction back to plan
for the upgrade!

But hey, why walking the extra mile here, would you ask me? After all, in
the second query we would already have had the information we needed should
we added the 
`indexdef` column, albeit in a human reader friendly way: the
*resultset* would then contain the 
`CREATE INDEX` command you need to issue to
build the index from scratch. That would be enough for checking only the
catalog, but the extra mile allows you to produce a 
`SQL` script to build the
indexes that need your attention post upgrade. That last step is left as an
exercise for the reader, though.
