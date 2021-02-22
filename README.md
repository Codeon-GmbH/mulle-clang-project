[![Codeon Gmbh](CodeonLogo.png)](//www.codeon.de)

# mulle-clang

This is an Objective-C compiler based on clang 11.0.0, written for the
[mulle-objc](//www.mulle-kybernetik.com/weblog/2015/mulle_objc_a_new_objective_c_.html)
runtime. It corresponds to mulle-objc-runtime v0.18 or better.

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
for all method calls. The resultant `.o` files are linkable like any
other compiled C code.


### AAM - Always Autorelease Mode

The compiler has a special mode called AAM. This changes the Objective-C
language in the following ways:

1. There is a tranformation done on selector names

    Name           | Transformed Name
    ---------------|---------------------
    alloc          | instantiate
    new            | instantiatedObject
    copy           | immutableInstance
    mutableCopy    | mutableInstance
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

Name                             | Compiler                 | Default | Description
---------------------------------|--------------------------|---------|--------------------
`__MULLE_OBJC__`                 | -                        | -       | Compiling for mulle-objc
`__MULLE_OBJC_UNIVERSEID__`      | -fobjc-universename=name | -       | id of the universe, or 0 for default universe
`__MULLE_OBJC_UNIVERSENAME__`    | -fobjc-universename=name | -       | name of the universe, or NULL for default universe


The following table represents option pairs, that logically exclude each other.
Either one is always defined.

Name                    | Compiler        | Default | Description
------------------------|-----------------|-------|--------------------
`__MULLE_OBJC_AAM__`    | .aam file       | -     | AAM is enabled
`__MULLE_OBJC_NO_AAM__` | .m file         | -     | AAM is not enabled
 &nbsp;                 | &nbsp;          | &nbsp;|
`__MULLE_OBJC_TPS__`    | -fobjc-tps      | YES   | TPS (tagged pointer support) is enabled
`__MULLE_OBJC_NO_TPS__` | -fno-objc-tps   | NO    | TPS is not enabled
&nbsp;                  | &nbsp;          | &nbsp;|
`__MULLE_OBJC_FCS__`    | -fobjc-fcs      | YES   | FCS fast method/class support is enabled
`__MULLE_OBJC_NO_FCS__` | -fno-objc-fcs   | NO    | FCS is not enabled


## Macros used in Code Generation


The compiler output can be tweaked with the following preprocessor macros.
All macros must be defined with a simple integer, no expressions. All of them
are optional, unless indicated otherwise. The runtime, the Objective-C
Foundation on top of the runtime and the user application, will define them.


Name                                  | Description
--------------------------------------|--------------------------------------
`MULLE_OBJC_RUNTIME_VERSION_MAJOR`    | Major of version of the runtime
`MULLE_OBJC_RUNTIME_VERSION_MINOR`    | Minor of version of the runtime
`MULLE_OBJC_RUNTIME_VERSION_PATCH`    | Patch of version of the runtime
`MULLE_OBJC_FOUNDATION_VERSION_MAJOR` | Major of version of the Foundation
`MULLE_OBJC_FOUNDATION_VERSION_MINOR` | Minor of version of the Foundation
`MULLE_OBJC_FOUNDATION_VERSION_PATCH` | Patch of version of the Foundation
`MULLE_OBJC_USER_VERSION_MAJOR`       | User supplied major of version
`MULLE_OBJC_USER_VERSION_MINOR`       | User supplied minor of version
`MULLE_OBJC_USER_VERSION_PATCH`       | User supplied patch of version .All these version information values will be stored in the emitted object file.
`MULLE_OBJC_FASTCLASSHASH_0`          | First unique ID of a fast class
... | ...
`MULLE_OBJC_FASTCLASSHASH_63`         | Last unique ID of a fast class


## Functions used in Code Generation

These are the runtime functions used for method calling, retain/release
management, class lookup and exception handling. They are defined in the
runtime.

### All

Function                        | Memo
--------------------------------|---------
`mulle_objc_exception_tryenter` | `@throw`
`mulle_objc_exception_tryexit`  | `@finally`
`mulle_objc_exception_extract`  | `@catch`
`mulle_objc_exception_match`    | `@catch`


### -O0, -Os

Function                                            | Memo
----------------------------------------------------|-------------
`mulle_objc_object_call`                            | `[self foo:bar]`
`mulle_objc_object_supercall`                       | `[super foo:bar]`
`mulle_objc_object_lookup_infraclass_nofail`        | `[Foo ...` for methods
`mulle_objc_object_lookup_infraclass_nofast_nofail` | `__MULLE_OBJC_NO_FCS__`
`mulle_objc_global_lookup_infraclass_nofail`        | `[Foo ...` for functions
`mulle_objc_global_lookup_infraclass_nofast_nofail` | `__MULLE_OBJC_NO_FCS__`


### -O1

Like -O0, but two functions are replaced with partially inlining versions:

Function                                            | Memo
----------------------------------------------------|-------------
`mulle_objc_object_call_inline_partial`             | `[self foo:bar]`
`mulle_objc_object_supercall_inline_partial`        | `[super foo:bar]`


### -O2

Like -O1, but four functions are replaced with four inlining functions, and
two new inlining functions are used:

Function                                            | Memo
----------------------------------------------------|-------------
`mulle_objc_object_lookup_infraclass_inline_nofail` | `[Foo ...` for methods
`mulle_objc_global_lookup_infraclass_inline_nofail` |  `__MULLE_OBJC_NO_FCS__`
`mulle_objc_object_lookup_infraclass_inline_nofail_nofast` | `[Foo ...` for functions
`mulle_objc_global_lookup_infraclass_inline_nofail_nofast` | `__MULLE_OBJC_NO_FCS__`
`mulle_objc_object_retain_inline`                   | `[foo retain]`
`mulle_objc_object_release_inline`                  | `[foo release]`


### -O3

Like -O3, but two functions are replaced with inlining functions:

Function                                            | Memo
----------------------------------------------------|-------------
`mulle_objc_object_call_inline`                     | `[self foo:bar]`
`mulle_objc_object_supercall_inline`                | `[super foo:bar]`


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

This directory and its sub-directories contain source code for LLVM,
a toolkit for the construction of highly optimized compilers,
optimizers, and run-time environments.

The README briefly describes how to get started with building LLVM.
For more information on how to contribute to the LLVM project, please
take a look at the
[Contributing to LLVM](https://llvm.org/docs/Contributing.html) guide.

## Getting Started with the LLVM System

Taken from https://llvm.org/docs/GettingStarted.html.

### Overview

Welcome to the LLVM project!

The LLVM project has multiple components. The core of the project is
itself called "LLVM". This contains all of the tools, libraries, and header
files needed to process intermediate representations and converts it into
object files.  Tools include an assembler, disassembler, bitcode analyzer, and
bitcode optimizer.  It also contains basic regression tests.

C-like languages use the [Clang](http://clang.llvm.org/) front end.  This
component compiles C, C++, Objective-C, and Objective-C++ code into LLVM bitcode
-- and from there into object files, using LLVM.

Other components include:
the [libc++ C++ standard library](https://libcxx.llvm.org),
the [LLD linker](https://lld.llvm.org), and more.

### Getting the Source Code and Building LLVM

The LLVM Getting Started documentation may be out of date.  The [Clang
Getting Started](http://clang.llvm.org/get_started.html) page might have more
accurate information.

This is an example work-flow and configuration to get and build the LLVM source:

1. Checkout LLVM (including related sub-projects like Clang):

     * ``git clone https://github.com/llvm/llvm-project.git``

     * Or, on windows, ``git clone --config core.autocrlf=false
    https://github.com/llvm/llvm-project.git``

2. Configure and build LLVM and Clang:

     * ``cd llvm-project``

     * ``mkdir build``

     * ``cd build``

     * ``cmake -G <generator> [options] ../llvm``

        Some common build system generators are:

        * ``Ninja`` --- for generating [Ninja](https://ninja-build.org)
          build files. Most llvm developers use Ninja.
        * ``Unix Makefiles`` --- for generating make-compatible parallel makefiles.
        * ``Visual Studio`` --- for generating Visual Studio projects and
          solutions.
        * ``Xcode`` --- for generating Xcode projects.

        Some Common options:

        * ``-DLLVM_ENABLE_PROJECTS='...'`` --- semicolon-separated list of the LLVM
          sub-projects you'd like to additionally build. Can include any of: clang,
          clang-tools-extra, libcxx, libcxxabi, libunwind, lldb, compiler-rt, lld,
          polly, or debuginfo-tests.

          For example, to build LLVM, Clang, libcxx, and libcxxabi, use
          ``-DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi"``.

        * ``-DCMAKE_INSTALL_PREFIX=directory`` --- Specify for *directory* the full
          path name of where you want the LLVM tools and libraries to be installed
          (default ``/usr/local``).

        * ``-DCMAKE_BUILD_TYPE=type`` --- Valid options for *type* are Debug,
          Release, RelWithDebInfo, and MinSizeRel. Default is Debug.

        * ``-DLLVM_ENABLE_ASSERTIONS=On`` --- Compile with assertion checks enabled
          (default is Yes for Debug builds, No for all other build types).

      * ``cmake --build . [-- [options] <target>]`` or your build system specified above
        directly.

        * The default target (i.e. ``ninja`` or ``make``) will build all of LLVM.

        * The ``check-all`` target (i.e. ``ninja check-all``) will run the
          regression tests to ensure everything is in working order.

        * CMake will generate targets for each tool and library, and most
          LLVM sub-projects generate their own ``check-<project>`` target.

        * Running a serial build will be **slow**.  To improve speed, try running a
          parallel build.  That's done by default in Ninja; for ``make``, use the option
          ``-j NNN``, where ``NNN`` is the number of parallel jobs, e.g. the number of
          CPUs you have.

      * For more information see [CMake](https://llvm.org/docs/CMake.html)

Consult the
[Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html#getting-started-with-llvm)
page for detailed information on configuring and compiling LLVM. You can visit
[Directory Layout](https://llvm.org/docs/GettingStarted.html#directory-layout)
to learn about the layout of the source code tree.
