+++
date = "2010-08-27T14:15:00.000000+02:00"
title = "welcome el-get scratch installer"
tags = ["Emacs", "el-get"]
categories = ["Emacs","el-get"]
thumbnailImage = "/img/old/el-get.big.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/el-get.big.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/08/27-welcome-el-get-scratch-installer",
           "/blog/2010/08/27-welcome-el-get-scratch-installer.html"]
+++

A very good remark from some users: installing and managing 
`el-get` should be
simpler. They wanted both an easy install of the thing, and a way to be able
to manage it afterwards (like, update the local copy against the
authoritative source). So I decided it was high time for getting the code
out of my 
`~/.emacs.d` git repository and up to a public place:
[http://github.com/dimitri/el-get](http://github.com/dimitri/el-get).

Then, I added some documentation (a 
`README`), and then, a 
`*scratch*
installer`, following great ideas from 
`ELPA`. So have at it, it's a copy paste
away! 

Don't forget to setup your 
`el-get-sources` and include there the 
`el-get`
source for updates, there's nothing magic about it so it's up to you. You
may notice that it's not yet possible to init 
`el-get` from 
`el-get-sources`,
though, that's the drawback of the lack of magic. So you will have to still
add an explicit 
`(require 'el-get)` before to go and define you own
`el-get-sources` then finally 
`(el-get)`. I don't think that's a problem I need
to solve, though.
