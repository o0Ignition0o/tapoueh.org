+++
date = "2009-01-28T00:00:00.000000+01:00"
title = "Controling HOT usage in 8.3"
tags = ["PostgreSQL", "Catalogs"]
categories = ["PostgreSQL","Catalogs"]
thumbnailImage = "/img/old/library-card-catalogs.small.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/library-card-catalogs.small.jpg"
coverSize = "partial"
coverMeta = "in"
aliases = ["/blog/2009/01/28-controling-hot-usage-in-83",
           "/blog/2009/01/28-controling-hot-usage-in-83.html"]
+++

As it happens, I've got some environments where I want to make sure *HOT* (
*aka Heap Only Tuples*) is in use. Because we're doing so much updates a
second that I want to get sure it's not killing my database server. I not
only wrote some checking view to see about it, but also made
a
[quick article](http://www.postgresql.fr/support:trucs_et_astuces:controler_l_utilisation_de_hot_a_partir_de_la_8.3) about
it in the [French PostgreSQL website](http://postgresql.fr/). Handling
around in `#postgresql` means that I'm now bound to write about it in
English too!

<!--more-->

So *HOT* will get used each time you update a row without changing an
indexed value of it, and the benefit is skipping index maintenance, and as
far as I understand it, easying *VACUUM* hard work too. To get the benefit,
*HOT* will need some place where to put new version of the *updated* tuple
in the same disk page, which means you'll probably want to set your
table
[fillfactor](http://www.postgresql.org/docs/8.3/static/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS) to
something much less than 100.

Now, here's how to check you're benefitting from *HOT*:

~~~ sql
SELECT schemaname, relname,
       n_tup_upd,n_tup_hot_upd,
       case when n_tup_upd > 0
            then (( n_tup_hot_upd::numeric
                   /n_tup_upd::numeric
                  )*100.0
                 )::numeric(5,2) 
            else NULL
       end AS hot_ratio
 
 FROM pg_stat_all_tables;
~~~

Here we have:

~~~ psql
 schemaname | relname | n_tup_upd | n_tup_hot_upd | hot_ratio
------------+---------+-----------+---------------+-----------
 public     | table1  |         6 |             6 |    100.00
 public     | table2  |   2551200 |       2549474 |     99.93
~~~


Here's even an extended version of the same request, displaying the
`fillfactor` option value for the tables you're inquiring about. This comes
separated from the first example because you get the `fillfactor` of a
relation into the `pg_class` catalog `reloptions` field, and to filter
against a schema qualified table name, you want to join against
`pg_namespace` too.

~~~ sql
SELECT t.schemaname, t.relname, c.reloptions, 
       t.n_tup_upd, t.n_tup_hot_upd, 
       case when n_tup_upd > 0 
            then (( n_tup_hot_upd::numeric
                   /n_tup_upd::numeric
                  )*100.0
                 )::numeric(5,2) 
            else NULL
        end AS hot_ratio
 FROM pg_stat_all_tables t 
      JOIN (pg_class c
                JOIN pg_namespace n
                  ON c.relnamespace = n.oid) 
        ON n.nspname = t.schemaname
       AND c.relname = t.relname
~~~

And this time:

~~~ psql
 schemaname | relname |   reloptions    | n_tup_upd | n_tup_hot_upd | hot_ratio
------------+---------+-----------------+-----------+---------------+-----------
 public     | table1  | {fillfactor=50} |   1585920 |       1585246 |     99.96
 public     | table2  | {fillfactor=50} |   2504880 |       2503154 |     99.93
~~~


Don't let the *HOT* question affect your sleeping no more!
