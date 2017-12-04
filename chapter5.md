## Chapter 5. Constructing a Temporary System

## 5.1. Introduction

This chapter shows how to build a minimal Linux system. This system will contain just enough tools to start constructing the final LFS system in [Chapter 6](Linux%20From%20Scratch.html#chapter-building-system) and allow a working environment with more user convenience than a minimum environment would.

There are two steps in building this minimal system. The first step is to build a new and host-independent toolchain (compiler, assembler, linker, libraries, and a few useful utilities). The second step uses this toolchain to build the other essential tools.

The files compiled in this chapter will be installed under the `$LFS/tools` directory to keep them separate from the files installed in the next chapter and the host production directories. Since the packages compiled here are temporary, we do not want them to pollute the soon-to-be LFS system.

## 5.2. Toolchain Technical Notes

This section explains some of the rationale and technical details behind the overall build method. It is not essential to immediately understand everything in this section. Most of this information will be clearer after performing an actual build. This section can be referred to at any time during the process.

The overall goal of [Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) is to produce a temporary area that contains a known-good set of tools that can be isolated from the host system. By using **chroot**, the commands in the remaining chapters will be contained within that environment, ensuring a clean, trouble-free build of the target LFS system. The build process has been designed to minimize the risks for new readers and to provide the most educational value at the same time.

### Note

Before continuing, be aware of the name of the working platform, often referred to as the target triplet. A simple way to determine the name of the target triplet is to run the **config.guess** script that comes with the source for many packages. Unpack the Binutils sources and run the script: **`./config.guess`** and note the output. For example, for a 32-bit Intel processor the output will be *i686-pc-linux-gnu*. On a 64-bit system it will be *x86_64-pc-linux-gnu*.

Also be aware of the name of the platform's dynamic linker, often referred to as the dynamic loader (not to be confused with the standard linker **ld** that is part of Binutils). The dynamic linker provided by Glibc finds and loads the shared libraries needed by a program, prepares the program to run, and then runs it. The name of the dynamic linker for a 32-bit Intel machine will be `ld-linux.so.2` (`ld-linux-x86-64.so.2` for 64-bit systems). A sure-fire way to determine the name of the dynamic linker is to inspect a random binary from the host system by running: **`readelf -l <name of binary> | grep interpreter`** and noting the output. The authoritative reference covering all platforms is in the `shlib-versions` file in the root of the Glibc source tree.

Some key technical points of how the [Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) build method works:

-
Slightly adjusting the name of the working platform, by changing the "vendor" field target triplet by way of the `LFS_TGT` variable, ensures that the first build of Binutils and GCC produces a compatible cross-linker and cross-compiler. Instead of producing binaries for another architecture, the cross-linker and cross-compiler will produce binaries compatible with the current hardware.

-
The temporary libraries are cross-compiled. Because a cross-compiler by its nature cannot rely on anything from its host system, this method removes potential contamination of the target system by lessening the chance of headers or libraries from the host being incorporated into the new tools. Cross-compilation also allows for the possibility of building both 32-bit and 64-bit libraries on 64-bit capable hardware.

-
Careful manipulation of the GCC source tells the compiler which target dynamic linker will be used.

Binutils is installed first because the **configure** runs of both GCC and Glibc perform various feature tests on the assembler and linker to determine which software features to enable or disable. This is more important than one might first realize. An incorrectly configured GCC or Glibc can result in a subtly broken toolchain, where the impact of such breakage might not show up until near the end of the build of an entire distribution. A test suite failure will usually highlight this error before too much additional work is performed.

Binutils installs its assembler and linker in two locations, `/tools/bin` and `/tools/$LFS_TGT/bin`. The tools in one location are hard linked to the other. An important facet of the linker is its library search order. Detailed information can be obtained from **ld** by passing it the *`--verbose`* flag. For example, an **`ld --verbose | grep SEARCH`** will illustrate the current search paths and their order. It shows which files are linked by **ld** by compiling a dummy program and passing the *`--verbose`* switch to the linker. For example, **`gcc dummy.c -Wl,--verbose 2>&1 | grep succeeded`** will show all the files successfully opened during the linking.

The next package installed is GCC. An example of what can be seen during its run of **configure** is:

    checking what assembler to use... /tools/i686-lfs-linux-gnu/bin/as
    checking what linker to use... /tools/i686-lfs-linux-gnu/bin/ld

This is important for the reasons mentioned above. It also demonstrates that GCC's configure script does not search the PATH directories to find which tools to use. However, during the actual operation of **gcc** itself, the same search paths are not necessarily used. To find out which standard linker **gcc** will use, run: **`gcc -print-prog-name=ld`**.

Detailed information can be obtained from **gcc** by passing it the *`-v`* command line option while compiling a dummy program. For example, **`gcc -v dummy.c`** will show detailed information about the preprocessor, compilation, and assembly stages, including **gcc**'s included search paths and their order.

Next installed are sanitized Linux API headers. These allow the standard C library (Glibc) to interface with features that the Linux kernel will provide.

The next package installed is Glibc. The most important considerations for building Glibc are the compiler, binary tools, and kernel headers. The compiler is generally not an issue since Glibc will always use the compiler relating to the *`--host`* parameter passed to its configure script; e.g. in our case, the compiler will be **i686-lfs-linux-gnu-gcc**. The binary tools and kernel headers can be a bit more complicated. Therefore, take no risks and use the available configure switches to enforce the correct selections. After the run of **configure**, check the contents of the `config.make` file in the `glibc-build` directory for all important details. Note the use of *`CC="i686-lfs-gnu-gcc"`* to control which binary tools are used and the use of the *`-nostdinc`* and *`-isystem`* flags to control the compiler's include search path. These items highlight an important aspect of the Glibc package—it is very self-sufficient in terms of its build machinery and generally does not rely on toolchain defaults.

During the second pass of Binutils, we are able to utilize the *`--with-lib-path`* configure switch to control **ld**'s library search path.

For the second pass of GCC, its sources also need to be modified to tell GCC to use the new dynamic linker. Failure to do so will result in the GCC programs themselves having the name of the dynamic linker from the host system's `/lib` directory embedded into them, which would defeat the goal of getting away from the host. From this point onwards, the core toolchain is self-contained and self-hosted. The remainder of the [Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) packages all build against the new Glibc in `/tools`.

Upon entering the chroot environment in [Chapter 6](Linux%20From%20Scratch.html#chapter-building-system), the first major package to be installed is Glibc, due to its self-sufficient nature mentioned above. Once this Glibc is installed into `/usr`, we will perform a quick changeover of the toolchain defaults, and then proceed in building the rest of the target LFS system.

## 5.3. General Compilation Instructions

When building packages there are several assumptions made within the instructions:

-
Several of the packages are patched before compilation, but only when the patch is needed to circumvent a problem. A patch is often needed in both this and the next chapter, but sometimes in only one or the other. Therefore, do not be concerned if instructions for a downloaded patch seem to be missing. Warning messages about *offset* or *fuzz* may also be encountered when applying a patch. Do not worry about these warnings, as the patch was still successfully applied.

-
During the compilation of most packages, there will be several warnings that scroll by on the screen. These are normal and can safely be ignored. These warnings are as they appear—warnings about deprecated, but not invalid, use of the C or C++ syntax. C standards change fairly often, and some packages still use the older standard. This is not a problem, but does prompt the warning.

-
Check one last time that the `LFS` environment variable is set up properly:

    echo $LFS

Make sure the output shows the path to the LFS partition's mount point, which is `/mnt/lfs`, using our example.

-
Finally, two last important items must be emphasized:

### Important

The build instructions assume that the [Host System Requirements](Linux%20From%20Scratch.html#pre-hostreqs), including symbolic links, have been set properly:

-
**bash** is the shell in use.

-
**sh** is a symbolic link to **bash**.

-
**/usr/bin/awk** is a symbolic link to **gawk**.

-
**/usr/bin/yacc** is a symbolic link to **bison** or a small script that executes bison.

### Important

To re-emphasize the build process:

1.
Place all the sources and patches in a directory that will be accessible from the chroot environment such as `/mnt/lfs/sources/`. Do *not* put sources in `/mnt/lfs/tools/`.

2.
Change to the sources directory.

3.
For each package:

1.
Using the **tar** program, extract the package to be built. In Chapter 5, ensure you are the *lfs* user when extracting the package.

2.
Change to the directory created when the package was extracted.

3.
Follow the book's instructions for building the package.

4.
Change back to the sources directory.

5.
Delete the extracted source directory unless instructed otherwise.

## 5.4. Binutils-2.29 - Pass 1

The Binutils package contains a linker, an assembler, and other tools for handling object files.

**Approximate build time:**1 SBU

**Required disk space:**547 MB

### 5.4.1. Installation of Cross Binutils

### Note

Go back and re-read the notes in the previous section. Understanding the notes labeled important will save you a lot of problems later.

It is important that Binutils be the first package compiled because both Glibc and GCC perform various tests on the available linker and assembler to determine which of their own features to enable.

The Binutils documentation recommends building Binutils in a dedicated build directory:

    mkdir -v build
    cd       build

### Note

In order for the SBU values listed in the rest of the book to be of any use, measure the time it takes to build this package from the configuration, up to and including the first install. To achieve this easily, wrap the commands in a **time** command like this: **`time { ./configure ... && ... && make install; }`**.

### Note

The approximate build SBU values and required disk space in Chapter 5 does not include test suite data.

Now prepare Binutils for compilation:

    ../configure --prefix=/tools            \
                 --with-sysroot=$LFS        \
                 --with-lib-path=/tools/lib \
                 --target=$LFS_TGT          \
                 --disable-nls              \
                 --disable-werror

**The meaning of the configure options:**
*`--prefix=/tools`*
This tells the configure script to prepare to install the Binutils programs in the `/tools` directory.
*`--with-sysroot=$LFS`*
For cross compilation, this tells the build system to look in $LFS for the target system libraries as needed.
*`--with-lib-path=/tools/lib`*
This specifies which library path the linker should be configured to use.
`--target=$LFS_TGT`
Because the machine description in the `LFS_TGT` variable is slightly different than the value returned by the **config.guess** script, this switch will tell the **configure** script to adjust Binutil's build system for building a cross linker.
*`--disable-nls`*
This disables internationalization as i18n is not needed for the temporary tools.
*`--disable-werror`*
This prevents the build from stopping in the event that there are warnings from the host's compiler.

Continue with compiling the package:

    make

Compilation is now complete. Ordinarily we would now run the test suite, but at this early stage the test suite framework (Tcl, Expect, and DejaGNU) is not yet in place. The benefits of running the tests at this point are minimal since the programs from this first pass will soon be replaced by those from the second.

If building on x86_64, create a symlink to ensure the sanity of the toolchain:

    case $(uname -m) in
      x86_64) mkdir -v /tools/lib && ln -sv lib /tools/lib64 ;;
    esac

Install the package:

    make install

Details on this package are located in [Section 6.16.2, “Contents of Binutils.”](Linux%20From%20Scratch.html#contents-binutils)

Last updated on

## 5.5. GCC-7.2.0 - Pass 1

The GCC package contains the GNU compiler collection, which includes the C and C++ compilers.

**Approximate build time:**8.9 SBU

**Required disk space:**2.2 GB

### 5.5.1. Installation of Cross GCC

GCC now requires the GMP, MPFR and MPC packages. As these packages may not be included in your host distribution, they will be built with GCC. Unpack each package into the GCC source directory and rename the resulting directories so the GCC build procedures will automatically use them:

### Note

There are frequent misunderstandings about this chapter. The procedures are the same as every other chapter as explained earlier ([Package build instructions](Linux%20From%20Scratch.html#buildinstr)). First extract the gcc tarball from the sources directory and then change to the directory created. Only then should you proceed with the instructions below.

    tar -xf ../mpfr-3.1.5.tar.xz
    mv -v mpfr-3.1.5 mpfr
    tar -xf ../gmp-6.1.2.tar.xz
    mv -v gmp-6.1.2 gmp
    tar -xf ../mpc-1.0.3.tar.gz
    mv -v mpc-1.0.3 mpc

The following command will change the location of GCC's default dynamic linker to use the one installed in `/tools`. It also removes `/usr/include`from GCC's include search path. Issue:

    for file in gcc/config/{linux,i386/linux{,64}}.h
    do
      cp -uv $file{,.orig}
      sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
          -e 's@/usr@/tools@g' $file.orig > $file
      echo '
    #undef STANDARD_STARTFILE_PREFIX_1
    #undef STANDARD_STARTFILE_PREFIX_2
    #define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
    #define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
      touch $file.orig
    done

In case the above seems hard to follow, let's break it down a bit. First we copy the files `gcc/config/linux.h`, `gcc/config/i386/linux.h`, and `gcc/config/i368/linux64.h`. to a file of the same name but with an added suffix of “.orig”. Then the first sed expression prepends “/tools” to every instance of “/lib/ld”, “/lib64/ld” or “/lib32/ld”, while the second one replaces hard-coded instances of “/usr”. Next, we add our define statements which alter the default startfile prefix to the end of the file. Note that the trailing “/” in “/tools/lib/” is required. Finally, we use **touch** to update the timestamp on the copied files. When used in conjunction with **cp -u**, this prevents unexpected changes to the original files in case the commands are inadvertently run twice.

Finally, on x86_64 hosts, set the default directory name for 64-bit libraries to “lib”:

    case $(uname -m) in
      x86_64)
        sed -e '/m64=/s/lib64/lib/' \
            -i.orig gcc/config/i386/t-linux64
     ;;
    esac

The GCC documentation recommends building GCC in a dedicated build directory:

    mkdir -v build
    cd       build

Prepare GCC for compilation:

    ../configure                                       \
        --target=$LFS_TGT                              \
        --prefix=/tools                                \
        --with-glibc-version=2.11                      \
        --with-sysroot=$LFS                            \
        --with-newlib                                  \
        --without-headers                              \
        --with-local-prefix=/tools                     \
        --with-native-system-header-dir=/tools/include \
        --disable-nls                                  \
        --disable-shared                               \
        --disable-multilib                             \
        --disable-decimal-float                        \
        --disable-threads                              \
        --disable-libatomic                            \
        --disable-libgomp                              \
        --disable-libmpx                               \
        --disable-libquadmath                          \
        --disable-libssp                               \
        --disable-libvtv                               \
        --disable-libstdcxx                            \
        --enable-languages=c,c++

**The meaning of the configure options:**
*`--with-newlib`*
Since a working C library is not yet available, this ensures that the inhibit_libc constant is defined when building libgcc. This prevents the compiling of any code that requires libc support.
*`--without-headers`*
When creating a complete cross-compiler, GCC requires standard headers compatible with the target system. For our purposes these headers will not be needed. This switch prevents GCC from looking for them.
*`--with-local-prefix=/tools`*
The local prefix is the location in the system that GCC will search for locally installed include files. The default is `/usr/local`. Setting this to `/tools` helps keep the host location of `/usr/local` out of this GCC's search path.
*`--with-native-system-header-dir=/tools/include`*
By default GCC searches `/usr/include` for system headers. In conjunction with the sysroot switch, this would translate normally to `$LFS/usr/include`. However the headers that will be installed in the next two sections will go to `$LFS/tools/include`. This switch ensures that gcc will find them correctly. In the second pass of GCC, this same switch will ensure that no headers from the host system are found.
*`--disable-shared`*
This switch forces GCC to link its internal libraries statically. We do this to avoid possible issues with the host system.
*`--disable-decimal-float, --disable-threads, --disable-libatomic, --disable-libgomp, --disable-libmpx, --disable-libquadmath, --disable-libssp, --disable-libvtv, --disable-libstdcxx`*
These switches disable support for the decimal floating point extension, threading, libatomic, libgomp, libmpx, libquadmath, libssp, libvtv, and the C++ standard library respectively. These features will fail to compile when building a cross-compiler and are not necessary for the task of cross-compiling the temporary libc.
*`--disable-multilib`*
On x86_64, LFS does not yet support a multilib configuration. This switch is harmless for x86.
*`--enable-languages=c,c++`*
This option ensures that only the C and C++ compilers are built. These are the only languages needed now.

Compile GCC by running:

    make

Compilation is now complete. At this point, the test suite would normally be run, but, as mentioned before, the test suite framework is not in place yet. The benefits of running the tests at this point are minimal since the programs from this first pass will soon be replaced.

Install the package:

    make install

Details on this package are located in [Section 6.20.2, “Contents of GCC.”](Linux%20From%20Scratch.html#contents-gcc)

Last updated on

## 5.6. Linux-4.12.7 API Headers

The Linux API Headers (in linux-4.12.7.tar.xz) expose the kernel's API for use by Glibc.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**861 MB

### 5.6.1. Installation of Linux API Headers

The Linux kernel needs to expose an Application Programming Interface (API) for the system's C library (Glibc in LFS) to use. This is done by way of sanitizing various C header files that are shipped in the Linux kernel source tarball.

Make sure there are no stale files embedded in the package:

    make mrproper

Now extract the user-visible kernel headers from the source. They are placed in an intermediate local directory and copied to the needed location because the extraction process removes any existing files in the target directory.

    make INSTALL_HDR_PATH=dest headers_install
    cp -rv dest/include/* /tools/include

Details on this package are located in [Section 6.7.2, “Contents of Linux API Headers.”](Linux%20From%20Scratch.html#contents-linux-headers)

Last updated on

## 5.7. Glibc-2.26

The Glibc package contains the main C library. This library provides the basic routines for allocating memory, searching directories, opening and closing files, reading and writing files, string handling, pattern matching, arithmetic, and so on.

**Approximate build time:**4.2 SBU

**Required disk space:**790 MB

### 5.7.1. Installation of Glibc

The Glibc documentation recommends building Glibc in a dedicated build directory:

    mkdir -v build
    cd       build

Next, prepare Glibc for compilation:

    ../configure                             \
          --prefix=/tools                    \
          --host=$LFS_TGT                    \
          --build=$(../scripts/config.guess) \
          --enable-kernel=3.2             \
          --with-headers=/tools/include      \
          libc_cv_forced_unwind=yes          \
          libc_cv_c_cleanup=yes

**The meaning of the configure options:**
*`--host=$LFS_TGT, --build=$(../scripts/config.guess)`*
The combined effect of these switches is that Glibc's build system configures itself to cross-compile, using the cross-linker and cross-compiler in `/tools`.
*`--enable-kernel=3.2`*
This tells Glibc to compile the library with support for 3.2 and later Linux kernels. Workarounds for older kernels are not enabled.
*`--with-headers=/tools/include`*
This tells Glibc to compile itself against the headers recently installed to the tools directory, so that it knows exactly what features the kernel has and can optimize itself accordingly.
*`libc_cv_forced_unwind=yes`*
The linker installed during [Section 5.4, “Binutils-2.29 - Pass 1”](Linux%20From%20Scratch.html#ch-tools-binutils-pass1) was cross-compiled and as such cannot be used until Glibc has been installed. This means that the configure test for force-unwind support will fail, as it relies on a working linker. The libc_cv_forced_unwind=yes variable is passed in order to inform **configure** that force-unwind support is available without it having to run the test.
*`libc_cv_c_cleanup=yes`*
Similarly, we pass libc_cv_c_cleanup=yes through to the **configure** script so that the test is skipped and C cleanup handling support is configured.

During this stage the following warning might appear:

>     configure: WARNING:
>     *** These auxiliary programs are missing or
>     *** incompatible versions: msgfmt
>     *** some features will be disabled.
>     *** Check the INSTALL file for required versions.

The missing or incompatible **msgfmt** program is generally harmless. This **msgfmt** program is part of the Gettext package which the host distribution should provide.

### Note

There have been reports that this package may fail when building as a "parallel make". If this occurs, rerun the make command with a "-j1" option.

Compile the package:

    make

Install the package:

    make install

### Caution

At this point, it is imperative to stop and ensure that the basic functions (compiling and linking) of the new toolchain are working as expected. To perform a sanity check, run the following commands:

    echo 'int main(){}' > dummy.c
    $LFS_TGT-gcc dummy.c
    readelf -l a.out | grep ': /tools'

If everything is working correctly, there should be no errors, and the output of the last command will be of the form:

    [Requesting program interpreter: /tools/lib/ld-linux.so.2]

Note that for 64-bit machines, the interpreter name will be `/tools/lib64/ld-linux-x86-64.so.2`.

If the output is not shown as above or there was no output at all, then something is wrong. Investigate and retrace the steps to find out where the problem is and correct it. This issue must be resolved before continuing on.

Once all is well, clean up the test files:

    rm -v dummy.c a.out

### Note

Building Binutils in the section after next will serve as an additional check that the toolchain has been built properly. If Binutils fails to build, it is an indication that something has gone wrong with the previous Binutils, GCC, or Glibc installations.

Details on this package are located in [Section 6.9.3, “Contents of Glibc.”](Linux%20From%20Scratch.html#contents-glibc)

Last updated on

## 5.8. Libstdc++-7.2.0

Libstdc++ is the standard C++ library. It is needed for the correct operation of the g++ compiler.

**Approximate build time:**0.4 SBU

**Required disk space:**750 MB

### 5.8.1. Installation of Target Libstdc++

### Note

Libstdc++ is part of the GCC sources. You should first unpack the GCC tarball and change to the `gcc-7.2.0` directory.

Create a separate build directory for Libstdc++ and enter it:

    mkdir -v build
    cd       build

Prepare Libstdc++ for compilation:

    ../libstdc++-v3/configure           \
        --host=$LFS_TGT                 \
        --prefix=/tools                 \
        --disable-multilib              \
        --disable-nls                   \
        --disable-libstdcxx-threads     \
        --disable-libstdcxx-pch         \
        --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/7.2.0

**The meaning of the configure options:**
*`--host=...`*
Indicates to use the cross compiler we have just built instead of the one in `/usr/bin`.
*`--disable-libstdcxx-threads`*
Since we have not yet built the C threads library, the C++ one cannot be built either.
*`--disable-libstdcxx-pch`*
This switch prevents the installation of precompiled include files, which are not needed at this stage.
*`--with-gxx-include-dir=/tools/$LFS_TGT/include/c++/7.2.0`*
This is the location where the standard include files are searched by the C++ compiler. In a normal build, this information is automatically passed to the Libstdc++ **configure** options from the top level directory. In our case, this information must be explicitly given.

Compile libstdc++ by running:

    make

Install the library:

    make install

Details on this package are located in [Section 6.20.2, “Contents of GCC.”](Linux%20From%20Scratch.html#contents-gcc)

Last updated on

## 5.9. Binutils-2.29 - Pass 2

The Binutils package contains a linker, an assembler, and other tools for handling object files.

**Approximate build time:**1.1 SBU

**Required disk space:**582 MB

### 5.9.1. Installation of Binutils

Create a separate build directory again:

    mkdir -v build
    cd       build

Prepare Binutils for compilation:

    CC=$LFS_TGT-gcc                \
    AR=$LFS_TGT-ar                 \
    RANLIB=$LFS_TGT-ranlib         \
    ../configure                   \
        --prefix=/tools            \
        --disable-nls              \
        --disable-werror           \
        --with-lib-path=/tools/lib \
        --with-sysroot

**The meaning of the new configure options:**
*`CC=$LFS_TGT-gcc AR=$LFS_TGT-ar RANLIB=$LFS_TGT-ranlib`*
Because this is really a native build of Binutils, setting these variables ensures that the build system uses the cross-compiler and associated tools instead of the ones on the host system.
*`--with-lib-path=/tools/lib`*
This tells the configure script to specify the library search path during the compilation of Binutils, resulting in `/tools/lib` being passed to the linker. This prevents the linker from searching through library directories on the host.
*`--with-sysroot`*
The sysroot feature enables the linker to find shared objects which are required by other shared objects explicitly included on the linker's command line. Without this, some packages may not build successfully on some hosts.

Compile the package:

    make

Install the package:

    make install

Now prepare the linker for the “Re-adjusting” phase in the next chapter:

    make -C ld clean
    make -C ld LIB_PATH=/usr/lib:/lib
    cp -v ld/ld-new /tools/bin

**The meaning of the make parameters:**
*`-C ld clean`*
This tells the make program to remove all compiled files in the `ld` subdirectory.
*`-C ld LIB_PATH=/usr/lib:/lib`*
This option rebuilds everything in the `ld` subdirectory. Specifying the `LIB_PATH` Makefile variable on the command line allows us to override the default value of the temporary tools and point it to the proper final path. The value of this variable specifies the linker's default library search path. This preparation is used in the next chapter.

Details on this package are located in [Section 6.16.2, “Contents of Binutils.”](Linux%20From%20Scratch.html#contents-binutils)

Last updated on

## 5.10. GCC-7.2.0 - Pass 2

The GCC package contains the GNU compiler collection, which includes the C and C++ compilers.

**Approximate build time:**11 SBU

**Required disk space:**2.6 GB

### 5.10.1. Installation of GCC

Our first build of GCC has installed a couple of internal system headers. Normally one of them, `limits.h`, will in turn include the corresponding system `limits.h` header, in this case, `/tools/include/limits.h`. However, at the time of the first build of gcc `/tools/include/limits.h` did not exist, so the internal header that GCC installed is a partial, self-contained file and does not include the extended features of the system header. This was adequate for building the temporary libc, but this build of GCC now requires the full internal header. Create a full version of the internal header using a command that is identical to what the GCC build system does in normal circumstances:

    cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
      `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include-fixed/limits.h

Once again, change the location of GCC's default dynamic linker to use the one installed in `/tools`.

    for file in gcc/config/{linux,i386/linux{,64}}.h
    do
      cp -uv $file{,.orig}
      sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
          -e 's@/usr@/tools@g' $file.orig > $file
      echo '
    #undef STANDARD_STARTFILE_PREFIX_1
    #undef STANDARD_STARTFILE_PREFIX_2
    #define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
    #define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
      touch $file.orig
    done

If building on x86_64, change the default directory name for 64-bit libraries to “lib”:

    case $(uname -m) in
      x86_64)
        sed -e '/m64=/s/lib64/lib/' \
            -i.orig gcc/config/i386/t-linux64
      ;;
    esac

As in the first build of GCC it requires the GMP, MPFR and MPC packages. Unpack the tarballs and move them into the required directory names:

    tar -xf ../mpfr-3.1.5.tar.xz
    mv -v mpfr-3.1.5 mpfr
    tar -xf ../gmp-6.1.2.tar.xz
    mv -v gmp-6.1.2 gmp
    tar -xf ../mpc-1.0.3.tar.gz
    mv -v mpc-1.0.3 mpc

Create a separate build directory again:

    mkdir -v build
    cd       build

Before starting to build GCC, remember to unset any environment variables that override the default optimization flags.

Now prepare GCC for compilation:

    CC=$LFS_TGT-gcc                                    \
    CXX=$LFS_TGT-g++                                   \
    AR=$LFS_TGT-ar                                     \
    RANLIB=$LFS_TGT-ranlib                             \
    ../configure                                       \
        --prefix=/tools                                \
        --with-local-prefix=/tools                     \
        --with-native-system-header-dir=/tools/include \
        --enable-languages=c,c++                       \
        --disable-libstdcxx-pch                        \
        --disable-multilib                             \
        --disable-bootstrap                            \
        --disable-libgomp

**The meaning of the new configure options:**
*`--enable-languages=c,c++`*
This option ensures that both the C and C++ compilers are built.
*`--disable-libstdcxx-pch`*
Do not build the pre-compiled header (PCH) for `libstdc++`. It takes up a lot of space, and we have no use for it.
*`--disable-bootstrap`*
For native builds of GCC, the default is to do a "bootstrap" build. This does not just compile GCC, but compiles it several times. It uses the programs compiled in a first round to compile itself a second time, and then again a third time. The second and third iterations are compared to make sure it can reproduce itself flawlessly. This also implies that it was compiled correctly. However, the LFS build method should provide a solid compiler without the need to bootstrap each time.

Compile the package:

    make

Install the package:

    make install

As a finishing touch, create a symlink. Many programs and scripts run **cc** instead of **gcc**, which is used to keep programs generic and therefore usable on all kinds of UNIX systems where the GNU C compiler is not always installed. Running **cc** leaves the system administrator free to decide which C compiler to install:

    ln -sv gcc /tools/bin/cc

### Caution

At this point, it is imperative to stop and ensure that the basic functions (compiling and linking) of the new toolchain are working as expected. To perform a sanity check, run the following commands:

    echo 'int main(){}' > dummy.c
    cc dummy.c
    readelf -l a.out | grep ': /tools'

If everything is working correctly, there should be no errors, and the output of the last command will be of the form:

    [Requesting program interpreter: /tools/lib/ld-linux.so.2]

Note that `/tools/lib`, or `/tools/lib64` for 64-bit machines appears as the prefix of the dynamic linker.

If the output is not shown as above or there was no output at all, then something is wrong. Investigate and retrace the steps to find out where the problem is and correct it. This issue must be resolved before continuing on. First, perform the sanity check again, using **gcc** instead of **cc**. If this works, then the `/tools/bin/cc` symlink is missing. Install the symlink as per above. Next, ensure that the `PATH` is correct. This can be checked by running **echo $PATH** and verifying that `/tools/bin` is at the head of the list. If the `PATH` is wrong it could mean that you are not logged in as user `lfs` or that something went wrong back in [Section 4.4, “Setting Up the Environment.”](Linux%20From%20Scratch.html#ch-tools-settingenviron)

Once all is well, clean up the test files:

    rm -v dummy.c a.out

Details on this package are located in [Section 6.20.2, “Contents of GCC.”](Linux%20From%20Scratch.html#contents-gcc)

Last updated on

## 5.11. Tcl-core-8.6.7

The Tcl package contains the Tool Command Language.

**Approximate build time:**0.4 SBU

**Required disk space:**42 MB

### 5.11.1. Installation of Tcl-core

This package and the next three (Expect, DejaGNU, and Check) are installed to support running the test suites for GCC and Binutils and other packages. Installing four packages for testing purposes may seem excessive, but it is very reassuring, if not essential, to know that the most important tools are working properly. Even if the test suites are not run in this chapter (they are not mandatory), these packages are required to run the test suites in [Chapter 6](Linux%20From%20Scratch.html#chapter-building-system).

Note that the Tcl package used here is a minimal version needed to run the LFS tests. For the full package, see the [BLFS Tcl procedures](http://www.linuxfromscratch.org/blfs/view/8.1/general/tcl.html).

Prepare Tcl for compilation:

    cd unix
    ./configure --prefix=/tools

Build the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Tcl test suite anyway, issue the following command:

    TZ=UTC make test

The Tcl test suite may experience failures under certain host conditions that are not fully understood. Therefore, test suite failures here are not surprising, and are not considered critical. The *`TZ=UTC`* parameter sets the time zone to Coordinated Universal Time (UTC), but only for the duration of the test suite run. This ensures that the clock tests are exercised correctly. Details on the `TZ` environment variable are provided in [Chapter 7](Linux%20From%20Scratch.html#chapter-bootscripts).

Install the package:

    make install

Make the installed library writable so debugging symbols can be removed later:

    chmod -v u+w /tools/lib/libtcl8.6.so

Install Tcl's headers. The next package, Expect, requires them to build.

    make install-private-headers

Now make a necessary symbolic link:

    ln -sv tclsh8.6 /tools/bin/tclsh

### 5.11.2. Contents of Tcl-core

**Installed programs:**tclsh (link to tclsh8.6) and tclsh8.6

**Installed library:**libtcl8.6.so, libtclstub8.6.a

#### Short Descriptions

**tclsh8.6**

The Tcl command shell

**tclsh**

A link to tclsh8.6

`libtcl8.6.so`

The Tcl library

`libtclstub8.6.a`

The Tcl Stub library

Last updated on

## 5.12. Expect-5.45

The Expect package contains a program for carrying out scripted dialogues with other interactive programs.

**Approximate build time:**0.1 SBU

**Required disk space:**4.3 MB

### 5.12.1. Installation of Expect

First, force Expect's configure script to use `/bin/stty` instead of a `/usr/local/bin/stty` it may find on the host system. This will ensure that our test suite tools remain sane for the final builds of our toolchain:

    cp -v configure{,.orig}
    sed 's:/usr/local/bin:/bin:' configure.orig > configure

Now prepare Expect for compilation:

    ./configure --prefix=/tools       \
                --with-tcl=/tools/lib \
                --with-tclinclude=/tools/include

**The meaning of the configure options:**
*`--with-tcl=/tools/lib`*
This ensures that the configure script finds the Tcl installation in the temporary tools location instead of possibly locating an existing one on the host system.
*`--with-tclinclude=/tools/include`*
This explicitly tells Expect where to find Tcl's internal headers. Using this option avoids conditions where **configure** fails because it cannot automatically discover the location of Tcl's headers.

Build the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Expect test suite anyway, issue the following command:

    make test

Note that the Expect test suite is known to experience failures under certain host conditions that are not within our control. Therefore, test suite failures here are not surprising and are not considered critical.

Install the package:

    make SCRIPTS="" install

**The meaning of the make parameter:**
*`SCRIPTS=""`*
This prevents installation of the supplementary Expect scripts, which are not needed.

### 5.12.2. Contents of Expect

**Installed program:**expect

**Installed library:**libexpect-5.45.so

#### Short Descriptions

**expect**

Communicates with other interactive programs according to a script

`libexpect-5.45.so`

Contains functions that allow Expect to be used as a Tcl extension or to be used directly from C or C++ (without Tcl)

Last updated on

## 5.13. DejaGNU-1.6

The DejaGNU package contains a framework for testing other programs.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**3.2 MB

### 5.13.1. Installation of DejaGNU

Prepare DejaGNU for compilation:

    ./configure --prefix=/tools

Build and install the package:

    make install

To test the results, issue:

    make check

### 5.13.2. Contents of DejaGNU

**Installed program:**runtest

#### Short Descriptions

**runtest**

A wrapper script that locates the proper **expect** shell and then runs DejaGNU

Last updated on

## 5.14. Check-0.11.0

Check is a unit testing framework for C.

**Approximate build time:**0.1 SBU

**Required disk space:**11 MB

### 5.14.1. Installation of Check

Prepare Check for compilation:

    PKG_CONFIG= ./configure --prefix=/tools

**The meaning of the configure parameter:**
*`PKG_CONFIG=`*
This tells the configure script to ignore any pkg-config options that may cause the system to try to link with libraries not in the `/tools`directory.

Build the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Check test suite anyway, issue the following command:

    make check

Note that the Check test suite may take a relatively long (up to 4 SBU) time.

Install the package:

    make install

### 5.14.2. Contents of Check

**Installed program:**checkmk

**Installed library:**libcheck.{a,so}

#### Short Descriptions

**checkmk**

Awk script for generating C unit tests for use with the Check unit testing framework

`libcheck.{a,so}`

Contains functions that allow Check to be called from a test program

Last updated on

## 5.15. Ncurses-6.0

The Ncurses package contains libraries for terminal-independent handling of character screens.

**Approximate build time:**0.5 SBU

**Required disk space:**38 MB

### 5.15.1. Installation of Ncurses

First, ensure that **gawk** is found first during configuration:

    sed -i s/mawk// configure

Prepare Ncurses for compilation:

    ./configure --prefix=/tools \
                --with-shared   \
                --without-debug \
                --without-ada   \
                --enable-widec  \
                --enable-overwrite

**The meaning of the configure options:**
*`--without-ada`*
This ensures that Ncurses does not build support for the Ada compiler which may be present on the host but will not be available once we enter the **chroot** environment.
*`--enable-overwrite`*
This tells Ncurses to install its header files into `/tools/include`, instead of `/tools/include/ncurses`, to ensure that other packages can find the Ncurses headers successfully.
*`--enable-widec`*
This switch causes wide-character libraries (e.g., `libncursesw.so.6.0`) to be built instead of normal ones (e.g., `libncurses.so.6.0`). These wide-character libraries are usable in both multibyte and traditional 8-bit locales, while normal libraries work properly only in 8-bit locales. Wide-character and normal libraries are source-compatible, but not binary-compatible.

Compile the package:

    make

This package has a test suite, but it can only be run after the package has been installed. The tests reside in the `test/` directory. See the`README` file in that directory for further details.

Install the package:

    make install

Details on this package are located in [Section 6.23.2, “Contents of Ncurses.”](Linux%20From%20Scratch.html#contents-ncurses)

Last updated on

## 5.16. Bash-4.4

The Bash package contains the Bourne-Again SHell.

**Approximate build time:**0.4 SBU

**Required disk space:**61 MB

### 5.16.1. Installation of Bash

Prepare Bash for compilation:

    ./configure --prefix=/tools --without-bash-malloc

**The meaning of the configure options:**
*`--without-bash-malloc`*
This option turns off the use of Bash's memory allocation (`malloc`) function which is known to cause segmentation faults. By turning this option off, Bash will use the `malloc` functions from Glibc which are more stable.

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Bash test suite anyway, issue the following command:

    make tests

Install the package:

    make install

Make a link for the programs that use **sh** for a shell:

    ln -sv bash /tools/bin/sh

Details on this package are located in [Section 6.34.2, “Contents of Bash.”](Linux%20From%20Scratch.html#contents-bash)

Last updated on

## 5.17. Bison-3.0.4

The Bison package contains a parser generator.

**Approximate build time:**0.3 SBU

**Required disk space:**32 MB

### 5.17.1. Installation of Bison

Prepare Bison for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.31.2, “Contents of Bison.”](Linux%20From%20Scratch.html#contents-bison)

Last updated on

## 5.18. Bzip2-1.0.6

The Bzip2 package contains programs for compressing and decompressing files. Compressing text files with **bzip2** yields a much better compression percentage than with the traditional **gzip**.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**5.2 MB

### 5.18.1. Installation of Bzip2

The Bzip2 package does not contain a **configure** script. Compile and test it with:

    make

Install the package:

    make PREFIX=/tools install

Details on this package are located in [Section 6.21.2, “Contents of Bzip2.”](Linux%20From%20Scratch.html#contents-bzip2)

Last updated on

## 5.19. Coreutils-8.27

The Coreutils package contains utilities for showing and setting the basic system characteristics.

**Approximate build time:**0.6 SBU

**Required disk space:**136 MB

### 5.19.1. Installation of Coreutils

Prepare Coreutils for compilation:

    ./configure --prefix=/tools --enable-install-program=hostname

**The meaning of the configure options:**
`--enable-install-program=hostname`
This enables the **hostname** binary to be built and installed – it is disabled by default but is required by the Perl test suite.

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Coreutils test suite anyway, issue the following command:

    make RUN_EXPENSIVE_TESTS=yes check

The *`RUN_EXPENSIVE_TESTS=yes`* parameter tells the test suite to run several additional tests that are considered relatively expensive (in terms of CPU power and memory usage) on some platforms, but generally are not a problem on Linux.

Install the package:

    make install

Details on this package are located in [Section 6.50.2, “Contents of Coreutils.”](Linux%20From%20Scratch.html#contents-coreutils)

Last updated on

## 5.20. Diffutils-3.6

The Diffutils package contains programs that show the differences between files or directories.

**Approximate build time:**0.2 SBU

**Required disk space:**22 MB

### 5.20.1. Installation of Diffutils

Prepare Diffutils for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Diffutils test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.51.2, “Contents of Diffutils.”](Linux%20From%20Scratch.html#contents-diffutils)

Last updated on

## 5.21. File-5.31

The File package contains a utility for determining the type of a given file or files.

**Approximate build time:**0.1 SBU

**Required disk space:**16 MB

### 5.21.1. Installation of File

Prepare File for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the File test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.12.2, “Contents of File.”](Linux%20From%20Scratch.html#contents-file)

Last updated on

## 5.22. Findutils-4.6.0

The Findutils package contains programs to find files. These programs are provided to recursively search through a directory tree and to create, maintain, and search a database (often faster than the recursive find, but unreliable if the database has not been recently updated).

**Approximate build time:**0.3 SBU

**Required disk space:**35 MB

### 5.22.1. Installation of Findutils

Prepare Findutils for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Findutils test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.53.2, “Contents of Findutils.”](Linux%20From%20Scratch.html#contents-findutils)

Last updated on

## 5.23. Gawk-4.1.4

The Gawk package contains programs for manipulating text files.

**Approximate build time:**0.2 SBU

**Required disk space:**35 MB

### 5.23.1. Installation of Gawk

Prepare Gawk for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Gawk test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.52.2, “Contents of Gawk.”](Linux%20From%20Scratch.html#contents-gawk)

Last updated on

## 5.24. Gettext-0.19.8.1

The Gettext package contains utilities for internationalization and localization. These allow programs to be compiled with NLS (Native Language Support), enabling them to output messages in the user's native language.

**Approximate build time:**0.8 SBU

**Required disk space:**164 MB

### 5.24.1. Installation of Gettext

For our temporary set of tools, we only need to build and install three programs from Gettext.

Prepare Gettext for compilation:

    cd gettext-tools
    EMACS="no" ./configure --prefix=/tools --disable-shared

**The meaning of the configure option:**
*`EMACS="no"`*
This prevents the configure script from determining where to install Emacs Lisp files as the test is known to hang on some hosts.
*`--disable-shared`*
We do not need to install any of the shared Gettext libraries at this time, therefore there is no need to build them.

Compile the package:

    make -C gnulib-lib
    make -C intl pluralx.c
    make -C src msgfmt
    make -C src msgmerge
    make -C src xgettext

As only three programs have been compiled, it is not possible to run the test suite without compiling additional support libraries from the Gettext package. It is therefore not recommended to attempt to run the test suite at this stage.

Install the **msgfmt**, **msgmerge** and **xgettext** programs:

    cp -v src/{msgfmt,msgmerge,xgettext} /tools/bin

Details on this package are located in [Section 6.47.2, “Contents of Gettext.”](Linux%20From%20Scratch.html#contents-gettext)

Last updated on

## 5.25. Grep-3.1

The Grep package contains programs for searching through files.

**Approximate build time:**0.2 SBU

**Required disk space:**19 MB

### 5.25.1. Installation of Grep

Prepare Grep for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Grep test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.33.2, “Contents of Grep.”](Linux%20From%20Scratch.html#contents-grep)

Last updated on

## 5.26. Gzip-1.8

The Gzip package contains programs for compressing and decompressing files.

**Approximate build time:**0.1 SBU

**Required disk space:**9 MB

### 5.26.1. Installation of Gzip

Prepare Gzip for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Gzip test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.57.2, “Contents of Gzip.”](Linux%20From%20Scratch.html#contents-gzip)

Last updated on

## 5.27. M4-1.4.18

The M4 package contains a macro processor.

**Approximate build time:**0.2 SBU

**Required disk space:**19 MB

### 5.27.1. Installation of M4

Prepare M4 for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the M4 test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.14.2, “Contents of M4.”](Linux%20From%20Scratch.html#contents-m4)

Last updated on

## 5.28. Make-4.2.1

The Make package contains a program for compiling packages.

**Approximate build time:**0.1 SBU

**Required disk space:**12.5 MB

### 5.28.1. Installation of Make

Prepare Make for compilation:

    ./configure --prefix=/tools --without-guile

**The meaning of the configure option:**
*`--without-guile`*
This ensures that Make-4.2.1 won't link against Guile libraries, which may be present on the host system, but won't be available within the **chroot** environment in the next chapter.

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Make test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.61.2, “Contents of Make.”](Linux%20From%20Scratch.html#contents-make)

Last updated on

## 5.29. Patch-2.7.5

The Patch package contains a program for modifying or creating files by applying a “patch” file typically created by the **diff** program.

**Approximate build time:**0.2 SBU

**Required disk space:**11 MB

### 5.29.1. Installation of Patch

Prepare Patch for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Patch test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.62.2, “Contents of Patch.”](Linux%20From%20Scratch.html#contents-patch)

Last updated on

## 5.30. Perl-5.26.0

The Perl package contains the Practical Extraction and Report Language.

**Approximate build time:**1.3 SBU

**Required disk space:**261 MB

### 5.30.1. Installation of Perl

First, fix a build issue that arises only in the LFS environment:

    sed -e '9751 a#ifndef PERL_IN_XSUB_RE' \
        -e '9808 a#endif'                  \
        -i regexec.c

Prepare Perl for compilation:

    sh Configure -des -Dprefix=/tools -Dlibs=-lm

Build the package:

    make

Although Perl comes with a test suite, it would be better to wait until it is installed in the next chapter.

Only a few of the utilities and libraries need to be installed at this time:

    cp -v perl cpan/podlators/scripts/pod2man /tools/bin
    mkdir -pv /tools/lib/perl5/5.26.0
    cp -Rv lib/* /tools/lib/perl5/5.26.0

Details on this package are located in [Section 6.40.2, “Contents of Perl.”](Linux%20From%20Scratch.html#contents-perl)

Last updated on

## 5.31. Sed-4.4

The Sed package contains a stream editor.

**Approximate build time:**0.2 SBU

**Required disk space:**16 MB

### 5.31.1. Installation of Sed

Prepare Sed for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Sed test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.27.2, “Contents of Sed.”](Linux%20From%20Scratch.html#contents-sed)

Last updated on

## 5.32. Tar-1.29

The Tar package contains an archiving program.

**Approximate build time:**0.3 SBU

**Required disk space:**33 MB

### 5.32.1. Installation of Tar

Prepare Tar for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Tar test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.68.2, “Contents of Tar.”](Linux%20From%20Scratch.html#contents-tar)

Last updated on

## 5.33. Texinfo-6.4

The Texinfo package contains programs for reading, writing, and converting info pages.

**Approximate build time:**0.2 SBU

**Required disk space:**99 MB

### 5.33.1. Installation of Texinfo

Prepare Texinfo for compilation:

    ./configure --prefix=/tools

### Note

As part of the configure process, a test is made that indicates an error for TestXS_la-TestXS.lo. This is not relevant for LFS and should be ignored.

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Texinfo test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.69.2, “Contents of Texinfo.”](Linux%20From%20Scratch.html#contents-texinfo)

Last updated on

## 5.34. Util-linux-2.30.1

The Util-linux package contains miscellaneous utility programs.

**Approximate build time:**0.8 SBU

**Required disk space:**123 MB

### 5.34.1. Installation of Util-linux

Prepare Util-linux for compilation:

    ./configure --prefix=/tools                \
                --without-python               \
                --disable-makeinstall-chown    \
                --without-systemdsystemunitdir \
                --without-ncurses              \
                PKG_CONFIG=""

**The meaning of the configure option:**
*`--without-python`*
This switch disables using Python if it is installed on the host system. It avoids trying to build unneeded bindings.
*`--disable-makeinstall-chown`*
This switch disables using the **chown** command during installation. This is not needed when installing into the /tools directory and avoids the necessity of installing as root.
*`--without-ncurses`*
This switch disables using the ncurses library for the build process. This is not needed when installing into the /tools directory and avoids problems on some host distros.
*`--without-systemdsystemunitdir`*
On systems that use systemd, the package tries to install a systemd specific file to a non-existent directory in /tools. This switch disables the unnecessary action.
`PKG_CONFIG=""`
Setting this environment variable prevents adding unneeded features that may be available on the host. Note that the location shown for setting this environment variable is different from other LFS sections where variables are set preceding the command. This location is shown to demonstrate an alternative way of setting an environment variable when using configure.

Compile the package:

    make

Install the package:

    make install

Last updated on

## 5.35. Xz-5.2.3

The Xz package contains programs for compressing and decompressing files. It provides capabilities for the lzma and the newer xz compression formats. Compressing text files with **xz** yields a better compression percentage than with the traditional **gzip** or **bzip2**commands.

**Approximate build time:**0.2 SBU

**Required disk space:**17 MB

### 5.35.1. Installation of Xz

Prepare Xz for compilation:

    ./configure --prefix=/tools

Compile the package:

    make

Compilation is now complete. As discussed earlier, running the test suite is not mandatory for the temporary tools here in this chapter. To run the Xz test suite anyway, issue the following command:

    make check

Install the package:

    make install

Details on this package are located in [Section 6.45.2, “Contents of Xz.”](Linux%20From%20Scratch.html#contents-xz)

Last updated on

## 5.36. Stripping

The steps in this section are optional, but if the LFS partition is rather small, it is beneficial to learn that unnecessary items can be removed. The executables and libraries built so far contain about 70 MB of unneeded debugging symbols. Remove those symbols with:

    strip --strip-debug /tools/lib/*
    /usr/bin/strip --strip-unneeded /tools/{,s}bin/*

These commands will skip a number of files, reporting that it does not recognize their file format. Most of these are scripts instead of binaries. Also use the system strip command to include the strip binary in /tools.

Take care *not* to use *`--strip-unneeded`* on the libraries. The static ones would be destroyed and the toolchain packages would need to be built all over again.

To save more, remove the documentation:

    rm -rf /tools/{,share}/{info,man,doc}

At this point, you should have at least 3 GB of free space in `$LFS` that can be used to build and install Glibc and Gcc in the next phase. If you can build and install Glibc, you can build and install the rest too.

## 5.37. Changing Ownership

### Note

The commands in the remainder of this book must be performed while logged in as user `root` and no longer as user `lfs`. Also, double check that `$LFS` is set in `root`'s environment.

Currently, the `$LFS/tools` directory is owned by the user `lfs`, a user that exists only on the host system. If the `$LFS/tools` directory is kept as is, the files are owned by a user ID without a corresponding account. This is dangerous because a user account created later could get this same user ID and would own the `$LFS/tools` directory and all the files therein, thus exposing these files to possible malicious manipulation.

To avoid this issue, you could add the `lfs` user to the new LFS system later when creating the `/etc/passwd` file, taking care to assign it the same user and group IDs as on the host system. Better yet, change the ownership of the `$LFS/tools` directory to user `root` by running the following command:

    chown -R root:root $LFS/tools

Although the `$LFS/tools` directory can be deleted once the LFS system has been finished, it can be retained to build additional LFS systems *of the same book version*. How best to backup `$LFS/tools` is a matter of personal preference.

### Caution

If you intend to keep the temporary tools for use in building future LFS systems, *now* is the time to back them up. Subsequent commands in chapter 6 will alter the tools currently in place, rendering them useless for future builds.
