# asdf-sbcl

Steel Bank Common Lisp plugin for [asdf](https://github.com/asdf-vm/asdf) version manager.

This plugin compiles from source... so it takes awhile, but it does let you run the exact version
that you want to run instead of whatever is published.  Be aware that it installs a temporary binary version of sbcl to use as the compiler for the version you actually want to run.  It's meta and it cleans up after itself!

## Install

```
asdf plugin-add sbcl https://github.com/smashedtoatoms/asdf-sbcl.git
```

## ASDF options

Check [asdf](https://github.com/asdf-vm/asdf) readme for instructions on how to install & manage versions of Steel Bank Common Lisp.

# How to use (easier version)
## Install
1. Create your .tool-versions file in the project that needs sbcl and add `sbcl 1.5.7` or whatever version that you want.
2. run `asdf install`

## Run
1. Once it is done, run `sbcl`
2. ???
3. Profit!
