+++
date = "2010-09-23T09:30:00.000000+02:00"
title = "Scratch that itch: M-x mailq"
tags = ["el-get", "cssh", "mailq", "postfix"]
categories = ["Software Programming","Emacs Lisp"]
thumbnailImage = "/img/old/el-get.big.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/el-get.big.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2010/09/23-scratch-that-itch-m-x-mailq",
           "/blog/2010/09/23-scratch-that-itch-m-x-mailq.html"]
+++

Nowadays, most people would think that email is something simple, you just
setup your preferred client (that's called a 
`MUA`) with some information such
as the 
`smtp` host you want it to talk to (that's call a 
`MTA` and this one is
your 
`relayhost`). Then there's all the receiving mails part, and that's 
`smtp`
again on the server side. Then there's how to get those mail, read them,
flag them, manage them, and that's better served by 
`IMAP`. Let's talk about
sending mails in 
`smtp` for this entry.

The traditional way to handle mail sending is to have your own 
`MTA` on each
system you use — there used to be a 
*sysadmin* team caring about all those
systems, but we're lost in the personal computer era now — that only means
***you*** are the sysadmin. So about any Unix tool that wants to send a mail will
do so with the command 
`/usr/bin/sendmail` to queue the outgoing message.

My typical 
*workstation* setup includes a full-blown 
`MTA` (my choice is
[Postfix](http://www.postfix.com/)) that will choose the next relay host depending on the message 
*From*
field: I don't want to trust any local default relayhost. Note that the next
relay is connected to with authentication and over an encrypted protocol.

> We're getting there, really. But I don't know a better way to present a
> software, little as it be, other than talking about the need that leads to
> its development.


Some relaying I do atop an 
`ssh` tunnel, and it happens that I send mail and
have forgotten about setting up the aforementioned tunnel. In this case, the
advantage is that it will not block my 
`MUA` (
[gnus](http://gnus.org/), in quite good shape those
days, receiving lots of love), as the queueing happens as usual. The
drawback is that 
[Postfix](http://www.postfix.com/) will 
*silently* queue the mail until it's able to
deliver it, which can take days.

Enters 
`M-x mailq`! Ok, I could be doing 
`M-! mailq` and see 
*Mail queue is empty*
in the message area, but then as soon as the queue's not empty I need to
resort to some 
*shell* or 
*terminal* in order to 
*flush* the queue — that's after
setting up the tunnel, as easy as 
`C-= remote` in my case, see
[cssh](http://github.com/dimitri/cssh). Scratching that itch, I now only have to hit 
`f` here, to flush the
queue. And from the 
*gnus* 
`*Group*` and 
`*Summary*` buffers, it's 
`M-q` to see the
mail queue.

Thanks to 
[http://forum.ubuntu-fr.org/viewtopic.php?id=218883](http://forum.ubuntu-fr.org/viewtopic.php?id=218883) here's a visual
sample of the 
`mailq` mode, where you see the mail queue in colors and the
*keymap* you're offered.

<center>
{{< image classes="fig50 fancybox dim-margin" src="/img/old/mailq-el.png" >}}
</center>

So you could even 
*flush* only a given 
`queue id` or a given 
`site`, or just 
*kill*
the current 
`id` or the current 
`site` so that it's a 
`C-y` away. I hope it's
useful for you too — oh, and it's already in the 
[el-get](http://github.com/dimitri/el-get) recipes, of course!
