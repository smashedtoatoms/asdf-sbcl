# asdf-sbcl ![Build](https://github.com/smashedtoatoms/asdf-sbcl/workflows/Build/badge.svg?branch=main)

Steel Bank Common Lisp plugin for [asdf](https://github.com/asdf-vm/asdf)
version manager.

Note: _This plugin currently only works on M1 macs in Rosetta mode.  Until SBCL
officially supports it, this will be the case.  To install, you will need to run
something like: `arch -x86_64 asdf install sbcl 2.0.11`_

This plugin installs SBCL from source. Depending on the speed of your system,
this can take some time.  In return, it will give you the flexibility to run and
manage different versions of SBCL, which will liberate you from the often
outdated official packages.

I have found [clpm](https://www.clpm.dev/) to be incredibly useful (It gives you
version pinning, git repo sources, quicklisp with pinning, hooks into ASDF
elegantly, and does it all without closing over things in a way that forces a
particular workflow or cruds up your standalone binaries).  I also use
[lake](https://github.com/takagi/lake) as a build tool for lisp projects.  I am
including them both by default, so you will not need to install or configure
either.  You can simply use them as normal command line utilities.  I HIGHLY
recommend reading clpm's docs to get familiar with why it's amazing, and why
everyone should be using it.

## Requirements

- [jq](https://stedolan.github.io/jq/)
- [curl](https://curl.haxx.se/)

## Recommended
- [rlwrap](https://github.com/hanslub42/rlwrap)

## Install

```
asdf plugin-add sbcl https://github.com/smashedtoatoms/asdf-sbcl.git
```

## Asdf-vm Usage

List candidate SBCLs:

```
asdf list-all sbcl
```

Install a candidate listed from the previous command like this:

```
asdf install sbcl 2.0.11
```

Select an installed candidate for use like this:

```
asdf global sbcl 2.0.11
```

## Custom compile flags

The default install will use `--with-sb-core-compression and --with-sb-thread`
so that you can create freestanding images for distribution that use multiple
cores. If you want different compile-time flags, you can set them by using the
following syntax on install:

```
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.0.10
```

## Disable CLPM and Lake

To disable clpm and lake, you can set `SKIP_INSTALLING_CLPM=true` and
`SKIP_INSTALLING_LAKE=true`.  For example, to install sbcl 2.0.11 with core
compression and threading and exclude lake, you would run:

```
SKIP_INSTALLING_LAKE=true \
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.0.11
```

## SBCL Tricks

Sbcl will work like normal following installation; however, If you want all the
perks, I recommend installing rlwrap and using clpm to start sbcl.

- You can start a swank repl on port 4005 which you can then hook an editor to
  (I use vscode with
  [Alive](https://marketplace.visualstudio.com/items?itemName=rheller.alive)):
    ```
    rlwrap clpm exec --context=your-project-name -- sbcl --eval '(asdf:load-system :swank)' --eval '(swank:create-server)'
    ```
- You can build a standalone binary (lake for this example) using clpm (which
  installs any required dependencies automatically and compresses the binary
  without including any unnecessary libraries in your binary) by doing something
  like this:
    ```
    # From within the lake project...
    clpm exec --context=lake -- sbcl \
        --eval '(setf clpm-client:*asdf-system-not-found-behavior* :install)' \
        --eval '(setf clpm-client:*context-diff-approval-method* t)' \
        --eval '(asdf:load-system :lake)' \
        --eval "(sb-ext:save-lisp-and-die \"lake\" :toplevel #'lake/main:uiop-main :executable t :compression 9)"
    ```
- clpm [caches files](https://common-lisp.net/project/clpm/docs/storage.html) in
  a few different places.  If things misbehave, it doesn't hurt to blow those
  caches with `rm -rf ~/.cache/clpm ~/.local/share/clpm`

## Possible issues
The addition of clpm and lake is relatively new.  clpm is configured to use the
[ql-clpi
source](https://gitlab.common-lisp.net/clpm/clpm/-/blob/master/docs/sources.org),
which allows access to quicklisp but also allows sourcing GitHub repos
(including private ones) and allows version specification and pinning, as well
as version locking per project.  It is integrated with asdf and intercepts the
exception caused when a library can't be found and will ask if you want to
install the library in those cases.  The way it is added to the ASDF
*central-registry* may cause unexpected behavior if you have a lot of custom
configuration in your .sbclrc file.
