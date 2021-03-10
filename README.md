# asdf-sbcl ![Build](https://github.com/smashedtoatoms/asdf-sbcl/workflows/Build/badge.svg?branch=main)

Steel Bank Common Lisp plugin for [asdf-vm](https://github.com/asdf-vm/asdf)
version manager.

This plugin installs SBCL from source. Depending on the speed of your system,
this can take some time.  In return, it will give you the flexibility to run and
manage different versions of SBCL, which will liberate you from the often
outdated official packages.

I have found [CLPM](https://www.clpm.dev/) to be incredibly useful.  It gives
you version pinning.  You can use git repos as sources.  You can pin quicklisp
dependencies.  It hooks into ASDF elegantly so that when a dependency is
missing, you can fix it in the debugger.  It does all of this without closing
over your lisp environment in a way that forces a particular workflow.  It
doesn't crud up your standalone binaries.  It doesn't force the consumers of
your library to adopt your workflow.  I have included it by default, so you will
not need to install or configure it.  I suspect I will end up pulling it out
though since it really should be separate from this installer.  I HIGHLY 
recommend reading CLPM's docs to get familiar with why it's excellent, and why
everyone should be using it.  All of that said, if you don't want to install 
it, you can exclude it by setting the SKIP_INSTALLING_CLPM environment to 1 
or true.  There are examples below.

## Requirements
- [jq](https://stedolan.github.io/jq/)
- [curl](https://curl.haxx.se/)

## Recommended
- [rlwrap](https://github.com/hanslub42/rlwrap)

## Install Plugin

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
asdf install sbcl 2.1.2
```

Select an installed candidate for use like this:

```
asdf global sbcl 2.1.2
```

## Custom compile flags

The default install will use `--with-sb-core-compression --with-sb-thread
--with-sb-linkable-runtime` so that you can create freestanding images for
distribution that use multiple cores. If you want different compile-time flags,
you can set them by using the following syntax on install:

```
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.0.10
```

## Disable CLPM

To disable CLPM, you can set `SKIP_INSTALLING_CLPM=true`.  For example, to
install sbcl 2.1.2 with core compression and threading and exclude CLPM, you
would run:

```
SKIP_INSTALLING_CLPM=true \
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.1.2
```

## SBCL Tricks

Sbcl will work like normal following installation; however, If you want all the
perks, I recommend installing rlwrap and using CLPM to start sbcl.

- You can start a swank repl on port 4005 which you can then hook an editor to
  (I use vscode with
  [Alive](https://marketplace.visualstudio.com/items?itemName=rheller.alive)):
    ```
    rlwrap clpm exec --context=your-project-name -- sbcl \
        --eval '(setf clpm-client:*asdf-system-not-found-behavior* :install)' \
        --eval '(setf clpm-client:*context-diff-approval-method* t)' \
        --eval '(asdf:load-system :swank)'
        --eval '(swank:create-server)'
    ```
- You can build a standalone binary ([lake](https://github.com/takagi/lake) for
  this example) using CLPM (which installs any required dependencies
  automatically and compresses the binary without including any unnecessary
  libraries in your binary) by doing something like this:
    ```
    # From within the lake project...
    clpm exec --context=lake -- sbcl \
        --eval '(setf clpm-client:*asdf-system-not-found-behavior* :install)' \
        --eval '(setf clpm-client:*context-diff-approval-method* t)' \
        --eval '(asdf:load-system :lake)' \
        --eval '(asdf:clear-system :clpm-client)' \
        --eval "(sb-ext:save-lisp-and-die \"lake\" :toplevel #'lake/main:uiop-main :executable t :compression 9)"
    ```
- CLPM [caches files](https://common-lisp.net/project/clpm/docs/storage.html) in
  a few different places.  If things misbehave, it doesn't hurt to blow those
  caches with `rm -rf ~/.cache/clpm ~/.local/share/clpm`
- Places where CLPM puts configuration:
  - ~/.config/clpm
  - ~/.config/common-lisp
  - ~/.cache/clpm
  - ~/.local/share/clpm
  - ~/.sbclrc

## Possible CLPM issues
CLPM dynamically links to OpenSSL.  OpenSSL on Macs is a dumpster fire.  It
currently checks
[the following paths](https://github.com/cl-plus-ssl/cl-plus-ssl/blob/5aed9cabc2a6394d9e35e377f154d8c882b865eb/src/reload.lisp#L44).
If it can't find your openssl lib, it will throw an error that looks like this:
```sh
Unhandled SIMPLE-ERROR in thread #<SB-THREAD:THREAD "main thread" RUNNING
                                    {7008F30243}>:
  Error opening shared object "/opt/local/lib/libcrypto.dylib":
  dlopen(/opt/local/lib/libcrypto.dylib, 10): image not found.

Backtrace for: #<SB-THREAD:THREAD "main thread" RUNNING {7008F30243}>
0: (SB-DEBUG::DEBUGGER-DISABLED-HOOK #<SIMPLE-ERROR "Error opening ~:[runtime~;shared object ~:*~S~]:
  ~A." {70033701C3}> #<unused argument> :QUIT T)
1: (SB-DEBUG::RUN-HOOK SB-EXT:*INVOKE-DEBUGGER-HOOK* #<SIMPLE-ERROR "Error opening ~:[runtime~;shared object ~:*~S~]:
  ~A." {70033701C3}>)
2: (INVOKE-DEBUGGER #<SIMPLE-ERROR "Error opening ~:[runtime~;shared object ~:*~S~]:
  ~A." {70033701C3}>)
3: (ERROR "Error opening ~:[runtime~;shared object ~:*~S~]:
  ~A." "/opt/local/lib/libcrypto.dylib" "dlopen(/opt/local/lib/libcrypto.dylib, 10): image not found")
4: (SB-SYS:DLOPEN-OR-LOSE #S(SB-ALIEN::SHARED-OBJECT :PATHNAME #P"/opt/local/lib/libcrypto.dylib" :NAMESTRING "/opt/local/lib/libcrypto.dylib" :HANDLE NIL :DONT-SAVE NIL))
5: (SB-ALIEN::TRY-REOPEN-SHARED-OBJECT #S(SB-ALIEN::SHARED-OBJECT :PATHNAME #P"/opt/local/lib/libcrypto.dylib" :NAMESTRING "/opt/local/lib/libcrypto.dylib" :HANDLE NIL :DONT-SAVE NIL))
6: (SB-SYS:REOPEN-SHARED-OBJECTS)
7: (SB-IMPL::FOREIGN-REINIT)
8: (SB-IMPL::REINIT T)
9: ((FLET SB-UNIX::BODY :IN SB-IMPL::START-LISP))
10: ((FLET "WITHOUT-INTERRUPTS-BODY-1" :IN SB-IMPL::START-LISP))
11: (SB-IMPL::START-LISP)

unhandled condition in --disable-debugger mode, quitting
```
If you encounter this, you need to get your libcrypto.dylib into one of [the
paths
specified](https://github.com/cl-plus-ssl/cl-plus-ssl/blob/5aed9cabc2a6394d9e35e377f154d8c882b865eb/src/reload.lisp#L44).
If you're using [MacPorts](https://www.macports.org), you likely won't have an
issue.  If you're using [Homebrew](https://brew.sh) on arm64, you're probably
going to need to symlink to the macports path or something until the explicit
pathing set
[here](https://github.com/cl-plus-ssl/cl-plus-ssl/blob/5aed9cabc2a6394d9e35e377f154d8c882b865eb/src/reload.lisp#L44)
is updated.  For example, I did this as a workaround:
```sh
sudo mkdir -p /opt/local && sudo ln -s /opt/homebrew/opt/openssl/lib /opt/local/lib
```
