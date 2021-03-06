+++
date = "2011-04-15T21:30:00.000000+02:00"
title = "Emacs Kicker"
tags = ["Emacs", "debian", "el-get", "switch-window", "emacs-kicker"]
categories = ["Projects","switch-window"]
thumbnailImage = "/img/old/debian-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/debian-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2011/04/15-emacs-kicker",
           "/blog/2011/04/15-emacs-kicker.html"]
+++

Following up on the very popular 
[emacs-starter-kit](https://github.com/technomancy/emacs-starter-kit), I'm now proposing the
[emacs-kicker](https://github.com/dimitri/emacs-kicker).  It's about the 
`.emacs` file you've seen in older posts here,
which I maintain for some colleagues.  After all, if they find it useful,
some more people might to, so I've decided to publish it.

What you'll find is a very simple 
`128` lines 
[Emacs](http://www.gnu.org/software/emacs/) user init file, based on
[el-get](https://github.com/dimitri/el-get) for external packages.  A not so 
*random* selection of those is used,
here's the list when you hide some details:

~~~
'(el-get			; el-get is self-hosting
   escreen            		; screen for emacs, C-\ C-h
   php-mode-improved		; if you're into php...
   psvn				; M-x svn-status
   switch-window		; takes over C-x o
   auto-complete		; complete as you type with overlays
   emacs-goodies-el		; the debian addons for emacs
   yasnippet			; powerful snippet mode
   zencoding-mode		; http://www.emacswiki.org/emacs/ZenCoding
   (:name buffer-move		; move buffers around in windows
   (:name smex			; a better (ido like) M-x
   (:name magit			; git meet emacs, and a binding
   (:name goto-last-change	; move pointer back to last change
~~~


Another interresting thing to note in this 
`kicker` is a choice of some key
bindings that are rather unusual (yet) I guess.

~~~
(global-set-key (kbd "C-x C-b") 'ido-switch-buffer)
(global-set-key (kbd "C-x C-c") 'ido-switch-buffer)
(global-set-key (kbd "C-x B") 'ibuffer)
~~~


Yes, you see that I've rebound 
`C-x C-c` to switching buffers.  That key is
really easy to use and I don't think that 
`M-x kill-emacs` deserves it.  Keys
that are so easy to use should be kept for frequent actions, and quiting
emacs is a once-a-day to once-a-month action here.  And you can still quit
from the window manager button or from the menu or from 
`M-x`.

Also 
*Mac* users are not left behind, you will see some settings that either
are adapted to the system (like choosing another 
*font*, keep displaying the
`menu-bar` or not installing the darkish 
`tango-color-mode` on this system,
where it renders poorly in my opinion), as you can see here:

~~~
(if (string-match "apple-darwin" system-configuration)
    (set-face-font 'default "Monaco-13")
  (set-frame-font "Monospace-10"))

(when (string-match "apple-darwin" system-configuration)
  (setq mac-allow-anti-aliasing t)
  (setq mac-command-modifier 'meta)
  (setq mac-option-modifier 'none))
~~~


So all in all, I don't expect this 
`emacs-kicker` to please everyone, but I
expect it to be simple and rich enough (thanks to 
[el-get](https://github.com/dimitri/el-get)), and it should be
a good 
*kick start* that's easy to adapt.

If you want to try it without installing it it's very easy to do so.  Just
clone the 
`git` repository then start an 
`Emacs` that will use this.  For
example that could be, using the excellent 
[Emacs For MacOSX](http://emacsformacosx.com/):

~~~
$ /Applications/Emacs.app/Contents/MacOS/Emacs -Q -l init.el 
~~~


I hope some readers will find it useful! :)
