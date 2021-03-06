+++
date = "2011-07-13T17:15:00.000000+02:00"
title = "Back From CHAR(11)"
tags = ["PostgreSQL", "skytools", "Conferences"]
categories = ["Conferences","PostgreSQL Confs"]
thumbnailImage = "/img/old/conferences.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/old/conferences.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2011/07/13-back-from-char11",
           "/blog/2011/07/13-back-from-char11.html"]
+++

[CHAR(11)](http://char11.org/schedule) finished somewhen in the night leading to today, if you consider
the 
*social events* to be part of it, which I definitely do.  This conference
has been a very good one, both on the organisation side of things and of
course for its content.

It began with a perspective about the evolution of replication solutions, by
***Jan Wieck*** himself.  In some way 
[Skytools](http://wiki.postgresql.org/wiki/SKytools) is an evolution of 
[Slony](http://slony.info/), in the
sense that it reuses the same concepts, a part of the design, and even share
bits of the implementation (like the 
[txid_snapshot](http://www.postgresql.org/docs/8.3/interactive/functions-info.html#FUNCTIONS-TXID-SNAPSHOT) datatype that were added
in PostgreSQL 8.3).  The evolution occured in choosing a subset of the
features of Slony and then simplifying the user interface as much as
possible.  And with Skytools 3.0, those features that were removed but still
are useful to solve real-life problems are now available too.

Of course the talk did approach the other replication solutions (not just
the trigger based ones), and did compare 
[RServ](http://wiki.postgresql.org/wiki/Setting_up_RServ_with_PostgreSQL_7.0.3) to 
[Bucardo](http://bucardo.org/) for example.  And
then all those were compared to the 
[PostgreSQL](http://www.postgresql.org/) core replication facilities,
which are quite a different animal.  It was a really nice 
*keynote* here,
preparing the audience minds to make the most out of all the other talks.

I will not review all the talks in details, as I'm pretty sure some other
attendees will turn into reporters themselves: scaling the write load!

Still 
[repmgr](http://projects.2ndquadrant.com/repmgr) got its share of attention.  
[Greg Smith](http://www.2ndquadrant.com/books/postgresql-9-0-high-performance/) and 
[Cédric Villemain](http://www.2ndquadrant.fr/)
did present both how to do 
**read scaling** and 
**auto failover** management with
this tool, going into fine details about how it works internally and how to
best design your services architecture for maximum 
**data availibility**.  The
question and answers section led to insist on the fact that you can not have
data availibility with less than 3 production nodes.

[Magnus Hagander](http://www.hagander.net/) detailed how flexible the core protocol support for
replication (and streaming) really is.  That flexibility means that you can
quite easily talk this protocol from any application, and the idea of a 
*wal
proxy* did pop out again (see 
[Back from PgCon2010](../../2010/05/27-back-from-pgcon2010.html) article for my first
mentionning of the idea).  The main difference is that we now have
*synchronous replication* support, so that the proxy could be trusted both for
archiving and serving standbys.

Of course 
[Simon](http://database-explorer.blogspot.com/) still has lots of ideas about next 10 years of replication
oriented projects for core PostgreSQL code, and his talk nicely summarized
the previous 7 years.  Future is bright, and guess what, it's beginning
today!

We also heard about 
[Heroku](http://www.heroku.com/), and these guys are doing crazy impressive
things.  Like running 
`150 000` PostgreSQL instances, for example, showing
that you can actually use our prefered database server in the hosting
business.  I expect that the maturing solution and tool sets providing data
availibility are soon to be a game changer here.  What they are doing is
designing a 
**flexible data architecture** with strong guarantees (
**no data
loss**).  The 
*cloud elasticity* is reaching out from the stateless services,
and 
*those guys* are making it happen now.

May you live in interresting times!
