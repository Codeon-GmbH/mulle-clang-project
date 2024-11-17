[![mulle-cc](https://avatars.githubusercontent.com/u/70896078?s=400&u=35dc427f9bb75c52c1d973cc3f895d2d18a461ae&v=4)](https://github.com/mulle-cc)

# mulle-clang

This is an Objective-C compiler based on clang 17.0.6, written for the, written for the
[mulle-objc](//www.mulle-kybernetik.com/weblog/2015/mulle_objc_a_new_objective_c_.html)
runtime. It corresponds to mulle-objc-runtime v0.20 or better.

> See [README.txt](README.txt) for more information about clang

The compiler can be used to:

1. compile Objective-C code for **mulle-objc**
2. compile C code

It is not recommended to use it for C++, since that is not tested.
It is not recommended to use it for other Objective-C runtimes, since there
have been some changes, that affect other runtimes.


## Operation

The compiler compiles Objective-C source for the **mulle-objc** runtime by
default. When compiling for **mulle-objc** the compiler will use the
[meta-ABI](//www.mulle-kybernetik.com/weblog/2015/mulle_objc_meta_call_convention.html)
for all Objective-C method calls. C function calls use the platform convention.
The resultant `.o` files are linkable like any other compiled C code.


### AAM - Always Autorelease Mode

The compiler has a special mode called AAM. This changes the Objective-C
language in the following ways:

1. There is a tranformation done on selector names

    | Name          | Transformed Name
    |---------------|---------------------
    | `alloc`       | `instantiate`
    | `new`         | `instantiatedObject`
    | `copy`        | `immutableInstance`
    | `mutableCopy` | `mutableInstance`
2. You can not access instance variables directly, but must use properties (or methods)
3. You can not do explicit memory management (like `-dealloc`, `-autorelease`,
`-release`, `-retain`, `-retainCount` etc.)

The transformed methods will return objects that are autoreleased. Hence the
name of the mode. The net effect is, that you have a mode that is ARC-like, yet
understandable and much simpler.

Your ARC code may not run in AAM, but AAM code should run in ARC with no
problems. If you can't do something in AAM, put it in a category in regular
Objective-C style.

The compiler handles `.aam` files, which enables AAM ("Always Autoreleased
Mode").

## Additional Compiler options and defined macros

| Name                             | Compiler                 | Default | Description
|----------------------------------|--------------------------|---------|--------------------
| `__MULLE_OBJC__`                 | -                        | -       | Compiling for mulle-objc
| `__MULLE_OBJC_UNIVERSEID__`      | -fobjc-universename=name | -       | id of the universe, or 0 for default | universe
| `__MULLE_OBJC_UNIVERSENAME__`    | -fobjc-universename=name | -       | name of the universe, or NULL for default | universe
| `__OBJC_CLASS__`                 | -                        | -       | name of the class thats currently being compiled
| `__OBJC_CATEGORY__`              | -                        | -       | name of the category that's currently being compiled
| `__MULLE_OBJC_CLASSID__`         | -                        | -       | classid of the class thats currently being compiled
| `__MULLE_OBJC_CATEGORYID__`      | -                        | -       | uniqueud of the category thats currently being compiled

The following table represents option pairs, that logically exclude each other.
Either one is always defined.

| Name                    | Compiler        | Default | Description
|-------------------------|-----------------|---------|--------------------
| `__MULLE_OBJC_AAM__`    | .aam file       | -       | AAM is enabled
| `__MULLE_OBJC_NO_AAM__` | .m file         | -       | AAM is not enabled
|  &nbsp;                 | &nbsp;          | &nbsp;  |
| `__MULLE_OBJC_TPS__`    | -fobjc-tps      | YES     | TPS (tagged pointer support) is enabled
| `__MULLE_OBJC_NO_TPS__` | -fno-objc-tps   | NO      | TPS is not enabled
| &nbsp;                  | &nbsp;          | &nbsp;  |
| `__MULLE_OBJC_FCS__`    | -fobjc-fcs      | YES     | FCS fast method/class support is enabled
| `__MULLE_OBJC_NO_FCS__` | -fno-objc-fcs   | NO      | FCS is not enabled
| &nbsp;                  | &nbsp;          | &nbsp;  |
| `__MULLE_OBJC_TAO__`    | -fobjc-tao      | -       | TAO (thread affine objects) is enabled
| `__MULLE_OBJC_NO_TAO__` | -fno-objc-tao   | -       | TAO is not enabled

The default is `-fobjc-tao` for `-O0` builds and `-fno-objc-tao` for optimized
builds.


## Macros used in Code Generation


The compiler output can be tweaked with the following preprocessor macros.
All macros must be defined with a simple integer, no expressions. All of them
are optional, unless indicated otherwise. The runtime, the Objective-C
Foundation on top of the runtime and the user application, will define them.


| Name                                  | Description
|---------------------------------------|--------------------------------------
| `MULLE_OBJC_RUNTIME_VERSION_MAJOR`    | Major of version of the runtime
| `MULLE_OBJC_RUNTIME_VERSION_MINOR`    | Minor of version of the runtime
| `MULLE_OBJC_RUNTIME_VERSION_PATCH`    | Patch of version of the runtime
| `MULLE_OBJC_FOUNDATION_VERSION_MAJOR` | Major of version of the Foundation
| `MULLE_OBJC_FOUNDATION_VERSION_MINOR` | Minor of version of the Foundation
| `MULLE_OBJC_FOUNDATION_VERSION_PATCH` | Patch of version of the Foundation
| `MULLE_OBJC_USER_VERSION_MAJOR`       | User supplied major of version
| `MULLE_OBJC_USER_VERSION_MINOR`       | User supplied minor of version
| `MULLE_OBJC_USER_VERSION_PATCH`       | User supplied patch of version. All these version information values will be stored in the emitted object file.
| `MULLE_OBJC_FASTCLASSHASH_0`          | First unique ID of a fast class
| ...                                   | ...
| `MULLE_OBJC_FASTCLASSHASH_63`         | Last unique ID of a fast class


## Functions used in Code Generation

These are the runtime functions used for method calling, retain/release
management, class lookup and exception handling. They are defined in the
runtime.

### All

| Function                        | Memo
|---------------------------------|---------
| `mulle_objc_exception_tryenter` | `@throw`
| `mulle_objc_exception_tryexit`  | `@finally`
| `mulle_objc_exception_extract`  | `@catch`
| `mulle_objc_exception_match`    | `@catch`



### Inlining method calls

With `-fobjc-inline-method-calls=[1-5]` you can control the level of inlining.
If you don't specify it, the optimization level is mapped to a inlining
setting. (See table below)

`cmake` sets `-O3` as the optimization level, for Release builds, though you
may want to prefer `-O2` or `-Oz` for Objective-C code.


| Inlining         | -fobjc-inline-method-calls | Optimization Level (if -fobjc-inline-method-calls is unset)
|------------------|----------------------------|------------------------
| No inlining      | 1                          | -O0
| Minimal inlining | 2                          | -O1/-Os
| Partial inlining | 3                          | -O2/-Oz
| Default inlining | 4                          | -O3/-O4
| Full inlining    | 5                          | -O5+


Also note, that if you combine -O0 with -fobjc-inline-method-calls, there
won't be any inlining, since -O0 prohibits it.

### No inlining

| Function                                            | Memo
|-----------------------------------------------------|-------------
| `mulle_objc_object_call`                            | `[self foo:bar]`
| `mulle_objc_object_supercall`                       | `[super foo:bar]`
| &nbsp;                                              |
| `mulle_objc_object_lookup_infraclass_nofail`        | `[Foo ...` for methods
| `mulle_objc_object_lookup_infraclass_nofast_nofail` | `__MULLE_OBJC_NO_FCS__`
| `mulle_objc_global_lookup_infraclass_nofail`        | `[Foo ...` for functions
| `mulle_objc_global_lookup_infraclass_nofast_nofail` | `__MULLE_OBJC_NO_FCS__`


### Minimal inlining

Like "No inlining" but one function is replaced with a minimally inlining
version:

| Function                                            | Memo
|-----------------------------------------------------|-------------
| `mulle_objc_object_call_inline_minimal`             | `[self foo:bar]`


### Partial inlining

Like "No inlining" but four functions are replaced with four inlining
functions, and two new inlining functions are used:

| Function                                                   | Memo
|------------------------------------------------------------|-------------
| `mulle_objc_object_call_inline_partial`                    | `[self foo:bar]`
| `mulle_objc_object_supercall_inline_partial`               | `[super foo:bar]`
| &nbsp;                                                     |
| `mulle_objc_object_lookup_infraclass_inline_nofail`        | `[Foo ...` for methods
| `mulle_objc_global_lookup_infraclass_inline_nofail`        |  `__MULLE_OBJC_NO_FCS__`
| `mulle_objc_object_lookup_infraclass_inline_nofail_nofast` | `[Foo ...` for functions
| `mulle_objc_global_lookup_infraclass_inline_nofail_nofast` | `__MULLE_OBJC_NO_FCS__`
| `mulle_objc_object_retain_inline`                          | `[foo retain]`
| `mulle_objc_object_release_inline`                         | `[foo release]`


The partial inlining code, tries to eliminate some constant setup cost by
moving it to the compilation. This can be good for multiple calls involving
the same object.


### Default inlining

Like "Partial inlining", but two functions are replaced with more inlined
functions:

| Function                                            | Memo
|-----------------------------------------------------|-------------
| `mulle_objc_object_call_inline`                     | `[self foo:bar]`
| `mulle_objc_object_call_super_inline`               | `[super foo:bar]`

The default inline code checks the cache for an immediate hit, then slower
shared coded is used for further cache lookup and cache misses.

### Full inlining

Like "Default inlining", but two functions are replaced with fully inlined
functions:

| Function                                            | Memo
|-----------------------------------------------------|-------------
| `mulle_objc_object_call_inline_full`                | `[self foo:bar]`
| `mulle_objc_object_call_super_inline_full`          | `[super foo:bar]`

The full inline code checks the cache for a hit, then slower
shared coded is used for cache misses.


## Build

You can use the original [llvm project instructions](#Getting-the-Source-Code-and-Building-LLVM)
to build the project. But do checkout [cmake-ninja](clang/bin/cmake-ninja) for
a small build script, that sets important build values.

If you are building inside a fresh system, try [install-prerequisites](clang/bin/install-prerequisites)
that can install all prerequisites for a macos or debian based system.


Afterwards head on over to [mulle-objc](//github.com/mulle-objc) to get the
runtime libraries.


## Author

[Nat!](//www.mulle-kybernetik.com/weblog) for
[Codeon GmbH](//www.codeon.de)

Original README.md follows
----

# The LLVM Compiler Infrastructure

Welcome to the LLVM project!

This repository contains the source code for LLVM, a toolkit for the
construction of highly optimized compilers, optimizers, and run-time
environments.

The LLVM project has multiple components. The core of the project is
itself called "LLVM". This contains all of the tools, libraries, and header
files needed to process intermediate representations and convert them into
object files. Tools include an assembler, disassembler, bitcode analyzer, and
bitcode optimizer.

C-like languages use the [Clang](http://clang.llvm.org/) frontend. This
component compiles C, C++, Objective-C, and Objective-C++ code into LLVM bitcode
-- and from there into object files, using LLVM.

Other components include:
the [libc++ C++ standard library](https://libcxx.llvm.org),
the [LLD linker](https://lld.llvm.org), and more.

## Getting the Source Code and Building LLVM

Consult the
[Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html#getting-the-source-code-and-building-llvm)
page for information on building and running LLVM.

For information on how to contribute to the LLVM project, please take a look at
the [Contributing to LLVM](https://llvm.org/docs/Contributing.html) guide.

## Getting in touch

Join the [LLVM Discourse forums](https://discourse.llvm.org/), [Discord
chat](https://discord.gg/xS7Z362), or #llvm IRC channel on
[OFTC](https://oftc.net/).

The LLVM project has adopted a [code of conduct](https://llvm.org/docs/CodeOfConduct.html) for
participants to all modes of communication within the project.
