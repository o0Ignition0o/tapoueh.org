+++
date = "2010-07-08T11:15:00.000000+02:00"
title = "Using indexes as column store?"
tags = ["PostgreSQL", "release"]
categories = ["PostgreSQL","Release"]
thumbnailImage = "/img/postgresql-512.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/postgresql-512.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/07/08-using-indexes-as-column-store",
           "/blog/2010/07/08-using-indexes-as-column-store.html"]
+++

There's a big trend nowadays about using column storage as opposed to what
PostgreSQL is doing, which would be row storage. The difference is that if
you have the same column value in a lot of rows, you could get to a point
where you have this value only once in the underlying storage file. That
means high compression. Then you tweak the 
*executor* to be able to load this
value only once, not once per row, and you win another huge source of data
traffic (often enough, from disk).

Well, it occurs to me that maybe we could have column oriented storage
support without adding any new storage facility into PostgreSQL itself, just
using in new ways what we already have now. Column oriented storage looks
somewhat like an index, where any given value is meant to appear only
once. And you have 
*links* to know where to find the full row associated in
the main storage.

There's a work in progress to allow for PostgreSQL to use indexes on their
own, without having to get to the main storage for checking the
visibility. That's known as the 
[Visibility Map](http://www.postgresql.org/docs/8.4/static/storage-vm.html), which is still only a hint
in released versions. The goal is to turn that into a crash-safe trustworthy
source in the future, so that we get 
*covering indexes*. That means we can use
an index and skip getting to the full row in main storage and get the
visibility information there.

Now, once we have that, we could consider using the indexes in more
queries. It could be a win to get the column values from the index when
possible and if you don't 
*output* more columns from the 
*heap*, return the
values from there. Scanning the index only once per value, not once per row.

There's a little more though on the point in the 
[Next Generation PostgreSQL](char10.html#sec10)
article I've been referencing already, should you be interested.
