+++
date = "2010-10-07T17:15:00.000000+02:00"
title = "Resuming work on Extensions, first little step"
tags = ["PostgreSQL", "Extensions"]
categories = ["PostgreSQL","Extensions"]
thumbnailImage = "/img/postgresql-512.jpg"
thumbnailImagePosition = "left"
coverImage = "/img/postgresql-512.jpg"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/10/07-resuming-work-on-extensions-first-little-step",
           "/blog/2010/10/07-resuming-work-on-extensions-first-little-step.html"]
+++

Yeah I'm back on working on my part of the extension thing in 
[PostgreSQL](http://www.postgresql.org/).

First step is a little one, but as it has public consequences, I figured I'd
talk about it already. I've just refreshed my 
`git` repository to follow the
new 
`master` one, and you can see that here
[http://git.postgresql.org/gitweb?p=postgresql-extension.git;a=commitdiff;h=9a88e9de246218e93c04b6b97e1ef61d97925430](http://git.postgresql.org/gitweb?p=postgresql-extension.git;a=commitdiff;h=9a88e9de246218e93c04b6b97e1ef61d97925430).

It's been easier than I feared, mainly:

~~~
$ git --no-pager diff master..extension
$ git --no-pager format-patch master..extension
$ cp 0001-First-stab-at-writing-pg_execute_from_file-function.patch ..
$ git checkout master
$ git pull -f pgmaster
$ git reset --hard pgmaster/master
$ git checkout extension
$ git reset --hard master
$ git am -s ../0001-First-stab-at-writing-pg_execute_from_file-function.edit.patch 
$ git status
$ git log --short | head
$ git log -n2 --oneline
$ git push -f
~~~


So that's still more steps that one want to call dead simple, but still. The
`format-patch` command is to save my work away (all patches that are in the
*extension* branch but not in the 
*master* — well that was only one of them
here). Then, as the master repository 
`URL` didn't change, I can simply 
`pull`
the changes in. Of course I had a nice message 
*warning: no common commits*.

Once pulled, I trashed my local copy and replaced it with the new official
one, that's 
`git reset --hard pgmaster/master`, then in the 
*extension* branch I
could trash it and have it linked to the local 
`master` again.

Of course, the 
`git am` method wouldn't apply my patch as-is, there was some
underlying changes in the source files, the identification tag changed from
`$PostgreSQL$` to, e.g., 
`src/backend/utils/adt/genfile.c`, and I had to cope
with that. Maybe there's some tool (
`git am -3` ?) to do it automatically, I
just copy edited the 
`.patch` file.

Lastly, it's all about checking the result and publishing the result. This
last line is 
`git push -f` and is when I just trashed and replaced my
[postgresql-extension](http://git.postgresql.org/gitweb?p=postgresql-extension.git;a=summary) community repository. I don't think anybody was
following it, but should it be the case, you will have to 
*reinit* your copy.

More blog posts to come about extensions, as I arranged to have some real
time to devote on the topic. At least I was able to arrange things so that I
can work on the subject for real, and the first thing I did, the very night
before it was meant to begin, is catch a 
*tonsillitis*. Lost about a week, not
the project! Stay tuned!
