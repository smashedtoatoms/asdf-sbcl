# asdf-sbcl ![Build](https://github.com/smashedtoatoms/asdf-sbcl/workflows/Build/badge.svg?branch=main)

Steel Bank Common Lisp plugin for [asdf-vm](https://github.com/asdf-vm/asdf)
version manager.

This plugin installs SBCL from source. Depending on the speed of your system,
this can take some time.  In return, it will give you the flexibility to run and
manage different versions of SBCL, which will liberate you from the often
outdated official packages.

### Requirements
- [jq](https://stedolan.github.io/jq/)
- [curl](https://curl.haxx.se/)

### Recommended
- [rlwrap](https://github.com/hanslub42/rlwrap)

### Install Plugin

```
asdf plugin-add sbcl https://github.com/smashedtoatoms/asdf-sbcl.git
```

### Asdf-vm Usage

List candidate SBCLs:

```
asdf list-all sbcl
```

Install a candidate listed from the previous command like this:

```
asdf install sbcl 2.1.5
```

Select an installed candidate for use like this:

```
asdf global sbcl 2.1.5
```

### Custom compile flags

The default install will use `--with-sb-core-compression --with-sb-thread
--with-sb-linkable-runtime --with-sb-rotate-byte` so that you can create
freestanding images for distribution that use multiple cores and use crypto. If
you want different compile-time flags you can set them by using the following
syntax on install:

```
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.1.5
```

## CLPM

I have found [CLPM](https://www.clpm.dev/) to be incredibly useful.  It gives
you version pinning.  You can use git repos as sources.  You can pin quicklisp
dependencies.  It hooks into ASDF elegantly so that when a dependency is
missing, you can fix it in the debugger.  It does all of this without closing
over your lisp environment in a way that forces a particular workflow.  It
doesn't crud up your standalone binaries.  It doesn't force the consumers of
your library to adopt your workflow.  I HIGHLY recommend reading CLPM's docs to
get familiar with why it's excellent, and why everyone should be using it.

I have written about how to use SBCL with CLPM and VSCode
[here](https://smashedtoatoms.com/dev-life/sbcl-with-vscode-via-clpm-2021/) and
why I like it
[here](https://smashedtoatoms.com/posts/2021-03-05t174019-0700-starting/)

I used to include it by default, but I have since removed it because there isn't
a good way to include it in this installer that doesn't affect other things.
If, after installing sbcl, you decide that you want to install CLPM, do this:

1. [Install the clpm binary](https://www.clpm.dev/#installing)
   - Note: there isn't a clpm binary for the M1 Mac yet.  If you need one now,
     you can either build it yourself from the source, or use [this one that I
     built](https://smashedtoatoms.com/rando/clpm-0.4.0-alpha.1-darwin-arm64.tar.gz)
     if you trust me.  If you choose to run this and get errors about clpm not being
     able to find openssl libraries that looks like either of these:
     ```sh
     Error opening shared object "/opt/local/lib/libcrypto.dylib"
     Error opening shared object "/opt/local/lib/libssl.dylib"
     ```
     you may need to add links to openssl in places where it is looking.  This is 
     a problem with sbcl/clpm that has been fixed but hasn't been mainlined yet.  
     ```sh
     sudo ln -s /opt/homebrew/opt/openssl/lib/libcrypto.dylib /opt/local/lib/
     sudo ln -s /opt/homebrew/opt/openssl/lib/libssl.dylib /opt/local/lib/
     ```
2. Configure CLPM
   - If you're comfortable running scripts off of the internet, you can run
      this:
      ```sh
      /bin/bash -c "$(curl -fsSL https://smashedtoatoms.com/rando/configure-clpm-to-work-with-sbcl.sh)"
      ```
   - If you want to do it manually, [follow these
      instructions](https://www.clpm.dev/#installing)

If you decide you want to remove the clpm config and clear the cached data, you can do this:
```sh
wget https://smashedtoatoms.com/rando/configure-clpm-to-work-with-sbcl.sh
chmod 755 configure-clpm-to-work-with-sbcl.sh
./configure-clpm-to-work-with-sbcl.sh cleanup
```

### CLPM Tricks

Sbcl will work like normal following installation; however, If you want all the
perks, I recommend installing rlwrap and CPLM, and use CLPM to start sbcl.  Once
you have all of those in place, you can do stuff like this:

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

### Possible CLPM issues
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
