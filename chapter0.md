# Preface

## Foreword <span id="1"></span>

My journey to learn and better understand Linux began over a decade ago, back in 1998. I had just installed my first Linux distribution and had quickly become intrigued with the whole concept and philosophy behind Linux.

There are always many ways to accomplish a single task. The same can be said about Linux distributions. A great many have existed over the years. Some still exist, some have morphed into something else, yet others have been relegated to our memories. They all do things differently to suit the needs of their target audience. Because so many different ways to accomplish the same end goal exist, I began to realize I no longer had to be limited by any one implementation. Prior to discovering Linux, we simply put up with issues in other Operating Systems as you had no choice. It was what it was, whether you liked it or not. With Linux, the concept of choice began to emerge. If you didn't like something, you were free, even encouraged, to change it.

I tried a number of distributions and could not decide on any one. They were great systems in their own right. It wasn't a matter of right and wrong anymore. It had become a matter of personal taste. With all that choice available, it became apparent that there would not be a single system that would be perfect for me. So I set out to create my own Linux system that would fully conform to my personal preferences.

To truly make it my own system, I resolved to compile everything from source code instead of using pre-compiled binary packages. This “perfect” Linux system would have the strengths of various systems without their perceived weaknesses. At first, the idea was rather daunting. I remained committed to the idea that such a system could be built.

After sorting through issues such as circular dependencies and compile-time errors, I finally built a custom-built Linux system. It was fully operational and perfectly usable like any of the other Linux systems out there at the time. But it was my own creation. It was very satisfying to have put together such a system myself. The only thing better would have been to create each piece of software myself. This was the next best thing.

As I shared my goals and experiences with other members of the Linux community, it became apparent that there was a sustained interest in these ideas. It quickly became plain that such custom-built Linux systems serve not only to meet user specific requirements, but also serve as an ideal learning opportunity for programmers and system administrators to enhance their (existing) Linux skills. Out of this broadened interest, the *Linux From Scratch Project* was born.

This Linux From Scratch book is the central core around that project. It provides the background and instructions necessary for you to design and build your own system. While this book provides a template that will result in a correctly working system, you are free to alter the instructions to suit yourself, which is, in part, an important part of this project. You remain in control; we just lend a helping hand to get you started on your own journey.

I sincerely hope you will have a great time working on your own Linux From Scratch system and enjoy the numerous benefits of having a system that is truly your own.

--
Gerard Beekmans
gerard AT linuxfromscratch D0T org

## Audience <span id="2"></span>

There are many reasons why you would want to read this book. One of the questions many people raise is, “why go through all the hassle of manually building a Linux system from scratch when you can just download and install an existing one?”

One important reason for this project's existence is to help you learn how a Linux system works from the inside out. Building an LFS system helps demonstrate what makes Linux tick, and how things work together and depend on each other. One of the best things that this learning experience can provide is the ability to customize a Linux system to suit your own unique needs.

Another key benefit of LFS is that it allows you to have more control over the system without relying on someone else's Linux implementation. With LFS, you are in the driver's seat and dictate every aspect of the system.

LFS allows you to create very compact Linux systems. When installing regular distributions, you are often forced to install a great many programs which are probably never used or understood. These programs waste resources. You may argue that with today's hard drive and CPUs, such resources are no longer a consideration. Sometimes, however, you are still constrained by size considerations if nothing else. Think about bootable CDs, USB sticks, and embedded systems. Those are areas where LFS can be beneficial.

Another advantage of a custom built Linux system is security. By compiling the entire system from source code, you are empowered to audit everything and apply all the security patches desired. It is no longer necessary to wait for somebody else to compile binary packages that fix a security hole. Unless you examine the patch and implement it yourself, you have no guarantee that the new binary package was built correctly and adequately fixes the problem.

The goal of Linux From Scratch is to build a complete and usable foundation-level system. If you do not wish to build your own Linux system from scratch, you may not entirely benefit from the information in this book.

There are too many other good reasons to build your own LFS system to list them all here. In the end, education is by far the most powerful of reasons. As you continue in your LFS experience, you will discover the power that information and knowledge truly bring.

## LFS Target Architectures <span id="3"></span>

The primary target architectures of LFS are the AMD/Intel x86 (32-bit) and x86_64 (64-bit) CPUs. On the other hand, the instructions in this book are also known to work, with some modifications, with the Power PC and ARM CPUs. To build a system that utilizes one of these CPUs, the main prerequisite, in addition to those on the next few pages, is an existing Linux system such as an earlier LFS installation, Ubuntu, Red Hat/Fedora, SuSE, or other distribution that targets the architecture that you have. Also note that a 32-bit distribution can be installed and used as a host system on a 64-bit AMD/Intel computer.

Some other facts about 64-bit systems need to be added here. When compared to a 32-bit system, the sizes of executable programs are slightly larger and the execution speeds are only slightly faster. For example, in a test build of LFS-6.5 on a Core2Duo CPU based system, the following statistics were measured:

    Architecture Build Time     Build Size
    32-bit       198.5 minutes  648 MB
    64-bit       190.6 minutes  709 MB

As you can see, the 64-bit build is only 4% faster and is 9% larger than the 32-bit build. The gain from going to a 64-bit system is relatively minimal. Of course, if you have more than 4GB of RAM or want to manipulate data that exceeds 4GB, the advantages of a 64-bit system are substantial.

The default 64-bit build that results from LFS is considered a "pure" 64-bit system. That is, it supports 64-bit executables only. Building a "multi-lib" system requires compiling many applications twice, once for a 32-bit system and once for a 64-bit system. This is not directly supported in LFS because it would interfere with the educational objective of providing the instructions needed for a straightforward base Linux system. You can refer to the [Cross Linux From Scratch](http://trac.clfs.org/) project for this advanced topic.

## LFS and Standards <span id="4"></span>

The structure of LFS follows Linux standards as closely as possible. The primary standards are:

  - [POSIX.1-2008](http://pubs.opengroup.org/onlinepubs/9699919799/).
  - [Filesystem Hierarchy Standard (FHS) Version 3.0](http://refspecs.linuxfoundation.org/fhs.shtml)
  - [Linux Standard Base (LSB) Version 5.0](http://refspecs.linuxfoundation.org/lsb.shtml)

The LSB has four separate standards: Core, Desktop, Runtime Languages, and Imaging. In addition to generic requirements there are also architecture specific requirements. There are also two areas for trial use: Gtk3 and Graphics. LFS attempts to conform to the architectures discussed in the previous section.

### Note

Many people do not agree with the requirements of the LSB. The main purpose of defining it is to ensure that proprietary software will be able to be installed and run properly on a compliant system. Since LFS is source based, the user has complete control over what packages are desired and many choose not to install some packages that are specified by the LSB.

Creating a complete LFS system capable of passing the LSB certifications tests is possible, but not without many additional packages that are beyond the scope of LFS. These additional packages have installation instructions in BLFS.

#### Packages supplied by LFS needed to satisfy the LSB Requirements

*LSB Core:*

Bash, Bc, Binutils, Coreutils, Diffutils, File, Findutils, Gawk, Grep, Gzip, M4, Man-DB, Ncurses, Procps, Psmisc, Sed, Shadow, Tar, Util-linux, Zlib

*LSB Desktop:*

None

*LSB Runtime Languages:*

Perl

*LSB Imaging:*

None

*LSB Gtk3 and LSB Graphics (Trial Use):*

None

#### Packages supplied by BLFS needed to satisfy the LSB Requirements

*LSB Core:*

At, Batch (a part of At), Cpio, Ed, Fcrontab, Initd-tools, Lsb_release, NSPR, NSS, PAM, Pax, Sendmail (or Postfix or Exim), time

*LSB Desktop:*

Alsa, ATK, Cairo, Desktop-file-utils, Freetype, Fontconfig, Gdk-pixbuf, Glib2, GTK+2, Icon-naming-utils, Libjpeg-turbo, Libpng, Libtiff, Libxml2, MesaLib, Pango, Qt4, Xdg-utils, Xorg

*LSB Runtime Languages:*

Python, Libxml2, Libxslt

*LSB Imaging:*

CUPS, Cups-filters, Ghostscript, SANE

*LSB Gtk3 and LSB Graphics (Trial Use):*

GTK+3

#### Packages not supplied by LFS or BLFS needed to satisfy the LSB Requirements

*LSB Core:*

None

*LSB Desktop:*

None

*LSB Runtime Languages:*

None

*LSB Imaging:*

None

*LSB Gtk3 and LSB Graphics (Trial Use):*

None

## Rationale for Packages in the Book <span id="5"></span>

As stated earlier, the goal of LFS is to build a complete and usable foundation-level system. This includes all packages needed to replicate itself while providing a relatively minimal base from which to customize a more complete system based on the choices of the user. This does not mean that LFS is the smallest system possible. Several important packages are included that are not strictly required. The lists below document the rationale for each package in the book.

- Acl

  This package contains utilities to administer Access Control Lists, which are used to define more fine-grained discretionary access rights for files and directories.

- Attr

  This package contains programs for administering extended attributes on filesystem objects.

- Autoconf

  This package contains programs for producing shell scripts that can automatically configure source code from a developer's template. It is often needed to rebuild a package after updates to the build procedures.

- Automake

  This package contains programs for generating Make files from a template. It is often needed to rebuild a package after updates to the build procedures.

- Bash

  This package satisfies an LSB core requirement to provide a Bourne Shell interface to the system. It was chosen over other shell packages because of its common usage and extensive capabilities beyond basic shell functions.

- Bc

  This package provides an arbitrary precision numeric processing language. It satisfies a requirement needed when building the Linux kernel.

- Binutils

  This package contains a linker, an assembler, and other tools for handling object files. The programs in this package are needed to compile most of the packages in an LFS system and beyond.

- Bison

  This package contains the GNU version of yacc (Yet Another Compiler Compiler) needed to build several other LFS programs.

- Bzip2

  This package contains programs for compressing and decompressing files. It is required to decompress many LFS packages.

- Check

  This package contains a test harness for other programs. It is only installed in the temporary toolchain.

- Coreutils

  This package contains a number of essential programs for viewing and manipulating files and directories. These programs are needed for command line file management, and are necessary for the installation procedures of every package in LFS.

- DejaGNU

  This package contains a framework for testing other programs. It is only installed in the temporary toolchain.

- Diffutils

  This package contains programs that show the differences between files or directories. These programs can be used to create patches, and are also used in many packages' build procedures.

- E2fsprogs

  This package contains the utilities for handling the ext2, ext3 and ext4 file systems. These are the most common and thoroughly tested file systems that Linux supports.

- Eudev

  This package is a device manager. It dynamically controls the entries in the /dev directory as devices are added or removed from the system.

- Expat

  This package contains a relatively small XML parsing library. It is required by the XML::Parser Perl module.

- Expect

  This package contains a program for carrying out scripted dialogues with other interactive programs. It is commonly used for testing other packages. It is only installed in the temporary toolchain.

- File

  This package contains a utility for determining the type of a given file or files. A few packages need it to build.

- Findutils

  This package contains programs to find files in a file system. It is used in many packages' build scripts.

- Flex

  This package contains a utility for generating programs that recognize patterns in text. It is the GNU version of the lex (lexical analyzer) program. It is required to build several LFS packages.

- Gawk

  This package contains programs for manipulating text files. It is the GNU version of awk (Aho-Weinberg-Kernighan). It is used in many other packages' build scripts.

- Gcc

  This package is the Gnu Compiler Collection. It contains the C and C++ compilers as well as several others not built by LFS.

- GDBM

  This package contains the GNU Database Manager library. It is used by one other LFS package, Man-DB.

- Gettext

  This package contains utilities and libraries for internationalization and localization of numerous packages.

- Glibc

  This package contains the main C library. Linux programs would not run without it.

- GMP

  This package contains math libraries that provide useful functions for arbitrary precision arithmetic. It is required to build Gcc.

- Gperf

  This package contains a program that generates a perfect hash function from a key set. It is required for Eudev.

- Grep

  This package contains programs for searching through files. These programs are used by most packages' build scripts.

- Groff

  This package contains programs for processing and formatting text. One important function of these programs is to format man pages.

- GRUB

  This package is the Grand Unified Boot Loader. It is one of several boot loaders available, but is the most flexible.

- Gzip

  This package contains programs for compressing and decompressing files. It is needed to decompress many packages in LFS and beyond.

- Iana-etc

  This package provides data for network services and protocols. It is needed to enable proper networking capabilities.

- Inetutils

  This package contains programs for basic network administration.

- Intltool

  This package contains tools for extracting translatable strings from source files.

- IProute2

  This package contains programs for basic and advanced IPv4 and IPv6 networking. It was chosen over the other common network tools package (net-tools) for its IPv6 capabilities.

- Kbd

  This package contains key-table files, keyboard utilities for non-US keyboards, and a number of console fonts.

- Kmod

  This package contains programs needed to administer Linux kernel modules.

- Less

  This package contains a very nice text file viewer that allows scrolling up or down when viewing a file. It is also used by Man-DB for viewing manpages.

- Libcap

  This package implements the user-space interfaces to the POSIX 1003.1e capabilities available in Linux kernels.

- Libpipeline

  The Libpipeline package contains a library for manipulating pipelines of subprocesses in a flexible and convenient way. It is required by the Man-DB package.

- Libtool

  This package contains the GNU generic library support script. It wraps the complexity of using shared libraries in a consistent, portable interface. It is needed by the test suites in other LFS packages.

- Linux Kernel

  This package is the Operating System. It is the Linux in the GNU/Linux environment.

- M4

  This package contains a general text macro processor useful as a build tool for other programs.

- Make

  This package contains a program for directing the building of packages. It is required by almost every package in LFS.

- Man-DB

  This package contains programs for finding and viewing man pages. It was chosen instead of the man package due to superior internationalization capabilities. It supplies the man program.

- Man-pages

  This package contains the actual contents of the basic Linux man pages.

- MPC

  This package contains functions for the arithmetic of complex numbers. It is required by Gcc.

- MPFR

  This package contains functions for multiple precision arithmetic. It is required by Gcc.

- Ncurses

  This package contains libraries for terminal-independent handling of character screens. It is often used to provide cursor control for a menuing system. It is needed by a number of packages in LFS.

- Patch

  This package contains a program for modifying or creating files by applying a *patch* file typically created by the diff program. It is needed by the build procedure for several LFS packages.

- Perl

  This package is an interpreter for the runtime language PERL. It is needed for the installation and test suites of several LFS packages.

- Pkg-config

  This package provides a program to return meta-data about an installed library or package.

- Procps-NG

  This package contains programs for monitoring processes. These programs are useful for system administration, and are also used by the LFS Bootscripts.

- Psmisc

  This package contains programs for displaying information about running processes. These programs are useful for system administration.

- Readline

  This package is a set of libraries that offers command-line editing and history capabilities. It is used by Bash.

- Sed

  This package allows editing of text without opening it in a text editor. It is also needed by most LFS packages' configure scripts.

- Shadow

  This package contains programs for handling passwords in a secure way.

- Sysklogd

  This package contains programs for logging system messages, such as those given by the kernel or daemon processes when unusual events occur.

- Sysvinit

  This package provides the init program, which is the parent of all other processes on the Linux system.

- Tar

  This package provides archiving and extraction capabilities of virtually all packages used in LFS.

- Tcl

  This package contains the Tool Command Language used in many test suites in LFS packages. It is only installed in the temporary toolchain.

- Texinfo

  This package contains programs for reading, writing, and converting info pages. It is used in the installation procedures of many LFS packages.

- Util-linux

  This package contains miscellaneous utility programs. Among them are utilities for handling file systems, consoles, partitions, and messages.

- Vim

  This package contains an editor. It was chosen because of its compatibility with the classic vi editor and its huge number of powerful capabilities. An editor is a very personal choice for many users and any other editor could be substituted if desired.

- XML::Parser

  This package is a Perl module that interfaces with Expat.

- XZ Utils

  This package contains programs for compressing and decompressing files. It provides the highest compression generally available and is useful for decompressing packages in XZ or LZMA format.

- Zlib

  This package contains compression and decompression routines used by some programs.


## Prerequisites <span id="6"></span>

Building an LFS system is not a simple task. It requires a certain level of existing knowledge of Unix system administration in order to resolve problems and correctly execute the commands listed. In particular, as an absolute minimum, you should already have the ability to use the command line (shell) to copy or move files and directories, list directory and file contents, and change the current directory. It is also expected that you have a reasonable knowledge of using and installing Linux software.

Because the LFS book assumes *at least* this basic level of skill, the various LFS support forums are unlikely to be able to provide you with much assistance in these areas. You will find that your questions regarding such basic knowledge will likely go unanswered or you will simply be referred to the LFS essential pre-reading list.

Before building an LFS system, we recommend reading the following:

-
Software-Building-HOWTO [http://www.tldp.org/HOWTO/Software-Building-HOWTO.html](http://www.tldp.org/HOWTO/Software-Building-HOWTO.html)

This is a comprehensive guide to building and installing “generic” Unix software packages under Linux. Although it was written some time ago, it still provides a good summary of the basic techniques needed to build and install software.

-
Beginner's Guide to Installing from Source [http://moi.vonos.net/linux/beginners-installing-from-source/](http://moi.vonos.net/linux/beginners-installing-from-source/)

This guide provides a good summary of basic skills and techniques needed to build software from source code.

## Typography <span id="7"></span>

To make things easier to follow, there are a few typographical conventions used throughout this book. This section contains some examples of the typographical format found throughout Linux From Scratch.

    ./configure --prefix=/usr

This form of text is designed to be typed exactly as seen unless otherwise noted in the surrounding text. It is also used in the explanation sections to identify which of the commands is being referenced.

In some cases, a logical line is extended to two or more physical lines with a backslash at the end of the line.

    CC="gcc -B/usr/bin/" ../binutils-2.18/configure \
      --prefix=/tools --disable-nls --disable-werror

Note that the backslash must be followed by an immediate return. Other whitespace characters like spaces or tab characters will create incorrect results.

    install-info: unknown option '--dir-file=/mnt/lfs/usr/info/dir'

This form of text (fixed-width text) shows screen output, usually as the result of commands issued. This format is also used to show filenames, such as `/etc/ld.so.conf`.

*Emphasis*

This form of text is used for several purposes in the book. Its main purpose is to emphasize important points or items.

[http://www.linuxfromscratch.org/](http://www.linuxfromscratch.org/)

This format is used for hyperlinks both within the LFS community and to external pages. It includes HOWTOs, download locations, and websites.

    cat > $LFS/etc/group << "EOF"
    root:x:0:
    bin:x:1:
    ......
    EOF

This format is used when creating configuration files. The first command tells the system to create the file `$LFS/etc/group` from whatever is typed on the following lines until the sequence End Of File (EOF) is encountered. Therefore, this entire section is generally typed as seen.

*`<REPLACED TEXT>`*

This format is used to encapsulate text that is not to be typed as seen or for copy-and-paste operations.

*`[OPTIONAL TEXT]`*

This format is used to encapsulate text that is optional.

`passwd(5)`

This format is used to refer to a specific manual (man) page. The number inside parentheses indicates a specific section inside the manuals. For example, **passwd** has two man pages. Per LFS installation instructions, those two man pages will be located at `/usr/share/man/man1/passwd.1` and `/usr/share/man/man5/passwd.5`. When the book uses `passwd(5)` it is specifically referring to `/usr/share/man/man5/passwd.5`. **man passwd** will print the first man page it finds that matches “passwd”, which will be `/usr/share/man/man1/passwd.1`. For this example, you will need to run **man 5 passwd** in order to read the specific page being referred to. It should be noted that most man pages do not have duplicate page names in different sections. Therefore, **man *`<program name>`*** is generally sufficient.


## Structure <span id="8"></span>

This book is divided into the following parts.

### Part I - Introduction

Part I explains a few important notes on how to proceed with the LFS installation. This section also provides meta-information about the book.

### Part II - Preparing for the Build

Part II describes how to prepare for the building process—making a partition, downloading the packages, and compiling temporary tools.

### Part III - Building the LFS System

Part III guides the reader through the building of the LFS system—compiling and installing all the packages one by one, setting up the boot scripts, and installing the kernel. The resulting Linux system is the foundation on which other software can be built to expand the system as desired. At the end of this book, there is an easy to use reference listing all of the programs, libraries, and important files that have been installed.

## Errata <span id="9"></span>

The software used to create an LFS system is constantly being updated and enhanced. Security warnings and bug fixes may become available after the LFS book has been released. To check whether the package versions or instructions in this release of LFS need any modifications to accommodate security vulnerabilities or other bug fixes, please visit [http://www.linuxfromscratch.org/lfs/errata/8.1/](http://www.linuxfromscratch.org/lfs/errata/8.1/) before proceeding with your build. You should note any changes shown and apply them to the relevant section of the book as you progress with building the LFS system.
