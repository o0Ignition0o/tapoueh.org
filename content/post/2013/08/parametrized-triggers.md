+++
date = "2013-08-23T12:08:00.000000+02:00"
title = "Trigger Parameters"
tags = ["PostgreSQL", "Triggers", "hstore", "Extensions", "YeSQL"]
categories = ["PostgreSQL","YeSQL"]
thumbnailImage = "/img/old/dunlop-trigger-84fg-gold.640.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/dunlop-trigger-84fg-gold.640.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2013/08/23-parametrized-triggers",
           "/blog/2013/08/23-parametrized-triggers.html"]
+++

Sometimes you want to compute values automatically at 
`INSERT` time, like for
example a 
*duration* column out of a 
*start* and an 
*end* column, both
*timestamptz*. It's easy enough to do with a 
`BEFORE TRIGGER` on your table.
What's more complex is to come up with a parametrized spelling of the
trigger, where you can attach the same 
*stored procedure* to any table even
when the column names are different from one another.

<!--more-->
<!--toc-->

The exact problem to solve here is how to code a 
*dynamic trigger* where the
trigger's function code doesn't have to hard code the field names it will
process. Basically, 
[PLpgSQL](http://www.postgresql.org/docs/9.2/static/plpgsql-overview.html) is a static language and wants to know all about
the function data types in use before it 
*compiles* it, so there's no easy way
to do that.

That said, we now have 
[hstore](http://www.postgresql.org/docs/9.2/static/hstore.html) and it's empowering us a lot here.


# The exemple

Let's start simple, with a table having a 
`d_start` and a 
`d_end` column where
to store, as you might have already guessed, a start timestamp (with
timezone) and an ending timezone. The goal will be to have a parametrized
trigger able to maintain a 
`duration` for us automatically, something we
should be able to reuse on other tables.

~~~
create table foo (
  id serial primary key,
  d_start timestamptz default now(),
  d_end timestamptz,
  duration interval
);

insert into foo(d_start, d_end)
     select now() - 10 * random() * interval '1 min',
            now() + 10 * random() * interval '1 min'
       from generate_series(1, 10);
~~~


So now I have a table with 10 lines containing random timestamps, but none
of them of course has the 
`duration` field set. Let's see about that now.


# Playing with hstore

The 
*hstore* extension is full of goodies, we will only have to discover a
handful of them now.

First thing to do is make 
`hstore` available in our test database:

~~~ sql
# create extension hstore;
CREATE EXTENSION
~~~


And now play with 
*hstore* in our table.

~~~ sql
> select hstore(foo) from foo limit 1;

 "id"=>"1",
 "d_end"=>"2013-08-23 11:34:53.129109+01",
 "d_start"=>"2013-08-23 11:16:04.869424+01",
 "duration"=>NULL
(1 row)
~~~


I edited the result for it to be easier to read, splitting it on more than
one line, so if you try that at home you will have a different result.

What's happening in that first example is that we are transforming a 
*row
type* into a value of type 
*hstore*. A 
*row type* is the result of 
`select foo
from foo;`. Each PostgreSQL relation defines a type of the same name, and you
can use it as a 
*composite type* if you want to.

Now, hstore also provides the 
`#=` operator which will replace a
given field in a row, look at that:

~~~ sql
> select (foo #= hstore('duration', '10 mins')).*
    from foo
   limit 1;

 id |            d_start            |             d_end             | duration 
----+-------------------------------+-------------------------------+----------
  1 | 2013-08-23 11:16:04.869424+01 | 2013-08-23 11:34:53.129109+01 | 00:10:00
(1 row)
~~~


We just replaced the 
`duration` field with the value 
`10 mins`, and to have a
better grasp at what just happened, we then use the 
`(...).*` notation to
expand the row type into its full definition.

We should be ready for the next step now...


# The generic trigger, using hstore

Now let's code the trigger:

~~~ sql
create or replace function tg_duration()
 -- (
 --  start_name    text,
 --  end_name      text,
 --  duration      interval
 -- )
 returns trigger
 language plpgsql
as $$
declare
   hash hstore := hstore(NEW);
   duration interval;
begin
   duration :=  (hash -> TG_ARGV[1])::timestamptz
              - (hash -> TG_ARGV[0])::timestamptz;

   NEW := NEW #= hstore(TG_ARGV[2], duration::text);

   RETURN NEW;
end;
$$;
~~~


And here's how to attach the trigger to our table. Don't forget the 
`FOR EACH
ROW` part or you will have a hard time understanding why you can't accedd the
details of the 
`OLD` and 
`NEW` records in your trigger: they default to being
*FOR EACH STATEMENT* triggers.

The other important point is how we pass down the column names as argument
to the stored procedure above.

~~~ sql
create trigger compute_duration
     before insert on foo
          for each row
 execute procedure tg_duration('d_start', 'd_end', 'duration');
~~~


Equiped with the trigger properly attached to our table, we can truncate it
and insert again some rows:

~~~ sql
> truncate foo;
> insert into foo(d_start, d_end)
       select now() - 10 * random() * interval '1 min',
              now() + 10 * random() * interval '1 min'
         from generate_series(1, 10);

> select d_start, d_end, duration from foo;

            d_start            |             d_end             |    duration     
-------------------------------+-------------------------------+-----------------
 2013-08-23 11:56:20.185563+02 | 2013-08-23 12:00:08.188698+02 | 00:03:48.003135
 2013-08-23 11:51:10.933982+02 | 2013-08-23 12:02:08.661389+02 | 00:10:57.727407
 2013-08-23 11:59:44.214844+02 | 2013-08-23 12:00:57.852027+02 | 00:01:13.637183
 2013-08-23 11:50:18.931533+02 | 2013-08-23 12:00:52.752111+02 | 00:10:33.820578
 2013-08-23 11:53:18.811819+02 | 2013-08-23 12:06:30.419106+02 | 00:13:11.607287
 2013-08-23 11:56:33.933842+02 | 2013-08-23 12:01:15.158055+02 | 00:04:41.224213
 2013-08-23 11:57:26.881887+02 | 2013-08-23 12:05:53.724116+02 | 00:08:26.842229
 2013-08-23 11:54:10.897691+02 | 2013-08-23 12:06:27.528534+02 | 00:12:16.630843
 2013-08-23 11:52:17.22929+02  | 2013-08-23 12:02:08.647837+02 | 00:09:51.418547
 2013-08-23 11:58:18.20224+02  | 2013-08-23 12:07:11.170435+02 | 00:08:52.968195
(10 rows)
~~~



# Conclusion

Thanks to the 
[hstore](http://www.postgresql.org/docs/current/static/hstore.html) extension we've been able to come up with a dynamic
solution where you can give the name of the columns you want to work with at
`CREATE TRIGGER` time rather than hard-code that in a series of stored
procedure that will end up alike and a pain to maintain.
