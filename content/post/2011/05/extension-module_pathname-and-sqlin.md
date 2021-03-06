+++
date = "2011-05-02T17:30:00.000000+02:00"
title = "Extension module_pathname and .sql.in"
tags = ["PostgreSQL", "debian", "Extensions", "release", "prefix", "9.1"]
categories = ["Projects","prefix"]
thumbnailImage = "/img/old/debian-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/debian-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2011/05/02-extension-module_pathname-and-sqlin",
           "/blog/2011/05/02-extension-module_pathname-and-sqlin.html"]
+++

While currently too busy at work to deliver much Open Source contributions,
let's debunk an old habit of 
[PostgreSQL](http://www.postgresql.org/) extension authors.  It's all down to
copy pasting from 
*contrib*, and there's no reason to continue doing 
`$libdir`
this way ever since 
`7.4` days.

Let's take an example here, with the 
[prefix](https://github.com/dimitri/prefix) extension.  This one too will
need some love, but is still behind on my spare time todo list, sorry about
that.  So, in the 
`prefix.sql.in` we read

~~~
CREATE OR REPLACE FUNCTION prefix_range_in(cstring)
  RETURNS prefix_range
  AS 'MODULE_PATHNAME'
  LANGUAGE 'C' IMMUTABLE STRICT;
~~~


Two things are to change here.  First, the PostgreSQL 
*backend* will
understand just fine if you just say 
`AS '$libdir/prefix'`.  So you have to
know in the 
`sql` script the name of the shared object library, but if you do,
you can maintain directly a 
`prefix.sql` script instead.

The advantage is that you now can avoid a compatibility problem when you
want to support PostgreSQL from 
`8.2` to 
`9.1` in your extension (older than
that and it's 
[no longer supported](http://wiki.postgresql.org/wiki/PostgreSQL_Release_Support_Policy)).  You directly ship your script.

For compatibility, you could also use the 
[control file](http://developer.postgresql.org/pgdocs/postgres/extend-extensions.html) 
`module_pathname`
property.  But for 
`9.1` you then have to add a implicit 
`Make` rule so that the
script is derived from your 
`.sql.in`. And as you are managing several scripts
— so that you can handle 
*versioning* and 
*upgrades* — it can get hairy (
*hint*,
you need to copy 
`prefix.sql` as 
`prefix--1.1.1.sql`, then change its name at
next revision, and think about 
*upgrade* scripts too).  The 
`module_pathname`
facility is better to keep for when managing more than a single extension in
the same directory, like the 
[SPI contrib](http://git.postgresql.org/gitweb?p=postgresql.git;a=blob;f=contrib/spi/Makefile;h=0c11bfcbbd47b0c3ed002874bfefd9e2022cf5ac;hb=HEAD) is doing.

Sure, maintaining an extension that targets both antique releases of
PostgreSQL and 
[CREATE EXTENSION](http://developer.postgresql.org/pgdocs/postgres/sql-createextension.html) super-powered one(s) (not yet released) is a
little more involved than that.  We'll get back to that, as some people are
still pioneering the movement.

On my side, I'm working with some 
[debian](http://www.debian.org/) 
[developer](http://qa.debian.org/developer.php?login=myon) on how to best manage the
packaging of those extensions, and this work could end up as a specialized
*policy* document and a coordinated 
*team* of maintainers for all things
PostgreSQL in 
`debian`.  This will also give some more steam to the PostgreSQL
effort for debian packages: the idea is to maintain packages for all
supported version (from 
`8.2` up to soon 
`9.1`), something 
`debian` itself can not
commit to.
