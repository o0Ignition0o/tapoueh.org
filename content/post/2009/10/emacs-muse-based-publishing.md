+++
date = "2009-10-06T17:23:00.000000+02:00"
title = "Emacs Muse based publishing"
tags = ["Emacs", "Muse"]
categories = ["Software Programming","Emacs Lisp"]
thumbnailImage = "/img/old/emacs-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/emacs-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2009/10/06-emacs-muse-based-publishing",
           "/blog/2009/10/06-emacs-muse-based-publishing.html"]
+++

As you might have noticed, this little blog of mine is not compromising much
and entirely maintained from Emacs. Until today, I had to resort to 
`term` to
upload my publications, though, as I've been too lazy to hack up the tools
integration for simply doing a single 
`rsync` command line. That was one time
to many:

~~~
(defvar dim:muse-rsync-options "-avz"
  "rsync options")

(defvar dim:muse-rsync-source "~/dev/muse/out"
  "local path from where to rsync, with no ending /")

(defvar dim:muse-rsync-target
  "dim@tapoueh.org:/home/www/tapoueh.org/blog.tapoueh.org"
  "Remote URL to use as rsync target, with no ending /")

(defvar dim:muse-rsync-extra-subdirs
  '("../css" "../images" "../pdf")
  "static subdirs to rsync too, path from dim:muse-rsync-source, no ending /")

(defun dim:muse-project-rsync (&optional static)
  "publish tapoueh.org using rsync"
  (interactive "P")
  (let* ((rsync-command (format "rsync %s %s %s" 
				dim:muse-rsync-options
				(concat dim:muse-rsync-source "/")
				(concat dim:muse-rsync-target "/"))))
    (with-current-buffer (get-buffer-create "*muse-rsync*")
      (erase-buffer)
      (insert (concat rsync-command "\n"))
      (message "%s" rsync-command)
      (insert (shell-command-to-string rsync-command))
      (insert "\n")

      (when static
	(dolist (subdir dim:muse-rsync-extra-subdirs)
	  (let ((cmd (format "rsync %s %s %s" 
			     dim:muse-rsync-options
			     (concat dim:muse-rsync-source "/" subdir)
			     dim:muse-rsync-target)))
	    (insert (concat cmd "\n"))
	    (message "%s" cmd)
	    (insert (shell-command-to-string cmd))
	    (insert "\n")))))))

(define-key muse-mode-map (kbd "C-c R") 'dim:muse-project-rsync)
~~~


So now to publish this blog, it's just a 
`C-c R` away! :)
