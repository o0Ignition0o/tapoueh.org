+++
date = "2013-02-12T11:17:00.000000+01:00"
title = "Playing with pgloader"
tags = ["PostgreSQL", "Common-Lisp", "python", "pgloader", "lparallel"]
categories = ["Projects","pgloader"]
thumbnailImage = "/img/old/made-with-lisp.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/made-with-lisp.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2013/02/12-playing-with-pgloader",
           "/blog/2013/02/12-playing-with-pgloader.html"]
+++

While making progress with both 
[Event Triggers](http://wiki.postgresql.org/wiki/Event_Triggers) and 
[Extension Templates](http://tapoueh.org/blog/2013/01/08-Extensions-Templates.html), I
needed to make a little break. My current keeping sane mental exercise seems
to mainly involve using 
*Common Lisp*, a programming language that ships with
about all the building blocks you need.

<center>*Yes, that old language brings so much on the table*</center>

When using 
*Common Lisp*, you have an awesome interactive development
environment where you can redefine function and objects 
*while testing them*.
That means you don't have to quit the interpreter, reload the new version of
the code and put the interactive test case together all over again after a
change. Just evaluate the change in the interactive environement: functions
are compiled incrementally over their previous definition, objects whose
classes have changed are migrated live.

See, I just said 
*objects* and 
*classes*. 
*Common Lisp* comes with some advanced
*Object Oriented Programming* facilities named 
[CLOS](http://www.aiai.ed.ac.uk/~jeff/clos-guide.html) and 
[MOP](http://www.alu.org/mop/index.html) where the 
*Java* and
*Python* and 
*C++* object models are just a subset of what you're being offered.
Hint, those don't have 
[Multiple Dispatch](http://en.wikipedia.org/wiki/Multiple_dispatch).

And you have a very sophisticated 
[Condition System](http://www.gigamonkeys.com/book/beyond-exception-handling-conditions-and-restarts.html) where 
*Exceptions* are just
a subset of what you can do (hint: have a look a 
[restarts](http://www.gigamonkeys.com/book/beyond-exception-handling-conditions-and-restarts.html#restarts) and tell me you
didn't wish your programming language of choice had them). And it continues
that way for about any basic building bloc you might want to be using.


## Loading data

Back to 
[pgloader](http://tapoueh.org/pgsql/pgloader.html) will you tell me. Right. I've been spending a couple of
evening on hacking on the new version of pgloader in 
*Common Lisp*, and wanted
to share some preliminary results.

<center>
{{< image classes="fig50 fancybox dim-margin" src="/img/old/toy-loader.320.jpg" >}}
</center>

<center>*Playing with the loader*</center>

The current status of the new 
*pgloader* still is pretty rough, if you're not
used to develop in Common Lisp you might not find it ready for use yet. I'm
still working on the internal APIs and trying to make something clean and
easy to use for a developer, and then I will provide some external ways to
play with it, user oriented. I missed that step once with the 
*Python* based
version of the tool, I don't want to do the same errors again this time.

So here's a test run with the current 
*pgloader*, on a small enough data set
of 
`226 MB` of 
`CSV` files.

~~~
time python pgloader.py -R.. --summary -Tc ../pgloader.dbname.conf

Table name        |    duration |    size |  copy rows |     errors
====================================================================
aaaaaaaaaa_aaaa   |      2.148s |       - |      24595 |          0
bbbbbbbbbb_bbbb...|      0.609s |       - |        326 |          0
cccccccccc_cccc...|      2.868s |       - |      25126 |          0
dddddddddd_dddd...|      0.638s |       - |          8 |          0
eeeeeeeeee_eeee...|      2.874s |       - |      36825 |          0
ffffffffff_ffffff |      0.667s |       - |        624 |          0
gggggggggg_gggg...|      0.847s |       - |       5638 |          0
hhh_hhhhhhh       |      9.907s |       - |     120159 |          0
iii_iiiiiiiiiiiii |      0.574s |       - |        661 |          0
jjjjjjj           |      6.647s |       - |      30027 |          0
kkk_kkkkkkkkk     |      0.439s |       - |         12 |          0
lll_llllll        |      0.308s |       - |          4 |          0
mmmm_mmm          |      2.139s |       - |      29669 |          0
nnnn_nnnnnn       |      8.555s |       - |     100197 |          0
oooo_ooooo        |     13.781s |       - |      93555 |          0
pppp_ppppppp      |      8.275s |       - |      76457 |          0
qqqq_qqqqqqqqqqqq |      8.568s |       - |     126159 |          0
====================================================================
Total             |  01m09.902s |       - |     670042 |          0
~~~



## Streaming data

With the new code in 
*Common Lisp*, I could benefit from real multi threading
and higher level abstraction to make it easy to use: 
[lparallel](http://lparallel.org/) is a lib
providing exactly what I need here, with 
*workers* and 
*queues* to communicate
data in between them.

What I'm doing is that two threads are separated, one is reading the data
from either a 
`CSV` file or a 
*MySQL* database directly, and pushing that data
in the queue; while the other thread is pulling data from the queue and
writing it into our 
[PostgreSQL](http://www.postgresql.org/) database.

~~~
CL-USER> (pgloader.csv:import-database "dbname"
            :csv-path-root "/path/to/csv/"
            :separator #\Tab
            :quote #\"
            :escape "\"\""
            :null-as ":null:")
                    table name       read   imported     errors       time
------------------------------  ---------  ---------  ---------  ---------
               aaaaaaaaaa_aaaa      24595      24595          0     0.995s
          bbbbbbbbbb_bbbbbbbbb        326        326          0     0.570s
       cccccccccc_cccccccccccc      25126      25126          0     1.461s
      dddddddddd_dddddddddd_dd          8          8          0     0.650s
eeeeeeeeee_eeeeeeeeee_eeeeeeee      36825      36825          0     1.664s
             ffffffffff_ffffff        624        624          0     0.707s
     gggggggggg_ggggg_gggggggg       5638       5638          0     0.655s
                   hhh_hhhhhhh     120159     120159          0     3.415s
             iii_iiiiiiiiiiiii        661        661          0     0.420s
                       jjjjjjj      30027      30027          0     2.743s
                 kkk_kkkkkkkkk         12         12          0     0.327s
                    lll_llllll          4          4          0     0.315s
                      mmmm_mmm      29669      29669          0     1.182s
                   nnnn_nnnnnn     100197     100197          0     2.206s
                    oooo_ooooo      93555      93555          0     9.683s
                  pppp_ppppppp      76457      76457          0     5.349s
             qqqq_qqqqqqqqqqqq     126159     126159          0     2.495s
------------------------------  ---------  ---------  ---------  ---------
             Total import time     670042     670042          0    34.836s
NIL
~~~


As you can see the control is still made for interactive developer usage,
which is fine for now but will have to change down the road, when the APIs
stabilize.

Now, let's compare to reading directly from 
*MySQL*:

~~~
CL-USER> (pgloader.mysql:stream-database "dbname")
                    table name       read   imported     errors       time
------------------------------  ---------  ---------  ---------  ---------
               aaaaaaaaaa_aaaa      24595      24595          0     0.887s
          bbbbbbbbbb_bbbbbbbbb        326        326          0     0.617s
       cccccccccc_cccccccccccc      25126      25126          0     1.497s
      dddddddddd_dddddddddd_dd          8          8          0     0.582s
eeeeeeeeee_eeeeeeeeee_eeeeeeee      36825      36825          0     1.697s
             ffffffffff_ffffff        624        624          0     0.748s
     gggggggggg_ggggg_gggggggg       5638       5638          0     0.923s
                   hhh_hhhhhhh     120159     120159          0     3.525s
             iii_iiiiiiiiiiiii        661        661          0     0.449s
                       jjjjjjj      30027      30027          0     2.546s
                 kkk_kkkkkkkkk         12         12          0     0.330s
                    lll_llllll          4          4          0     0.323s
                      mmmm_mmm      29669      29669          0     1.227s
                   nnnn_nnnnnn     100197     100197          0     2.489s
                    oooo_ooooo      93555      93555          0     9.148s
                  pppp_ppppppp      76457      76457          0     6.713s
             qqqq_qqqqqqqqqqqq     126159     126159          0     4.571s
------------------------------  ---------  ---------  ---------  ---------
          Total streaming time     670042     670042          0    38.272s
NIL
~~~


The 
*streaming* here is a tad slower than the 
*importing* from files. Now if you
want to be fair when comparing those, you would have to take into account
the time it takes to 
*export* the data out from its source. When doing that
*export/import* dance, a quick test shows a timing of 
`1m4.745s`. Now, if we do
an 
*export only* test, it runs in 
`31.822s`. So yes streaming is a good thing to
have here.


## Conclusion

We just got twice as fast as the python version.

Some will say that I'm not comparing fairly to the 
*Python* version of
pgloader here, because I could have implemented the streaming facility in
*Python* too. Well actually I did, the option are called 
[section_threads](http://tapoueh.org/pgsql/pgloader.html#sec13) and
[split_file_reading](http://tapoueh.org/pgsql/pgloader.html#sec15), that you can set so that a reader is pushing data into a
set of queues and several workers are feeding each from its own queue. It
didn't help with performances at all. Once again, read about the infamous
[Global Interpreter Lock](http://docs.python.org/3/c-api/init.html#threads) to understand why not.

<center>
{{< image classes="fig50 fancybox dim-margin" src="/img/old/lisplogo_flag_128.png" >}}
</center>

So actually it's a fair comparison here where the new code is twice as fast
as the previous one, with only some hours of hacking and before spending any
time on optimisation. Well, apart from using a 
*producer*, a 
*consumer* and a
*queue*, which I almost had to have for streaming in between two database
connections anyways.
