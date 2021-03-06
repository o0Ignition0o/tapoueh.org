+++
date = "2013-02-04T09:55:00.000000+01:00"
title = "Another Great FOSDEM"
tags = ["PostgreSQL", "Conferences", "FOSDEM", "Event-Triggers"]
categories = ["Conferences","PostgreSQL Confs"]
thumbnailImage = "/img/old/fosdem-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/fosdem-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2013/02/04-Another-great-FOSDEM",
           "/blog/2013/02/04-Another-great-FOSDEM.html"]
+++

This year's FOSDEM has been a great edition, in particular the
[FOSDEM PGDAY 2013](http://fosdem2013.pgconf.eu/) was a great way to begin a 3 days marathon of talking
about PostgreSQL with people not only from our community but also from
plenty other Open Source communities too: users!

<center>
<div class="figure dim-margin">
  <a href="https://fosdem.org/2013/">
    <img src="/img/old/fosdem-logo.png">
  </a>
</div>

<div class="figure dim-margin">
  <a href="http://fosdem2013.pgconf.eu/">
    <img src="/img/old/postgresql-elephant.small.png">
  </a>
</div>
</center>

<center>*PostgreSQL at FOSDEM made for a great event*</center>

Having had the opportunity to meet more people from those other development
communities, I really think we should go and reach for them in their own
conferences. About any PostgreSQL community member I've been talking about
with about that idea seemed to agree and generally already was thinking the
same thing. And most are already doing it, in fact...

I had the pleasure to run two conferences there, both in the 
[PostgreSQL devroom](https://fosdem.org/2013/schedule/track/postgresql/).


## Event Triggers

I'm currently in the middle of implementing 
*Event Triggers* for PostgreSQL
and I have been for about the last 2 years. It's a quite complex feature to
get right and so the patch itself is complex and large, which means the
reviewing process is complex and takes time.

That also means that some parts of the design have already been redone
completely at least 3 times, and that what got 
*commited* to the PostgreSQL
code is nothing like what the design we decided should go in looks like.
That's just a fact of life, maybe, but that makes for a very long
development process.

We're now getting to the end of it though, and this talk is showing both
where we want to go with 
*Event Triggers*, where we are now and what remains
to be done for 9.3 if we want the feature to be any useful.

If you're interested into that development, have a look at the slide deck
and possibly ask me some questions about what's not clear on the
[pgsql-hackers](http://www.postgresql.org/list/pgsql-hackers/) mailing list (preferably).

<center>
<div class="figure dim-margin">
  <a href="/images/confs/Fosdem2013_Event_Triggers.pdf">
    <img src="/img/old/Fosdem2013_Event_Triggers.png">
  </a>
</div>
</center>

<center>*Event Triggers, The Real Mess™*</center>

The other way to get summarized and clear information about Event Triggers
is the wiki page by the same name: 
[Event Triggers](http://wiki.postgresql.org/wiki/Event_Triggers).

You will see that while a lot has been done (internal refactoring, adding
new infrastructure and SQL level commands, and the minimum 
`PLpgSQL` support);
a lot remains to be done where the code has already been submitted several
times, following several designs directions given by careful review on
hackers, and still we have some choices to make.


## Implementing High-Availability

This talk is showing several ways to implement 
*High Availability* with
PostgreSQL. The fact is that that term is overloaded already, and usually
covers two very different things which are 
*Service Availability* and 
*Data
Availability*.

In the talk, we're showing up several techniques that you can use to address
different set of compromises in between 
*scaling*, 
*load balancing*, 
*data
availability* and 
*durability*, and 
*service availability*. The first two points
could seem unrelated to the main topic, but 
*scaling* often is a simple enough
way to achieve 
*service availability*... until you need to think about
*sharding*, that is.

<center>
<div class="figure dim-margin">
  <a href="/images/confs/Fosdem2013_High_Availability.pdf">
    <img src="/img/old/Fosdem2013_High_Availability.640.png">
  </a>
</div>
</center>

<center>*Implementing High Availability of Services and Data with PostgreSQL*</center>

So the talk is all about making compromises in between them and getting to
an architecture able to implement the choosen compromises. While the talk
has been pretty well received, it was delivered in a 50 mins slot where we
usually take a whole day or three when addressing that problems at a
customer's site.

Some parts of how to get to the right architecture for the compromises that
are important for you can't be fully covered in that time slot, while still
being able to actually present the techniques that we're using.

I think it might be useful to extract a single use-case or two from that
talk then have a full 50 mins version reduced to a single or a couple of
very clear compromises and how to achieve them in details, rather than
trying to present a full range of techniques and how to use them in
different scenarios.


## FOSDEM

After having been talking with many people, it appears that for next year's
edition I should be proposing a more general talk that aims at helping
developpers in other communities (python, ruby, etc) discover what's in for
them in PostgreSQL. This database is full of advanced features that are
really easy to use, and the only problem when preparing such a talk is
choosing the right subset...

If you're running a local developper user group and are interested into
learning some more about how PostgreSQL can help you in a daily basis,
please do get in touch with me and let's schedule a presentation together!
