+++
date = "2013-01-08T17:53:00.000000+01:00"
title = "Extensions Templates"
tags = ["PostgreSQL", "Extensions", "9.3", "Templates"]
categories = ["PostgreSQL","Extensions"]
thumbnailImage = "/img/old/community.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/community.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2013/01/08-Extensions-Templates",
           "/blog/2013/01/08-Extensions-Templates.html"]
+++

In a recent article titled 
[Inline Extensions](../../2012/12/13-Inline-Extensions.html) we detailed the problem of how
to distribute an extension's 
*package* to a remote server without having
access to its file system at all. The solution to that problem is non
trivial, let's say. But thanks to the awesome 
[PostgreSQL Community](http://www.postgresql.org/community/) we finaly
have some practical ideas on how to address the problem as discussed on
[pgsql-hackers](http://archives.postgresql.org/pgsql-hackers/), our development mailing list.

<center>*PostgreSQL is first an Awesome Community*</center>

The solution we talked about is to use 
*templates*, and so I've been working
on a patch to bring 
*templates for extensions* to PostgreSQL. As we're talking
about 3 new system catalogs, that's a big patch in term of lines of code. In
term of features though, it's quite an easy one.

Here's how it goes. Let's say you want to prepare the system to be able to
`CREATE EXTENSION pair;` without having to install it as an 
*OS package* for
which you would need to get 
`root` access on the server where your PostgreSQL
instance is running, which is not always easy, and sometimes not a good
idea.


## Installing an extension template

With the 
[template patch](http://www.postgresql.org/message-id/m2wqvoha0p.fsf%402ndQuadrant.fr) I just sent on the lists, what you can do is prepare
a template with your extension's script and properties, then use it to
install the extensions.

~~~
create template
   for extension pair default version '1.0'
  with (nosuperuser, norelocatable, schema public)
as $$
  CREATE TYPE pair AS ( k text, v text );
  
  CREATE OR REPLACE FUNCTION pair(anyelement, text)
  RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair';
  
  CREATE OR REPLACE FUNCTION pair(text, anyelement)
  RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair';
  
  CREATE OR REPLACE FUNCTION pair(anyelement, anyelement)
  RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair';
  
  CREATE OR REPLACE FUNCTION pair(text, text)
  RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair;';      
$$;
~~~



## Installing an extension from a template

With the template installed in the catalogs, now you can go and install your
extension:

~~~
foo> create extension pair;
CREATE EXTENSION

foo> \dx pair
     List of installed extensions
 Name | Version | Schema | Description 
------+---------+--------+-------------
 pair | 1.0     | public | 
(1 row)

foo> \dx+ pair
     Objects in extension "pair"
          Object Description          
--------------------------------------
 function pair(anyelement,anyelement)
 function pair(anyelement,text)
 function pair(text,anyelement)
 function pair(text,text)
 type pair
(5 rows)
~~~


The extension installation is now happening from the catalog templates
rather than the file system, which means you didn't need to be 
`root` on the
system where the server is running. Also note that this example above did
happen when connected as the 
*database owner*, a user who is not the
*superuser*. Requiring less privileges is always good news, right?


## Managing upgrade scripts and extension update

Now that the extension is installed, you might want to update it with some
new awesome features. Let's have a look at that.

<center>
{{< image classes="fig50 fancybox dim-margin" src="/img/old/extension-update.png" >}}
</center>

<center>*Upload your Extension Update Scripts*</center>

Rather than make a new version of the extension package with the new files
in there, then asking the operations team to make the new package available
on the internal repositories then install them on the servers, you could now
prepare and 
*QA* the new setup that way:

~~~
create template for extension pair from '1.0' to '1.1'
as $$
  CREATE OPERATOR ~> (LEFTARG = text,
                      RIGHTARG = anyelement,
                      PROCEDURE = pair);
                      
  CREATE OPERATOR ~> (LEFTARG = anyelement,
                      RIGHTARG = text,
                      PROCEDURE = pair);

  CREATE OPERATOR ~> (LEFTARG = anyelement,
                      RIGHTARG = anyelement,
                      PROCEDURE = pair);
                      
  CREATE OPERATOR ~> (LEFTARG = text,
                      RIGHTARG = text,
                      PROCEDURE = pair);           
$$;

create template
   for extension pair from '1.1' to '1.2'
as $$
	    comment on extension pair is 'Simple Key Value Text Type';
$$;
~~~


Of course it's not the most realistic example when you look at the content.
In particular the 
`1.2` version that only adds a comment to the extension. I
needed another version to test the automatic upgrade path with more than one
step though, so here we go.

~~~
foo> alter extension pair update to '1.2';
ALTER EXTENSION

foo> \dx pair
             List of installed extensions
 Name | Version | Schema |        Description         
------+---------+--------+----------------------------
 pair | 1.2     | public | Simple Key Value Text Type
(1 row)

foo> \dx+ pair
     Objects in extension "pair"
          Object Description          
--------------------------------------
 function pair(anyelement,anyelement)
 function pair(anyelement,text)
 function pair(text,anyelement)
 function pair(text,text)
 operator ~>(anyelement,anyelement)
 operator ~>(anyelement,text)
 operator ~>(text,anyelement)
 operator ~>(text,text)
 type pair
(9 rows)
~~~


We did it!


## Internals

Let's have a look at those new catalogs:

<center>
{{< image classes="fig50 fancybox dim-margin" src="/img/old/octopus-anatomy.jpg" >}}
</center>

<center>*Oh, that's not quite the internals I expected...*</center>

Here we go now:

~~~
foo> select * from pg_extension_control;
select * from pg_extension_control;
-[ RECORD 1 ]--+-------
ctlname        | pair
ctlowner       | 32926
ctldefault     | t
ctlrelocatable | f
ctlsuperuser   | f
ctlnamespace   | public
ctlversion     | 1.0
ctlrequires    | 

foo> select * from pg_extension_template;
select * from pg_extension_template;
-[ RECORD 1 ]-----------------------------------------------------------------
tplname    | pair
tplowner   | 32926
tplversion | 1.0
tplscript  | 
           |   CREATE TYPE pair AS ( k text, v text );
           |   
           |   CREATE OR REPLACE FUNCTION pair(anyelement, text)
           |   RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair';
           |   
           |   CREATE OR REPLACE FUNCTION pair(text, anyelement)
           |   RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair';
           |   
           |   CREATE OR REPLACE FUNCTION pair(anyelement, anyelement)
           |   RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair';
           |   
           |   CREATE OR REPLACE FUNCTION pair(text, text)
           |   RETURNS pair LANGUAGE SQL AS 'SELECT ROW($1, $2)::pair;';      
           | 

foo> select * from pg_extension_uptmpl;
select * from pg_extension_uptmpl;
-[ RECORD 1 ]-------------------------------------------------------------
uptname   | pair
uptowner  | 32926
uptfrom   | 1.0
uptto     | 1.1
uptscript | 
          |   CREATE OPERATOR ~> (LEFTARG = text,
          |                       RIGHTARG = anyelement,
          |                       PROCEDURE = pair);
          |                       
          |   CREATE OPERATOR ~> (LEFTARG = anyelement,
          |                       RIGHTARG = text,
          |                       PROCEDURE = pair);
          | 
          |   CREATE OPERATOR ~> (LEFTARG = anyelement,
          |                       RIGHTARG = anyelement,
          |                       PROCEDURE = pair);
          |                       
          |   CREATE OPERATOR ~> (LEFTARG = text,
          |                       RIGHTARG = text,
          |                       PROCEDURE = pair);           
          | 
-[ RECORD 2 ]-------------------------------------------------------------
uptname   | pair
uptowner  | 32926
uptfrom   | 1.1
uptto     | 1.2
uptscript | 
          |     comment on extension pair is 'Simple Key Value Text Type';
          | 
~~~


As you can see there's nothing too complex here, it's quite straightforward.
We need to separate away the 
*creating* templates from the 
*updating* templates
because we need 
***unique*** keys and we can't have that on 
`NULL` columns.

~~~
foo> \d pg_extension_template
\d pg_extension_template
Table "pg_catalog.pg_extension_template"
   Column   | Type | Modifiers 
------------+------+-----------
 tplname    | name | not null
 tplowner   | oid  | not null
 tplversion | text | 
 tplscript  | text | 
Indexes:
    "pg_extension_template_name_version_index" UNIQUE, btree (tplname, tplversion)
    "pg_extension_template_oid_index" UNIQUE, btree (oid)

foo> \d pg_extension_uptmpl
\d pg_extension_uptmpl
Table "pg_catalog.pg_extension_uptmpl"
  Column   | Type | Modifiers 
-----------+------+-----------
 uptname   | name | not null
 uptowner  | oid  | not null
 uptfrom   | text | 
 uptto     | text | 
 uptscript | text | 
Indexes:
    "pg_extension_uptmpl_name_from_to_index" UNIQUE, btree (uptname, uptfrom, uptto)
    "pg_extension_uptmpl_oid_index" UNIQUE, btree (oid)
~~~



## Next steps

Now that we have the basics in place, the patch is far from finished still.
It needs 
`pg_dump` and 
`psql` support, support for the function
`pg_available_extension_versions()`, implementing some 
`ALTER TEMPLATE FOR
EXTENSION` commands for which I only sketched the syntax in the grammar, and
some more infrastructure to be able to have 
`ALTER OWNER` and 
`ALTER RENAME`
commands.

<center>
{{< image classes="fig50 fancybox dim-margin" src="/img/old/patch-brewing.jpg" >}}
</center>

<center>*Warning: patch brewing here! Syntax and other key elements will change.*</center>

All that is pretty technical though, the real thing that patch needs is some
quality review and maybe some adjustments. I would be surprised if it didn't
need adjustments, really. Because the way the community works, we always
need some. That's why the PostgreSQL product is so good!
