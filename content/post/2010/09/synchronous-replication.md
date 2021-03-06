+++
date = "2010-09-06T18:05:00.000000+02:00"
title = "Synchronous Replication"
tags = ["PostgreSQL", "release"]
categories = ["PostgreSQL","Release"]
thumbnailImage = "/img/postgresql-512.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/postgresql-512.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/09/06-synchronous-replication",
           "/blog/2010/09/06-synchronous-replication.html"]
+++

Although the new asynchronous replication facility that ships with 9.0 ain't
released to the wide public yet, our hackers hero are already working on the
synchronous version of it. A part of the facility is rather easy to design,
we want something comparable to 
[DRBD](http://www.drbd.org/) flexibility, but specific to our
database world.  So 
*synchronous* would either mean 
*recv*, 
*fsync* or 
*apply*,
depending on what you need the 
*standby* to have already done when the master
acknowledges the 
`COMMIT`. Let's call that the 
*service level*.

The part of the design that's not so easy is more interesting. Do we need to
register standbys and have the 
*service level* setup per standby? Can we get
some more flexibility and have the 
*service level* set on a per-transaction
basis? The idea here would be that the application knows which transactions
are meant to be extra-safe and which are not, the same way that you can set
`synchronous_commit to off` when dealing with web sessions, for example.

*Why choosing?* I hear you ask. Well, it's all about having more data safety,
and a typical setup would contain an asynchronous reporting server and a
local 
*failover* synchronous server. Then add a remote one, too. So even if we
pick the transaction based facility, we still want to be able to choose at
setup time which server to failover to. Than means we don't want that much
flexibility now, we want to know where the data is safe, we don't want to
have to guess.

Some way to solve that is to be able to setup a slave as being the failover
one, or say, the 
`sync` one. Now, the detail that ruins it all is that we need
a 
*timeout* to handle worst cases when a given slave loses its connectivity
(or power, say). Now, the slave ain't in 
*sync* any more and some people will
require that the service is still available (
*timeout* but 
`COMMIT`) and some
will require that the service is down: don't accept a new transaction if you
can't make its data safe to the slave too.

The answer would be to have the master arbitrate between what the
transaction wants and what the slave is setup to provide, and what it's able
to provide at the time of the transaction. Given a transaction with a
*service level* of 
*apply* and a slave setup for being 
*async*, the 
`COMMIT` does
not have to wait, because there's no known slave able to offer the needed
level. Or the 
`COMMIT` can not happen, for the very same reason.

Then I think it all flows quite naturally from there, and while arbitrating
the master could record which slave is currently offering what 
*service
level*. And offering the information in a system view too, of course.

The big question that's not answered in this proposal is how to setup that
being unable to reach the wanted 
*service level* is an error or a
warning?

That too would need to be for the master to arbitrate based on a per standby
and a per transaction setting, and in the general case it could be a 
*quorum*
setup: each slave is given a 
*weight* and each transaction a 
*quorum* to
reach. The master sums up the weights of the standby that ack the
transaction at the needed 
*service level* and the 
`COMMIT` happens as soon as
the quorum is reached, or is canceled as soon as the 
*timeout* is reached,
whichever comes first.

Such a model allows for very flexible setups, where each standby has a
*weight* and offers a given 
*service level*, and each transaction waits until a
*quorum* is reached. Giving the right weights to your standbys (like, powers
of two) allow you to set the quorum in a way that only one given standby is
able to acknowledge the most important transactions. But that's flexible
enough you can change it at any time, it's just a 
*weight* that allows a 
*sum*
to be made, so my guess would be it ends up in the 
*feedback loop* between the
standby and its master.

The most appealing part of this proposal is that it doesn't look complex to
implement, and should allow for highly flexible setups. Of course, the devil
is in the details, and we're talking about latencies in the distributed
system here. That's also being discussed on the 
[mailing list](http://archives.postgresql.org/pgsql-hackers/).
