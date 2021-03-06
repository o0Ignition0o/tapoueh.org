+++
date = "2010-07-06T10:50:00.000000+02:00"
title = "MVCC in the Cloud"
tags = ["PostgreSQL", "Conferences"]
categories = ["Conferences","PostgreSQL Confs"]
thumbnailImage = "/img/old/conferences.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/conferences.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/07/06-mvcc-in-the-cloud",
           "/blog/2010/07/06-mvcc-in-the-cloud.html"]
+++

At 
[CHAR(10)](http://char10.org/) 
***Markus*** had a talk about
[Using MVCC for Clustered Database Systems](http://char10.org/talk-schedule-details#talk13) and explained how 
[Postgres-R](http://postgres-r.org/) does
it. The scope of his project is to maintain a set of database servers in the
same state, eventually.

Now, what does it mean to get "In the Cloud"? Well there are more than one
answer I'm sure, mine would insist on including this "Elasticity" bit. What
I mean here is that it'd be great to be able to add or lose nodes and stay
*online*. Granted, that what's 
*Postgres-R* is providing. Does that make it
ready for the "Cloud"? Well it happens so that I don't think so.

Once you have elasticity, you also want 
*scalability*. That could mean lots of
thing, and 
*Postgres-R* already provides a great deal of it, at the connect
and reads level: you can do your business 
*unlimited* on any node, the others
will eventually (
*eagerly*) catch-up, and you can do your 
`select` on any node
too, reading from the same data set. Eventually.

What's still missing here is the hard sell, 
*write scalability*. This is the
idea that you don't want to sustain the same 
*write load* on all the members
of the "Cloud cluster". It happens that I have some idea about how to go on
this, and this time I've been trying to write them down. You might be
interested into the 
[MVCC in the Cloud](http://tapoueh.org/char10.html#sec3) part of my 
[Next Generation PostgreSQL](http://tapoueh.org/char10.html)
notes.

My opinion is that if you want to distribute the data, this is a problem
that falls in the category of finding the data on disk. This problem is
already solved in the executor, it knows which operating system level file
to open and where to seek inside that in order to find a row value for a
given relation. So it should be possible to teach it that some relation's
storage ain't local, to get the data it needs to communicate to another
PostgreSQL instance. 

I would call that a 
*remote tablespace*. It allows for distributing both the
data and their processing, which could happen in parallel. Of course that
means there's now some latency concerns, and that some 
*JOIN* will get slow if
you need to retrieve the data from the network each time. For that what I'm
thinking about is the possibility to manage a local copy of a remote
tablespace, which would be a 
*mirror tablespace*. But that's for another blog
post.

Oh, if that makes you think a lot of 
[SQL/MED](http://wiki.postgresql.org/wiki/SQL/MED), that would mean I did a good
enough job at explaining the idea. The main difference though would be to
ensure transaction boundaries over the local and remote data: it's one
single distributed database we're talking about here.
