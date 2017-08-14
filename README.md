# toolchains

A collection of standalone scripts to build GCC for various targets.
Available targets:
 * x86-64-linux-gnu
 * i686-mingw32

Authors
=======

- Ace17 (Sebastien Alaiwan: sebastien.alaiwan@gmail.com)

Purpose
=======

These are scripts for building a GCC compiler targetting TARGET.
Included languages are:
 * C
 * C++
 * D

The resulting compiler will run under the system it was built from.
In one word, you will get a natively running compiler, targetting TARGET.
The script will automatically download all dependencies, excepted for the build tools (native gcc, GNU make, etc.).

Usage
-----

* Download the latest head.

  $ git clone https://github.com/Ace17/toolchains.git

* Create a temporary directory, and make it the current working directory.
  (Using a tmpfs directory here will considerably speedup the build).

  $ mkdir /tmp/my-gcc && cd /tmp/my-gcc

* Then, launch one "deploy" script. For example:

  $ /path/to/toolchains/depoy_toolchain_i686-ace-mingw32

* Once the script is complete, you have a standalone gcc toolchain
  in the /tmp/toolchains/TARGET directory.

The script keeps track of what has already been done.
In case of an interruption, you may just run it again from the same directory:
the build will resume where it was left.

Common Issues
-------------

* Spaces in PATH. I have found that having spaces in your PATH variable may
  cause the build of some packages to fail. The script detects this and will
  fail in this case. Solution: clean your PATH variable for this shell.

* Although the script has not been tested on MS Windows platforms,
  you might have some luck with MSYS2 [[http://mingw.org]]

Internals
---------

To build a GCC compiler targetting TARGET, the script goes through the
following steps:

* Build binutils.
  This step provides "ld", "as", "objdump", etc. for the target.
  Technically, binutils isn't a GCC dependency, but the build process will need
  it later.

* Download GCC dependencies:
  This is done by simply calling the dependency download script provided by gcc.

* Build GCC: host part.
  This step will is achieved by running:
  
  `$ make all-host`
  
  A working compiler will be created. This compiler will enable to create ".o"
  files for the target. But at this stage you won't be able to link a full
  executable program, because the runtime libraries (crt1.o) are not built yet.
  And you can't build the runtime libraries without the Win32 API libraries,
  which are not available yet at this point.

* Build glibc (if targetting GNU/Linux).

* Build mingw64 (if targetting MS Windows).
  This step provides 'windows.h', libkernel32.a, etc.
  mingw64 is split in several parts, we only use here the following parts:
  * mingw-w64-headers
  * mingw-w64-crt
  * winpthread

  All of these parts generate code which will be run under MS Windows. So they
  must be compiled using the GCC created in the previous step, which targets
  MS Windows.

  mingw-w64-headers must be built and installed first. Otherwise, mingw-w64-crt
  configuration will fail. Then mingw-w64-crt can be built and installed, then
  winpthread library.

* Finish GCC: target part.
  This step will is achieved by running:
  $ make all-target
  The build process will rely on the freshly built crt to generate a libgcc to
  be run under TARGET.

Lastest working
---------------
   * x86_64-linux-gnu: D2.068, GCC 5.3.0, BINUTILS 2.26
   * i686-mingw: D2.067, GCC 5.3.0, BINUTILS 2.26

