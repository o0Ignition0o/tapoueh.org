+++
date = "2010-02-18T11:37:00.000000+01:00"
title = "Getting out of SQL_ASCII, part 1"
tags = ["PostgreSQL", "Encoding"]
categories = ["PostgreSQL","Encoding"]
thumbnailImage = "/img/encodings.png"
thumbnailImagePosition = "left"
coverImage = "/img/encodings.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/02/18-getting-out-of-sql_ascii-part-1",
           "/blog/2010/02/18-getting-out-of-sql_ascii-part-1.html"]
+++

It happens that you have to manage databases 
*designed* by your predecessor,
and it even happens that the team used to not have a 
*DBA*. Those 
*histerical
raisins* can lead to having a 
`SQL_ASCII` database. The horror!

What 
`SQL_ASCII` means, if you're not already familiar with the consequences
of such a choice, is that all the 
`text` and 
`varchar` data that you put in the
database is accepted as-is. No checks. At all. It's pretty nice when you're
lazy enough to not dealing with 
*strange* errors in your application, but if
you think that t's a smart move, please go read
[The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](http://www.joelonsoftware.com/articles/Unicode.html)
by 
[Joel Spolsky](http://www.joelonsoftware.com/) now. I said now, I'm waiting for you to get back here. Yes,
I'll wait.

The problem of course is not being able to read the data you just stored,
which is seldom the use case anywhere you use a database solution such as
[PostgreSQL](http://www.postgresql.org/).

Now, it happens too that it's high time to get off of 
`SQL_ASCII`, the
infamous. In our case we're lucky enough in that the data are all in fact
`latin1` or about that, and this comes from the fact that all the applications
connecting to the database are sharing some common code and setup. Then we
have some tables that can be tagged 
*archives* and some other 
*live*. This blog
post will only deal with the former category.

For those tables that are not receiving changes anymore, we will migrate
them by using a simple but time hungry method: 
`COPY OUT|recode|COPY IN`. I've
tried to use 
`iconv` for recoding our data, but it failed to do so in lots of
cases, so I've switched to using the 
[GNU recode](http://www.gnu.org/software/recode/recode.html) tool, which works just fine.

The fact that it takes so much time doing the conversion is not really a
problem here, as you can do it 
*offline*, while the applications are still
using the 
`SQL_ASCII` database. So, here's the program's help:

~~~
recode.sh [-npdf0TI] [-U user ] -s schema [-m mintable] pattern
     -d debug
     -n dry run, only print table names and expected files
     -s schema
     -m mintable, to skip already processed once
     -U connect to PostgreSQL as user
     -f force table loading even when export files do exist
     -0 only (re)load tables with zero-sized copy files
     -T Truncate the tables before COPYing recoded data
     -I Temporarily drop the indexes of the table while COPYing
pattern ^table_name_, e.g.
~~~


The 
`-I` option is neat enough to create the indexes in parallel, but with no
upper limit on the number of index creation launched. In our case it worked
well, so I didn't have to bother.

Take a look at the 
[recode.sh](/resources/recode.sh) script, and don't hesitate editing it for your
purpose. It's missing some obvious options to get useful in the large, such

We'll get back to the subject of this entry in 
*part 2*, dealing with how to
recode your data in the database itself, thanks to some insane regexp based
queries and helper functions. And thanks to a great deal of IRC based
helping, too.
