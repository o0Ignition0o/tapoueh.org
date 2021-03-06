+++
date = "2010-09-09T16:35:00.000000+02:00"
title = "Window Functions example"
tags = ["PostgreSQL", "YeSQL"]
categories = ["PostgreSQL","YeSQL"]
thumbnailImage = "/img/old/change.gif"
thumbnailImagePosition = "left"
coverImage = "/img/old/change.gif"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/09/09-window-functions-example",
           "/blog/2010/09/09-window-functions-example.html"]
+++

So, when 
`8.4` came out there was all those comments about how getting
[window functions](http://www.postgresql.org/docs/8.4/interactive/tutorial-window.html) was an awesome addition. Now, it seems that a lot of people
seeking for help in 
[#postgresql](http://wiki.postgresql.org/index.php?title=IRC) just don't know what kind of problem this
feature helps solving. I've already been using them in some cases here in
this blog, for getting some nice overview about
[Partitioning: relation size per “group”](http://tapoueh.org/articles/blog/_Partitioning:_relation_size_per_%E2%80%9Cgroup%E2%80%9D.html).

<center>*That's another way to count change*</center>

Now, another example use case rose on 
`IRC` today. I'll quote directly our user here:

>   hey there, how can i count the number of (value) changes in one column?


Now, several of us began talking about 
*window functions* and about the fact
that you need some other column to identify the ordering of those weights,
obviously, because that's the only way to define what a change is in this
context. Let's have a first try at it.

~~~
=# select o, w, 
          case when lag(w) over(order by o) is distinct from w then 1 end as change
     from (values (1, 5), (2, 10), (3, 7), (4, 7), (5, 7)) as data(o, w);
 o | w  | change 
---+----+--------
 1 |  5 |      1
 2 | 10 |      1
 3 |  7 |      1
 4 |  7 |       
 5 |  7 |       
(5 rows)
~~~


Not too bad, but of course we are seeing a false change on the first line,
as for any 
*window* of rows you define the previous one, given by 
`lag()
over()`, will be 
`NULL`. The easiest way to accommodate is the following:

~~~
=# select sum(change) -1 as changes 
     from (select case when lag(w) over(order by o) is distinct from w
                       then 1
                   end as change
             from (values (1, 5),
                          (2, 10),
                          (3, 7),
                          (4, 7),
                          (5, 7)) as t(o, w)) as x;
 changes 
---------
       2
(1 row)
~~~


So don't be shy and go read about 
[window functions in SQL expressions](http://www.postgresql.org/docs/8.4/interactive/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS) and
[window function processing](http://www.postgresql.org/docs/8.4/interactive/queries-table-expressions.html#QUERIES-WINDOW) in the query table expressions. That's a very
nice tool to have and my guess is that you will soon enough realize the only
reason why you could think you don't have a need for them is that you didn't
know it existed, and what you can do with it. 
*Sharpen your saw!* :)
