+++
date = "2009-01-21T00:00:00.000000+01:00"
title = "Londiste Trick"
tags = ["PostgreSQL", "Skytools"]
categories = ["PostgreSQL","Skytools"]
thumbnailImage = "/img/old/londiste_logo.gif"
thumbnailImagePosition = "left"
coverImage = "/img/old/londiste_logo.gif"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2009/01/21-londiste-trick",
           "/blog/2009/01/21-londiste-trick.html"]
+++

So, you're using 
`londiste` and the 
`ticker` has not been running all night
long, due to some restart glitch in your procedures, and the 
*on call* admin
didn't notice the restart failure. If you blindly restart the replication
daemon, it will load in memory all those events produced during the night,
at once, because you now have only one tick where to put them all.

The following query allows you to count how many events that represents,
with the magic tick numbers coming from 
`pgq.subscription` in columns
`sub_last_tick` and 
`sub_next_tick`.

~~~
SELECT count(*) 
  FROM pgq.event_1, 
      (SELECT tick_snapshot
         FROM pgq.tick
        WHERE tick_id BETWEEN 5715138 AND 5715139
      ) as t(snapshots)
WHERE txid_visible_in_snapshot(ev_txid, snapshots);
~~~


In our case, this was more than 
*5 millions and 400 thousands* of events. With
this many events to care about, if you start londiste, it'll eat as many
memory as needed to have them all around, which might be more that what your
system is able to give it. So you want a way to tell 
`londiste` not to load
all events at once. Here's how: add the following knob to your 
*.ini*
configuration file before to restart the londiste daemon:

~~~
pgq_lazy_fetch = 500
~~~


Now, 
`londiste` will lazyly fetch 
`500` events at once or less, even if a single
`batch` (which contains all 
*events* between two 
*ticks*) contains a huge number
of events. This number seems a good choice as it's the default 
`PGQ` setting
of number of events in a single 
*batch*. This number is only outgrown when the
ticker is not running or when you're producing more 
*events* than that in a
single transaction.

Hope you'll find the tip useful!
