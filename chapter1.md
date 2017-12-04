# Part I. Introduction

## Chapter 1. Introduction

## 1.1. How to Build an LFS System

The LFS system will be built by using an already installed Linux distribution (such as Debian, OpenMandriva, Fedora, or openSUSE). This existing Linux system (the host) will be used as a starting point to provide necessary programs, including a compiler, linker, and shell, to build the new system. Select the “development” option during the distribution installation to be able to access these tools.

As an alternative to installing a separate distribution onto your machine, you may wish to use a LiveCD from a commercial distribution.

[Chapter 2](Linux%20From%20Scratch.html#chapter-partitioning) of this book describes how to create a new Linux native partition and file system. This is the place where the new LFS system will be compiled and installed. [Chapter 3](Linux%20From%20Scratch.html#chapter-getting-materials) explains which packages and patches need to be downloaded to build an LFS system and how to store them on the new file system. [Chapter 4](Linux%20From%20Scratch.html#chapter-final-preps) discusses the setup of an appropriate working environment. Please read [Chapter 4](Linux%20From%20Scratch.html#chapter-final-preps) carefully as it explains several important issues you need be aware of before beginning to work your way through [Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) and beyond.

[Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) explains the installation of a number of packages that will form the basic development suite (or toolchain) which is used to build the actual system in [Chapter 6](Linux%20From%20Scratch.html#chapter-building-system). Some of these packages are needed to resolve circular dependencies—for example, to compile a compiler, you need a compiler.

[Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) also shows you how to build a first pass of the toolchain, including Binutils and GCC (first pass basically means these two core packages will be reinstalled). The next step is to build Glibc, the C library. Glibc will be compiled by the toolchain programs built in the first pass. Then, a second pass of the toolchain will be built. This time, the toolchain will be dynamically linked against the newly built Glibc. The remaining [Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) packages are built using this second pass toolchain. When this is done, the LFS installation process will no longer depend on the host distribution, with the exception of the running kernel.

This effort to isolate the new system from the host distribution may seem excessive. A full technical explanation as to why this is done is provided in [Section 5.2, “Toolchain Technical Notes”](Linux%20From%20Scratch.html#ch-tools-toolchaintechnotes).

In [Chapter 6](Linux%20From%20Scratch.html#chapter-building-system), the full LFS system is built. The **chroot** (change root) program is used to enter a virtual environment and start a new shell whose root directory will be set to the LFS partition. This is very similar to rebooting and instructing the kernel to mount the LFS partition as the root partition. The system does not actually reboot, but instead uses **chroot** because creating a bootable system requires additional work which is not necessary just yet. The major advantage is that “chrooting” allows you to continue using the host system while LFS is being built. While waiting for package compilations to complete, you can continue using your computer as normal.

To finish the installation, the basic system configuration is set up in [Chapter 7](Linux%20From%20Scratch.html#chapter-bootscripts), and the kernel and boot loader are set up in [Chapter 8](Linux%20From%20Scratch.html#chapter-bootable). [Chapter 9](Linux%20From%20Scratch.html#chapter-finalizing) contains information on continuing the LFS experience beyond this book. After the steps in this book have been implemented, the computer will be ready to reboot into the new LFS system.

This is the process in a nutshell. Detailed information on each step is discussed in the following chapters and package descriptions. Items that may seem complicated will be clarified, and everything will fall into place as you embark on the LFS adventure.

## 1.2. What's new since the last release

Below is a list of package updates made since the previous release of the book.

**Upgraded to:**

-
Automake 1.15.1

-
Bc 1.07.1

-
Binutils 2.29

-
Coreutils 8.27

-
Diffutils 3.6

-
Eudev 3.2.2

-
E2fsprogs 1.43.5

-
Expat-2.2.3

-
File 5.31

-
Flex 2.6.4

-
GCC 7.2.0

-
GDBM 1.13

-
Glibc 2.26

-
Gperf-3.1

-
Grep 3.1

-
GRUB 2.02

-
IPRoute2 4.12.0

-
Kmod 24

-
Less 487

-
Libpipeline 1.4.2

-
Linux 4.12.7

-
Man-pages 4.12

-
Perl 5.26.0

-
Pkg-config 0.29.2

-
Psmisc 23.1

-
Shadow 4.5

-
Tcl-core-8.6.7

-
Texinfo 6.4

-
Tzdata 2017b

-
Util-Linux 2.30.1

-
Vim 8.0.586

**Added:**

**Removed:**

-
bc-1.06.95-memory_leak-1.patch

## 1.3. Changelog

This is version 8.1 of the Linux From Scratch book, dated September 1, 2017. If this book is more than six months old, a newer and better version is probably already available. To find out, please check one of the mirrors via [http://www.linuxfromscratch.org/mirrors.html](http://www.linuxfromscratch.org/mirrors.html).

Below is a list of changes made since the previous release of the book.

**Changelog Entries:**

-
2017-09-01

-
[bdubbs] - LFS-8.1 released.

-
2017-08-27

-
[bdubbs] - Remove invalid option from systemd book's util-linux.

-
2017-08-24

-
[dj] - Remove rpcgen from GlibC index.

-
[ken] - Fix acl's testsuite for perl-5.26.

-
2017-08-16

-
[bdubbs] - Remove unneeded options from glibc.

-
2017-08-16

-
[bdubbs] - Add a note to gmp that shows how to create generic gmp libraries.

-
2017-08-15

-
[bdubbs] - Update to gcc-7.2.0. Fixes [#4125](http://wiki.linuxfromscratch.org/lfs/ticket/4125).

-
[bdubbs] - Update to linux-4.12.7. Fixes [#4124](http://wiki.linuxfromscratch.org/lfs/ticket/4124).

-
[bdubbs] - Update to glibc-2.26. Fixes [#4120](http://wiki.linuxfromscratch.org/lfs/ticket/4120).

-
[bdubbs] - Update to binutils-2.29. Fixes [#4117](http://wiki.linuxfromscratch.org/lfs/ticket/4117).

-
2017-08-11

-
[bdubbs] - Update to tcl-core-8.6.7. Fixes [#4123](http://wiki.linuxfromscratch.org/lfs/ticket/4123).

-
2017-08-08

-
[bdubbs] - Update to e2fsprogs-1.43.5. Fixes [#4122](http://wiki.linuxfromscratch.org/lfs/ticket/4122).

-
[bdubbs] - Update to expat-2.2.3. Fixes [#4121](http://wiki.linuxfromscratch.org/lfs/ticket/4121).

-
[bdubbs] - Update to linux-4.12.5. Fixes [#4119](http://wiki.linuxfromscratch.org/lfs/ticket/4119).

-
2017-07-23

-
[dj] - Update to util-linux-2.30.1. Fixes [#4114](http://wiki.linuxfromscratch.org/lfs/ticket/4114).

-
[dj] - Update to linux-4.12.3. Fixes [#4115](http://wiki.linuxfromscratch.org/lfs/ticket/4115).

-
[dj] - Update to man-pages-4.12. Fixes [#4116](http://wiki.linuxfromscratch.org/lfs/ticket/4116).

-
2017-07-18

-
[dj] - Update to linux-4.12.2. Fixes [#4113](http://wiki.linuxfromscratch.org/lfs/ticket/4113).

-
[dj] - Update to expat-2.2.2. Fixes [#4112](http://wiki.linuxfromscratch.org/lfs/ticket/4112).

-
[dj] - Postpone systemd test suite until BLFS. Fixes [#4107](http://wiki.linuxfromscratch.org/lfs/ticket/4107).

-
2017-07-13

-
[bdubbs] - Update to linux-4.12.1. Fixes [#4110](http://wiki.linuxfromscratch.org/lfs/ticket/4110).

-
[bdubbs] - Update to libpipeline-1.4.2. Fixes [#4109](http://wiki.linuxfromscratch.org/lfs/ticket/4109).

-
[bdubbs] - Update to iproute2-4.12.0. Fixes [#4108](http://wiki.linuxfromscratch.org/lfs/ticket/4108).

-
[bdubbs] - Fix the test procedure in make. Fixes [#4105](http://wiki.linuxfromscratch.org/lfs/ticket/4105).

-
[bdubbs] - Fix the test procedure in procps-ng. Fixes [#4106](http://wiki.linuxfromscratch.org/lfs/ticket/4106).

-
[bdubbs] - Update to glibc-2.25+adc7e06. Fixes [#4097](http://wiki.linuxfromscratch.org/lfs/ticket/4097).

-
2017-07-03

-
[bdubbs] - Update to linux-4.12.

-
2017-07-02

-
[bdubbs] - Fix the test procedure in attr. Fixes [#4103](http://wiki.linuxfromscratch.org/lfs/ticket/4103).

-
[bdubbs] - Update to grep-3.1. Fixes [#4104](http://wiki.linuxfromscratch.org/lfs/ticket/4104).

-
2017-06-30

-
[bdubbs] - Update to linux-4.11.8. Fixes [#4099](http://wiki.linuxfromscratch.org/lfs/ticket/4090).

-
2017-06-26

-
[bdubbs] - Fix an error in the mountfs bootscript.

-
[bdubbs] - Remove outdated seds for diffutils in both Chapters 5 and 6.

-
2017-06-24

-
[bdubbs] - Update to texinfo-6.4. Fixes [#4100](http://wiki.linuxfromscratch.org/lfs/ticket/4100).

-
[bdubbs] - Update to linux-4.11.7. Fixes [#4099](http://wiki.linuxfromscratch.org/lfs/ticket/4090).

-
[bdubbs] - Remove section disussing configuration without a network card.

-
[bdubbs] - Update boot scripts to unmount network file systems before bringing down the network.

-
2017-06-21

-
[bdubbs] - Update to automake-1.15.1. Fixes [#4098](http://wiki.linuxfromscratch.org/lfs/ticket/4098).

-
[bdubbs] - Update to expat-2.2.1. Fixes [#4096](http://wiki.linuxfromscratch.org/lfs/ticket/4096).

-
[bdubbs] - Update to psmisc-23.1. Fixes [#4094](http://wiki.linuxfromscratch.org/lfs/ticket/4094).

-
[bdubbs] - Update to linux-4.11.6. Fixes [#4095](http://wiki.linuxfromscratch.org/lfs/ticket/4095).

-
2017-06-07

-
[bdubbs] - Update to linux-4.11.4. Fixes [#4093](http://wiki.linuxfromscratch.org/lfs/ticket/4093).

-
2017-06-02

-
[bdubbs] - Update to util-linux-2.30. Fixes [#4092](http://wiki.linuxfromscratch.org/lfs/ticket/4092).

-
[bdubbs] - Update to perl-5.26.0. Fixes [#4091](http://wiki.linuxfromscratch.org/lfs/ticket/4091).

-
[bdubbs] - Update to file-5.31. Fixes [#4090](http://wiki.linuxfromscratch.org/lfs/ticket/4090).

-
[bdubbs] - Update to diffutils-3.6. Fixes [#4089](http://wiki.linuxfromscratch.org/lfs/ticket/4089).

-
[bdubbs] - Update to linux-4.11.3. Fixes [#4088](http://wiki.linuxfromscratch.org/lfs/ticket/4088).

-
2017-05-25

-
[renodr] - Fix the build for i686 systems. Includes a modification to Chapter 06 Glibc.

-
2017-05-19

-
[bdubbs] - Move library versions for presevation of debugging symbols to packages.ent. Fixes [#4085](http://wiki.linuxfromscratch.org/lfs/ticket/4085).

-
[bdubbs] - Update to linux-4.11.1. Fixes [#4086](http://wiki.linuxfromscratch.org/lfs/ticket/4086).

-
[bdubbs] - Update to shadow-4.5. Fixes [#4087](http://wiki.linuxfromscratch.org/lfs/ticket/4087).

-
2017-05-13

-
[dj] - Introduce -isystem to use the final system location of gcc's internal headers in the Glibc build.

-
[ken] - Update some library versions in Stripping Again, partly fixes [#4085](http://wiki.linuxfromscratch.org/lfs/ticket/4085).

-
[dj] - Add additional symlinks to avoid "/tools" references in final system.

-
2017-05-07

-
[bdubbs] - Update to flex-2.6.4. Fixes [#4084](http://wiki.linuxfromscratch.org/lfs/ticket/4084).

-
2017-05-05

-
[bdubbs] - Update to man-pages-4.11. Fixes [#4083](http://wiki.linuxfromscratch.org/lfs/ticket/4083).

-
2017-05-03

-
[bdubbs] - Update to gcc-7.1.0. Fixes [#4082](http://wiki.linuxfromscratch.org/lfs/ticket/4082).

-
[bdubbs] - Update to iproute2-4.11.0. Fixes [#4081](http://wiki.linuxfromscratch.org/lfs/ticket/4081).

-
[bdubbs] - Fix a problem with glibc tests and add some explanations to the configure options.

-
[bdubbs] - Add a command to touch /root/.vimrc so that the default vim options don't override those in /etc/vimrc.

-
2017-05-01

-
[bdubbs] - Update to linux-4.11. Fixes [#4080](http://wiki.linuxfromscratch.org/lfs/ticket/4080).

-
[bdubbs] - Update flex patch.

-
2017-04-26

-
[bdubbs] - Add a patch to fix a flex regresion in version 2.6.3.

-
[bdubbs] - Update to linux-4.10.13. Fixes [#4079](http://wiki.linuxfromscratch.org/lfs/ticket/4079).

-
2017-04-26

-
[bdubbs] - Update to grub-2.02. Fixes [#4042](http://wiki.linuxfromscratch.org/lfs/ticket/4042).

-
2017-04-25

-
[bdubbs] - Update to vim-8.0.586. Fixes [#4078](http://wiki.linuxfromscratch.org/lfs/ticket/4078).

-
[bdubbs] - Update to eudev-3.2.2. Fixes [#4077](http://wiki.linuxfromscratch.org/lfs/ticket/4077).

-
[bdubbs] - Update to linux-4.10.12. Fixes [#4075](http://wiki.linuxfromscratch.org/lfs/ticket/4075).

-
[bdubbs] - Update to gperf-3.1. Fixes [#4053](http://wiki.linuxfromscratch.org/lfs/ticket/4053).

-
[bdubbs] - Improve instructions to save debugging information for selected libraries when stripping at the end of Chapter 6. Fixes [#4076](http://wiki.linuxfromscratch.org/lfs/ticket/4076) (again).

-
2017-04-22

-
[bdubbs] - Add instructions to save debugging information for selected libraries when stripping at the end of Chapter 6. Fixes [#4076](http://wiki.linuxfromscratch.org/lfs/ticket/4076).

-
2017-04-11

-
[bdubbs] - Remove unneeded --disable-compile-warnings from pkg-config instructions. Thanks to Jeffery Smith for pointing this out.

-
2017-04-10

-
[bdubbs] - Update to linux-4.10.9. Fixes [#4073](http://wiki.linuxfromscratch.org/lfs/ticket/4073).

-
[bdubbs] - Update to bc-1.07.1. Fixes [#4074](http://wiki.linuxfromscratch.org/lfs/ticket/4074).

-
2017-04-07

-
[bdubbs] - Fix an error in bc-1.07.

-
2017-04-03

-
[bdubbs] - Update to bc-1.07. Fixes [#4071](http://wiki.linuxfromscratch.org/lfs/ticket/4071).

-
2017-03-31

-
[bdubbs] - Update to linux-4.10.8. Fixes [#4070](http://wiki.linuxfromscratch.org/lfs/ticket/4070).

-
[bdubbs] - Update to less-487. Fixes [#4069](http://wiki.linuxfromscratch.org/lfs/ticket/4069).

-
2017-03-28

-
[bdubbs] - Move readline to be before bc in Chapter 6. Fixes [#4068](http://wiki.linuxfromscratch.org/lfs/ticket/4068).

-
[bdubbs] - Update to linux-4.10.6. Fixes [#4065](http://wiki.linuxfromscratch.org/lfs/ticket/4065).

-
[bdubbs] - Update to pkg-config-0.29.2. Fixes [#4066](http://wiki.linuxfromscratch.org/lfs/ticket/4066).

-
[bdubbs] - Update to tzdata-2017b. Fixes [#4067](http://wiki.linuxfromscratch.org/lfs/ticket/4067).

-
[bdubbs] - Add option -Dusethreads to perl in Chapter 6.

-
2017-03-25

-
[dj] - Update to lfs-bootscripts-20170825. Fix a scope issue in the rc script. Thanks to "quesker" in #lfs-support for the report and subsequent testing.

-
2017-03-18

-
[bdubbs] - Update formats of error messages in checkfs boot script. Fixes [#4064](http://wiki.linuxfromscratch.org/lfs/ticket/4064).

-
[bdubbs] - Update to man-pages-4.10. Fixes [#4063](http://wiki.linuxfromscratch.org/lfs/ticket/4063).

-
[bdubbs] - Update to linux-4.10.3. Fixes [#4062](http://wiki.linuxfromscratch.org/lfs/ticket/4062).

-
[bdubbs] - Update to gdbm-1.13. Fixes [#4061](http://wiki.linuxfromscratch.org/lfs/ticket/4061).

-
[bdubbs] - Update to coreutils-8.27. Fixes [#4060](http://wiki.linuxfromscratch.org/lfs/ticket/4060).

-
2017-03-11

-
[dj] - Fix description of modifications to `gcc/config/{linux,i386/linux{,64}}.h` in gcc pass 1.

-
2017-03-08

-
[dj] - Update /etc/hosts in network configuration.

-
2017-03-03

-
[bdubbs] - Update to binutils-2.28. Moved m4 and bc to before binutils to accommodate the gold linker regression tests. Fixes [#4059](http://wiki.linuxfromscratch.org/lfs/ticket/4059).

-
[renodr] - Update to linux-4.10.1. Fixes [#4056](http://wiki.linuxfromscratch.org/lfs/ticket/4056).

-
[renodr] - Update to tzdata2017a. Fixes [#4057](http://wiki.linuxfromscratch.org/lfs/ticket/4057).

-
2017-02-28

-
[bdubbs] - Update to kmod-24. Fixes [#4054](http://wiki.linuxfromscratch.org/lfs/ticket/4054).

-
[bdubbs] - Update to util-linux 2.29.2. Fixes [#4052](http://wiki.linuxfromscratch.org/lfs/ticket/4052).

-
[bdubbs] - Update to iproute2-4.10.0. Fixes [#4051](http://wiki.linuxfromscratch.org/lfs/ticket/4051).

-
[bdubbs] - Update to linux-4.10. Fixes [#4049](http://wiki.linuxfromscratch.org/lfs/ticket/4049).

-
2017-02-25

-
[bdubbs] - LFS-8.0 released.

## 1.4. Resources

### 1.4.1. FAQ

If during the building of the LFS system you encounter any errors, have any questions, or think there is a typo in the book, please start by consulting the Frequently Asked Questions (FAQ) that is located at [http://www.linuxfromscratch.org/faq/](http://www.linuxfromscratch.org/faq/).

### 1.4.2. Mailing Lists

The `linuxfromscratch.org` server hosts a number of mailing lists used for the development of the LFS project. These lists include the main development and support lists, among others. If the FAQ does not solve the problem you are having, the next step would be to search the mailing lists at [http://www.linuxfromscratch.org/search.html](http://www.linuxfromscratch.org/search.html).

For information on the different lists, how to subscribe, archive locations, and additional information, visit[http://www.linuxfromscratch.org/mail.html](http://www.linuxfromscratch.org/mail.html).

### 1.4.3. IRC

Several members of the LFS community offer assistance on Internet Relay Chat (IRC). Before using this support, please make sure that your question is not already answered in the LFS FAQ or the mailing list archives. You can find the IRC network at `irc.freenode.net`. The support channel is named #LFS-support.

### 1.4.4. Mirror Sites

The LFS project has a number of world-wide mirrors to make accessing the website and downloading the required packages more convenient. Please visit the LFS website at [http://www.linuxfromscratch.org/mirrors.html](http://www.linuxfromscratch.org/mirrors.html) for a list of current mirrors.

### 1.4.5. Contact Information

Please direct all your questions and comments to one of the LFS mailing lists (see above).

## 1.5. Help

If an issue or a question is encountered while working through this book, please check the FAQ page at [http://www.linuxfromscratch.org/faq/#generalfaq](http://www.linuxfromscratch.org/faq/#generalfaq). Questions are often already answered there. If your question is not answered on this page, try to find the source of the problem. The following hint will give you some guidance for troubleshooting: [http://www.linuxfromscratch.org/hints/downloads/files/errors.txt](http://www.linuxfromscratch.org/hints/downloads/files/errors.txt).

If you cannot find your problem listed in the FAQ, search the mailing lists at [http://www.linuxfromscratch.org/search.html](http://www.linuxfromscratch.org/search.html).

We also have a wonderful LFS community that is willing to offer assistance through the mailing lists and IRC (see the [Section 1.4, “Resources”](Linux%20From%20Scratch.html#ch-intro-resources) section of this book). However, we get several support questions every day and many of them can be easily answered by going to the FAQ and by searching the mailing lists first. So, for us to offer the best assistance possible, you need to do some research on your own first. That allows us to focus on the more unusual support needs. If your searches do not produce a solution, please include all relevant information (mentioned below) in your request for help.

### 1.5.1. Things to Mention

Apart from a brief explanation of the problem being experienced, the essential things to include in any request for help are:

-
The version of the book being used (in this case 8.1 )

-
The host distribution and version being used to create LFS

-
The output from the [Host System Requirements](Linux%20From%20Scratch.html#version-check) script

-
The package or section the problem was encountered in

-
The exact error message or symptom being received

-
Note whether you have deviated from the book at all

### Note

Deviating from this book does *not* mean that we will not help you. After all, LFS is about personal preference. Being upfront about any changes to the established procedure helps us evaluate and determine possible causes of your problem.

### 1.5.2. Configure Script Problems

If something goes wrong while running the **configure** script, review the `config.log` file. This file may contain errors encountered during **configure**which were not printed to the screen. Include the *relevant* lines if you need to ask for help.

### 1.5.3. Compilation Problems

Both the screen output and the contents of various files are useful in determining the cause of compilation problems. The screen output from the **configure** script and the **make** run can be helpful. It is not necessary to include the entire output, but do include enough of the relevant information. Below is an example of the type of information to include from the screen output from **make**:

    gcc -DALIASPATH=\"/mnt/lfs/usr/share/locale:.\"
    -DLOCALEDIR=\"/mnt/lfs/usr/share/locale\"
    -DLIBDIR=\"/mnt/lfs/usr/lib\"
    -DINCLUDEDIR=\"/mnt/lfs/usr/include\" -DHAVE_CONFIG_H -I. -I.
    -g -O2 -c getopt1.c
    gcc -g -O2 -static -o make ar.o arscan.o commands.o dir.o
    expand.o file.o function.o getopt.o implicit.o job.o main.o
    misc.o read.o remake.o rule.o signame.o variable.o vpath.o
    default.o remote-stub.o version.o opt1.o
    -lutil job.o: In function `load_too_high':
    /lfs/tmp/make-3.79.1/job.c:1565: undefined reference
    to `getloadavg'
    collect2: ld returned 1 exit status
    make[2]: *** [make] Error 1
    make[2]: Leaving directory `/lfs/tmp/make-3.79.1'
    make[1]: *** [all-recursive] Error 1
    make[1]: Leaving directory `/lfs/tmp/make-3.79.1'
    make: *** [all-recursive-am] Error 2

In this case, many people would just include the bottom section:

    make [2]: *** [make] Error 1

This is not enough information to properly diagnose the problem because it only notes that something went wrong, not *what* went wrong. The entire section, as in the example above, is what should be saved because it includes the command that was executed and the associated error message(s).

An excellent article about asking for help on the Internet is available online at [http://catb.org/~esr/faqs/smart-questions.html](http://catb.org/~esr/faqs/smart-questions.html). Read and follow the hints in this document to increase the likelihood of getting the help you need.

