+++
date = "2010-10-07T13:30:00.000000+02:00"
title = "el-get reaches 1.0"
tags = ["Emacs", "Muse", "el-get", "Extensions", "release", "switch-window", "cssh", "mailq", "rcirc"]
categories = ["Projects","switch-window"]
thumbnailImage = "/img/old/el-get.big.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/el-get.big.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/10/07-el-get-reaches-10",
           "/blog/2010/10/07-el-get-reaches-10.html"]
+++

It's been a week since the last commits in the 
[el-get repository](http://github.com/dimitri/el-get), and those
were all about fixing and adding recipes, and about notifications. Nothing
like 
*core plumbing* you see. Also, 
`0.9` was released on 
*2010-08-24* and felt
pretty complete already, then received lots of improvements. It's high time
to cross the line and call it 
`1.0`!

Now existing users will certainly just be moderatly happy to see the tool
reach that version number, depending whether they think more about the bugs
they want to see fixed (ftp is supported, only called http) and the new
features they want to see in (
*info* documentation) or more about what 
`el-get`
does for them already today...

For the new users, or the yet-to-be-convinced users, let's take some time
and talk about 
`el-get`. A 
*FAQ* like session might be best.


## How is el-get different from ELPA?

[ELPA](http://tromey.com/elpa/) is the 
*Emacs Lisp Package Archive* and is also known as 
`package.el`, to
be included in Emacs 24. This allows emacs list extension authors to 
*package*
their work. That means they have to follow some guidelines and format their
contribution, then propose it for upload.

This requires licence checks (good) and for the 
[new official ELPA mirror](http://elpa.gnu.org/) it
even requires dead-tree papers exchange and contracts and copyright
assignments, I believe.


## Why have both?

While 
*ELPA* is a great thing to have, it's so easy to find some high quality
Emacs extension out there that are not part of the offer. Either authors are
not interrested into uploading to ELPA, or they don't know how to properly
*package* for it (it's only simple for single file extensions, see).

So 
`el-get` is a pragmatic answer here. It's there because it so happens that
I don't depend only on emacs extensions that are available with Emacs
itself, in my distribution 
`site-lisp` and in 
`ELPA`. I need some more, and I
don't need it to be complex to find it, fetch it, init it and use it.

Of course I could try and package any extension I find I need and submit it
to 
`ELPA`, but really, to do that nicely I'd need to contact the extension
author (
*upstream*) for him to accept my patch, and then consider a fork.

With 
`el-get` I propose distributed packaging if you will. Let's have a look
at two 
*recipes* here. First, the 
`el-get` one itself:

~~~
(:name el-get
       :type git
       :url "git://github.com/dimitri/el-get.git"
       :features el-get
       :compile "el-get.el")
~~~


Then a much more complex one, the 
[bbdb](http://bbdb.sourceforge.net/) one:

~~~
(:name bbdb
       :type git
       :url "git://github.com/barak/BBDB.git"
       :load-path ("./lisp" "./bits")
       :build ("./configure" "make autoloads" "make")
       :build/darwin ("./configure --with-emacs=/Applications/Emacs.app/Contents/MacOS/Emacs" "make autoloads" "make")
       :features bbdb
       :after (lambda () (bbdb-initialize))
       :info "texinfo")
~~~


The idea is that it's much simpler to just come up with a recipe like this
than to patch existing code and upload it to 
`ELPA`. And anybody can share
their 
*recipes* very easily, with or without proposing them to me, even if I
very much like to add some more in the official 
`el-get` list.

As a user, you don't even need to twiddle with recipes, mostly, because we
already have them for you. What you do instead is list them in
`el-get-sources`.


## So, show me how you use it?

Yeah, sure. Here's a sample of my 
`dim-packages.el` file, part of my 
`.emacs`
*suite*. Yeah a single 
`.emacs` does not suit me anymore, it's a complete
`.emacs.d` now, but that's because that's how I like it organised, you
know. So, here's the example:

~~~
;;; dim-packages.el --- Dimitri Fontaine
;;
;; Set el-get-sources and call el-get to init all those packages we need.
(require 'el-get)
(add-to-list 'el-get-recipe-path "~/dev/emacs/el-get/recipes")

(setq el-get-sources
      '(cssh el-get switch-window vkill google-maps yasnippet verbiste mailq sicp

	(:name magit
	       :after (lambda () (global-set-key (kbd "C-x C-z") 'magit-status)))

	(:name asciidoc
	       :type elpa
	       :after (lambda ()
			(autoload 'doc-mode "doc-mode" nil t)
			(add-to-list 'auto-mode-alist '("\\.adoc$" . doc-mode))
			(add-hook 'doc-mode-hook '(lambda ()
						    (turn-on-auto-fill)
						    (require 'asciidoc)))))

	(:name goto-last-change
	       :after (lambda ()
			(global-set-key (kbd "C-x C-/") 'goto-last-change)))

	(:name auto-dictionary :type elpa)
	(:name gist            :type elpa)
	(:name lisppaste       :type elpa)))

(el-get) ; that could/should be (el-get 'sync)
(provide 'dim-packages)
~~~


Ok that's not all of it, but it should give you a nice idea about what
problem I solve with 
`el-get` and how. In my emacs startup sequence, somewhere
inside my 
`~/.emacs.d/init.el` file, I have a line that says 
`(require
'dim-packages)`. This will set 
`el-get-sources` to the list just above, then
call 
`(el-get)`, the main function.

This main function will check each given package and install it if necessary
(including 
*build* the package, as in 
`make autoloads; make`), then 
*init*
it. What 
*init* means exactly depends on what the recipe says. That can
include 
*byte-compiling* some files, caring about 
*load-path*, 
*load* and 
*require*
commands, caring about 
*Info-directory-list* and 
`ginstall-info` too, and some
more.

So in short, it will make it so that your emacs instance is ready for you to
use. And you get the choice to use the given 
`el-get` recipes as-is, like I
did for 
`cssh`, 
`el-get`, 
`switch-window` and others, up to 
`sicp`, or to tweak them
partly, like in the 
`magit` example where I've added a user init function (the
`:after` property) to bind 
`magit-status` to 
`C-x C-z` here. You can even embed a
full recipe inline in the 
`el-get-sources` variable, that's the case for each
item that gives its 
`:type` property, like 
`asciidoc` or 
`gist`.

And, as you see, we're using 
`ELPA` a lot in this sources, so 
`el-get` isn't
striving to replace it at all, it's just trying to accomodate to a broader
world.


## I read that the el-get-install is asynchronous, tell me more.

Yeah, right, the example above says 
`(el-get)` at its end, and in the cases
when 
`el-get` has to install or build sources, this will be done
asynchronously. Which means that not only several sources will get processed
at once (using your multi cores, yeah) but that it will let emacs start up
as if it was ready.

It happens that's usually what I want, because I seldom add sources in my
setup, but in theory that can break your emacs. What I do is start it again
or fix by hand, what you can do instead is 
`(el-get 'sync)` so that emacs is
blocked waiting for 
`el-get` to properly install and initialize all the
sources you've setup. Your choice, just add the 
`'sync` parameter there.


## Now, explain me why it is better this way, again, please?

Well, before I wrote 
`el-get`, trying out a new extension, setting it up etc
was something quite involved, and that I had to redo on several
machines. The only way not to redo it was to include the extension's code
into my own 
`git` repository (my 
`emacs.d` is in 
`git`, of course).

And putting code I don't maintain into my own 
`git` repository is something I
frown upon. I have no business pretending I'll maintain the code, and I know
I will never think to check the 
`URL` where I've found it for updates. That's
when I though noting down the 
`URL` somewhere.

Also, what about sharing the extension with friends. Uneasy, at best.

Enters 
`el-get` and I can just add an entry to 
`el-get-sources`, based on a file
somewhere in my own 
`el-get-recipe-path`. When I'm happy with this file, I can
contribute it to 
`el-get` proper or just send it over to any interested
recipient. Adding it to your sources is easy. Copy the file in your
`el-get-recipe-path` somewhere, add its name to your 
`el-get-sources`, then 
`M-x
el-get-install` it. Done. If you were given the 
`:after` function, it's all
setup already.

If you contribute the recipe to 
`el-get`, then 
`M-x el-get-update RET el-get
RET` and you get it on this other machine where you also use Emacs. Or you
can tell your friend to do the same and benefit from your 
*packaging*.


## Well, sounds good. What recipes do you have already?

I count 
`67` of them already. One of them is just a book in 
*info* format, with
no 
*elisp* at all, can you spot it?

~~~
ELISP> (directory-files "~/dev/emacs/el-get/recipes/" nil "el$")

("auctex.el" "auto-complete-etags.el" "auto-complete-extension.el"
"auto-complete.el" "auto-install.el" "autopair.el" "bbdb.el"
"blender-python-mode.el" "color-theme-twilight.el" "color-theme.el"
"cssh.el" "django-mode.el" "el-get.el" "emacs-w3m.el" "emacschrome.el"
"emms.el" "ensime.el" "erc-highlight-nicknames.el" "erc-track-score.el"
"escreen.el" "filladapt.el" "flyguess.el" "gist.el" "google-maps.el"
"google-weather.el" "goto-last-change.el" "haskell-mode.el"
"highlight-parentheses.el" "hl-sexp.el" "levenshtein.el" "magit.el"
"mailq.el" "maxframe.el" "multi-term.el" "muse-blog.el" "nognus.el"
"nterm.el" "nxhtml.el" "offlineimap.el" "package.el" "popup-kill-ring.el"
"pos-tip.el" "pov-mode.el" "psvn.el" "pymacs.el" "rainbow-mode.el"
"rcirc-groups.el" "rinari.el" "ropemacs.el" "rt-liberation.el" "scratch.el"
"session.el" "sicp.el" "smex.el" "switch-window.el" "textile-mode.el"
"todochiku.el" "twitter.el" "twittering-mode.el" "undo-tree.el"
"verbiste.el" "vimpulse-surround.el" "vimpulse.el" "vkill.el" "xcscope.el"
"xml-rpc-el.el" "yasnippet.el")
~~~



## Ok, I want to try it, what's next?

Visit the following 
`URL` 
[http://github.com/dimitri/el-get](http://github.com/dimitri/el-get) and follow the
install instructions. You're given a 
*scratch installer* there, that's some
*elisp* code you copy paste into 
`*scratch*` then execute there, and you have
`el-get` ready to serve.

An excellent idea I stole at 
`ELPA`!


## Hey, I already know what el-get is, what's new in 1.0?

The 
*changelog* is quite full of good stuff, really:

  - Implement el-get recipes so that el-get-sources can be a simple list

   of symbols. Now that there's an authoritative git repository, where
   to share the recipes is easy.

  - Add support for emacswiki directly, save from having to enter the URL

  - Implement package status on-disk saving so that installing over a

   previously failed install is in theory possible. Currently `el-get'
   will refrain from removing your package automatically, though.

  - Fix ELPA remove method, adding a "removed" state too.

  - Implement CVS login support.

  - Add lots of recipes

  - Add support for `system-type' specific build commands

  - Byte compile files from the load-path entries or :compile files

  - Implement support for git submodules with the command

   `git submodule update --init --recursive`

  - Add catch-all post-install and post-update hooks

  - Add desktop notification on install/update.


## I'm still using the deprecated emacswiki version, what now?

That version didn't have recipes, and the new version should be perfectly
happy with your current 
`el-get-sources`, so that I recommend using the
*scratch installer* too. Don't forget to add 
`el-get` itself into your
`el-get-sources` list, of course!
