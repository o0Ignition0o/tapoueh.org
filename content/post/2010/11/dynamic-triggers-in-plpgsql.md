+++
date = "2010-11-24T16:45:00.000000+01:00"
title = "Dynamic Triggers in PLpgSQL"
tags = ["plpgsql", "YeSQL"]
categories = ["PostgreSQL","YeSQL"]
thumbnailImage = "/img/old/dynamic_trigger.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/dynamic_trigger.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/11/24-dynamic-triggers-in-plpgsql",
           "/blog/2010/11/24-dynamic-triggers-in-plpgsql.html"]
+++

You certainly know that implementing 
*dynamic* triggers in 
`PLpgSQL` is
impossible. But I had a very bad night, being up from as soon as 3:30 am
today, so that when a developer asked me about reusing the same trigger
function code from more than one table and for a dynamic column name, I
didn't remember about it being impossible.

Here's what happens in such cases, after a long time on the problem (yes,
overall, that's a slow day). Note that I'm abusing the 
`(record_literal).*`
notation a lot in there, and even the 
`(record_literal).column_name` too.

~~~
CREATE OR REPLACE FUNCTION public.update_timestamp()
 RETURNS TRIGGER
 LANGUAGE plpgsql
AS $f$
DECLARE
    ts_column varchar;
    old_timestamp timestamptz;
    attname name;
    n text;
    v text;
BEGIN
    IF TG_NARGS != 1
    THEN
        RAISE EXCEPTION 'Trigger public.update_timestamp() called with % args',
                         TG_NARGS;
    END IF;

    ts_column := TG_ARGV[0];

    EXECUTE 'SELECT n.' || ts_column
         || ' FROM (SELECT (' 
         || quote_literal(OLD) || '::' || TG_RELID::regclass
         || ').*) as n'
       INTO old_timestamp;

    -- build NEW record text
    n := '(';
    FOR attname IN
      EXECUTE 'SELECT attname '
           || '  FROM pg_class c left join pg_attribute a on a.attrelid = c.oid'
           || ' WHERE c.oid = $1 and attnum > 0 order by attnum'
       USING TG_RELID
    LOOP
        EXECUTE 'SELECT (' || quote_literal(NEW) || '::' || TG_RELID::regclass || ').' || attname INTO v;

        IF n != '(' THEN n := n || ','; END IF;

        IF attname = ts_column 
           AND v::timestamptz IS NOT DISTINCT FROM old_timestamp
        THEN
                n := n || now();
        ELSE
                n := n || COALESCE(v, '');              
        END IF;
    END LOOP;
    n := n || ')';

    EXECUTE 'SELECT ($1::' || TG_RELID::regclass || ').*'
      INTO NEW
     USING n;

    RETURN NEW;
END;
$f$;
~~~


It's not pretty, and not fast. It's about 
`2 ms` per call on a table with 
`15`
columns, in some preliminary tests. But it sure was a nice challenge!
