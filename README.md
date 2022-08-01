# asdf-sbcl ![Build](https://github.com/smashedtoatoms/asdf-sbcl/workflows/Build/badge.svg?branch=main)

Steel Bank Common Lisp plugin for [asdf-vm](https://github.com/asdf-vm/asdf)
version manager.

This plugin installs SBCL from source. Depending on the speed of your
system, this can take some time. In return, it will give you the
flexibility to run and manage different versions of SBCL with
configurable build options. This will liberate you from the tyranny of
outdated official packages or missing compile-time flags that you might
need.

It chooses a few useful options by default to make it easier to generate
compressed images of multithreaded apps that make use of external
libraries and crypto (mostly so you can generate small binaries that use
openssl out of the box):

- `--with-sb-core-compression` for creating compressable images
- `--with-sb-thread` for threading
- `--with-sb-linkable-runtime` for linking outside libs like openssl
- `--with-sb-rotate-byte` for cryptography and hashing

All versions from 2.1.2 to 2.1.11 have a patch applied to allow the
linking of external libs on M1 macs. To the best of my knowledge,
versions >= 2.2.0 on no longer require the patch.

`--with-sb-core-compression` relies on zstd for compression in the
latest versions of SBCL. You will need to make sure you have zstd
installed via your favorite package manager and make sure your CPATH and
LIBRARY_PATH point to them. If you're on a mac, you also need to have
your command line tools in your path in your ~/.zshrc. For example, if
you're using brew on an M1 or M2 mac, install zstd with `brew install zstd` and then install sbcl with `CPATH=/opt/homebrew/include:$CPATH LIBRARY_PATH=/opt/homebrew/lib:$LIBRARY_PATH PATH=/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib:$PATH asdf install 2.2.7`. Be sure to thank apple for this hellscape. If
you're using asdf, install zstd with `sudo apt-get install libzstd-dev`
and let it rip. Other distros will need to install it in whatever way
they normally install.

### Requirements

- [jq](https://stedolan.github.io/jq/)
- [curl](https://curl.haxx.se/)
- [zstd](https://github.com/facebook/zstd)

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
asdf install sbcl 2.2.7
```

Select an installed candidate for use like this:

```
asdf global sbcl 2.2.7
```

### Custom compile flags

The default install will use `--with-sb-core-compression --with-sb-thread --with-sb-linkable-runtime --with-sb-rotate-byte`. If
you want different compile-time flags, you can set them by using the
following syntax on install:

```
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.2.0
```

## CLPM

I have found [CLPM](https://www.clpm.dev/) to be incredibly useful. It gives
you version pinning. You can use git repos as sources. You can pin quicklisp
dependencies. It hooks into ASDF elegantly so that when a dependency is
missing, you can fix it in the debugger. It does all of this without closing
over your lisp environment in a way that forces a particular workflow. It
doesn't crud up your standalone binaries. It doesn't force the consumers of
your library to adopt your workflow. I HIGHLY recommend reading CLPM's docs to
get familiar with why it's excellent and why everyone should be using it.

I used to include it by default, but I have since removed it because
there isn't a good way to include it in this installer without
hard-to-manage side effects. If, after installing sbcl, you decide that
you want to install CLPM, do this:

1. [Install the clpm binary](https://www.clpm.dev/#installing)
2. Configure CLPM
   - If you're comfortable running scripts off of the internet, you can run
     this:
     ```sh
     curl -L https://smashedtoatoms.com/rando/configure-clpm-to-work-with-sbcl.sh | bash
     ```
   - If you want to do it manually, [follow these
     instructions](https://www.clpm.dev/#installing)

If you decide you want to remove the clpm config and clear the cached data, you can do this:

```sh
curl -L https://smashedtoatoms.com/rando/configure-clpm-to-work-with-sbcl.sh | bash -s -- cleanup
```

### CLPM Tricks

- You can start a swank repl on port 4005 which you can then hook an editor to
  (I use vscode with
  [Alive](https://marketplace.visualstudio.com/items?itemName=rheller.alive)):
  ```
  clpm exec --context=your-project-name -- sbcl \
      --eval '(setf clpm-client:*asdf-system-not-found-behavior* :install)' \
      --eval '(setf clpm-client:*context-diff-approval-method* t)' \
      --eval '(asdf:load-system :swank)' \
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
  a few different places. If things misbehave, it doesn't hurt to blow those
  caches with `rm -rf ~/.cache/clpm ~/.local/share/clpm`
- Places where CLPM puts configuration:
  - ~/.config/clpm
  - ~/.config/common-lisp
  - ~/.cache/clpm
  - ~/.local/share/clpm
  - ~/.sbclrc

### Possible CLPM issues

CLPM dynamically links to OpenSSL. OpenSSL on Macs is a dumpster fire.
With SBCL versions between 2.1.2 and 2.1.7 it will most certainly have
problems (no M1 support pre 2.1.2 and it was fixed in 2.1.8). Those
versions check [the following
paths](https://github.com/cl-plus-ssl/cl-plus-ssl/blob/5aed9cabc2a6394d9e35e377f154d8c882b865eb/src/reload.lisp#L44).
If it can't find your openssl lib, it will throw an error that looks
like this:

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

If you encounter this, you need to get your libcrypto.dylib and
libssl.dylib into one of [the paths
specified](https://github.com/cl-plus-ssl/cl-plus-ssl/blob/5aed9cabc2a6394d9e35e377f154d8c882b865eb/src/reload.lisp#L44).
If you're using [MacPorts](https://www.macports.org), you likely won't
have an issue. If you're using [Homebrew](https://brew.sh), you're
probably going to need to symlink to /opt/local/lib. For example, I did
this as a workaround:

```sh
sudo mkdir -p /opt/local
sudo ln -s /opt/homebrew/opt/openssl/lib/libcrypto.dylib /opt/local/lib/
sudo ln -s /opt/homebrew/opt/openssl/lib/libssl.dylib /opt/local/lib/
```
