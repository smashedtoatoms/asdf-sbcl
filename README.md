# asdf-sbcl

Steel Bank Common Lisp plugin for [asdf](https://github.com/asdf-vm/asdf) version manager.

This plugin compiles SBCL from source. Depending on the speed of your
system, this can take some time. In return, it will give you the
flexibility to run and manage versions of SBCL you'd like to run,
which will liberate you from the often outdated packages published for
your system.  

Since creating this project, I have discovered that 
[roswell](https://github.com/roswell/roswell) is actually a better
way to manage common lisp  and plugins for me.  The being said, the way 
that roswell does the initial build on mac relies on the published
latest binary from sbcl.  For mac, that version is ancient 1.2.11, which
is incompatible with lots of stuff these days, including qlot.  For 
that reason, I still recommend roswell for managing lisp versions and 
plugins, but on mac, you can use asdf-sbcl to install the latest version 
of sbcl, which you can then use inside of roswell to make your life 
easier.  You could also just compile sbcl from source to accomplish the
same thing.  To do what I just explained, you can do the following:

- Manually install sbcl 2.0.7 with `https://github.com/smashedtoatoms/asdf-sbcl`
   - `asdf plugin add sbcl`
   - `SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" asdf install sbcl 2.0.7`
- Install roswell with brew and initialize it (which will install sbcl 1.2.11 by default on mac).
   - `brew install roswell && ros`
- Update default sbcl in roswell from 1.2.11 to 2.0.7
   - `mv ~/.asdf/installs/sbcl/2.0.7 ~/.roswell/impls/x86-64/darwin/sbcl-bin`
   - `ros config set sbcl-bin.version 2.0.7`
- Install qlot
   - `ros install qlot`

Now you can install [qlot](https://github.com/fukamachi/qlot) and have
a very elegant way to handle cross-platform installs and get
repeatable builds.

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

## Custom compile flags

The default install will use `--with-sb-core-compression` so that you can create freestanding images for distribution.
If you want different compile-time flags, you can set them by using the following syntax on install:

`SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" asdf install sbcl 2.0.3`
