+++
date = "2010-08-09T14:45:00.000000+02:00"
title = "Editing constants in constraints"
tags = ["PostgreSQL", "plpgsql", "Catalogs"]
categories = ["PostgreSQL","Catalogs"]
thumbnailImage = "/img/old/library-card-catalogs.small.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/library-card-catalogs.small.jpg"
coverSize = "partial"
coverMeta = "in"
aliases = ["/blog/2010/08/09-editing-constants-in-constraints",
           "/blog/2010/08/09-editing-constants-in-constraints.html"]
+++

We're using constants in some constraints here, for example in cases where
several servers are replicating to the same 
*federating* one: each origin
server has his own schema, and all is replicated nicely on the central host,
thanks to 
[Londiste](http://wiki.postgresql.org/wiki/Londiste_Tutorial#Federated_database), as you might have guessed already.

<!--more-->

For bare-metal recovery scripts, I'm working on how to change those
constants in the constraints, so that 
`pg_dump -s` plus some schema tweaking
would kick-start a server. Here's a 
`PLpgSQL` snippet to do just that:

~~~ plpgsql
FOR rec IN EXECUTE
$s$
SELECT schemaname, tablename, conname, attnames, def
  FROM (
   SELECT n.nspname, c.relname, r.conname, 
          (select array_accum(attname)
             from pg_attribute 
            where attrelid = c.oid and r.conkey @> array[attnum]) as attnames, 
          pg_catalog.pg_get_constraintdef(r.oid, true)
   FROM pg_catalog.pg_constraint r 
        JOIN pg_class c on c.oid = r.conrelid 
        JOIN pg_namespace n ON n.oid = c.relnamespace
   WHERE r.contype = 'c'
ORDER BY 1, 2, 3
       ) as cons(schemaname, tablename, conname, attnames, def)
WHERE attnames @> array['server']::name[]
$s$
  LOOP
    rec.def := replace(rec.def, 'server = ' || old_id,
                                'server = ' || new_id);

    sql := 'ALTER TABLE ' || rec.schemaname || '.' || rec.tablename
        || ' DROP CONSTRAINT ' || rec.conname;
    RAISE NOTICE '%', sql;
    RETURN NEXT;
    EXECUTE sql;

    sql := 'ALTER TABLE ' || rec.schemaname || '.' || rec.tablename
        || ' ADD ' || rec.def;
    RAISE NOTICE '%', sql;
    RETURN NEXT;
    EXECUTE sql;

  END LOOP;
~~~


This relies on the fact that our constraints are on the column 
`server`. Why
would this be any better than a 
`sed` one-liner, would you ask me? I'm fed up
with having pseudo-parsing scripts and taking the risk that the simple
command will change data I didn't want to edit. I want context aware tools,
pretty please, to 
*feel* safe.

Otherwise I'd might have gone with

~~~ bash
pg_dump -s| sed -e 's:\(server =\) 17:\1 18:'
~~~

but this one-liner already contains too much useless magic for my taste (the
space before *17* ain't in the group match to allow for having *\1 18* in
the right hand side. And this isn't yet parametrized, and there I'll need to
talk to the database, as that's were I store the servers name and their id
(a *bigserial* — yes, the constraints are all generated from scripts). I
don't want to write an *SQL parser* and I don't want to play loose, so the
*PLpgSQL* approach is what I'm thinking as the best tool here. Opinionated
answers get to my mailbox!
