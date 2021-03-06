+++
date = "2009-09-07T12:50:00.000000+02:00"
title = "Follow-up on dim:mailrc-add-entry"
tags = ["Emacs", "prefix"]
categories = ["Emacs","Emacs Tips"]
thumbnailImage = "/img/old/emacs-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/emacs-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2009/09/07-follow-up-on-dimmailrc-add-entry",
           "/blog/2009/09/07-follow-up-on-dimmailrc-add-entry.html"]
+++

The function didn't allow for using more than one 
`mailrc` file, which isn't a
good idea, so I've just added that. Oh and for 
`gnus` integration what I need
is 
`(add-hook 'message-mode-hook 'mail-abbrevs-setup)` it seems... so that if
I type the alias it'll get automatically expanded. And to be real lazy and
avoid having to type in the entire alias, 
`mail-abbrev-complete-alias` to the
rescue, assigned to some easy to reach keys.

~~~
(require 'message)
(define-key message-mode-map (kbd "C-'") 'mail-abbrev-complete-alias)

(defun dim:mailrc-add-entry (&optional prefix alias)
  "read email at point and add it to an ~/.mailrc file"
  (interactive "P\nMalias: ")
  (let* ((default-mailrc (file-name-nondirectory mail-personal-alias-file))
	 (mailrc (if prefix (expand-file-name
			     (read-file-name 
			      "Add alias into file: " 
			      "~/" 
			      default-mailrc
			      t
			      default-mailrc))
		   mail-personal-alias-file))
	 (address (thing-at-point 'email-address))
	 (buffer (find-file-noselect mailrc t)))
    (when address
      (with-current-buffer buffer
	;; we don't support updating existing alias in the file
	(save-excursion
	  (goto-char (point-min))
	  (if (search-forward (concat "alias " alias) nil t)
	      (error "Alias %s is already present in .mailrc" alias)))

	(save-current-buffer
	  (save-excursion
	    (goto-char (point-max))
	    (insert (format "\nalias %s \"%s <%s>\"" alias (cdr address) (car address)))))))))
~~~

