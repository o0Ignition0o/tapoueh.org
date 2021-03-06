+++
date = "2011-06-29T09:50:00.000000+02:00"
title = "Multi-Version support for Extensions"
tags = ["PostgreSQL", "debian", "Extensions", "release", "ip4r", "9.1"]
categories = ["debian"]
thumbnailImage = "/img/old/debian-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/debian-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2011/06/29-multi-version-support-for-extensions",
           "/blog/2011/06/29-multi-version-support-for-extensions.html"]
+++

We still have this problem to solve with extensions and their packaging.
How to best organize things so that your extension is compatible with before
`9.1` and 
`9.1` and following releases of 
[PostgreSQL](http://www.postgresql.org/)?

Well, I had to do it for the 
[ip4r](http://pgfoundry.org/projects/ip4r/) contribution, and I wanted the following
to happen:

~~~
dpkg-deb: building package `postgresql-8.3-ip4r' ...
dpkg-deb: building package `postgresql-8.4-ip4r' ...
dpkg-deb: building package `postgresql-9.0-ip4r' ...
dpkg-deb: building package `postgresql-9.1-ip4r' ...
~~~


And here's a simple enough way to achieve that.  First, you have to get your
packaging ready the usual way, and to install the build dependencies.  Then
realizing that 
`/usr/share/postgresql-common/supported-versions` from the
latest 
`postgresql-common` package will only return 
`8.3` in 
`lenny` (yes, I'm
doing some 
*backporting* here), we have to tweak it.

~~~
postgresql-server-dev-8.4
postgresql-server-dev-9.0
postgresql-server-dev-9.1
postgresql-server-dev-all

$ sudo dpkg-divert \
--divert /usr/share/postgresql-common/supported-versions.distrib \
--rename /usr/share/postgresql-common/supported-versions

$ cat /usr/share/postgresql-common/supported-versions
#! /bin/bash

dpkg -l postgresql-server-dev-* \
| awk -F '[ -]' '/^ii/ && ! /server-dev-all/ {print $6}'
~~~


Now we are allowed to build our extension for all those versions, so we add
`9.1` to the 
`debian/pgversions` file.  And 
`debuild` will do the right thing now,
thanks to 
[pg_buildext](http://manpages.debian.net/cgi-bin/man.cgi?query=pg_buildext) from 
[postgresql-server-dev-all](http://packages.debian.org/sid/postgresql-server-dev-all).

The problem we face is that the built is not an 
[extension](http://www.postgresql.org/docs/9.1/static/extend-extensions.html) as in 
`9.1`, so
things like 
`\dx` in 
`psql` and 
[CREATE EXTENSION](http://www.postgresql.org/docs/9.1/static/sql-createextension.html) will not work out of the box.
First, we need a control file.  Then we need to remove the transaction
control from the install script (here, 
`ip4r.sql`), and finally, this script
needs to be called 
`ip4r--1.05.sql`.  Here's how I did it:

~~~
$ cat ip4r.control
comment = 'IPv4 and IPv4 range index types'
default_version = '1.05'
relocatable = yes

$ cat debian/postgresql-9.1-ip4r.install
debian/ip4r-9.1/ip4r.so usr/lib/postgresql/9.1/lib
ip4r.control usr/share/postgresql/9.1/extension
debian/ip4r-9.1/ip4r.sql usr/share/postgresql/9.1/extension

$ cat debian/postgresql-9.1-ip4r.links
usr/share/postgresql/9.1/extension/ip4r.sql usr/share/postgresql/9.1/extension/ip4r--1.05.sql
~~~


Be careful not to forget to remove any and all 
`BEGIN;` and 
`COMMIT;` lines from
the 
`ip4r.sql` file, which meant that I also removed support for 
*Rtree*, which
is not relevant for modern versions of PostgreSQL saith the script (post
`8.2`).  That means I'm not publishing this very work yet, but I wanted to
share the 
`debian/postgresql-9.1-extension.links` idea.

Notice that I didn't change anything about the 
`.sql.in` make rule, so I
didn't have to use the support for 
`module_pathname` in the control file.

Now, after the usual 
`debuild` step, I can just 
`sudo debi` to install all the
just build packages and 
`CREATE EXTENSION` will run fine.  And in 
`9.0` you get
the old way to install it, but it still works:

~~~
$ psql -U postgres --cluster 9.0/main -1 \
-f /usr/share/postgresql/9.0/contrib/ip4r.sql
<lots of chatter>

$ psql -U postgres --cluster 9.1/main -c 'create extension ip4r;'
CREATE EXTENSION
~~~


That's it :)
