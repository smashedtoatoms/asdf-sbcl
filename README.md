# asdf-sbcl

Steel Bank Common Lisp plugin for [asdf](https://github.com/asdf-vm/asdf) version manager.

This plugin compiles SBCL from source. Depending on the speed of your
system, this can take some time. In return, it will give you the
flexibility to run and manage versions of SBCL you'd like to run,
which will liberate you from the often outdated packages published for
your system.

It also installs [quicklisp](https://www.quicklisp.org/beta/) and
[qlot](https://github.com/fukamachi/qlot) for the installed lisp because I am
tired of messing with something that should be part of the default install.
Every version of sbcl that you install will get its own associated quicklisp
and qlot that is not shared across installs. If you need their installation
to be configurable, let me know. As of now, I think I am the only person
using this.

## Requirements

- [jq](https://stedolan.github.io/jq/)
- [curl](https://curl.haxx.se/)

## Install

```
asdf plugin-add sbcl https://github.com/smashedtoatoms/asdf-sbcl.git
```

## Use

List candidate SBCLs:

`asdf list-all sbcl`

Install a candidate listed from the previous command like this:

`asdf install sbcl 2.0.0`

Select an installed candidate for use like this:

`asdf global sbcl 2.0.0`
