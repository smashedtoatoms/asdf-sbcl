# asdf-sbcl ![Build](https://github.com/smashedtoatoms/asdf-sbcl/workflows/Build/badge.svg?branch=main)

Steel Bank Common Lisp plugin for [asdf-vm](https://github.com/asdf-vm/asdf)
version manager.

You're honestly probably better served loading it from your local
package manager.  Unless you need the linkable runtime or cryptography,
just use your package manager.

This plugin installs SBCL from source. Depending on the speed of your
system, this can take some time. In return, it will give you the
flexibility to run and manage different versions of SBCL with
configurable build options. This will liberate you from the tyranny of
outdated official packages or missing compile-time flags that you might
need.

SBCL is compiled using itself, or any other Common Lisp.  I would love
to use a publicly available older SBCL version to bootstrap the new one
like i have done for years, and as I still do for linux, but on MacOS
Ventura the old builds don't run anymore due to mmap errors.  To deal
with that, I now use ecl for mac builds which is an embeddable common
lisp that is widely available via package managers.

I have chosen a few useful options by default to make it easier to
generate compressed images of multithreaded apps that make use of
external libraries and crypto (mostly so you can generate small binaries
that use openssl out of the box):

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
you're using brew on an M1 or M2 mac, install zstd with `brew install zstd` and then install sbcl with `CPATH=/opt/homebrew/include:$CPATH LIBRARY_PATH=/opt/homebrew/lib:$LIBRARY_PATH PATH=/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib:$PATH asdf install 2.3.9`. Be sure to thank apple for this hellscape. If
you're using asdf, install zstd with `sudo apt-get install libzstd-dev`
and let it rip. Other distros will need to install it in whatever way
they normally install.

### Requirements

- [jq](https://stedolan.github.io/jq/)
- [curl](https://curl.haxx.se/)
- [libzstd-dev](https://github.com/facebook/zstd)
- [ecl](https://ecl.common-lisp.dev) on macs

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
asdf install sbcl 2.3.9
```

Select an installed candidate for use like this:

```
asdf global sbcl 2.3.9
```

### Custom compile flags

The default install will use `--with-sb-core-compression --with-sb-thread --with-sb-linkable-runtime --with-sb-rotate-byte`. If
you want different compile-time flags, you can set them by using the
following syntax on install:

```
SBCL_CONFIGURE_OPTIONS="--with-sb-core-compression --with-sb-thread" \
asdf install sbcl 2.3.9
```
