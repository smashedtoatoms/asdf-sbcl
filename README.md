# asdf-sbcl

Steel Bank Common Lisp plugin for [asdf](https://github.com/asdf-vm/asdf) version manager.

This plugin compiles SBCL from source. Depending on the speed of your
system, this can take some time. In return, it will give you the
flexibility to run and manage versions of SBCL you'd like to run,
which will liberate you from the often outdated packages published for
your system.

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

`asdf install sbcl 1.5.7`

Select an installed candidate for use like this:

`asdf global sbcl 1.5.7`
