# Heroku buildpack: Emacs Lisp

This is an Emacs Lisp Heroku buildpack. It is probably only useful
with [Elnode](http://www.emacswiki.org/emacs/Elnode).

## Usage

```lisp
;;; init.el
(require 'elnode (concat load-file-name "elnode.el"))

(defun handler (httpcon)
  "Demonstration function"
  (elnode-http-start httpcon "200"
                     '("Content-type" . "text/html")
                     `("Server" . ,(concat "GNU Emacs " emacs-version)))
  (elnode-http-return httpcon "<html><body><h1>Hello from Emacs!</h1></body></html>"))

(elnode-start 'handler (string-to-number (or (getenv "PORT") "8080")) "0.0.0.0")

(while t (accept-process-output nil 1))
```

    # requires heroku gem version 2.11.0+
    $ heroku create --stack cedar --buildpack http://github.com/technomancy/heroku-buildpack-emacs.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Emacs Lisp app detected
    -----> Downloading Emacs 24.0.50.1
           Downloading Emacs 24 pretest from http://p.hagelb.org/emacs.tar.gz...
           ...done

The buildpack will detect that your app has an `init.el` in the root
and download a copy of the Emacs 24 pretest to bundle in your slug.
The Procfile for Emacs Lisp apps currently needs to untar that copy of
Emacs before running and symlink it to /tmp/emacs, like this:

    web: tar xzf emacs.tar.gz && ln -s $PWD/emacs /tmp/emacs; emacs/bin/emacs --daemon --load init.el

This is obviously undesired since it means other users could interfere
with this symlink, so a way to compile Emacs without referring to the
/tmp directory needs to be discovered. Until then, don't use this
buildpack for anything you would need to keep secure.

The current Emacs tarball is compiled from the pretest of version 24
with this invocation:

    $ ./configure --without-x --without-sound --without-xpm \
    --without-jpeg --without-tiff --without-gif --without-png \
    --without-rsvg --without-xml2 --without-imagemagick --without-xft \
    --without-toolkit-scroll-bars --without-gconf --without-dbus \
    --without-gsettings --without-selinux --without-gnutls --without-gpm \
    --without-makeinfo --without-compress-info --prefix=/tmp/emacs
