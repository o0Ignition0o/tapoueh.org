+++
date = "2013-02-08T17:15:00.000000+01:00"
title = "Marking whole word"
tags = ["Emacs"]
categories = ["Emacs","Emacs Tips"]
thumbnailImage = "/img/old/sexp.gif"
thumbnailImagePosition = "left"
coverImage = "/img/old/sexp.gif"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2013/02/04-Emacs-mark-word",
           "/blog/2013/02/04-Emacs-mark-word.html"]
+++

I've discovered recently another Emacs facility that I since then use
several times a day, and I wonder how I did without it before: 
`C-M-SPC runs
the command mark-sexp`.

<center>*Well, `mark-sexp` apparently is related to the Sex Pistols*</center>

It's pretty simple actually, when you have the 
*point* at the beginning of a
word or an identifier (containing numbers, dashes, underscores and other
punctuation signs), you can select the 
*whole* of it in a single key chord!

The best thing is that if you press the same key chord again, it will expand
to include the next expression. And that works in plain text and most
programming languages where I've tried it, which is not so much recently. It
does not depend that much on the programming language anyway.

The full general solution here is to use something like 
[expand region](https://github.com/magnars/expand-region.el), don't
miss the 
[Emacs Rocks Expand Region Episode](http://emacsrocks.com/e09.html), it's less than 3 minutes and you
will want to install 
*expand-region* after that. For easy installing, of
course you are already using 
[el-get](http://tapoueh.org/emacs/el-get.html) right?

Now, a friend just asked this morning how to select the 
*current word* even
when the the point is currently in the middle of it. Going manually back to
the beginning of it is no fun. I knew about 
`thing-at-point` and a little
about how it works, but didn't find anything readily made for that use case
(hint: it needs to be an 
*interactive* command).

Here's what I came up with, then:

~~~
(defun mha:select-current-word ()
  "Select the current word."
  (interactive)
  (beginning-of-thing 'symbol)
  (push-mark (point) nil t)
  (end-of-thing 'symbol)
  (exchange-point-and-mark))

(global-set-key (kbd "C-S-M-SPC") 'mha:select-current-word)
~~~


I picked 
`C-M-S-SPC` not because it's the easiest way to invoke the new
command, but because to me it's a quite natural extension to the 
`C-M-SPC`
that I use so often. Again, each time you want to 
*select* a identifier in
some code of yours, you'd most certainly be better off using 
`C-M-SPC`.
