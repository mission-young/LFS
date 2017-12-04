# Part III. Building the LFS System

## Chapter 6. Installing Basic System Software

## 6.1. Introduction

In this chapter, we enter the building site and start constructing the LFS system in earnest. That is, we chroot into the temporary mini Linux system, make a few final preparations, and then begin installing the packages.

The installation of this software is straightforward. Although in many cases the installation instructions could be made shorter and more generic, we have opted to provide the full instructions for every package to minimize the possibilities for mistakes. The key to learning what makes a Linux system work is to know what each package is used for and why you (or the system) may need it.

We do not recommend using optimizations. They can make a program run slightly faster, but they may also cause compilation difficulties and problems when running the program. If a package refuses to compile when using optimization, try to compile it without optimization and see if that fixes the problem. Even if the package does compile when using optimization, there is the risk it may have been compiled incorrectly because of the complex interactions between the code and build tools. Also note that the `-march` and `-mtune` options using values not specified in the book have not been tested. This may cause problems with the toolchain packages (Binutils, GCC and Glibc). The small potential gains achieved in using compiler optimizations are often outweighed by the risks. First-time builders of LFS are encouraged to build without custom optimizations. The subsequent system will still run very fast and be stable at the same time.

The order that packages are installed in this chapter needs to be strictly followed to ensure that no program accidentally acquires a path referring to `/tools` hard-wired into it. For the same reason, do not compile separate packages in parallel. Compiling in parallel may save time (especially on dual-CPU machines), but it could result in a program containing a hard-wired path to `/tools`, which will cause the program to stop working when that directory is removed.

Before the installation instructions, each installation page provides information about the package, including a concise description of what it contains, approximately how long it will take to build, and how much disk space is required during this building process. Following the installation instructions, there is a list of programs and libraries (along with brief descriptions of these) that the package installs.

### Note

The SBU values and required disk space includes test suite data for all applicable packages in Chapter 6.

### 6.1.1. About libraries

In general, the LFS editors discourage building and installing static libraries. The original purpose for most static libraries has been made obsolete in a modern Linux system. In addition linking a static library into a program can be detrimental. If an update to the library is needed to remove a security problem, all programs that use the static library will need to be relinked to the new library. Since the use of static libraries is not always obvious, the relevant programs (and the procedures needed to do the linking) may not even be known.

In the procedures in Chapter 6, we remove or disable installation of most static libraries. In a few cases, especially glibc and gcc, the use of static libraries remains essential to the general package building process. Usually this is done by passing a `--disable-static` option to **configure**. In other cases, alternate means are needed.

For a more complete discussion of libraries, see the discussion [Libraries: Static or shared?](http://www.linuxfromscratch.org/blfs//view/8.1/introduction/libraries.html) in the BLFS book.

## 6.2. Preparing Virtual Kernel File Systems

Various file systems exported by the kernel are used to communicate to and from the kernel itself. These file systems are virtual in that no disk space is used for them. The content of the file systems resides in memory.

Begin by creating directories onto which the file systems will be mounted:

    mkdir -pv $LFS/{dev,proc,sys,run}

### 6.2.1. Creating Initial Device Nodes

When the kernel boots the system, it requires the presence of a few device nodes, in particular the `console` and `null` devices. The device nodes must be created on the hard disk so that they are available before **udevd** has been started, and additionally when Linux is started with*`init=/bin/bash`*. Create the devices by running the following commands:

    mknod -m 600 $LFS/dev/console c 5 1
    mknod -m 666 $LFS/dev/null c 1 3

### 6.2.2. Mounting and Populating /dev

The recommended method of populating the `/dev` directory with devices is to mount a virtual filesystem (such as `tmpfs`) on the `/dev` directory, and allow the devices to be created dynamically on that virtual filesystem as they are detected or accessed. Device creation is generally done during the boot process by Udev. Since this new system does not yet have Udev and has not yet been booted, it is necessary to mount and populate `/dev` manually. This is accomplished by bind mounting the host system's `/dev` directory. A bind mount is a special type of mount that allows you to create a mirror of a directory or mount point to some other location. Use the following command to achieve this:

    mount -v --bind /dev $LFS/dev

### 6.2.3. Mounting Virtual Kernel File Systems

Now mount the remaining virtual kernel filesystems:

    mount -vt devpts devpts $LFS/dev/pts -o gid=5,mode=620
    mount -vt proc proc $LFS/proc
    mount -vt sysfs sysfs $LFS/sys
    mount -vt tmpfs tmpfs $LFS/run

**The meaning of the mount options for devpts:**
*`gid=5`*
This ensures that all devpts-created device nodes are owned by group ID 5. This is the ID we will use later on for the `tty` group. We use the group ID instead of a name, since the host system might use a different ID for its `tty` group.
*`mode=0620`*
This ensures that all devpts-created device nodes have mode 0620 (user readable and writable, group writable). Together with the option above, this ensures that devpts will create device nodes that meet the requirements of grantpt(), meaning the Glibc **pt_chown**helper binary (which is not installed by default) is not necessary.

In some host systems, `/dev/shm` is a symbolic link to `/run/shm`. The /run tmpfs was mounted above so in this case only a directory needs to be created.

    if [ -h $LFS/dev/shm ]; then
      mkdir -pv $LFS/$(readlink $LFS/dev/shm)
    fi

## 6.3. Package Management

Package Management is an often requested addition to the LFS Book. A Package Manager allows tracking the installation of files making it easy to remove and upgrade packages. As well as the binary and library files, a package manager will handle the installation of configuration files. Before you begin to wonder, NO—this section will not talk about nor recommend any particular package manager. What it provides is a roundup of the more popular techniques and how they work. The perfect package manager for you may be among these techniques or may be a combination of two or more of these techniques. This section briefly mentions issues that may arise when upgrading packages.

Some reasons why no package manager is mentioned in LFS or BLFS include:

-
Dealing with package management takes the focus away from the goals of these books—teaching how a Linux system is built.

-
There are multiple solutions for package management, each having its strengths and drawbacks. Including one that satisfies all audiences is difficult.

There are some hints written on the topic of package management. Visit the [Hints Project](http://www.linuxfromscratch.org/hints/list.html) and see if one of them fits your need.

### 6.3.1. Upgrade Issues

A Package Manager makes it easy to upgrade to newer versions when they are released. Generally the instructions in the LFS and BLFS Book can be used to upgrade to the newer versions. Here are some points that you should be aware of when upgrading packages, especially on a running system.

-
If Glibc needs to be upgraded to a newer version, (e.g. from glibc-2.19 to glibc-2.20, it is safer to rebuild LFS. Though you *may* be able to rebuild all the packages in their dependency order, we do not recommend it.

-
If a package containing a shared library is updated, and if the name of the library changes, then all the packages dynamically linked to the library need to be recompiled to link against the newer library. (Note that there is no correlation between the package version and the name of the library.) For example, consider a package foo-1.2.3 that installs a shared library with name `libfoo.so.1`. Say you upgrade the package to a newer version foo-1.2.4 that installs a shared library with name `libfoo.so.2`. In this case, all packages that are dynamically linked to `libfoo.so.1` need to be recompiled to link against `libfoo.so.2`. Note that you should not remove the previous libraries until the dependent packages are recompiled.

### 6.3.2. Package Management Techniques

The following are some common package management techniques. Before making a decision on a package manager, do some research on the various techniques, particularly the drawbacks of the particular scheme.

#### 6.3.2.1. It is All in My Head!

Yes, this is a package management technique. Some folks do not find the need for a package manager because they know the packages intimately and know what files are installed by each package. Some users also do not need any package management because they plan on rebuilding the entire system when a package is changed.

#### 6.3.2.2. Install in Separate Directories

This is a simplistic package management that does not need any extra package to manage the installations. Each package is installed in a separate directory. For example, package foo-1.1 is installed in `/usr/pkg/foo-1.1` and a symlink is made from `/usr/pkg/foo` to `/usr/pkg/foo-1.1`. When installing a new version foo-1.2, it is installed in `/usr/pkg/foo-1.2` and the previous symlink is replaced by a symlink to the new version.

Environment variables such as `PATH`, `LD_LIBRARY_PATH`, `MANPATH`, `INFOPATH` and `CPPFLAGS` need to be expanded to include `/usr/pkg/foo`. For more than a few packages, this scheme becomes unmanageable.

#### 6.3.2.3. Symlink Style Package Management

This is a variation of the previous package management technique. Each package is installed similar to the previous scheme. But instead of making the symlink, each file is symlinked into the `/usr` hierarchy. This removes the need to expand the environment variables. Though the symlinks can be created by the user to automate the creation, many package managers have been written using this approach. A few of the popular ones include Stow, Epkg, Graft, and Depot.

The installation needs to be faked, so that the package thinks that it is installed in `/usr` though in reality it is installed in the `/usr/pkg` hierarchy. Installing in this manner is not usually a trivial task. For example, consider that you are installing a package libfoo-1.1. The following instructions may not install the package properly:

    ./configure --prefix=/usr/pkg/libfoo/1.1
    make
    make install

The installation will work, but the dependent packages may not link to libfoo as you would expect. If you compile a package that links against libfoo, you may notice that it is linked to `/usr/pkg/libfoo/1.1/lib/libfoo.so.1` instead of `/usr/lib/libfoo.so.1` as you would expect. The correct approach is to use the `DESTDIR` strategy to fake installation of the package. This approach works as follows:

    ./configure --prefix=/usr
    make
    make DESTDIR=/usr/pkg/libfoo/1.1 install

Most packages support this approach, but there are some which do not. For the non-compliant packages, you may either need to manually install the package, or you may find that it is easier to install some problematic packages into `/opt`.

#### 6.3.2.4. Timestamp Based

In this technique, a file is timestamped before the installation of the package. After the installation, a simple use of the **find** command with the appropriate options can generate a log of all the files installed after the timestamp file was created. A package manager written with this approach is install-log.

Though this scheme has the advantage of being simple, it has two drawbacks. If, during installation, the files are installed with any timestamp other than the current time, those files will not be tracked by the package manager. Also, this scheme can only be used when one package is installed at a time. The logs are not reliable if two packages are being installed on two different consoles.

#### 6.3.2.5. Tracing Installation Scripts

In this approach, the commands that the installation scripts perform are recorded. There are two techniques that one can use:

The `LD_PRELOAD` environment variable can be set to point to a library to be preloaded before installation. During installation, this library tracks the packages that are being installed by attaching itself to various executables such as **cp**, **install**, **mv** and tracking the system calls that modify the filesystem. For this approach to work, all the executables need to be dynamically linked without the suid or sgid bit. Preloading the library may cause some unwanted side-effects during installation. Therefore, it is advised that one performs some tests to ensure that the package manager does not break anything and logs all the appropriate files.

The second technique is to use **strace**, which logs all system calls made during the execution of the installation scripts.

#### 6.3.2.6. Creating Package Archives

In this scheme, the package installation is faked into a separate tree as described in the Symlink style package management. After the installation, a package archive is created using the installed files. This archive is then used to install the package either on the local machine or can even be used to install the package on other machines.

This approach is used by most of the package managers found in the commercial distributions. Examples of package managers that follow this approach are RPM (which, incidentally, is required by the [Linux Standard Base Specification](http://refspecs.linuxfoundation.org/lsb.shtml)), pkg-utils, Debian's apt, and Gentoo's Portage system. A hint describing how to adopt this style of package management for LFS systems is located at [http://www.linuxfromscratch.org/hints/downloads/files/fakeroot.txt](http://www.linuxfromscratch.org/hints/downloads/files/fakeroot.txt).

Creation of package files that include dependency information is complex and is beyond the scope of LFS.

Slackware uses a **tar** based system for package archives. This system purposely does not handle package dependencies as more complex package managers do. For details of Slackware package management, see [http://www.slackbook.org/html/package-management.html](http://www.slackbook.org/html/package-management.html).

#### 6.3.2.7. User Based Management

This scheme, unique to LFS, was devised by Matthias Benkmann, and is available from the [Hints Project](http://www.linuxfromscratch.org/hints/list.html). In this scheme, each package is installed as a separate user into the standard locations. Files belonging to a package are easily identified by checking the user ID. The features and shortcomings of this approach are too complex to describe in this section. For the details please see the hint at [http://www.linuxfromscratch.org/hints/downloads/files/more_control_and_pkg_man.txt](http://www.linuxfromscratch.org/hints/downloads/files/more_control_and_pkg_man.txt).

### 6.3.3. Deploying LFS on Multiple Systems

One of the advantages of an LFS system is that there are no files that depend on the position of files on a disk system. Cloning an LFS build to another computer with the same architecture as the base system is as simple as using **tar** on the LFS partition that contains the root directory (about 250MB uncompressed for a base LFS build), copying that file via network transfer or CD-ROM to the new system and expanding it. From that point, a few configuration files will have to be changed. Configuration files that may need to be updated include: `/etc/hosts`, `/etc/fstab`, `/etc/passwd`, `/etc/group`, `/etc/shadow`, `/etc/ld.so.conf`, `/etc/sysconfig/rc.site`, `/etc/sysconfig/network`, and `/etc/sysconfig/ifconfig.eth0`.

A custom kernel may need to be built for the new system depending on differences in system hardware and the original kernel configuration.

### Note

There have been some reports of issues when copying between similar but not identical architectures. For instance, the instruction set for an Intel system is not identical with an AMD processor and later versions of some processors may have instructions that are unavailable in earlier versions.

Finally the new system has to be made bootable via [Section 8.4, “Using GRUB to Set Up the Boot Process”](Linux%20From%20Scratch.html#ch-bootable-grub).

## 6.4. Entering the Chroot Environment

It is time to enter the chroot environment to begin building and installing the final LFS system. As user `root`, run the following command to enter the realm that is, at the moment, populated with only the temporary tools:

    chroot "$LFS" /tools/bin/env -i \
        HOME=/root                  \
        TERM="$TERM"                \
        PS1='\u:\w\$ '              \
        PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
        /tools/bin/bash --login +h

The *`-i`* option given to the **env** command will clear all variables of the chroot environment. After that, only the `HOME`, `TERM`, `PS1`, and `PATH` variables are set again. The *`TERM=$TERM`* construct will set the `TERM` variable inside chroot to the same value as outside chroot. This variable is needed for programs like **vim** and **less** to operate properly. If other variables are needed, such as `CFLAGS` or `CXXFLAGS`, this is a good place to set them again.

From this point on, there is no need to use the `LFS` variable anymore, because all work will be restricted to the LFS file system. This is because the Bash shell is told that `$LFS` is now the root (`/`) directory.

Notice that `/tools/bin` comes last in the `PATH`. This means that a temporary tool will no longer be used once its final version is installed. This occurs when the shell does not “remember” the locations of executed binaries—for this reason, hashing is switched off by passing the *`+h`*option to **bash**.

Note that the **bash** prompt will say `I have no name!` This is normal because the `/etc/passwd` file has not been created yet.

### Note

It is important that all the commands throughout the remainder of this chapter and the following chapters are run from within the chroot environment. If you leave this environment for any reason (rebooting for example), ensure that the virtual kernel filesystems are mounted as explained in [Section 6.2.2, “Mounting and Populating /dev”](Linux%20From%20Scratch.html#ch-system-bindmount) and [Section 6.2.3, “Mounting Virtual Kernel File Systems”](Linux%20From%20Scratch.html#ch-system-kernfsmount) and enter chroot again before continuing with the installation.

## 6.5. Creating Directories

It is time to create some structure in the LFS file system. Create a standard directory tree by issuing the following commands:

    mkdir -pv /{bin,boot,etc/{opt,sysconfig},home,lib/firmware,mnt,opt}
    mkdir -pv /{media/{floppy,cdrom},sbin,srv,var}
    install -dv -m 0750 /root
    install -dv -m 1777 /tmp /var/tmp
    mkdir -pv /usr/{,local/}{bin,include,lib,sbin,src}
    mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
    mkdir -v  /usr/{,local/}share/{misc,terminfo,zoneinfo}
    mkdir -v  /usr/libexec
    mkdir -pv /usr/{,local/}share/man/man{1..8}

    case $(uname -m) in
     x86_64) mkdir -v /lib64 ;;
    esac

    mkdir -v /var/{log,mail,spool}
    ln -sv /run /var/run
    ln -sv /run/lock /var/lock
    mkdir -pv /var/{opt,cache,lib/{color,misc,locate},local}

Directories are, by default, created with permission mode 755, but this is not desirable for all directories. In the commands above, two changes are made—one to the home directory of user `root`, and another to the directories for temporary files.

The first mode change ensures that not just anybody can enter the `/root` directory—the same as a normal user would do with his or her home directory. The second mode change makes sure that any user can write to the `/tmp` and `/var/tmp` directories, but cannot remove another user's files from them. The latter is prohibited by the so-called “sticky bit,” the highest bit (1) in the 1777 bit mask.

### 6.5.1. FHS Compliance Note

The directory tree is based on the Filesystem Hierarchy Standard (FHS) (available at [https://wiki.linuxfoundation.org/en/FHS](https://wiki.linuxfoundation.org/en/FHS)). The FHS also specifies the optional existence of some directories such as `/usr/local/games` and `/usr/share/games`. We create only the directories that are needed. However, feel free to create these directories.

## 6.6. Creating Essential Files and Symlinks

Some programs use hard-wired paths to programs which do not exist yet. In order to satisfy these programs, create a number of symbolic links which will be replaced by real files throughout the course of this chapter after the software has been installed:

    ln -sv /tools/bin/{bash,cat,dd,echo,ln,pwd,rm,stty} /bin
    ln -sv /tools/bin/{install,perl} /usr/bin
    ln -sv /tools/lib/libgcc_s.so{,.1} /usr/lib
    ln -sv /tools/lib/libstdc++.{a,so{,.6}} /usr/lib
    sed 's/tools/usr/' /tools/lib/libstdc++.la > /usr/lib/libstdc++.la
    ln -sv bash /bin/sh

**The purpose of each link:**
*``/bin/bash``*
Many **bash** scripts specify `/bin/bash`.
*``/bin/cat``*
This pathname is hard-coded into Glibc's configure script.
*``/bin/dd``*
The path to `dd` will be hard-coded into the `/usr/bin/libtool` utility.
*``/bin/echo``*
This is to satisfy one of the tests in Glibc's test suite, which expects `/bin/echo`.
*``/usr/bin/install``*
The path to `install` will be hard-coded into the `/usr/lib/bash/Makefile.inc` file.
*``/bin/ln``*
The path to `ln` will be hard-coded into the `/usr/lib/perl5/5.26.0/<target-triplet>/Config_heavy.pl` file.
*``/bin/pwd``*
Some **configure** scripts, particularly Glibc's, have this pathname hard-coded.
*``/bin/rm``*
The path to `rm` will be hard-coded into the `/usr/lib/perl5/5.26.0/<target-triplet>/Config_heavy.pl` file.
*``/bin/stty``*
This pathname is hard-coded into Expect, therefore it is needed for Binutils and GCC test suites to pass.
*``/usr/bin/perl``*
Many Perl scripts hard-code this path to the **perl** program.
*``/usr/lib/libgcc_s.so{,.1}``*
Glibc needs this for the pthreads library to work.
*``/usr/lib/libstdc++{,.6}``*
This is needed by several tests in Glibc's test suite, as well as for C++ support in GMP.
*``/usr/lib/libstdc++.la``*
This prevents a `/tools` reference that would otherwise be in `/usr/lib/libstdc++.la` after GCC is installed.
*``/bin/sh``*
Many shell scripts hard-code `/bin/sh`.

Historically, Linux maintains a list of the mounted file systems in the file `/etc/mtab`. Modern kernels maintain this list internally and exposes it to the user via the `/proc` filesystem. To satisfy utilities that expect the presence of `/etc/mtab`, create the following symbolic link:

    ln -sv /proc/self/mounts /etc/mtab

In order for user `root` to be able to login and for the name “root” to be recognized, there must be relevant entries in the `/etc/passwd` and `/etc/group` files.

Create the `/etc/passwd` file by running the following command:

    cat > /etc/passwd << "EOF"
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/dev/null:/bin/false
    daemon:x:6:6:Daemon User:/dev/null:/bin/false
    messagebus:x:18:18:D-Bus Message Daemon User:/var/run/dbus:/bin/false
    nobody:x:99:99:Unprivileged User:/dev/null:/bin/false
    EOF

The actual password for `root` (the “x” used here is just a placeholder) will be set later.

Create the `/etc/group` file by running the following command:

    cat > /etc/group << "EOF"
    root:x:0:
    bin:x:1:daemon
    sys:x:2:
    kmem:x:3:
    tape:x:4:
    tty:x:5:
    daemon:x:6:
    floppy:x:7:
    disk:x:8:
    lp:x:9:
    dialout:x:10:
    audio:x:11:
    video:x:12:
    utmp:x:13:
    usb:x:14:
    cdrom:x:15:
    adm:x:16:
    messagebus:x:18:
    systemd-journal:x:23:
    input:x:24:
    mail:x:34:
    nogroup:x:99:
    users:x:999:
    EOF

The created groups are not part of any standard—they are groups decided on in part by the requirements of the Udev configuration in this chapter, and in part by common convention employed by a number of existing Linux distributions. In addition, some test suites rely on specific users or groups. The Linux Standard Base (LSB, available at [http://www.linuxbase.org](http://www.linuxbase.org/)) recommends only that, besides the group `root` with a Group ID (GID) of 0, a group `bin` with a GID of 1 be present. All other group names and GIDs can be chosen freely by the system administrator since well-written programs do not depend on GID numbers, but rather use the group's name.

To remove the “I have no name!” prompt, start a new shell. Since a full Glibc was installed in [Chapter 5](Linux%20From%20Scratch.html#chapter-temporary-tools) and the `/etc/passwd` and `/etc/group` files have been created, user name and group name resolution will now work:

    exec /tools/bin/bash --login +h

Note the use of the *`+h`* directive. This tells **bash** not to use its internal path hashing. Without this directive, **bash** would remember the paths to binaries it has executed. To ensure the use of the newly compiled binaries as soon as they are installed, the *`+h`* directive will be used for the duration of this chapter.

The **login**, **agetty**, and **init** programs (and others) use a number of log files to record information such as who was logged into the system and when. However, these programs will not write to the log files if they do not already exist. Initialize the log files and give them proper permissions:

    touch /var/log/{btmp,lastlog,faillog,wtmp}
    chgrp -v utmp /var/log/lastlog
    chmod -v 664  /var/log/lastlog
    chmod -v 600  /var/log/btmp

The `/var/log/wtmp` file records all logins and logouts. The `/var/log/lastlog` file records when each user last logged in. The `/var/log/faillog` file records failed login attempts. The `/var/log/btmp` file records the bad login attempts.

### Note

The `/run/utmp` file records the users that are currently logged in. This file is created dynamically in the boot scripts.

## 6.7. Linux-4.12.7 API Headers

The Linux API Headers (in linux-4.12.7.tar.xz) expose the kernel's API for use by Glibc.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**865 MB

### 6.7.1. Installation of Linux API Headers

The Linux kernel needs to expose an Application Programming Interface (API) for the system's C library (Glibc in LFS) to use. This is done by way of sanitizing various C header files that are shipped in the Linux kernel source tarball.

Make sure there are no stale files and dependencies lying around from previous activity:

    make mrproper

Now extract the user-visible kernel headers from the source. They are placed in an intermediate local directory and copied to the needed location because the extraction process removes any existing files in the target directory. There are also some hidden files used by the kernel developers and not needed by LFS that are removed from the intermediate directory.

    make INSTALL_HDR_PATH=dest headers_install
    find dest/include \( -name .install -o -name ..install.cmd \) -delete
    cp -rv dest/include/* /usr/include

### 6.7.2. Contents of Linux API Headers

**Installed headers:**/usr/include/asm/*.h, /usr/include/asm-generic/*.h, /usr/include/drm/*.h, /usr/include/linux/*.h, /usr/include/misc/*.h, /usr/include/mtd/*.h, /usr/include/rdma/*.h, /usr/include/scsi/*.h, /usr/include/sound/*.h, /usr/include/video/*.h, and /usr/include/xen/*.h

**Installed directories:**/usr/include/asm, /usr/include/asm-generic, /usr/include/drm, /usr/include/linux, /usr/include/misc, /usr/include/mtd, /usr/include/rdma, /usr/include/scsi, /usr/include/sound, /usr/include/video, and /usr/include/xen

#### Short Descriptions

`/usr/include/asm/*.h`

The Linux API ASM Headers

`/usr/include/asm-generic/*.h`

The Linux API ASM Generic Headers

`/usr/include/drm/*.h`

The Linux API DRM Headers

`/usr/include/linux/*.h`

The Linux API Linux Headers

`/usr/include/mtd/*.h`

The Linux API MTD Headers

`/usr/include/rdma/*.h`

The Linux API RDMA Headers

`/usr/include/scsi/*.h`

The Linux API SCSI Headers

`/usr/include/sound/*.h`

The Linux API Sound Headers

`/usr/include/video/*.h`

The Linux API Video Headers

`/usr/include/xen/*.h`

The Linux API Xen Headers

Last updated on

## 6.8. Man-pages-4.12

The Man-pages package contains over 2,200 man pages.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**27 MB

### 6.8.1. Installation of Man-pages

Install Man-pages by running:

    make install

### 6.8.2. Contents of Man-pages

**Installed files:**various man pages

#### Short Descriptions

`man pages`

Describe C programming language functions, important device files, and significant configuration files

Last updated on

## 6.9. Glibc-2.26

The Glibc package contains the main C library. This library provides the basic routines for allocating memory, searching directories, opening and closing files, reading and writing files, string handling, pattern matching, arithmetic, and so on.

**Approximate build time:**20 SBU

**Required disk space:**2.0 GB

### 6.9.1. Installation of Glibc

### Note

The Glibc build system is self-contained and will install perfectly, even though the compiler specs file and linker are still pointing to `/tools`. The specs and linker cannot be adjusted before the Glibc install because the Glibc autoconf tests would give false results and defeat the goal of achieving a clean build.

Some of the Glibc programs use non-FHS compilant `/var/db` directory to store their runtime data. Apply the following patch to make such programs store their runtime data in the FHS-compliant locations:

    patch -Np1 -i ../glibc-2.26-fhs-1.patch

First create a compatibility symlink to avoid references to /tools in our final glibc:

    ln -sfv /tools/lib/gcc /usr/lib

Determine the GCC include directory and create a symlink for LSB compliance. Additionally, for x86_64, create a compatibility symlink required for the dynamic loader to function correctly:

    case $(uname -m) in
        i?86)    GCC_INCDIR=/usr/lib/gcc/$(uname -m)-pc-linux-gnu/7.2.0/include
                ln -sfv ld-linux.so.2 /lib/ld-lsb.so.3
        ;;
        x86_64) GCC_INCDIR=/usr/lib/gcc/x86_64-pc-linux-gnu/7.2.0/include
                ln -sfv ../lib/ld-linux-x86-64.so.2 /lib64
                ln -sfv ../lib/ld-linux-x86-64.so.2 /lib64/ld-lsb-x86-64.so.3
        ;;
    esac

Remove a file that may be left over from a previous build attempt:

    rm -f /usr/include/limits.h

The Glibc documentation recommends building Glibc in a dedicated build directory:

    mkdir -v build
    cd       build

Prepare Glibc for compilation:

    CC="gcc -isystem $GCC_INCDIR -isystem /usr/include" \
    ../configure --prefix=/usr                          \
                 --disable-werror                       \
                 --enable-kernel=3.2                    \
                 --enable-stack-protector=strong        \
                 libc_cv_slibdir=/lib
    unset GCC_INCDIR

**The meaning of the options and new configure parameters:**
*`CC="gcc -isystem $GCC_INCDIR -isystem /usr/include"`*
Setting the location of both gcc and system include directories avoids introduction of invalid paths in debugging symbols.
*`--disable-werror`*
This option disables the -Werror option passed to GCC. This is necessary for running the test suite.
*`--enable-stack-protector=strong`*
This option increases system security by adding a known canary (a random integer) to the stack during a function preamble, and checks it when the function returns. If it changed, there was a stack overflow, and the program aborts.
*`libc_cv_slibdir=/lib`*
This variable sets the correct library for all systems. We do not want lib64 to be used.

Compile the package:

    make

### Important

In this section, the test suite for Glibc is considered critical. Do not skip it under any circumstance.

Generally a few tests do not pass, but you can generally ignore any of the test failures listed below. Now test the build results:

    make check

You may see some test failures. The Glibc test suite is somewhat dependent on the host system. This is a list of the most common issues seen for some versions of LFS:

-
*posix/tst-getaddrinfo4* and *posix/tst-getaddrinfo5* may fail on some architectures.

-
The *rt/tst-cputimer1* and *rt/tst-cpuclock2* tests have been known to fail. The reason is not completely understood, but indications are that minor timing issues can trigger these failures.

-
The math tests sometimes fail when running on systems where the CPU is not a relatively new Intel or AMD processor.

-
The *nptl/tst-thread-affinity-{pthread,pthread2,sched}* tests may fail for reasons that have not been determined.

-
Other tests known to fail on some architectures are malloc/tst-malloc-usable and nptl/tst-cleanupx4.

Though it is a harmless message, the install stage of Glibc will complain about the absence of `/etc/ld.so.conf`. Prevent this warning with:

    touch /etc/ld.so.conf

Fix the generated Makefile to skip an uneeded sanity check that fails in the LFS partial environment:

    sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile

Install the package:

    make install

Install the configuration file and runtime directory for **nscd**:

    cp -v ../nscd/nscd.conf /etc/nscd.conf
    mkdir -pv /var/cache/nscd

Next, install the locales that can make the system respond in a different language. None of the locales are required, but if some of them are missing, the test suites of future packages would skip important testcases.

Individual locales can be installed using the **localedef** program. E.g., the first **localedef** command below combines the `/usr/share/i18n/locales/cs_CZ`charset-independent locale definition with the `/usr/share/i18n/charmaps/UTF-8.gz` charmap definition and appends the result to the `/usr/lib/locale/locale-archive` file. The following instructions will install the minimum set of locales necessary for the optimal coverage of tests:

    mkdir -pv /usr/lib/locale
    localedef -i cs_CZ -f UTF-8 cs_CZ.UTF-8
    localedef -i de_DE -f ISO-8859-1 de_DE
    localedef -i de_DE@euro -f ISO-8859-15 de_DE@euro
    localedef -i de_DE -f UTF-8 de_DE.UTF-8
    localedef -i en_GB -f UTF-8 en_GB.UTF-8
    localedef -i en_HK -f ISO-8859-1 en_HK
    localedef -i en_PH -f ISO-8859-1 en_PH
    localedef -i en_US -f ISO-8859-1 en_US
    localedef -i en_US -f UTF-8 en_US.UTF-8
    localedef -i es_MX -f ISO-8859-1 es_MX
    localedef -i fa_IR -f UTF-8 fa_IR
    localedef -i fr_FR -f ISO-8859-1 fr_FR
    localedef -i fr_FR@euro -f ISO-8859-15 fr_FR@euro
    localedef -i fr_FR -f UTF-8 fr_FR.UTF-8
    localedef -i it_IT -f ISO-8859-1 it_IT
    localedef -i it_IT -f UTF-8 it_IT.UTF-8
    localedef -i ja_JP -f EUC-JP ja_JP
    localedef -i ru_RU -f KOI8-R ru_RU.KOI8-R
    localedef -i ru_RU -f UTF-8 ru_RU.UTF-8
    localedef -i tr_TR -f UTF-8 tr_TR.UTF-8
    localedef -i zh_CN -f GB18030 zh_CN.GB18030

In addition, install the locale for your own country, language and character set.

Alternatively, install all locales listed in the `glibc-2.26/localedata/SUPPORTED` file (it includes every locale listed above and many more) at once with the following time-consuming command:

    make localedata/install-locales

Then use the **localedef** command to create and install locales not listed in the `glibc-2.26/localedata/SUPPORTED` file in the unlikely case you need them.

### 6.9.2. Configuring Glibc

#### 6.9.2.1. Adding nsswitch.conf

The `/etc/nsswitch.conf` file needs to be created because the Glibc defaults do not work well in a networked environment.

Create a new file `/etc/nsswitch.conf` by running the following:

    cat > /etc/nsswitch.conf << "EOF"
    # Begin /etc/nsswitch.conf

    passwd: files
    group: files
    shadow: files

    hosts: files dns
    networks: files

    protocols: files
    services: files
    ethers: files
    rpc: files

    # End /etc/nsswitch.conf
    EOF

#### 6.9.2.2. Adding time zone data

Install and set up the time zone data with the following:

    tar -xf ../../tzdata2017b.tar.gz

    ZONEINFO=/usr/share/zoneinfo
    mkdir -pv $ZONEINFO/{posix,right}

    for tz in etcetera southamerica northamerica europe africa antarctica  \
              asia australasia backward pacificnew systemv; do
        zic -L /dev/null   -d $ZONEINFO       -y "sh yearistype.sh" ${tz}
        zic -L /dev/null   -d $ZONEINFO/posix -y "sh yearistype.sh" ${tz}
        zic -L leapseconds -d $ZONEINFO/right -y "sh yearistype.sh" ${tz}
    done

    cp -v zone.tab zone1970.tab iso3166.tab $ZONEINFO
    zic -d $ZONEINFO -p America/New_York
    unset ZONEINFO

**The meaning of the zic commands:**
*`zic -L /dev/null ...`*
This creates posix time zones, without any leap seconds. It is conventional to put these in both `zoneinfo` and `zoneinfo/posix`. It is necessary to put the POSIX time zones in `zoneinfo`, otherwise various test-suites will report errors. On an embedded system, where space is tight and you do not intend to ever update the time zones, you could save 1.9MB by not using the `posix` directory, but some applications or test-suites might produce some failures.
*`zic -L leapseconds ...`*
This creates right time zones, including leap seconds. On an embedded system, where space is tight and you do not intend to ever update the time zones, or care about the correct time, you could save 1.9MB by omitting the `right` directory.
*`zic ... -p ...`*
This creates the `posixrules` file. We use New York because POSIX requires the daylight savings time rules to be in accordance with US rules.

One way to determine the local time zone is to run the following script:

    tzselect

After answering a few questions about the location, the script will output the name of the time zone (e.g., *America/Edmonton*). There are also some other possible time zones listed in `/usr/share/zoneinfo` such as *Canada/Eastern* or *EST5EDT* that are not identified by the script but can be used.

Then create the `/etc/localtime` file by running:

    cp -v /usr/share/zoneinfo/*<xxx>* /etc/localtime

Replace *`<xxx>`* with the name of the time zone selected (e.g., Canada/Eastern).

#### 6.9.2.3. Configuring the Dynamic Loader

By default, the dynamic loader (`/lib/ld-linux.so.2`) searches through `/lib` and `/usr/lib` for dynamic libraries that are needed by programs as they are run. However, if there are libraries in directories other than `/lib` and `/usr/lib`, these need to be added to the `/etc/ld.so.conf` file in order for the dynamic loader to find them. Two directories that are commonly known to contain additional libraries are `/usr/local/lib` and `/opt/lib`, so add those directories to the dynamic loader's search path.

Create a new file `/etc/ld.so.conf` by running the following:

    cat > /etc/ld.so.conf << "EOF"
    # Begin /etc/ld.so.conf
    /usr/local/lib
    /opt/lib

    EOF

If desired, the dynamic loader can also search a directory and include the contents of files found there. Generally the files in this include directory are one line specifying the desired library path. To add this capability run the following commands:

    cat >> /etc/ld.so.conf << "EOF"
    # Add an include directory
    include /etc/ld.so.conf.d/*.conf

    EOF
    mkdir -pv /etc/ld.so.conf.d

### 6.9.3. Contents of Glibc

**Installed programs:**catchsegv, gencat, getconf, getent, iconv, iconvconfig, ldconfig, ldd, lddlibc4, locale, localedef, makedb, mtrace, nscd, pldd, sln, sotruss, sprof, tzselect, xtrace, zdump, and zic

**Installed libraries:**ld-2.26.so, libBrokenLocale.{a,so}, libSegFault.so, libanl.{a,so}, libc.{a,so}, libc_nonshared.a, libcidn.so, libcrypt.{a,so}, libdl.{a,so}, libg.a, libieee.a, libm.{a,so}, libmcheck.a, libmemusage.so, libnsl.{a,so}, libnss_compat.so, libnss_dns.so, libnss_files.so, libnss_hesiod.so, libnss_nis.so, libnss_nisplus.so, libpthread.{a,so}, libpthread_nonshared.a, libresolv.{a,so}, librpcsvc.a, librt.{a,so}, libthread_db.so, and libutil.{a,so}

**Installed directories:**/usr/include/arpa, /usr/include/bits, /usr/include/gnu, /usr/include/net, /usr/include/netash, /usr/include/netatalk, /usr/include/netax25, /usr/include/neteconet, /usr/include/netinet, /usr/include/netipx, /usr/include/netiucv, /usr/include/netpacket, /usr/include/netrom, /usr/include/netrose, /usr/include/nfs, /usr/include/protocols, /usr/include/rpc, /usr/include/rpcsvc, /usr/include/sys, /usr/lib/audit, /usr/lib/gconv, /usr/lib/locale, /usr/libexec/getconf, /usr/share/i18n, /usr/share/zoneinfo, /var/cache/nscd, and /var/lib/nss_db

#### Short Descriptions

**catchsegv**

Can be used to create a stack trace when a program terminates with a segmentation fault

**gencat**

Generates message catalogues

**getconf**

Displays the system configuration values for file system specific variables

**getent**

Gets entries from an administrative database

**iconv**

Performs character set conversion

**iconvconfig**

Creates fastloading **iconv** module configuration files

**ldconfig**

Configures the dynamic linker runtime bindings

**ldd**

Reports which shared libraries are required by each given program or shared library

**lddlibc4**

Assists **ldd** with object files

**locale**

Prints various information about the current locale

**localedef**

Compiles locale specifications

**makedb**

Creates a simple database from textual input

**mtrace**

Reads and interprets a memory trace file and displays a summary in human-readable format

**nscd**

A daemon that provides a cache for the most common name service requests

**pldd**

Lists dynamic shared objects used by running processes

**sln**

A statically linked **ln** program

**sotruss**

Traces shared library procedure calls of a specified command

**sprof**

Reads and displays shared object profiling data

**tzselect**

Asks the user about the location of the system and reports the corresponding time zone description

**xtrace**

Traces the execution of a program by printing the currently executed function

**zdump**

The time zone dumper

**zic**

The time zone compiler

`ld-2.26.so`

The helper program for shared library executables

`libBrokenLocale`

Used internally by Glibc as a gross hack to get broken programs (e.g., some Motif applications) running. See comments in `glibc-2.26/locale/broken_cur_max.c` for more information

`libSegFault`

The segmentation fault signal handler, used by **catchsegv**

`libanl`

An asynchronous name lookup library

`libc`

The main C library

`libcidn`

Used internally by Glibc for handling internationalized domain names in the `getaddrinfo()` function

`libcrypt`

The cryptography library

`libdl`

The dynamic linking interface library

`libg`

Dummy library containing no functions. Previously was a runtime library for **g++**

`libieee`

Linking in this module forces error handling rules for math functions as defined by the Institute of Electrical and Electronic Engineers (IEEE). The default is POSIX.1 error handling

`libm`

The mathematical library

`libmcheck`

Turns on memory allocation checking when linked to

`libmemusage`

Used by **memusage** to help collect information about the memory usage of a program

`libnsl`

The network services library

`libnss`

The Name Service Switch libraries, containing functions for resolving host names, user names, group names, aliases, services, protocols, etc.

`libpthread`

The POSIX threads library

`libresolv`

Contains functions for creating, sending, and interpreting packets to the Internet domain name servers

`librpcsvc`

Contains functions providing miscellaneous RPC services

`librt`

Contains functions providing most of the interfaces specified by the POSIX.1b Realtime Extension

`libthread_db`

Contains functions useful for building debuggers for multi-threaded programs

`libutil`

Contains code for “standard” functions used in many different Unix utilities

Last updated on

## 6.10. Adjusting the Toolchain

Now that the final C libraries have been installed, it is time to adjust the toolchain so that it will link any newly compiled program against these new libraries.

First, backup the `/tools` linker, and replace it with the adjusted linker we made in chapter 5. We'll also create a link to its counterpart in `/tools/$(uname -m)-pc-linux-gnu/bin`:

    mv -v /tools/bin/{ld,ld-old}
    mv -v /tools/$(uname -m)-pc-linux-gnu/bin/{ld,ld-old}
    mv -v /tools/bin/{ld-new,ld}
    ln -sv /tools/bin/ld /tools/$(uname -m)-pc-linux-gnu/bin/ld

Next, amend the GCC specs file so that it points to the new dynamic linker. Simply deleting all instances of “/tools” should leave us with the correct path to the dynamic linker. Also adjust the specs file so that GCC knows where to find the correct headers and Glibc start files. A **sed**command accomplishes this:

    gcc -dumpspecs | sed -e 's@/tools@@g'                   \
        -e '/\*startfile_prefix_spec:/{n;s@.*@/usr/lib/ @}' \
        -e '/\*cpp:/{n;s@$@ -isystem /usr/include@}' >      \
        `dirname $(gcc --print-libgcc-file-name)`/specs

It is a good idea to visually inspect the specs file to verify the intended change was actually made.

It is imperative at this point to ensure that the basic functions (compiling and linking) of the adjusted toolchain are working as expected. To do this, perform the following sanity checks:

    echo 'int main(){}' > dummy.c
    cc dummy.c -v -Wl,--verbose &> dummy.log
    readelf -l a.out | grep ': /lib'

There should be no errors, and the output of the last command will be (allowing for platform-specific differences in dynamic linker name):

    [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]

Note that on 64-bit systems `/lib` is the location of our dynamic linker, but is accessed via a symbolic link in /lib64.

### Note

On 32-bit systems the interpreter should be /lib/ld-linux.so.2.

Now make sure that we're setup to use the correct start files:

    grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log

The output of the last command should be:

    /usr/lib/../lib/crt1.o succeeded
    /usr/lib/../lib/crti.o succeeded
    /usr/lib/../lib/crtn.o succeeded

Verify that the compiler is searching for the correct header files:

    grep -B1 '^ /usr/include' dummy.log

This command should return the following output:

    #include <...> search starts here:
     /usr/include

Next, verify that the new linker is being used with the correct search paths:

    grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'

References to paths that have components with '-linux-gnu' should be ignored, but otherwise the output of the last command should be:

    SEARCH_DIR("/usr/lib")
    SEARCH_DIR("/lib")

Next make sure that we're using the correct libc:

    grep "/lib.*/libc.so.6 " dummy.log

The output of the last command should be:

    attempt to open /lib/libc.so.6 succeeded

Lastly, make sure GCC is using the correct dynamic linker:

    grep found dummy.log

The output of the last command should be (allowing for platform-specific differences in dynamic linker name):

    found ld-linux-x86-64.so.2 at /lib/ld-linux-x86-64.so.2

If the output does not appear as shown above or is not received at all, then something is seriously wrong. Investigate and retrace the steps to find out where the problem is and correct it. The most likely reason is that something went wrong with the specs file adjustment. Any issues will need to be resolved before continuing on with the process.

Once everything is working correctly, clean up the test files:

    rm -v dummy.c a.out dummy.log

## 6.11. Zlib-1.2.11

The Zlib package contains compression and decompression routines used by some programs.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**4.5 MB

### 6.11.1. Installation of Zlib

Prepare Zlib for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

The shared library needs to be moved to `/lib`, and as a result the `.so` file in `/usr/lib` will need to be recreated:

    mv -v /usr/lib/libz.so.* /lib
    ln -sfv ../../lib/$(readlink /usr/lib/libz.so) /usr/lib/libz.so

### 6.11.2. Contents of Zlib

**Installed libraries:**libz.{a,so}

#### Short Descriptions

`libz`

Contains compression and decompression functions used by some programs

Last updated on

## 6.12. File-5.31

The File package contains a utility for determining the type of a given file or files.

**Approximate build time:**0.1 SBU

**Required disk space:**16 MB

### 6.12.1. Installation of File

Prepare File for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.12.2. Contents of File

**Installed programs:**file

**Installed library:**libmagic.so

#### Short Descriptions

**file**

Tries to classify each given file; it does this by performing several tests—file system tests, magic number tests, and language tests

`libmagic`

Contains routines for magic number recognition, used by the **file** program

Last updated on

## 6.13. Readline-7.0

The Readline package is a set of libraries that offers command-line editing and history capabilities.

**Approximate build time:**0.1 SBU

**Required disk space:**15 MB

### 6.13.1. Installation of Readline

Reinstalling Readline will cause the old libraries to be moved to <libraryname>.old. While this is normally not a problem, in some cases it can trigger a linking bug in **ldconfig**. This can be avoided by issuing the following two seds:

    sed -i '/MV.*old/d' Makefile.in
    sed -i '/{OLDSUFF}/c:' support/shlib-install

Prepare Readline for compilation:

    ./configure --prefix=/usr    \
                --disable-static \
                --docdir=/usr/share/doc/readline-7.0

Compile the package:

    make SHLIB_LIBS="-L/tools/lib -lncursesw"

**The meaning of the make option:**
*`SHLIB_LIBS="-L/tools/lib -lncursesw"`*
This option forces Readline to link against the `libncursesw` library.

This package does not come with a test suite.

Install the package:

    make SHLIB_LIBS="-L/tools/lib -lncurses" install

Now move the dynamic libraries to a more appropriate location and fix up some symbolic links:

    mv -v /usr/lib/lib{readline,history}.so.* /lib
    ln -sfv ../../lib/$(readlink /usr/lib/libreadline.so) /usr/lib/libreadline.so
    ln -sfv ../../lib/$(readlink /usr/lib/libhistory.so ) /usr/lib/libhistory.so

If desired, install the documentation:

    install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-7.0

### 6.13.2. Contents of Readline

**Installed libraries:**libhistory.so and libreadline.so

**Installed directories:**/usr/include/readline, /usr/share/readline, and /usr/share/doc/readline-7.0

#### Short Descriptions

`libhistory`

Provides a consistent user interface for recalling lines of history

`libreadline`

Aids in the consistency of user interface across discrete programs that need to provide a command line interface

Last updated on

## 6.14. M4-1.4.18

The M4 package contains a macro processor.

**Approximate build time:**0.4 SBU

**Required disk space:**30 MB

### 6.14.1. Installation of M4

Prepare M4 for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.14.2. Contents of M4

**Installed program:**m4

#### Short Descriptions

**m4**

copies the given files while expanding the macros that they contain [These macros are either built-in or user-defined and can take any number of arguments. Besides performing macro expansion, **m4** has built-in functions for including named files, running Unix commands, performing integer arithmetic, manipulating text, recursion, etc. The **m4** program can be used either as a front-end to a compiler or as a macro processor in its own right.]

Last updated on

## 6.15. Bc-1.07.1

The Bc package contains an arbitrary precision numeric processing language.

**Approximate build time:**0.1 SBU

**Required disk space:**3.6 MB

### 6.15.1. Installation of Bc

First, change an internal script to use **sed** instead of **ed**:

    cat > bc/fix-libmath_h << "EOF"
    #! /bin/bash
    sed -e '1   s/^/{"/' \
        -e     's/$/",/' \
        -e '2,$ s/^/"/'  \
        -e   '$ d'       \
        -i libmath.h

    sed -e '$ s/$/0}/' \
        -i libmath.h
    EOF

Create temporary symbolic links so the package can find the readline library and confirm that its required libncurses library is available. Even though the libraries are in /tools/lib at this point, the system will use /usr/lib at the end of this chapter.

    ln -sv /tools/lib/libncursesw.so.6 /usr/lib/libncursesw.so.6
    ln -sfv libncurses.so.6 /usr/lib/libncurses.so

Fix an issue in **configure** due to missing files in the early stages of LFS:

    sed -i -e '/flex/s/as_fn_error/: ;; # &/' configure

Prepare Bc for compilation:

    ./configure --prefix=/usr           \
                --with-readline         \
                --mandir=/usr/share/man \
                --infodir=/usr/share/info

**The meaning of the configure options:**
*`--with-readline`*
This option tells Bc to use the `readline` library that is already installed on the system rather than using its own readline version.

Compile the package:

    make

To test bc, run the commands below. There is quite a bit of output, so you may want to redirect it to a file. There are a very small percentage of tests (10 of 12,144) that will indicate a round off error at the last digit.

    echo "quit" | ./bc/bc -l Test/checklib.b

Install the package:

    make install

### 6.15.2. Contents of Bc

**Installed programs:**bc and dc

#### Short Descriptions

**bc**

is a command line calculator

**dc**

is a reverse-polish command line calculator

Last updated on

## 6.16. Binutils-2.29

The Binutils package contains a linker, an assembler, and other tools for handling object files.

**Approximate build time:**5.8 SBU

**Required disk space:**4.2 GB

### 6.16.1. Installation of Binutils

Verify that the PTYs are working properly inside the chroot environment by performing a simple test:

    expect -c "spawn ls"

This command should output the following:

    spawn ls

If, instead, the output includes the message below, then the environment is not set up for proper PTY operation. This issue needs to be resolved before running the test suites for Binutils and GCC:

    The system has no more ptys.
    Ask your system administrator to create more.

The Binutils documentation recommends building Binutils in a dedicated build directory:

    mkdir -v build
    cd       build

Prepare Binutils for compilation:

    ../configure --prefix=/usr       \
                 --enable-gold       \
                 --enable-ld=default \
                 --enable-plugins    \
                 --enable-shared     \
                 --disable-werror    \
                 --with-system-zlib

**The meaning of the configure parameters:**
*`--enable-gold`*
Build the gold linker and install it as ld.gold (along side the default linker).
*`--enable-ld=default`*
Build the original bdf linker and install it as both ld (the default linker) and ld.bfd.
*`--enable-plugins`*
Enables plugin support for the linker.
*`--with-system-zlib`*
Use the installed zlib library rather than building the included version.

Compile the package:

    make tooldir=/usr

**The meaning of the make parameter:**
*`tooldir=/usr`*
Normally, the tooldir (the directory where the executables will ultimately be located) is set to `$(exec_prefix)/$(target_alias)`. For example, x86_64 machines would expand that to `/usr/x86_64-unknown-linux-gnu`. Because this is a custom system, this target-specific directory in `/usr` is not required. `$(exec_prefix)/$(target_alias)` would be used if the system was used to cross-compile (for example, compiling a package on an Intel machine that generates code that can be executed on PowerPC machines).

### Important

The test suite for Binutils in this section is considered critical. Do not skip it under any circumstances.

Test the results:

    make -k check

One test, debug_msg.sh, is known to fail.

Install the package:

    make tooldir=/usr install

### 6.16.2. Contents of Binutils

**Installed programs:**addr2line, ar, as, c++filt, elfedit, gprof, ld, ld.bfd, ld.gold, nm, objcopy, objdump, ranlib, readelf, size, strings, and strip

**Installed libraries:**libbfd.{a,so} and libopcodes.{a,so}

**Installed directory:**/usr/lib/ldscripts

#### Short Descriptions

**addr2line**

Translates program addresses to file names and line numbers; given an address and the name of an executable, it uses the debugging information in the executable to determine which source file and line number are associated with the address

**ar**

Creates, modifies, and extracts from archives

**as**

An assembler that assembles the output of **gcc** into object files

**c++filt**

Used by the linker to de-mangle C++ and Java symbols and to keep overloaded functions from clashing

**elfedit**

Updates the ELF header of ELF files

**gprof**

Displays call graph profile data

**ld**

A linker that combines a number of object and archive files into a single file, relocating their data and tying up symbol references

**ld.gold**

A cut down version of ld that only supports the elf object file format

**ld.bfd**

Hard link to **ld**

**nm**

Lists the symbols occurring in a given object file

**objcopy**

Translates one type of object file into another

**objdump**

Displays information about the given object file, with options controlling the particular information to display; the information shown is useful to programmers who are working on the compilation tools

**ranlib**

Generates an index of the contents of an archive and stores it in the archive; the index lists all of the symbols defined by archive members that are relocatable object files

**readelf**

Displays information about ELF type binaries

**size**

Lists the section sizes and the total size for the given object files

**strings**

Outputs, for each given file, the sequences of printable characters that are of at least the specified length (defaulting to four); for object files, it prints, by default, only the strings from the initializing and loading sections while for other types of files, it scans the entire file

**strip**

Discards symbols from object files

`libbfd`

The Binary File Descriptor library

`libopcodes`

A library for dealing with opcodes—the “readable text” versions of instructions for the processor; it is used for building utilities like **objdump**

Last updated on

## 6.17. GMP-6.1.2

The GMP package contains math libraries. These have useful functions for arbitrary precision arithmetic.

**Approximate build time:**1.2 SBU

**Required disk space:**59 MB

### 6.17.1. Installation of GMP

### Note

If you are building for 32-bit x86, but you have a CPU which is capable of running 64-bit code *and* you have specified `CFLAGS`in the environment, the configure script will attempt to configure for 64-bits and fail. Avoid this by invoking the configure command below with

    *ABI=32* ./configure ...

### Note

The default settings of GMP produce libraries optimized for the host processor. If libraries suitable for processors less capable than the host's CPU are desired, generic libraries can be created by running the following:

    cp -v configfsf.guess config.guess
    cp -v configfsf.sub   config.sub

Prepare GMP for compilation:

    ./configure --prefix=/usr    \
                --enable-cxx     \
                --disable-static \
                --docdir=/usr/share/doc/gmp-6.1.2

**The meaning of the new configure options:**
*`--enable-cxx`*
This parameter enables C++ support
*`--docdir=/usr/share/doc/gmp-6.1.2`*
This variable specifies the correct place for the documentation.

Compile the package and generate the HTML documentation:

    make
    make html

### Important

The test suite for GMP in this section is considered critical. Do not skip it under any circumstances.

Test the results:

    make check 2>&1 | tee gmp-check-log

### Caution

The code in gmp is highly optimized for the processor where it is built. Occasionally, the code that detects the processor misidentifies the system capabilities and there will be errors in the tests or other applications using the gmp libraries with the message "Illegal instruction". In this case, gmp should be reconfigured with the option --build=x86_64-unknown-linux-gnu and rebuilt.

Ensure that all 190 tests in the test suite passed. Check the results by issuing the following command:

    awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log

Install the package and its documentation:

    make install
    make install-html

### 6.17.2. Contents of GMP

**Installed Libraries:**libgmp.so and libgmpxx.so

**Installed directory:**/usr/share/doc/gmp-6.1.2

#### Short Descriptions

`libgmp`

Contains precision math functions

`libgmpxx`

Contains C++ precision math functions

Last updated on

## 6.18. MPFR-3.1.5

The MPFR package contains functions for multiple precision math.

**Approximate build time:**0.8 SBU

**Required disk space:**45 MB

### 6.18.1. Installation of MPFR

Prepare MPFR for compilation:

    ./configure --prefix=/usr        \
                --disable-static     \
                --enable-thread-safe \
                --docdir=/usr/share/doc/mpfr-3.1.5

Compile the package and generate the HTML documentation:

    make
    make html

### Important

The test suite for MPFR in this section is considered critical. Do not skip it under any circumstances.

Test the results and ensure that all tests passed:

    make check

Install the package and its documentation:

    make install
    make install-html

### 6.18.2. Contents of MPFR

**Installed Libraries:**libmpfr.so

**Installed directory:**/usr/share/doc/mpfr-3.1.5

#### Short Descriptions

`libmpfr`

Contains multiple-precision math functions

Last updated on

## 6.19. MPC-1.0.3

The MPC package contains a library for the arithmetic of complex numbers with arbitrarily high precision and correct rounding of the result.

**Approximate build time:**0.3 SBU

**Required disk space:**17 MB

### 6.19.1. Installation of MPC

Prepare MPC for compilation:

    ./configure --prefix=/usr    \
                --disable-static \
                --docdir=/usr/share/doc/mpc-1.0.3

Compile the package and generate the HTML documentation:

    make
    make html

To test the results, issue:

    make check

Install the package and its documentation:

    make install
    make install-html

### 6.19.2. Contents of MPC

**Installed Libraries:**libmpc.so

**Installed Directory:**/usr/share/doc/mpc-1.0.3

#### Short Descriptions

`libmpc`

Contains complex math functions

Last updated on

## 6.20. GCC-7.2.0

The GCC package contains the GNU compiler collection, which includes the C and C++ compilers.

**Approximate build time:**82 SBU (with tests)

**Required disk space:**3.2 GB

### 6.20.1. Installation of GCC

If building on x86_64, change the default directory name for 64-bit libraries to “lib”:

    case $(uname -m) in
      x86_64)
        sed -e '/m64=/s/lib64/lib/' \
            -i.orig gcc/config/i386/t-linux64
      ;;
    esac

Remove the symlink created earlier as the final gcc includes will be installed here:

    rm -f /usr/lib/gcc

The GCC documentation recommends building GCC in a dedicated build directory:

    mkdir -v build
    cd       build

Prepare GCC for compilation:

    SED=sed                               \
    ../configure --prefix=/usr            \
                 --enable-languages=c,c++ \
                 --disable-multilib       \
                 --disable-bootstrap      \
                 --with-system-zlib

Note that for other languages, there are some prerequisites that are not yet available. See the [BLFS Book](http://www.linuxfromscratch.org/blfs/view/8.1/general/gcc.html) for instructions on how to build all of GCC's supported languages.

**The meaning of the new configure parameters:**
`SED=sed`
Setting this environment variable prevents a hard-coded path to /tools/bin/sed.
*`--with-system-zlib`*
This switch tells GCC to link to the system installed copy of the Zlib library, rather than its own internal copy.

Compile the package:

    make

### Important

In this section, the test suite for GCC is considered critical. Do not skip it under any circumstance.

One set of tests in the GCC test suite is known to exhaust the stack, so increase the stack size prior to running the tests:

    ulimit -s 32768

Test the results, but do not stop at errors:

    make -k check

To receive a summary of the test suite results, run:

    ../contrib/test_summary

For only the summaries, pipe the output through **`grep -A7 Summ`**.

Results can be compared with those located at [http://www.linuxfromscratch.org/lfs/build-logs/8.1/](http://www.linuxfromscratch.org/lfs/build-logs/8.1/) and [http://gcc.gnu.org/ml/gcc-testresults/](http://gcc.gnu.org/ml/gcc-testresults/).

A few unexpected failures cannot always be avoided. The GCC developers are usually aware of these issues, but have not resolved them yet. In particular, five tests in the libstdc++ test suite are known to fail when running as the root user as we do here. Unless the test results are vastly different from those at the above URL, it is safe to continue.

### Note

On some combinations of kernel configuration and AMD processors there may be more than 1100 failures in the gcc.target/i386/mpx tests (which are designed to test the MPX option on recent Intel processors). These can safely be ignored on AMD processors.

Install the package:

    make install

Create a symlink required by the [FHS](http://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s09.html) for "historical" reasons.

    ln -sv ../usr/bin/cpp /lib

Many packages use the name **cc** to call the C compiler. To satisfy those packages, create a symlink:

    ln -sv gcc /usr/bin/cc

Add a compatibility symlink to enable building programs with Link Time Optimization (LTO):

    install -v -dm755 /usr/lib/bfd-plugins
    ln -sfv ../../libexec/gcc/$(gcc -dumpmachine)/7.2.0/liblto_plugin.so \
            /usr/lib/bfd-plugins/

Now that our final toolchain is in place, it is important to again ensure that compiling and linking will work as expected. We do this by performing the same sanity checks as we did earlier in the chapter:

    echo 'int main(){}' > dummy.c
    cc dummy.c -v -Wl,--verbose &> dummy.log
    readelf -l a.out | grep ': /lib'

There should be no errors, and the output of the last command will be (allowing for platform-specific differences in dynamic linker name):

    [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]

Now make sure that we're setup to use the correct start files:

    grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log

The output of the last command should be:

    /usr/lib/gcc/x86_64-pc-linux-gnu/7.2.0/../../../../lib/crt1.o succeeded
    /usr/lib/gcc/x86_64-pc-linux-gnu/7.2.0/../../../../lib/crti.o succeeded
    /usr/lib/gcc/x86_64-pc-linux-gnu/7.2.0/../../../../lib/crtn.o succeeded

Depending on your machine architecture, the above may differ slightly, the difference usually being the name of the directory after `/usr/lib/gcc`. The important thing to look for here is that **gcc** has found all three `crt*.o` files under the `/usr/lib` directory.

Verify that the compiler is searching for the correct header files:

    grep -B4 '^ /usr/include' dummy.log

This command should return the following output:

    #include <...> search starts here:
     /usr/lib/gcc/x86_64-pc-linux-gnu/7.2.0/include
     /usr/local/include
     /usr/lib/gcc/x86_64-pc-linux-gnu/7.2.0/include-fixed
     /usr/include

Again, note that the directory named after your target triplet may be different than the above, depending on your architecture.

### Note

As of version 4.3.0, GCC now unconditionally installs the `limits.h` file into the private `include-fixed` directory, and that directory is required to be in place.

Next, verify that the new linker is being used with the correct search paths:

    grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'

References to paths that have components with '-linux-gnu' should be ignored, but otherwise the output of the last command should be:

    SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib64")
    SEARCH_DIR("/usr/local/lib64")
    SEARCH_DIR("/lib64")
    SEARCH_DIR("/usr/lib64")
    SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib")
    SEARCH_DIR("/usr/local/lib")
    SEARCH_DIR("/lib")
    SEARCH_DIR("/usr/lib");

A 32-bit system may see a few different directories. For example, here is the output from an i686 machine:

    SEARCH_DIR("/usr/i686-pc-linux-gnu/lib32")
    SEARCH_DIR("/usr/local/lib32")
    SEARCH_DIR("/lib32")
    SEARCH_DIR("/usr/lib32")
    SEARCH_DIR("/usr/i686-pc-linux-gnu/lib")
    SEARCH_DIR("/usr/local/lib")
    SEARCH_DIR("/lib")
    SEARCH_DIR("/usr/lib");

Next make sure that we're using the correct libc:

    grep "/lib.*/libc.so.6 " dummy.log

The output of the last command should be:

    attempt to open /lib/libc.so.6 succeeded

Lastly, make sure GCC is using the correct dynamic linker:

    grep found dummy.log

The output of the last command should be (allowing for platform-specific differences in dynamic linker name):

    found ld-linux-x86-64.so.2 at /lib/ld-linux-x86-64.so.2

If the output does not appear as shown above or is not received at all, then something is seriously wrong. Investigate and retrace the steps to find out where the problem is and correct it. The most likely reason is that something went wrong with the specs file adjustment. Any issues will need to be resolved before continuing on with the process.

Once everything is working correctly, clean up the test files:

    rm -v dummy.c a.out dummy.log

Finally, move a misplaced file:

    mkdir -pv /usr/share/gdb/auto-load/usr/lib
    mv -v /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib

### 6.20.2. Contents of GCC

**Installed programs:**c++, cc (link to gcc), cpp, g++, gcc, gcc-ar, gcc-nm, gcc-ranlib, and gcov

**Installed libraries:**libasan.{a,so}, libatomic.{a,so}, libgcc.a, libgcc_eh.a, libgcc_s.so, libgcov.a, libgomp.{a,so}, libiberty.a, libitm.{a,so}, liblto_plugin.so, libquadmath.{a,so}, libssp.{a,so}, libssp_nonshared.a, libstdc++.{a,so}, libsupc++.a, and libtsan.{a,so}

**Installed directories:**/usr/include/c++, /usr/lib/gcc, /usr/libexec/gcc, and /usr/share/gcc-7.2.0

#### Short Descriptions

**c++**

The C++ compiler

**cc**

The C compiler

**cpp**

The C preprocessor; it is used by the compiler to expand the #include, #define, and similar statements in the source files

**g++**

The C++ compiler

**gcc**

The C compiler

**gcc-ar**

A wrapper around **ar** that adds a plugin to the command line. This program is only used to add "link time optimization" and is not useful with the default build options

**gcc-nm**

A wrapper around **nm** that adds a plugin to the command line. This program is only used to add "link time optimization" and is not useful with the default build options

**gcc-ranlib**

A wrapper around **ranlib** that adds a plugin to the command line. This program is only used to add "link time optimization" and is not useful with the default build options

**gcov**

A coverage testing tool; it is used to analyze programs to determine where optimizations will have the most effect

**libasan**

The Address Sanitizer runtime library

`libgcc`

Contains run-time support for **gcc**

`libgcov`

This library is linked in to a program when GCC is instructed to enable profiling

`libgomp`

GNU implementation of the OpenMP API for multi-platform shared-memory parallel programming in C/C++ and Fortran

`libiberty`

Contains routines used by various GNU programs, including **getopt**, **obstack**, **strerror**, **strtol**, and **strtoul**

`liblto_plugin`

GCC's Link Time Optimization (LTO) plugin allows GCC to perform optimizations across compilation units

`libquadmath`

GCC Quad Precision Math Library API

`libssp`

Contains routines supporting GCC's stack-smashing protection functionality

`libstdc++`

The standard C++ library

`libsupc++`

Provides supporting routines for the C++ programming language

`libtsan`

The Thread Sanitizer runtime library

Last updated on

## 6.21. Bzip2-1.0.6

The Bzip2 package contains programs for compressing and decompressing files. Compressing text files with **bzip2** yields a much better compression percentage than with the traditional **gzip**.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**2.3 MB

### 6.21.1. Installation of Bzip2

Apply a patch that will install the documentation for this package:

    patch -Np1 -i ../bzip2-1.0.6-install_docs-1.patch

The following command ensures installation of symbolic links are relative:

    sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile

Ensure the man pages are installed into the correct location:

    sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile

Prepare Bzip2 for compilation with:

    make -f Makefile-libbz2_so
    make clean

**The meaning of the make parameter:**
*`-f Makefile-libbz2_so`*
This will cause Bzip2 to be built using a different `Makefile` file, in this case the `Makefile-libbz2_so` file, which creates a dynamic `libbz2.so` library and links the Bzip2 utilities against it.

Compile and test the package:

    make

Install the programs:

    make PREFIX=/usr install

Install the shared **bzip2** binary into the `/bin` directory, make some necessary symbolic links, and clean up:

    cp -v bzip2-shared /bin/bzip2
    cp -av libbz2.so* /lib
    ln -sv ../../lib/libbz2.so.1.0 /usr/lib/libbz2.so
    rm -v /usr/bin/{bunzip2,bzcat,bzip2}
    ln -sv bzip2 /bin/bunzip2
    ln -sv bzip2 /bin/bzcat

### 6.21.2. Contents of Bzip2

**Installed programs:**bunzip2 (link to bzip2), bzcat (link to bzip2), bzcmp (link to bzdiff), bzdiff, bzegrep (link to bzgrep), bzfgrep (link to bzgrep), bzgrep, bzip2, bzip2recover, bzless (link to bzmore), and bzmore

**Installed libraries:**libbz2.{a,so}

**Installed directory:**/usr/share/doc/bzip2-1.0.6

#### Short Descriptions

**bunzip2**

Decompresses bzipped files

**bzcat**

Decompresses to standard output

**bzcmp**

Runs **cmp** on bzipped files

**bzdiff**

Runs **diff** on bzipped files

**bzegrep**

Runs **egrep** on bzipped files

**bzfgrep**

Runs **fgrep** on bzipped files

**bzgrep**

Runs **grep** on bzipped files

**bzip2**

Compresses files using the Burrows-Wheeler block sorting text compression algorithm with Huffman coding; the compression rate is better than that achieved by more conventional compressors using “Lempel-Ziv” algorithms, like **gzip**

**bzip2recover**

Tries to recover data from damaged bzipped files

**bzless**

Runs **less** on bzipped files

**bzmore**

Runs **more** on bzipped files

`libbz2`

The library implementing lossless, block-sorting data compression, using the Burrows-Wheeler algorithm

Last updated on

## 6.22. Pkg-config-0.29.2

The pkg-config package contains a tool for passing the include path and/or library paths to build tools during the configure and make file execution.

**Approximate build time:**0.3 SBU

**Required disk space:**28 MB

### 6.22.1. Installation of Pkg-config

Prepare Pkg-config for compilation:

    ./configure --prefix=/usr              \
                --with-internal-glib       \
                --disable-host-tool        \
                --docdir=/usr/share/doc/pkg-config-0.29.2

**The meaning of the new configure options:**
*`--with-internal-glib`*
This will allow pkg-config to use its internal version of Glib because an external version is not available in LFS.
*`--disable-host-tool`*
This option disables the creation of an undesired hard link to the pkg-config program.

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.22.2. Contents of Pkg-config

**Installed program:**pkg-config

**Installed directory:**/usr/share/doc/pkg-config-0.29.2

#### Short Descriptions

**pkg-config**

returns meta information for the specified library or package

Last updated on

## 6.23. Ncurses-6.0

The Ncurses package contains libraries for terminal-independent handling of character screens.

**Approximate build time:**0.3 SBU

**Required disk space:**39 MB

### 6.23.1. Installation of Ncurses

Don't install a static library that is not handled by configure:

    sed -i '/LIBTOOL_INSTALL/d' c++/Makefile.in

Prepare Ncurses for compilation:

    ./configure --prefix=/usr           \
                --mandir=/usr/share/man \
                --with-shared           \
                --without-debug         \
                --without-normal        \
                --enable-pc-files       \
                --enable-widec

**The meaning of the new configure options:**
*`--enable-widec`*
This switch causes wide-character libraries (e.g., `libncursesw.so.6.0`) to be built instead of normal ones (e.g., `libncurses.so.6.0`). These wide-character libraries are usable in both multibyte and traditional 8-bit locales, while normal libraries work properly only in 8-bit locales. Wide-character and normal libraries are source-compatible, but not binary-compatible.
*`--enable-pc-files`*
This switch generates and installs .pc files for pkg-config.
*`--without-normal`*
This switch disables building and installing most static libraries.

Compile the package:

    make

This package has a test suite, but it can only be run after the package has been installed. The tests reside in the `test/` directory. See the`README` file in that directory for further details.

Install the package:

    make install

Move the shared libraries to the `/lib` directory, where they are expected to reside:

    mv -v /usr/lib/libncursesw.so.6* /lib

Because the libraries have been moved, one symlink points to a non-existent file. Recreate it:

    ln -sfv ../../lib/$(readlink /usr/lib/libncursesw.so) /usr/lib/libncursesw.so

Many applications still expect the linker to be able to find non-wide-character Ncurses libraries. Trick such applications into linking with wide-character libraries by means of symlinks and linker scripts:

    for lib in ncurses form panel menu ; do
        rm -vf                    /usr/lib/lib${lib}.so
        echo "INPUT(-l${lib}w)" > /usr/lib/lib${lib}.so
        ln -sfv ${lib}w.pc        /usr/lib/pkgconfig/${lib}.pc
    done

Finally, make sure that old applications that look for `-lcurses` at build time are still buildable:

    rm -vf                     /usr/lib/libcursesw.so
    echo "INPUT(-lncursesw)" > /usr/lib/libcursesw.so
    ln -sfv libncurses.so      /usr/lib/libcurses.so

If desired, install the Ncurses documentation:

    mkdir -v       /usr/share/doc/ncurses-6.0
    cp -v -R doc/* /usr/share/doc/ncurses-6.0

### Note

The instructions above don't create non-wide-character Ncurses libraries since no package installed by compiling from sources would link against them at runtime. However, the only known binary-only applications that link against non-wide-character Ncurses libraries require version 5. If you must have such libraries because of some binary-only application or to be compliant with LSB, build the package again with the following commands:

    make distclean
    ./configure --prefix=/usr    \
                --with-shared    \
                --without-normal \
                --without-debug  \
                --without-cxx-binding \
                --with-abi-version=5
    make sources libs
    cp -av lib/lib*.so.5* /usr/lib

### 6.23.2. Contents of Ncurses

**Installed programs:**captoinfo (link to tic), clear, infocmp, infotocap (link to tic), ncursesw6-config, reset (link to tset), tabs, tic, toe, tput, and tset

**Installed libraries:**libcursesw.so (symlink and linker script to libncursesw.so), libformw.so, libmenuw.so, libncursesw.so, libncurses++w.a, libpanelw.so, and their non-wide-character counterparts without "w" in the library names.

**Installed directories:**/usr/share/tabset, /usr/share/terminfo, and /usr/share/doc/ncurses-6.0

#### Short Descriptions

**captoinfo**

Converts a termcap description into a terminfo description

**clear**

Clears the screen, if possible

**infocmp**

Compares or prints out terminfo descriptions

**infotocap**

Converts a terminfo description into a termcap description

**ncursesw6-config**

Provides configuration information for ncurses

**reset**

Reinitializes a terminal to its default values

**tabs**

Clears and sets tab stops on a terminal

**tic**

The terminfo entry-description compiler that translates a terminfo file from source format into the binary format needed for the ncurses library routines [A terminfo file contains information on the capabilities of a certain terminal.]

**toe**

Lists all available terminal types, giving the primary name and description for each

**tput**

Makes the values of terminal-dependent capabilities available to the shell; it can also be used to reset or initialize a terminal or report its long name

**tset**

Can be used to initialize terminals

`libcursesw`

A link to `libncursesw`

`libncursesw`

Contains functions to display text in many complex ways on a terminal screen; a good example of the use of these functions is the menu displayed during the kernel's **make menuconfig**

`libformw`

Contains functions to implement forms

`libmenuw`

Contains functions to implement menus

`libpanelw`

Contains functions to implement panels

Last updated on

## 6.24. Attr-2.4.47

The attr package contains utilities to administer the extended attributes on filesystem objects.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**3.3 MB

### 6.24.1. Installation of Attr

Modify the documentation directory so that it is a versioned directory:

    sed -i -e 's|/@pkg_name@|&-@pkg_version@|' include/builddefs.in

Prevent installation of manual pages that were already installed by the [`man pages`](Linux%20From%20Scratch.html#man-pages) package:

    sed -i -e "/SUBDIRS/s|man[25]||g" man/Makefile

Fix a problem in the test procedures caused by changes in perl-5.26:

    sed -i 's:{(:\\{(:' test/run

Prepare Attr for compilation:

    ./configure --prefix=/usr \
                --bindir=/bin \
                --disable-static

Compile the package:

    make

The tests need to be run on a filesystem that supports extended attributes such as the ext2, ext3, or ext4 filesystems. The tests are also known to fail if running multiple simultaneous tests (-j option greater than 1). To test the results, issue:

    make -j1 tests root-tests

Install the package:

    make install install-dev install-lib
    chmod -v 755 /usr/lib/libattr.so

The shared library needs to be moved to `/lib`, and as a result the `.so` file in `/usr/lib` will need to be recreated:

    mv -v /usr/lib/libattr.so.* /lib
    ln -sfv ../../lib/$(readlink /usr/lib/libattr.so) /usr/lib/libattr.so

### 6.24.2. Contents of Attr

**Installed programs:**attr, getfattr, and setattr

**Installed library:**libattr.so

**Installed directories:**/usr/include/attr and /usr/share/doc/attr-2.4.47

#### Short Descriptions

**attr**

Extends attributes on filesystem objects

**getfattr**

Gets the extended attributes of filesystem objects

**setattr**

Sets the extended attributes of filesystem objects

`libattr`

Contains the libbrary functions for manipulating extended attributes

Last updated on

## 6.25. Acl-2.2.52

The Acl package contains utilities to administer Access Control Lists, which are used to define more fine-grained discretionary access rights for files and directories.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**4.8 MB

### 6.25.1. Installation of Acl

Modify the documentation directory so that it is a versioned directory:

    sed -i -e 's|/@pkg_name@|&-@pkg_version@|' include/builddefs.in

Fix some broken tests:

    sed -i "s:| sed.*::g" test/{sbits-restore,cp,misc}.test

Fix a problem in the test procedures caused by changes in perl-5.26:

    sed -i 's/{(/\\{(/' test/run

Additionally, fix a bug that causes **getfacl -e** to segfault on overly long group name:

    sed -i -e "/TABS-1;/a if (x > (TABS-1)) x = (TABS-1);" \
        libacl/__acl_to_any_text.c

Prepare Acl for compilation:

    ./configure --prefix=/usr    \
                --bindir=/bin    \
                --disable-static \
                --libexecdir=/usr/lib

Compile the package:

    make

The Acl tests need to be run on a filesystem that supports access controls after Coreutils has been built with the Acl libraries. If desired, return to this package and run **make -j1 tests** after Coreutils has been built later in this chapter.

Install the package:

    make install install-dev install-lib
    chmod -v 755 /usr/lib/libacl.so

The shared library needs to be moved to `/lib`, and as a result the `.so` file in `/usr/lib` will need to be recreated:

    mv -v /usr/lib/libacl.so.* /lib
    ln -sfv ../../lib/$(readlink /usr/lib/libacl.so) /usr/lib/libacl.so

### 6.25.2. Contents of Acl

**Installed programs:**chacl, getfacl, and setfacl

**Installed library:**libacl.so

**Installed directories:**/usr/include/acl and /usr/share/doc/acl-2.2.52

#### Short Descriptions

**chacl**

Changes the access control list of a file or directory

**getfacl**

Gets file access control lists

**setfacl**

Sets file access control lists

`libacl`

Contains the library functions for manipulating Access Control Lists

Last updated on

## 6.26. Libcap-2.25

The Libcap package implements the user-space interfaces to the POSIX 1003.1e capabilities available in Linux kernels. These capabilities are a partitioning of the all powerful root privilege into a set of distinct privileges.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**1.3 MB

### 6.26.1. Installation of Libcap

Prevent a static library from being installed:

    sed -i '/install.*STALIBNAME/d' libcap/Makefile

Compile the package:

    make

This package does not come with a test suite.

Install the package:

    make RAISE_SETFCAP=no lib=lib prefix=/usr install
    chmod -v 755 /usr/lib/libcap.so

**The meaning of the make option:**
*`RAISE_SETFCAP=no`*
This parameter skips trying to use **setcap** on itself. This avoids an installation error if the kernel or file system does not support extended capabilities.
*`lib=lib`*
This parameter installs the library in `$prefix/lib` rather than `$prefix/lib64` on x86_64. It has no effect on x86.

The shared library needs to be moved to `/lib`, and as a result the `.so` file in `/usr/lib` will need to be recreated:

    mv -v /usr/lib/libcap.so.* /lib
    ln -sfv ../../lib/$(readlink /usr/lib/libcap.so) /usr/lib/libcap.so

### 6.26.2. Contents of Libcap

**Installed programs:**capsh, getcap, getpcaps, and setcap

**Installed library:**libcap.so

#### Short Descriptions

**capsh**

A shell wrapper to explore and constrain capability support

**getcap**

Examines file capabilities

**getpcaps**

Displays the capabilities on the queried process(es)

`libcap`

Contains the library functions for manipulating POSIX 1003.1e capabilities

**setcap**

Sets file capabilities

Last updated on

## 6.27. Sed-4.4

The Sed package contains a stream editor.

**Approximate build time:**0.3 SBU

**Required disk space:**25 MB

### 6.27.1. Installation of Sed

First fix an issue in the LFS environment and remove a failing test:

    sed -i 's/usr/tools/'                 build-aux/help2man
    sed -i 's/testsuite.panic-tests.sh//' Makefile.in

Prepare Sed for compilation:

    ./configure --prefix=/usr --bindir=/bin

Compile the package and generate the HTML documentation:

    make
    make html

To test the results, issue:

    make check

Install the package and its documentation:

    make install
    install -d -m755           /usr/share/doc/sed-4.4
    install -m644 doc/sed.html /usr/share/doc/sed-4.4

### 6.27.2. Contents of Sed

**Installed program:**sed

**Installed directory:**/usr/share/doc/sed-4.4

#### Short Descriptions

**sed**

Filters and transforms text files in a single pass

Last updated on

## 6.28. Shadow-4.5

The Shadow package contains programs for handling passwords in a secure way.

**Approximate build time:**0.2 SBU

**Required disk space:**42 MB

### 6.28.1. Installation of Shadow

### Note

If you would like to enforce the use of strong passwords, refer to [http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/cracklib.html](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/cracklib.html) for installing CrackLib prior to building Shadow. Then add *`--with-libcrack`* to the **configure** command below.

Disable the installation of the **groups** program and its man pages, as Coreutils provides a better version. Also Prevent the installation of manual pages that were already installed by the [`man pages`](Linux%20From%20Scratch.html#man-pages) package:

    sed -i 's/groups$(EXEEXT) //' src/Makefile.in
    find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
    find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
    find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;

Instead of using the default *crypt* method, use the more secure *SHA-512* method of password encryption, which also allows passwords longer than 8 characters. It is also necessary to change the obsolete `/var/spool/mail` location for user mailboxes that Shadow uses by default to the `/var/mail` location used currently:

    sed -i -e 's@#ENCRYPT_METHOD DES@ENCRYPT_METHOD SHA512@' \
           -e 's@/var/spool/mail@/var/mail@' etc/login.defs

### Note

If you chose to build Shadow with Cracklib support, run the following:

    sed -i 's@DICTPATH.*@DICTPATH\t/lib/cracklib/pw_dict@' etc/login.defs

Make a minor change to make the default useradd consistent with the LFS groups file:

    sed -i 's/1000/999/' etc/useradd

Prepare Shadow for compilation:

    ./configure --sysconfdir=/etc --with-group-name-max-length=32

**The meaning of the configure option:**
*`--with-group-name-max-length=32`*
The maximum user name is 32 characters. Make the maximum group name the same.

Compile the package:

    make

This package does not come with a test suite.

Install the package:

    make install

Move a misplaced program to its proper location:

    mv -v /usr/bin/passwd /bin

### 6.28.2. Configuring Shadow

This package contains utilities to add, modify, and delete users and groups; set and change their passwords; and perform other administrative tasks. For a full explanation of what *password shadowing* means, see the `doc/HOWTO` file within the unpacked source tree. If using Shadow support, keep in mind that programs which need to verify passwords (display managers, FTP programs, pop3 daemons, etc.) must be Shadow-compliant. That is, they need to be able to work with shadowed passwords.

To enable shadowed passwords, run the following command:

    pwconv

To enable shadowed group passwords, run:

    grpconv

Shadow's stock configuration for the **useradd** utility has a few caveats that need some explanation. First, the default action for the **useradd**utility is to create the user and a group of the same name as the user. By default the user ID (UID) and group ID (GID) numbers will begin with 1000. This means if you don't pass parameters to **useradd**, each user will be a member of a unique group on the system. If this behavior is undesirable, you'll need to pass the *`-g`* parameter to **useradd**. The default parameters are stored in the `/etc/default/useradd` file. You may need to modify two parameters in this file to suit your particular needs.

**`/etc/default/useradd` Parameter Explanations**
*`GROUP=1000`*
This parameter sets the beginning of the group numbers used in the /etc/group file. You can modify it to anything you desire. Note that **useradd** will never reuse a UID or GID. If the number identified in this parameter is used, it will use the next available number after this. Note also that if you don't have a group 1000 on your system the first time you use **useradd** without the *`-g`* parameter, you'll get a message displayed on the terminal that says: `useradd: unknown GID 1000`. You may disregard this message and group number 1000 will be used.
*`CREATE_MAIL_SPOOL=yes`*
This parameter causes **useradd** to create a mailbox file for the newly created user. **useradd** will make the group ownership of this file to the `mail` group with 0660 permissions. If you would prefer that these mailbox files are not created by **useradd**, issue the following command:

    sed -i 's/yes/no/' /etc/default/useradd

### 6.28.3. Setting the root password

Choose a password for user *root* and set it by running:

    passwd root

### 6.28.4. Contents of Shadow

**Installed programs:**chage, chfn, chgpasswd, chpasswd, chsh, expiry, faillog, gpasswd, groupadd, groupdel, groupmems, groupmod, grpck, grpconv, grpunconv, lastlog, login, logoutd, newgidmap, newgrp, newuidmap, newusers, nologin, passwd, pwck, pwconv, pwunconv, sg (link to newgrp), su, useradd, userdel, usermod, vigr (link to vipw), and vipw

**Installed directory:**/etc/default

#### Short Descriptions

**chage**

Used to change the maximum number of days between obligatory password changes

**chfn**

Used to change a user's full name and other information

**chgpasswd**

Used to update group passwords in batch mode

**chpasswd**

Used to update user passwords in batch mode

**chsh**

Used to change a user's default login shell

**expiry**

Checks and enforces the current password expiration policy

**faillog**

Is used to examine the log of login failures, to set a maximum number of failures before an account is blocked, or to reset the failure count

**gpasswd**

Is used to add and delete members and administrators to groups

**groupadd**

Creates a group with the given name

**groupdel**

Deletes the group with the given name

**groupmems**

Allows a user to administer his/her own group membership list without the requirement of super user privileges.

**groupmod**

Is used to modify the given group's name or GID

**grpck**

Verifies the integrity of the group files `/etc/group` and `/etc/gshadow`

**grpconv**

Creates or updates the shadow group file from the normal group file

**grpunconv**

Updates `/etc/group` from `/etc/gshadow` and then deletes the latter

**lastlog**

Reports the most recent login of all users or of a given user

**login**

Is used by the system to let users sign on

**logoutd**

Is a daemon used to enforce restrictions on log-on time and ports

**newgidmap**

Is used to set the gid mapping of a user namespace

**newgrp**

Is used to change the current GID during a login session

**newuidmap**

Is used to set the uid mapping of a user namespace

**newusers**

Is used to create or update an entire series of user accounts

**nologin**

Displays a message that an account is not available; it is designed to be used as the default shell for accounts that have been disabled

**passwd**

Is used to change the password for a user or group account

**pwck**

Verifies the integrity of the password files `/etc/passwd` and `/etc/shadow`

**pwconv**

Creates or updates the shadow password file from the normal password file

**pwunconv**

Updates `/etc/passwd` from `/etc/shadow` and then deletes the latter

**sg**

Executes a given command while the user's GID is set to that of the given group

**su**

Runs a shell with substitute user and group IDs

**useradd**

Creates a new user with the given name, or updates the default new-user information

**userdel**

Deletes the given user account

**usermod**

Is used to modify the given user's login name, User Identification (UID), shell, initial group, home directory, etc.

**vigr**

Edits the `/etc/group` or `/etc/gshadow` files

**vipw**

Edits the `/etc/passwd` or `/etc/shadow` files

Last updated on

## 6.29. Psmisc-23.1

The Psmisc package contains programs for displaying information about running processes.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**4.2 MB

### 6.29.1. Installation of Psmisc

Prepare Psmisc for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

This package does not come with a test suite.

Install the package:

    make install

Finally, move the **killall** and **fuser** programs to the location specified by the FHS:

    mv -v /usr/bin/fuser   /bin
    mv -v /usr/bin/killall /bin

### 6.29.2. Contents of Psmisc

**Installed programs:**fuser, killall, peekfd, prtstat, pstree, and pstree.x11 (link to pstree)

#### Short Descriptions

**fuser**

Reports the Process IDs (PIDs) of processes that use the given files or file systems

**killall**

Kills processes by name; it sends a signal to all processes running any of the given commands

**peekfd**

Peek at file descriptors of a running process, given its PID

**prtstat**

Prints information about a process

**pstree**

Displays running processes as a tree

**pstree.x11**

Same as **pstree**, except that it waits for confirmation before exiting

Last updated on

## 6.30. Iana-Etc-2.30

The Iana-Etc package provides data for network services and protocols.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**2.3 MB

### 6.30.1. Installation of Iana-Etc

The following command converts the raw data provided by IANA into the correct formats for the `/etc/protocols` and `/etc/services` data files:

    make

This package does not come with a test suite.

Install the package:

    make install

### 6.30.2. Contents of Iana-Etc

**Installed files:**/etc/protocols and /etc/services

#### Short Descriptions

`/etc/protocols`

Describes the various DARPA Internet protocols that are available from the TCP/IP subsystem

`/etc/services`

Provides a mapping between friendly textual names for internet services, and their underlying assigned port numbers and protocol types

Last updated on

## 6.31. Bison-3.0.4

The Bison package contains a parser generator.

**Approximate build time:**0.3 SBU

**Required disk space:**32 MB

### 6.31.1. Installation of Bison

Prepare Bison for compilation:

    ./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.0.4

Compile the package:

    make

There is a circular dependency between bison and flex with regard to the checks. If desired, after installing flex in the next section, the bison checks can be run with **make check**. Three tests related to lalr1.cc fail for unknown reasons.

Install the package:

    make install

### 6.31.2. Contents of Bison

**Installed programs:**bison and yacc

**Installed library:**liby.a

**Installed directory:**/usr/share/bison

#### Short Descriptions

**bison**

Generates, from a series of rules, a program for analyzing the structure of text files; Bison is a replacement for Yacc (Yet Another Compiler Compiler)

**yacc**

A wrapper for **bison**, meant for programs that still call **yacc** instead of **bison**; it calls **bison** with the *`-y`* option

`liby`

The Yacc library containing implementations of Yacc-compatible `yyerror` and `main` functions; this library is normally not very useful, but POSIX requires it

Last updated on

## 6.32. Flex-2.6.4

The Flex package contains a utility for generating programs that recognize patterns in text.

**Approximate build time:**0.4 SBU

**Required disk space:**32 MB

### 6.32.1. Installation of Flex

First, fix a problem introduced with glibc-2.26:

    sed -i "/math.h/a #include <malloc.h>" src/flexdef.h

The build procedure assumes the help2man program is available to create a man page from the executable --help option. This is not present, so we use an environment variable to skip this process. Now, prepare Flex for compilation:

    HELP2MAN=/tools/bin/true \
    ./configure --prefix=/usr --docdir=/usr/share/doc/flex-2.6.4

Compile the package:

    make

To test the results (about 0.5 SBU), issue:

    make check

Install the package:

    make install

A few programs do not know about **flex** yet and try to run its predecessor, **lex**. To support those programs, create a symbolic link named `lex`that runs `flex` in **lex** emulation mode:

    ln -sv flex /usr/bin/lex

### 6.32.2. Contents of Flex

**Installed programs:**flex, flex++ (link to flex), and lex (link to flex)

**Installed libraries:**libfl.so and libfl_pic.so

**Installed directory:**/usr/share/doc/flex-2.6.4

#### Short Descriptions

**flex**

A tool for generating programs that recognize patterns in text; it allows for the versatility to specify the rules for pattern-finding, eradicating the need to develop a specialized program

**flex++**

An extension of flex, is used for generating C++ code and classes. It is a symbolic link to **flex**

**lex**

A script that runs **flex** in **lex** emulation mode

`libfl`

The `flex` library

Last updated on

## 6.33. Grep-3.1

The Grep package contains programs for searching through files.

**Approximate build time:**0.4 SBU

**Required disk space:**29 MB

### 6.33.1. Installation of Grep

Prepare Grep for compilation:

    ./configure --prefix=/usr --bindir=/bin

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.33.2. Contents of Grep

**Installed programs:**egrep, fgrep, and grep

#### Short Descriptions

**egrep**

Prints lines matching an extended regular expression

**fgrep**

Prints lines matching a list of fixed strings

**grep**

Prints lines matching a basic regular expression

Last updated on

## 6.34. Bash-4.4

The Bash package contains the Bourne-Again SHell.

**Approximate build time:**2.0 SBU

**Required disk space:**56 MB

### 6.34.1. Installation of Bash

Incorporate some upstream fixes:

    patch -Np1 -i ../bash-4.4-upstream_fixes-1.patch

Prepare Bash for compilation:

    ./configure --prefix=/usr                       \
                --docdir=/usr/share/doc/bash-4.4 \
                --without-bash-malloc               \
                --with-installed-readline

**The meaning of the new configure option:**
*`--with-installed-readline`*
This option tells Bash to use the `readline` library that is already installed on the system rather than using its own readline version.

Compile the package:

    make

Skip down to “Install the package” if not running the test suite.

To prepare the tests, ensure that the `nobody` user can write to the sources tree:

    chown -Rv nobody .

Now, run the tests as the `nobody` user:

    su nobody -s /bin/bash -c "PATH=$PATH make tests"

Install the package and move the main executable to `/bin`:

    make install
    mv -vf /usr/bin/bash /bin

Run the newly compiled **bash** program (replacing the one that is currently being executed):

    exec /bin/bash --login +h

### Note

The parameters used make the **bash** process an interactive login shell and continue to disable hashing so that new programs are found as they become available.

### 6.34.2. Contents of Bash

**Installed programs:**bash, bashbug, and sh (link to bash)

**Installed directory:**/usr/share/doc/bash-4.4

#### Short Descriptions

**bash**

A widely-used command interpreter; it performs many types of expansions and substitutions on a given command line before executing it, thus making this interpreter a powerful tool

**bashbug**

A shell script to help the user compose and mail standard formatted bug reports concerning **bash**

**sh**

A symlink to the **bash** program; when invoked as **sh**, **bash** tries to mimic the startup behavior of historical versions of **sh**as closely as possible, while conforming to the POSIX standard as well

Last updated on

## 6.35. Libtool-2.4.6

The Libtool package contains the GNU generic library support script. It wraps the complexity of using shared libraries in a consistent, portable interface.

**Approximate build time:**1.8 SBU

**Required disk space:**43 MB

### 6.35.1. Installation of Libtool

Prepare Libtool for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results (about 11.0 SBU), issue:

    make check

### Note

The test time for libtool can ibe reduced significantly on a system with multiple cores. To do this, append **TESTSUITEFLAGS=-j<N>**to the line above. For instance, using -j4 can reduce the test time by over 60 percent.

Five tests are known to fail in the LFS build environment due to a circular dependency, but all tests pass if rechecked after automake is installed.

Install the package:

    make install

### 6.35.2. Contents of Libtool

**Installed programs:**libtool and libtoolize

**Installed libraries:**libltdl.so

**Installed directories:**/usr/include/libltdl and /usr/share/libtool

#### Short Descriptions

**libtool**

Provides generalized library-building support services

**libtoolize**

Provides a standard way to add **libtool** support to a package

`libltdl`

Hides the various difficulties of dlopening libraries

Last updated on

## 6.36. GDBM-1.13

The GDBM package contains the GNU Database Manager. It is a library of database functions that use extensible hashing and work similar to the standard UNIX dbm. The library provides primitives for storing key/data pairs, searching and retrieving the data by its key and deleting a key along with its data.

**Approximate build time:**0.1 SBU

**Required disk space:**10 MB

### 6.36.1. Installation of GDBM

Prepare GDBM for compilation:

    ./configure --prefix=/usr \
                --disable-static \
                --enable-libgdbm-compat

**The meaning of the configure option:**
`--enable-libgdbm-compat`
This switch enables the libgdbm compatibility library to be built, as some packages outside of LFS may require the older DBM routines it provides.

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.36.2. Contents of GDBM

**Installed programs:**gdbm_dump, gdbm_load, and gdbmtool

**Installed libraries:**libgdbm.so and libgdbm_compat.so

#### Short Descriptions

**gdbm_dump**

Dumps a GDBM database to a file

**gdbm_load**

Recreates a GDBM database from a dump file

**gdbmtool**

Tests and modifies a GDBM database

`libgdbm`

Contains functions to manipulate a hashed database

`libgdbm_compat`

Compatibility library containing older DBM functions

Last updated on

## 6.37. Gperf-3.1

Gperf generates a perfect hash function from a key set.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**5.8 MB

### 6.37.1. Installation of Gperf

Prepare Gperf for compilation:

    ./configure --prefix=/usr --docdir=/usr/share/doc/gperf-3.1

Compile the package:

    make

The tests are known to fail if running multiple simultaneous tests (-j option greater than 1). To test the results, issue:

    make -j1 check

Install the package:

    make install

### 6.37.2. Contents of Gperf

**Installed program:**gperf

**Installed directory:**/usr/share/doc/gperf-3.1

#### Short Descriptions

**gperf**

Generates a perfect hash from a key set

Last updated on

## 6.38. Expat-2.2.3

The Expat package contains a stream oriented C library for parsing XML.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**9.5 MB

### 6.38.1. Installation of Expat

First fix a problem with the regession tests in the LFS environment:

    sed -i 's|usr/bin/env |bin/|' run.sh.in

Prepare Expat for compilation:

    ./configure --prefix=/usr --disable-static

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

If desired, install the documentation:

    install -v -dm755 /usr/share/doc/expat-2.2.3
    install -v -m644 doc/*.{html,png,css} /usr/share/doc/expat-2.2.3

### 6.38.2. Contents of Expat

**Installed program:**xmlwf

**Installed libraries:**libexpat.so

**Installed directory:**/usr/share/doc/expat-2.2.3

#### Short Descriptions

**xmlwf**

is a non-validating utility to check whether or not XML documents are well formed

`libexpat`

contains API functions for parsing XML

Last updated on

## 6.39. Inetutils-1.9.4

The Inetutils package contains programs for basic networking.

**Approximate build time:**0.3 SBU

**Required disk space:**27 MB

### 6.39.1. Installation of Inetutils

Prepare Inetutils for compilation:

    ./configure --prefix=/usr        \
                --localstatedir=/var \
                --disable-logger     \
                --disable-whois      \
                --disable-rcp        \
                --disable-rexec      \
                --disable-rlogin     \
                --disable-rsh        \
                --disable-servers

**The meaning of the configure options:**
*`--disable-logger`*
This option prevents Inetutils from installing the **logger** program, which is used by scripts to pass messages to the System Log Daemon. Do not install it because Util-linux installs a more recent version.
*`--disable-whois`*
This option disables the building of the Inetutils **whois** client, which is out of date. Instructions for a better **whois** client are in the BLFS book.
*`--disable-r*`*
These parameters disable building obsolete programs that should not be used due to security issues. The functions provided by these programs can be provided by the openssh package in the BLFS book.
*`--disable-servers`*
This disables the installation of the various network servers included as part of the Inetutils package. These servers are deemed not appropriate in a basic LFS system. Some are insecure by nature and are only considered safe on trusted networks. Note that better replacements are available for many of these servers.

Compile the package:

    make

To test the results, issue:

    make check

### Note

One test, libls.sh, may fail in the initial chroot environment but will pass if the test is rerun after the LFS system is complete.

Install the package:

    make install

Move some programs so they are available if `/usr` is not accessible:

    mv -v /usr/bin/{hostname,ping,ping6,traceroute} /bin
    mv -v /usr/bin/ifconfig /sbin

### 6.39.2. Contents of Inetutils

**Installed programs:**dnsdomainname, ftp, ifconfig, hostname, ping, ping6, talk, telnet, tftp, and traceroute

#### Short Descriptions

**dnsdomainname**

Show the system's DNS domain name

**ftp**

Is the file transfer protocol program

**hostname**

Reports or sets the name of the host

**ifconfig**

Manages network interfaces

**ping**

Sends echo-request packets and reports how long the replies take

**ping6**

A version of **ping** for IPv6 networks

**talk**

Is used to chat with another user

**telnet**

An interface to the TELNET protocol

**tftp**

A trivial file transfer program

**traceroute**

Traces the route your packets take from the host you are working on to another host on a network, showing all the intermediate hops (gateways) along the way

Last updated on

## 6.40. Perl-5.26.0

The Perl package contains the Practical Extraction and Report Language.

**Approximate build time:**8.6 SBU

**Required disk space:**257 MB

### 6.40.1. Installation of Perl

First create a basic `/etc/hosts` file to be referenced in one of Perl's configuration files as well as the optional test suite:

    echo "127.0.0.1 localhost $(hostname)" > /etc/hosts

This version of Perl now builds the Compress::Raw::Zlib and Compress::Raw::BZip2 modules. By default Perl will use an internal copy of the sources for the build. Issue the following command so that Perl will use the libraries installed on the system:

    export BUILD_ZLIB=False
    export BUILD_BZIP2=0

To have full control over the way Perl is set up, you can remove the “-des” options from the following command and hand-pick the way this package is built. Alternatively, use the command exactly as below to use the defaults that Perl auto-detects:

    sh Configure -des -Dprefix=/usr                 \
                      -Dvendorprefix=/usr           \
                      -Dman1dir=/usr/share/man/man1 \
                      -Dman3dir=/usr/share/man/man3 \
                      -Dpager="/usr/bin/less -isR"  \
                      -Duseshrplib                  \
                      -Dusethreads

**The meaning of the configure options:**
*`-Dvendorprefix=/usr`*
This ensures **perl** knows how to tell packages where they should install their perl modules.
*`-Dpager="/usr/bin/less -isR"`*
This ensures that **`less`** is used instead of **`more`**.
*`-Dman1dir=/usr/share/man/man1 -Dman3dir=/usr/share/man/man3`*
Since Groff is not installed yet, **Configure** thinks that we do not want man pages for Perl. Issuing these parameters overrides this decision.
*`-Duseshrplib`*
Build a shared libperl needed by some perl modules.
*`-Dusethreads`*
Build perl with support for threads.

Compile the package:

    make

To test the results (approximately 2.5 SBU), issue:

    make -k test

### Note

Several tests related to zlib will fail due to using the system version of zlib instead of the internal version.

Install the package and clean up:

    make install
    unset BUILD_ZLIB BUILD_BZIP2

### 6.40.2. Contents of Perl

**Installed programs:**c2ph, corelist, cpan, enc2xs, encguess, h2ph, h2xs, instmodsh, json_pp, libnetcfg, perl, perl5.26.0 (hard link to perl), perlbug, perldoc, perlivp, perlthanks (hard link to perlbug), piconv, pl2pm, pod2html, pod2man, pod2text, pod2usage, podchecker, podselect, prove, pstruct (hard link to c2ph), ptar, ptardiff, ptargrep, shasum, splain, xsubpp, and zipdetails

**Installed libraries:**Many which cannot all be listed here

**Installed directory:**/usr/lib/perl5

#### Short Descriptions

**c2ph**

Dumps C structures as generated from **cc -g -S**

**corelist**

A commandline frontend to Module::CoreList

**cpan**

Interact with the Comprehensive Perl Archive Network (CPAN) from the command line

**enc2xs**

Builds a Perl extension for the Encode module from either Unicode Character Mappings or Tcl Encoding Files

**encguess**

Guess the encoding type of one or several files

**h2ph**

Converts `.h` C header files to `.ph` Perl header files

**h2xs**

Converts `.h` C header files to Perl extensions

**instmodsh**

Shell script for examining installed Perl modules, and can even create a tarball from an installed module

**json_pp**

Converts data between certain input and output formats

**libnetcfg**

Can be used to configure the `libnet` Perl module

**perl**

Combines some of the best features of C, **sed**, **awk** and **sh** into a single swiss-army language

**perl5.26.0**

A hard link to **perl**

**perlbug**

Used to generate bug reports about Perl, or the modules that come with it, and mail them

**perldoc**

Displays a piece of documentation in pod format that is embedded in the Perl installation tree or in a Perl script

**perlivp**

The Perl Installation Verification Procedure; it can be used to verify that Perl and its libraries have been installed correctly

**perlthanks**

Used to generate thank you messages to mail to the Perl developers

**piconv**

A Perl version of the character encoding converter **iconv**

**pl2pm**

A rough tool for converting Perl4 `.pl` files to Perl5 `.pm` modules

**pod2html**

Converts files from pod format to HTML format

**pod2man**

Converts pod data to formatted *roff input

**pod2text**

Converts pod data to formatted ASCII text

**pod2usage**

Prints usage messages from embedded pod docs in files

**podchecker**

Checks the syntax of pod format documentation files

**podselect**

Displays selected sections of pod documentation

**prove**

Command line tool for running tests against the Test::Harness module

**pstruct**

Dumps C structures as generated from **cc -g -S** stabs

**ptar**

A **tar**-like program written in Perl

**ptardiff**

A Perl program that compares an extracted archive with an unextracted one

**ptargrep**

A Perl program that applies pattern matching to the contents of files in a tar archive

**shasum**

Prints or checks SHA checksums

**splain**

Is used to force verbose warning diagnostics in Perl

**xsubpp**

Converts Perl XS code into C code

**zipdetails**

Displays details about the internal structure of a Zip file

Last updated on

## 6.41. XML::Parser-2.44

The XML::Parser module is a Perl interface to James Clark's XML parser, Expat.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**2.0 MB

### 6.41.1. Installation of XML::Parser

Prepare XML::Parser for compilation:

    perl Makefile.PL

Compile the package:

    make

To test the results, issue:

    make test

Install the package:

    make install

### 6.41.2. Contents of XML::Parser

**Installed module:**Expat.so

#### Short Descriptions

`Expat`

provides the Perl Expat interface

Last updated on

## 6.42. Intltool-0.51.0

The Intltool is an internationalization tool used for extracting translatable strings from source files.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**1.5 MB

### 6.42.1. Installation of Intltool

First fix a warning that is caused by perl-5.22 and later:

    sed -i 's:\\\${:\\\$\\{:' intltool-update.in

Prepare Intltool for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install
    install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO

### 6.42.2. Contents of Intltool

**Installed programs:**intltool-extract, intltool-merge, intltool-prepare, intltool-update, and intltoolize

**Installed directories:**/usr/share/doc/intltool-0.51.0 and /usr/share/intltool

#### Short Descriptions

**intltoolize**

Prepares a package to use intltool

**intltool-extract**

Generates header files that can be read by **gettext**

**intltool-merge**

Merges translated strings into various file types

**intltool-prepare**

Updates pot files and merges them with translation files

**intltool-update**

Updates the po template files and merges them with the translations

Last updated on

## 6.43. Autoconf-2.69

The Autoconf package contains programs for producing shell scripts that can automatically configure source code.

**Approximate build time:**less than 0.1 SBU (about 3.3 SBU with tests)

**Required disk space:**17.3 MB

### 6.43.1. Installation of Autoconf

Prepare Autoconf for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

This takes a long time, about 3.3 SBUs. In addition, several tests are skipped that use Automake. For full test coverage, Autoconf can be re-tested after Automake has been installed. In addition, two tests fail due to changes in libtool-2.4.3 and later.

### Note

The test time for autoconf can be reduced significantly on a system with multiple cores. To do this, append **TESTSUITEFLAGS=-j<N>**to the line above. For instance, using -j4 can reduce the test time by over 60 percent.

Install the package:

    make install

### 6.43.2. Contents of Autoconf

**Installed programs:**autoconf, autoheader, autom4te, autoreconf, autoscan, autoupdate, and ifnames

**Installed directory:**/usr/share/autoconf

#### Short Descriptions

**autoconf**

Produces shell scripts that automatically configure software source code packages to adapt to many kinds of Unix-like systems; the configuration scripts it produces are independent—running them does not require the **autoconf** program

**autoheader**

A tool for creating template files of C *#define* statements for configure to use

**autom4te**

A wrapper for the M4 macro processor

**autoreconf**

Automatically runs **autoconf**, **autoheader**, **aclocal**, **automake**, **gettextize**, and **libtoolize** in the correct order to save time when changes are made to **autoconf** and **automake** template files

**autoscan**

Helps to create a `configure.in` file for a software package; it examines the source files in a directory tree, searching them for common portability issues, and creates a `configure.scan` file that serves as as a preliminary `configure.in` file for the package

**autoupdate**

Modifies a `configure.in` file that still calls **autoconf** macros by their old names to use the current macro names

**ifnames**

Helps when writing `configure.in` files for a software package; it prints the identifiers that the package uses in C preprocessor conditionals [If a package has already been set up to have some portability, this program can help determine what **configure** needs to check for. It can also fill in gaps in a `configure.in` file generated by **autoscan**.]

Last updated on

## 6.44. Automake-1.15.1

The Automake package contains programs for generating Makefiles for use with Autoconf.

**Approximate build time:**less than 0.1 SBU (about 8.5 SBU with tests)

**Required disk space:**110 MB

### 6.44.1. Installation of Automake

Prepare Automake for compilation:

    ./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.15.1

Compile the package:

    make

There are a couple of tests that incorrectly link to the wrong version of the flex library, so we temporarily work around the problem. Also, using the -j4 make option speeds up the tests, even on systems with only one processor, due to internal delays in individual tests. To test the results, issue:

    sed -i "s:./configure:LEXLIB=/usr/lib/libfl.a &:" t/lex-{clean,depend}-cxx.sh
    make -j4 check

Three tests are known to fail in the LFS environment: check12.sh, subobj.sh, and check12-w.sh.

Install the package:

    make install

### 6.44.2. Contents of Automake

**Installed programs:**aclocal, aclocal-1.15 (hard linked with aclocal), automake, and automake-1.15 (hard linked with automake)

**Installed directories:**/usr/share/aclocal-1.15, /usr/share/automake-1.15, and /usr/share/doc/automake-1.15.1

#### Short Descriptions

**aclocal**

Generates `aclocal.m4` files based on the contents of `configure.in` files

**aclocal-1.15**

A hard link to **aclocal**

**automake**

A tool for automatically generating `Makefile.in` files from `Makefile.am` files [To create all the `Makefile.in` files for a package, run this program in the top-level directory. By scanning the `configure.in` file, it automatically finds each appropriate `Makefile.am`file and generates the corresponding `Makefile.in` file.]

**automake-1.15**

A hard link to **automake**

Last updated on

## 6.45. Xz-5.2.3

The Xz package contains programs for compressing and decompressing files. It provides capabilities for the lzma and the newer xz compression formats. Compressing text files with **xz** yields a better compression percentage than with the traditional **gzip** or **bzip2**commands.

**Approximate build time:**0.2 SBU

**Required disk space:**15 MB

### 6.45.1. Installation of Xz

Prepare Xz for compilation with:

    ./configure --prefix=/usr    \
                --disable-static \
                --docdir=/usr/share/doc/xz-5.2.3

Compile the package:

    make

To test the results, issue:

    make check

Install the package and make sure that all essential files are in the correct directory:

    make install
    mv -v   /usr/bin/{lzma,unlzma,lzcat,xz,unxz,xzcat} /bin
    mv -v /usr/lib/liblzma.so.* /lib
    ln -svf ../../lib/$(readlink /usr/lib/liblzma.so) /usr/lib/liblzma.so

### 6.45.2. Contents of Xz

**Installed programs:**lzcat (link to xz), lzcmp (link to xzdiff), lzdiff (link to xzdiff), lzegrep (link to xzgrep), lzfgrep (link to xzgrep), lzgrep (link to xzgrep), lzless (link to xzless), lzma (link to xz), lzmadec, lzmainfo, lzmore (link to xzmore), unlzma (link to xz), unxz (link to xz), xz, xzcat (link to xz), xzcmp (link to xzdiff), xzdec, xzdiff, xzegrep (link to xzgrep), xzfgrep (link to xzgrep), xzgrep, xzless, and xzmore

**Installed libraries:**liblzma.so

**Installed directories:**/usr/include/lzma and /usr/share/doc/xz-5.2.3

#### Short Descriptions

**lzcat**

Decompresses to standard output

**lzcmp**

Runs **cmp** on LZMA compressed files

**lzdiff**

Runs **diff** on LZMA compressed files

**lzegrep**

Runs **egrep** on LZMA compressed files

**lzfgrep**

Runs **fgrep** on LZMA compressed files

**lzgrep**

Runs **grep** on LZMA compressed files

**lzless**

Runs **less** on LZMA compressed files

**lzma**

Compresses or decompresses files using the LZMA format

**lzmadec**

A small and fast decoder for LZMA compressed files

**lzmainfo**

Shows information stored in the LZMA compressed file header

**lzmore**

Runs **more** on LZMA compressed files

**unlzma**

Decompresses files using the LZMA format

**unxz**

Decompresses files using the XZ format

**xz**

Compresses or decompresses files using the XZ format

**xzcat**

Decompresses to standard output

**xzcmp**

Runs **cmp** on XZ compressed files

**xzdec**

A small and fast decoder for XZ compressed files

**xzdiff**

Runs **diff** on XZ compressed files

**xzegrep**

Runs **egrep** on XZ compressed files files

**xzfgrep**

Runs **fgrep** on XZ compressed files

**xzgrep**

Runs **grep** on XZ compressed files

**xzless**

Runs **less** on XZ compressed files

**xzmore**

Runs **more** on XZ compressed files

`liblzma`

The library implementing lossless, block-sorting data compression, using the Lempel-Ziv-Markov chain algorithm

Last updated on

## 6.46. Kmod-24

The Kmod package contains libraries and utilities for loading kernel modules

**Approximate build time:**0.1 SBU

**Required disk space:**12 MB

### 6.46.1. Installation of Kmod

Prepare Kmod for compilation:

    ./configure --prefix=/usr          \
                --bindir=/bin          \
                --sysconfdir=/etc      \
                --with-rootlibdir=/lib \
                --with-xz              \
                --with-zlib

**The meaning of the configure options:**
*`--with-xz, --with-zlib`*
These options enable Kmod to handle compressed kernel modules.
*`--with-rootlibdir=/lib`*
This option ensures different library related files are placed in the correct directories.

Compile the package:

    make

This package does not come with a test suite that can be run in the LFS chroot environment. At a minimum the git program is required and several tests will not run outside of a git repository.

Install the package, and create symlinks for compatibility with Module-Init-Tools (the package that previously handled Linux kernel modules):

    make install

    for target in depmod insmod lsmod modinfo modprobe rmmod; do
      ln -sfv ../bin/kmod /sbin/$target
    done

    ln -sfv kmod /bin/lsmod

### 6.46.2. Contents of Kmod

**Installed programs:**depmod (link to kmod), insmod (link to kmod), kmod, lsmod (link to kmod), modinfo (link to kmod), modprobe (link to kmod), and rmmod (link to kmod)

**Installed library:**libkmod.so

#### Short Descriptions

**depmod**

Creates a dependency file based on the symbols it finds in the existing set of modules; this dependency file is used by **modprobe** to automatically load the required modules

**insmod**

Installs a loadable module in the running kernel

**kmod**

Loads and unloads kernel modules

**lsmod**

Lists currently loaded modules

**modinfo**

Examines an object file associated with a kernel module and displays any information that it can glean

**modprobe**

Uses a dependency file, created by **depmod**, to automatically load relevant modules

**rmmod**

Unloads modules from the running kernel

`libkmod`

This library is used by other programs to load and unload kernel modules

Last updated on

## 6.47. Gettext-0.19.8.1

The Gettext package contains utilities for internationalization and localization. These allow programs to be compiled with NLS (Native Language Support), enabling them to output messages in the user's native language.

**Approximate build time:**2.4 SBU

**Required disk space:**199 MB

### 6.47.1. Installation of Gettext

First, suppress two invocations of test-lock which on some machines can loop forever:

    sed -i '/^TESTS =/d' gettext-runtime/tests/Makefile.in &&
    sed -i 's/test-lock..EXEEXT.//' gettext-tools/gnulib-tests/Makefile.in

Prepare Gettext for compilation:

    ./configure --prefix=/usr    \
                --disable-static \
                --docdir=/usr/share/doc/gettext-0.19.8.1

Compile the package:

    make

To test the results (this takes a long time, around 3 SBUs), issue:

    make check

Install the package:

    make install
    chmod -v 0755 /usr/lib/preloadable_libintl.so

### 6.47.2. Contents of Gettext

**Installed programs:**autopoint, envsubst, gettext, gettext.sh, gettextize, msgattrib, msgcat, msgcmp, msgcomm, msgconv, msgen, msgexec, msgfilter, msgfmt, msggrep, msginit, msgmerge, msgunfmt, msguniq, ngettext, recode-sr-latin, and xgettext

**Installed libraries:**libasprintf.so, libgettextlib.so, libgettextpo.so, libgettextsrc.so, and preloadable_libintl.so

**Installed directories:**/usr/lib/gettext, /usr/share/doc/gettext-0.19.8.1, and /usr/share/gettext

#### Short Descriptions

**autopoint**

Copies standard Gettext infrastructure files into a source package

**envsubst**

Substitutes environment variables in shell format strings

**gettext**

Translates a natural language message into the user's language by looking up the translation in a message catalog

**gettext.sh**

Primarily serves as a shell function library for gettext

**gettextize**

Copies all standard Gettext files into the given top-level directory of a package to begin internationalizing it

**msgattrib**

Filters the messages of a translation catalog according to their attributes and manipulates the attributes

**msgcat**

Concatenates and merges the given `.po` files

**msgcmp**

Compares two `.po` files to check that both contain the same set of msgid strings

**msgcomm**

Finds the messages that are common to the given `.po` files

**msgconv**

Converts a translation catalog to a different character encoding

**msgen**

Creates an English translation catalog

**msgexec**

Applies a command to all translations of a translation catalog

**msgfilter**

Applies a filter to all translations of a translation catalog

**msgfmt**

Generates a binary message catalog from a translation catalog

**msggrep**

Extracts all messages of a translation catalog that match a given pattern or belong to some given source files

**msginit**

Creates a new `.po` file, initializing the meta information with values from the user's environment

**msgmerge**

Combines two raw translations into a single file

**msgunfmt**

Decompiles a binary message catalog into raw translation text

**msguniq**

Unifies duplicate translations in a translation catalog

**ngettext**

Displays native language translations of a textual message whose grammatical form depends on a number

**recode-sr-latin**

Recodes Serbian text from Cyrillic to Latin script

**xgettext**

Extracts the translatable message lines from the given source files to make the first translation template

`libasprintf`

defines the *autosprintf* class, which makes C formatted output routines usable in C++ programs, for use with the *<string>* strings and the *<iostream>* streams

`libgettextlib`

a private library containing common routines used by the various Gettext programs; these are not intended for general use

`libgettextpo`

Used to write specialized programs that process `.po` files; this library is used when the standard applications shipped with Gettext (such as **msgcomm**, **msgcmp**, **msgattrib**, and **msgen**) will not suffice

`libgettextsrc`

A private library containing common routines used by the various Gettext programs; these are not intended for general use

`preloadable_libintl`

A library, intended to be used by LD_PRELOAD that assists `libintl` in logging untranslated messages

Last updated on

## 6.48. Procps-ng-3.3.12

The Procps-ng package contains programs for monitoring processes.

**Approximate build time:**0.1 SBU

**Required disk space:**14 MB

### 6.48.1. Installation of Procps-ng

Now prepare procps-ng for compilation:

    ./configure --prefix=/usr                            \
                --exec-prefix=                           \
                --libdir=/usr/lib                        \
                --docdir=/usr/share/doc/procps-ng-3.3.12 \
                --disable-static                         \
                --disable-kill

**The meaning of the configure options:**
*`--disable-kill`*
This switch disables building the **kill** command that will be installed by the Util-linux package.

Compile the package:

    make

The test suite needs some custom modifications for LFS. Remove a test that fails when scripting does not use a tty device and fix two others. To run the test suite, run the following commands:

    sed -i -r 's|(pmap_initname)\\\$|\1|' testsuite/pmap.test/pmap.exp
    sed -i '/set tty/d' testsuite/pkill.test/pkill.exp
    rm testsuite/pgrep.test/pgrep.exp
    make check

One ps test may fail, but passes if the tests are rerun at the end of Chapter 6.

Install the package:

    make install

Finally, move essential libraries to a location that can be found if `/usr` is not mounted.

    mv -v /usr/lib/libprocps.so.* /lib
    ln -sfv ../../lib/$(readlink /usr/lib/libprocps.so) /usr/lib/libprocps.so

### 6.48.2. Contents of Procps-ng

**Installed programs:**free, pgrep, pidof, pkill, pmap, ps, pwdx, slabtop, sysctl, tload, top, uptime, vmstat, w, and watch

**Installed library:**libprocps.so

**Installed directories:**/usr/include/proc and /usr/share/doc/procps-ng-3.3.12

#### Short Descriptions

**free**

Reports the amount of free and used memory (both physical and swap memory) in the system

**pgrep**

Looks up processes based on their name and other attributes

**pidof**

Reports the PIDs of the given programs

**pkill**

Signals processes based on their name and other attributes

**pmap**

Reports the memory map of the given process

**ps**

Lists the current running processes

**pwdx**

Reports the current working directory of a process

**slabtop**

Displays detailed kernel slap cache information in real time

**sysctl**

Modifies kernel parameters at run time

**tload**

Prints a graph of the current system load average

**top**

Displays a list of the most CPU intensive processes; it provides an ongoing look at processor activity in real time

**uptime**

Reports how long the system has been running, how many users are logged on, and the system load averages

**vmstat**

Reports virtual memory statistics, giving information about processes, memory, paging, block Input/Output (IO), traps, and CPU activity

**w**

Shows which users are currently logged on, where, and since when

**watch**

Runs a given command repeatedly, displaying the first screen-full of its output; this allows a user to watch the output change over time

`libprocps`

Contains the functions used by most programs in this package

Last updated on

## 6.49. E2fsprogs-1.43.5

The E2fsprogs package contains the utilities for handling the `ext2` file system. It also supports the `ext3` and `ext4` journaling file systems.

**Approximate build time:**3.3 SBU

**Required disk space:**58 MB

### 6.49.1. Installation of E2fsprogs

The E2fsprogs documentation recommends that the package be built in a subdirectory of the source tree:

    mkdir -v build
    cd build

Prepare E2fsprogs for compilation:

    LIBS=-L/tools/lib                    \
    CFLAGS=-I/tools/include              \
    PKG_CONFIG_PATH=/tools/lib/pkgconfig \
    ../configure --prefix=/usr           \
                 --bindir=/bin           \
                 --with-root-prefix=""   \
                 --enable-elf-shlibs     \
                 --disable-libblkid      \
                 --disable-libuuid       \
                 --disable-uuidd         \
                 --disable-fsck

**The meaning of the environment variable and configure options:**
*`PKG_CONFIG_PATH, LIBS, CFLAGS`*
These variables enable e2fsprogs to be built using the [Section 5.34, “Util-linux-2.30.1”](Linux%20From%20Scratch.html#ch-tools-util-linux) package built earlier.
*`--with-root-prefix=""`* and *`--bindir=/bin`*
Certain programs (such as the **e2fsck** program) are considered essential programs. When, for example, `/usr` is not mounted, these programs still need to be available. They belong in directories like `/lib` and `/sbin`. If this option is not passed to E2fsprogs' configure, the programs are installed into the `/usr` directory.
*`--enable-elf-shlibs`*
This creates the shared libraries which some programs in this package use.
*`--disable-*`*
This prevents E2fsprogs from building and installing the `libuuid` and `libblkid` libraries, the `uuidd` daemon, and the **fsck** wrapper, as Util-Linux installs more recent versions.

Compile the package:

    make

To set up and run the test suite we need to first link some libraries from /tools/lib to a location where the test programs look. To run the tests, issue:

    ln -sfv /tools/lib/lib{blk,uu}id.so.1 lib
    make LD_LIBRARY_PATH=/tools/lib check

One of the E2fsprogs tests will attempt to allocate 256 MB of memory. If you do not have significantly more RAM than this, be sure to enable sufficient swap space for the test. See [Section 2.5, “Creating a File System on the Partition”](Linux%20From%20Scratch.html#space-creatingfilesystem) and [Section 2.7, “Mounting the New Partition”](Linux%20From%20Scratch.html#space-mounting) for details on creating and enabling swap space.

Install the binaries, documentation, and shared libraries:

    make install

Install the static libraries and headers:

    make install-libs

Make the installed static libraries writable so debugging symbols can be removed later:

    chmod -v u+w /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a

This package installs a gzipped `.info` file but doesn't update the system-wide `dir` file. Unzip this file and then update the system `dir` file using the following commands.

    gunzip -v /usr/share/info/libext2fs.info.gz
    install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info

If desired, create and install some additional documentation by issuing the following commands:

    makeinfo -o      doc/com_err.info ../lib/et/com_err.texinfo
    install -v -m644 doc/com_err.info /usr/share/info
    install-info --dir-file=/usr/share/info/dir /usr/share/info/com_err.info

### 6.49.2. Contents of E2fsprogs

**Installed programs:**badblocks, chattr, compile_et, debugfs, dumpe2fs,e2freefrag, e2fsck, e2image, e2label, e2undo, e4defrag, filefrag, fsck.ext2, fsck.ext3, fsck.ext4, fsck.ext4dev, logsave, lsattr, mk_cmds, mke2fs, mkfs.ext2, mkfs.ext3, mkfs.ext4, mkfs.ext4dev, mklost+found, resize2fs, and tune2fs

**Installed libraries:**libcom_err.so, libe2p.so, libext2fs.so, and libss.so

**Installed directories:**/usr/include/e2p, /usr/include/et, /usr/include/ext2fs, /usr/include/ss, /usr/share/et, and /usr/share/ss

#### Short Descriptions

**badblocks**

Searches a device (usually a disk partition) for bad blocks

**chattr**

Changes the attributes of files on an `ext2` file system; it also changes `ext3` file systems, the journaling version of `ext2`file systems

**compile_et**

An error table compiler; it converts a table of error-code names and messages into a C source file suitable for use with the `com_err` library

**debugfs**

A file system debugger; it can be used to examine and change the state of an `ext2` file system

**dumpe2fs**

Prints the super block and blocks group information for the file system present on a given device

**e2freefrag**

Reports free space fragmentation information

**e2fsck**

Is used to check, and optionally repair `ext2` file systems and `ext3` file systems

**e2image**

Is used to save critical `ext2` file system data to a file

**e2label**

Displays or changes the file system label on the `ext2` file system present on a given device

**e2undo**

Replays the undo log undo_log for an ext2/ext3/ext4 filesystem found on a device [This can be used to undo a failed operation by an e2fsprogs program.]

**e4defrag**

Online defragmenter for ext4 filesystems

**filefrag**

Reports on how badly fragmented a particular file might be

**fsck.ext2**

By default checks `ext2` file systems and is a hard link to **e2fsck**

**fsck.ext3**

By default checks `ext3` file systems and is a hard link to **e2fsck**

**fsck.ext4**

By default checks `ext4` file systems and is a hard link to **e2fsck**

**fsck.ext4dev**

By default checks `ext4` development file systems and is a hard link to **e2fsck**

**logsave**

Saves the output of a command in a log file

**lsattr**

Lists the attributes of files on a second extended file system

**mk_cmds**

Converts a table of command names and help messages into a C source file suitable for use with the `libss` subsystem library

**mke2fs**

Creates an `ext2` or `ext3` file system on the given device

**mkfs.ext2**

By default creates `ext2` file systems and is a hard link to **mke2fs**

**mkfs.ext3**

By default creates `ext3` file systems and is a hard link to **mke2fs**

**mkfs.ext4**

By default creates `ext4` file systems and is a hard link to **mke2fs**

**mkfs.ext4dev**

By default creates `ext4` development file systems and is a hard link to **mke2fs**

**mklost+found**

Used to create a `lost+found` directory on an `ext2` file system; it pre-allocates disk blocks to this directory to lighten the task of **e2fsck**

**resize2fs**

Can be used to enlarge or shrink an `ext2` file system

**tune2fs**

Adjusts tunable file system parameters on an `ext2` file system

`libcom_err`

The common error display routine

`libe2p`

Used by **dumpe2fs**, **chattr**, and **lsattr**

`libext2fs`

Contains routines to enable user-level programs to manipulate an `ext2` file system

`libss`

Used by **debugfs**

Last updated on

## 6.50. Coreutils-8.27

The Coreutils package contains utilities for showing and setting the basic system characteristics.

**Approximate build time:**2.4 SBU

**Required disk space:**171 MB

### 6.50.1. Installation of Coreutils

POSIX requires that programs from Coreutils recognize character boundaries correctly even in multibyte locales. The following patch fixes this non-compliance and other internationalization-related bugs.

    patch -Np1 -i ../coreutils-8.27-i18n-1.patch

### Note

In the past, many bugs were found in this patch. When reporting new bugs to Coreutils maintainers, please check first if they are reproducible without this patch.

Suppress a test which on some machines can loop forever:

    sed -i '/test.lock/s/^/#/' gnulib-tests/gnulib.mk

Now prepare Coreutils for compilation:

    FORCE_UNSAFE_CONFIGURE=1 ./configure \
                --prefix=/usr            \
                --enable-no-install-program=kill,uptime

**The meaning of the configure options:**
`FORCE_UNSAFE_CONFIGURE=1`
This environment variable allows the package to be built as the root user.
*`--enable-no-install-program=kill,uptime`*
The purpose of this switch is to prevent Coreutils from installing binaries that will be installed by other packages later.

Compile the package:

    FORCE_UNSAFE_CONFIGURE=1 make

Skip down to “Install the package” if not running the test suite.

Now the test suite is ready to be run. First, run the tests that are meant to be run as user `root`:

    make NON_ROOT_USERNAME=nobody check-root

We're going to run the remainder of the tests as the `nobody` user. Certain tests, however, require that the user be a member of more than one group. So that these tests are not skipped we'll add a temporary group and make the user `nobody` a part of it:

    echo "dummy:x:1000:nobody" >> /etc/group

Fix some of the permissions so that the non-root user can compile and run the tests:

    chown -Rv nobody .

Now run the tests. Make sure the PATH in the **`su`** environment includes /tools/bin.

    su nobody -s /bin/bash \
              -c "PATH=$PATH make RUN_EXPENSIVE_TESTS=yes check"

The test programs test-getlogin and date-debug are known to fail in a partially built system environment like the chroot environment here, but pass if run at the end of this chapter.

Remove the temporary group:

    sed -i '/dummy/d' /etc/group

Install the package:

    make install

Move programs to the locations specified by the FHS:

    mv -v /usr/bin/{cat,chgrp,chmod,chown,cp,date,dd,df,echo} /bin
    mv -v /usr/bin/{false,ln,ls,mkdir,mknod,mv,pwd,rm} /bin
    mv -v /usr/bin/{rmdir,stty,sync,true,uname} /bin
    mv -v /usr/bin/chroot /usr/sbin
    mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
    sed -i s/\"1\"/\"8\"/1 /usr/share/man/man8/chroot.8

Some of the scripts in the LFS-Bootscripts package depend on **head**, **sleep**, and **nice**. As `/usr` may not be available during the early stages of booting, those binaries need to be on the root partition:

    mv -v /usr/bin/{head,sleep,nice,test,[} /bin

### 6.50.2. Contents of Coreutils

**Installed programs:**[, base32, base64, basename, cat, chcon, chgrp, chmod, chown, chroot, cksum, comm, cp, csplit, cut, date, dd, df, dir, dircolors, dirname, du, echo, env, expand, expr, factor, false, fmt, fold, groups, head, hostid, id, install, join, link, ln, logname, ls, md5sum, mkdir, mkfifo, mknod, mktemp, mv, nice, nl, nohup, nproc, numfmt, od, paste, pathchk, pinky, pr, printenv, printf, ptx, pwd, readlink, realpath, rm, rmdir, runcon, seq, sha1sum, sha224sum, sha256sum, sha384sum, sha512sum, shred, shuf, sleep, sort, split, stat, stdbuf, stty, sum, sync, tac, tail, tee, test, timeout, touch, tr, true, truncate, tsort, tty, uname, unexpand, uniq, unlink, users, vdir, wc, who, whoami, and yes

**Installed library:**libstdbuf.so

**Installed directory:**/usr/libexec/coreutils

#### Short Descriptions

**base32**

Encodes and decodes data according to the base32 specification (RFC 4648)

**base64**

Encodes and decodes data according to the base64 specification (RFC 4648)

**basename**

Strips any path and a given suffix from a file name

**cat**

Concatenates files to standard output

**chcon**

Changes security context for files and directories

**chgrp**

Changes the group ownership of files and directories

**chmod**

Changes the permissions of each file to the given mode; the mode can be either a symbolic representation of the changes to make or an octal number representing the new permissions

**chown**

Changes the user and/or group ownership of files and directories

**chroot**

Runs a command with the specified directory as the `/` directory

**cksum**

Prints the Cyclic Redundancy Check (CRC) checksum and the byte counts of each specified file

**comm**

Compares two sorted files, outputting in three columns the lines that are unique and the lines that are common

**cp**

Copies files

**csplit**

Splits a given file into several new files, separating them according to given patterns or line numbers and outputting the byte count of each new file

**cut**

Prints sections of lines, selecting the parts according to given fields or positions

**date**

Displays the current time in the given format, or sets the system date

**dd**

Copies a file using the given block size and count, while optionally performing conversions on it

**df**

Reports the amount of disk space available (and used) on all mounted file systems, or only on the file systems holding the selected files

**dir**

Lists the contents of each given directory (the same as the **ls** command)

**dircolors**

Outputs commands to set the `LS_COLOR` environment variable to change the color scheme used by **ls**

**dirname**

Strips the non-directory suffix from a file name

**du**

Reports the amount of disk space used by the current directory, by each of the given directories (including all subdirectories) or by each of the given files

**echo**

Displays the given strings

**env**

Runs a command in a modified environment

**expand**

Converts tabs to spaces

**expr**

Evaluates expressions

**factor**

Prints the prime factors of all specified integer numbers

**false**

Does nothing, unsuccessfully; it always exits with a status code indicating failure

**fmt**

Reformats the paragraphs in the given files

**fold**

Wraps the lines in the given files

**groups**

Reports a user's group memberships

**head**

Prints the first ten lines (or the given number of lines) of each given file

**hostid**

Reports the numeric identifier (in hexadecimal) of the host

**id**

Reports the effective user ID, group ID, and group memberships of the current user or specified user

**install**

Copies files while setting their permission modes and, if possible, their owner and group

**join**

Joins the lines that have identical join fields from two separate files

**link**

Creates a hard link with the given name to a file

**ln**

Makes hard links or soft (symbolic) links between files

**logname**

Reports the current user's login name

**ls**

Lists the contents of each given directory

**md5sum**

Reports or checks Message Digest 5 (MD5) checksums

**mkdir**

Creates directories with the given names

**mkfifo**

Creates First-In, First-Outs (FIFOs), a "named pipe" in UNIX parlance, with the given names

**mknod**

Creates device nodes with the given names; a device node is a character special file, a block special file, or a FIFO

**mktemp**

Creates temporary files in a secure manner; it is used in scripts

**mv**

Moves or renames files or directories

**nice**

Runs a program with modified scheduling priority

**nl**

Numbers the lines from the given files

**nohup**

Runs a command immune to hangups, with its output redirected to a log file

**nproc**

Prints the number of processing units available to a process

**numfmt**

Converts numbers to or from human-readable strings

**od**

Dumps files in octal and other formats

**paste**

Merges the given files, joining sequentially corresponding lines side by side, separated by tab characters

**pathchk**

Checks if file names are valid or portable

**pinky**

Is a lightweight finger client; it reports some information about the given users

**pr**

Paginates and columnates files for printing

**printenv**

Prints the environment

**printf**

Prints the given arguments according to the given format, much like the C printf function

**ptx**

Produces a permuted index from the contents of the given files, with each keyword in its context

**pwd**

Reports the name of the current working directory

**readlink**

Reports the value of the given symbolic link

**realpath**

Prints the resolved path

**rm**

Removes files or directories

**rmdir**

Removes directories if they are empty

**runcon**

Runs a command with specified security context

**seq**

Prints a sequence of numbers within a given range and with a given increment

**sha1sum**

Prints or checks 160-bit Secure Hash Algorithm 1 (SHA1) checksums

**sha224sum**

Prints or checks 224-bit Secure Hash Algorithm checksums

**sha256sum**

Prints or checks 256-bit Secure Hash Algorithm checksums

**sha384sum**

Prints or checks 384-bit Secure Hash Algorithm checksums

**sha512sum**

Prints or checks 512-bit Secure Hash Algorithm checksums

**shred**

Overwrites the given files repeatedly with complex patterns, making it difficult to recover the data

**shuf**

Shuffles lines of text

**sleep**

Pauses for the given amount of time

**sort**

Sorts the lines from the given files

**split**

Splits the given file into pieces, by size or by number of lines

**stat**

Displays file or filesystem status

**stdbuf**

Runs commands with altered buffering operations for its standard streams

**stty**

Sets or reports terminal line settings

**sum**

Prints checksum and block counts for each given file

**sync**

Flushes file system buffers; it forces changed blocks to disk and updates the super block

**tac**

Concatenates the given files in reverse

**tail**

Prints the last ten lines (or the given number of lines) of each given file

**tee**

Reads from standard input while writing both to standard output and to the given files

**test**

Compares values and checks file types

**timeout**

Runs a command with a time limit

**touch**

Changes file timestamps, setting the access and modification times of the given files to the current time; files that do not exist are created with zero length

**tr**

Translates, squeezes, and deletes the given characters from standard input

**true**

Does nothing, successfully; it always exits with a status code indicating success

**truncate**

Shrinks or expands a file to the specified size

**tsort**

Performs a topological sort; it writes a completely ordered list according to the partial ordering in a given file

**tty**

Reports the file name of the terminal connected to standard input

**uname**

Reports system information

**unexpand**

Converts spaces to tabs

**uniq**

Discards all but one of successive identical lines

**unlink**

Removes the given file

**users**

Reports the names of the users currently logged on

**vdir**

Is the same as **ls -l**

**wc**

Reports the number of lines, words, and bytes for each given file, as well as a total line when more than one file is given

**who**

Reports who is logged on

**whoami**

Reports the user name associated with the current effective user ID

**yes**

Repeatedly outputs “y” or a given string until killed

`libstdbuf`

Library used by **stdbuf**

Last updated on

## 6.51. Diffutils-3.6

The Diffutils package contains programs that show the differences between files or directories.

**Approximate build time:**0.4 SBU

**Required disk space:**30 MB

### 6.51.1. Installation of Diffutils

Prepare Diffutils for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.51.2. Contents of Diffutils

**Installed programs:**cmp, diff, diff3, and sdiff

#### Short Descriptions

**cmp**

Compares two files and reports whether or in which bytes they differ

**diff**

Compares two files or directories and reports which lines in the files differ

**diff3**

Compares three files line by line

**sdiff**

Merges two files and interactively outputs the results

Last updated on

## 6.52. Gawk-4.1.4

The Gawk package contains programs for manipulating text files.

**Approximate build time:**0.3 SBU

**Required disk space:**36 MB

### 6.52.1. Installation of Gawk

Prepare Gawk for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

If desired, install the documentation:

    mkdir -v /usr/share/doc/gawk-4.1.4
    cp    -v doc/{awkforai.txt,*.{eps,pdf,jpg}} /usr/share/doc/gawk-4.1.4

### 6.52.2. Contents of Gawk

**Installed programs:**awk (link to gawk), gawk, gawk-4.1.4, and igawk

**Installed libraries:**filefuncs.so, fnmatch.so, fork.so, inplace.so, ordchr.so, readdir.so, readfile.so, revoutput.so, revtwoway.so, rwarray.so, testext.so, and time.so

**Installed directories:**/usr/lib/gawk, /usr/libexec/awk, /usr/share/awk, and /usr/share/doc/gawk-4.1.4

#### Short Descriptions

**awk**

A link to **gawk**

**gawk**

A program for manipulating text files; it is the GNU implementation of **awk**

**gawk-4.1.4**

A hard link to **gawk**

**igawk**

Gives **gawk** the ability to include files

Last updated on

## 6.53. Findutils-4.6.0

The Findutils package contains programs to find files. These programs are provided to recursively search through a directory tree and to create, maintain, and search a database (often faster than the recursive find, but unreliable if the database has not been recently updated).

**Approximate build time:**0.7 SBU

**Required disk space:**48 MB

### 6.53.1. Installation of Findutils

First, suppress a test which on some machines can loop forever:

    sed -i 's/test-lock..EXEEXT.//' tests/Makefile.in

Prepare Findutils for compilation:

    ./configure --prefix=/usr --localstatedir=/var/lib/locate

**The meaning of the configure options:**
*`--localstatedir`*
This option changes the location of the **locate** database to be in `/var/lib/locate`, which is FHS-compliant.

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

Some of the scripts in the LFS-Bootscripts package depend on **find**. As `/usr` may not be available during the early stages of booting, this program needs to be on the root partition. The **updatedb** script also needs to be modified to correct an explicit path:

    mv -v /usr/bin/find /bin
    sed -i 's|find:=${BINDIR}|find:=/bin|' /usr/bin/updatedb

### 6.53.2. Contents of Findutils

**Installed programs:**code, find, locate, oldfind, updatedb, and xargs

#### Short Descriptions

**code**

Was formerly used to produce **locate** databases; it is the ancestor of **frcode**

**find**

Searches given directory trees for files matching the specified criteria

**locate**

Searches through a database of file names and reports the names that contain a given string or match a given pattern

**oldfind**

Older version of find, using a different algorithm

**updatedb**

Updates the **locate** database; it scans the entire file system (including other file systems that are currently mounted, unless told not to) and puts every file name it finds into the database

**xargs**

Can be used to apply a given command to a list of files

Last updated on

## 6.54. Groff-1.22.3

The Groff package contains programs for processing and formatting text.

**Approximate build time:**0.4 SBU

**Required disk space:**83 MB

### 6.54.1. Installation of Groff

Groff expects the environment variable `PAGE` to contain the default paper size. For users in the United States, *`PAGE=letter`* is appropriate. Elsewhere, *`PAGE=A4`* may be more suitable. While the default paper size is configured during compilation, it can be overridden later by echoing either “A4” or “letter” to the `/etc/papersize` file.

Prepare Groff for compilation:

    PAGE=*<paper_size>* ./configure --prefix=/usr

This package does not support parallel build. Compile the package:

    make -j1

This package does not come with a test suite.

Install the package:

    make install

### 6.54.2. Contents of Groff

**Installed programs:**addftinfo, afmtodit, chem, eqn, eqn2graph, gdiffmk, glilypond, gperl, gpinyin, grap2graph, grn, grodvi, groff, groffer, grog, grolbp, grolj4, gropdf, grops, grotty, hpftodit, indxbib, lkbib, lookbib, mmroff, neqn, nroff, pdfmom, pdfroff, pfbtops, pic, pic2graph, post-grohtml, preconv, pre-grohtml, refer, roff2dvi, roff2html, roff2pdf, roff2ps, roff2text, roff2x, soelim, tbl, tfmtodit, and troff

**Installed directories:**/usr/lib/groff and /usr/share/doc/groff-1.22.3, /usr/share/groff

#### Short Descriptions

**addftinfo**

Reads a troff font file and adds some additional font-metric information that is used by the **groff** system

**afmtodit**

Creates a font file for use with **groff** and **grops**

**chem**

Groff preprocessor for producing chemical structure diagrams

**eqn**

Compiles descriptions of equations embedded within troff input files into commands that are understood by **troff**

**eqn2graph**

Converts a troff EQN (equation) into a cropped image

**gdiffmk**

Marks differences between groff/nroff/troff files

**glilypond**

Transforms sheet music written in the lilypond language into the groff language

**gperl**

Preprocesor for groff, allowing addition of perl code into groff files

**gpinyin**

Preprocesor for groff, allowing addition of Chinese European-like language Pinyin into groff files.

**grap2graph**

Converts a grap diagram into a cropped bitmap image

**grn**

A **groff** preprocessor for gremlin files

**grodvi**

A driver for **groff** that produces TeX dvi format

**groff**

A front-end to the groff document formatting system; normally, it runs the **troff** program and a post-processor appropriate for the selected device

**groffer**

Displays groff files and man pages on X and tty terminals

**grog**

Reads files and guesses which of the **groff** options `-e`, `-man`, `-me`, `-mm`, `-ms`, `-p`, `-s`, and `-t` are required for printing files, and reports the **groff** command including those options

**grolbp**

Is a **groff** driver for Canon CAPSL printers (LBP-4 and LBP-8 series laser printers)

**grolj4**

Is a driver for **groff** that produces output in PCL5 format suitable for an HP LaserJet 4 printer

**gropdf**

Translates the output of GNU **troff** to PDF

**grops**

Translates the output of GNU **troff** to PostScript

**grotty**

Translates the output of GNU **troff** into a form suitable for typewriter-like devices

**hpftodit**

Creates a font file for use with **groff -Tlj4** from an HP-tagged font metric file

**indxbib**

Creates an inverted index for the bibliographic databases with a specified file for use with **refer**, **lookbib**, and **lkbib**

**lkbib**

Searches bibliographic databases for references that contain specified keys and reports any references found

**lookbib**

Prints a prompt on the standard error (unless the standard input is not a terminal), reads a line containing a set of keywords from the standard input, searches the bibliographic databases in a specified file for references containing those keywords, prints any references found on the standard output, and repeats this process until the end of input

**mmroff**

A simple preprocessor for **groff**

**neqn**

Formats equations for American Standard Code for Information Interchange (ASCII) output

**nroff**

A script that emulates the **nroff** command using **groff**

**pdfmom**

Is a wrapper around groff that facilitates the production of PDF documents from files formatted with the mom macros.

**pdfroff**

Creates pdf documents using groff

**pfbtops**

Translates a PostScript font in `.pfb` format to ASCII

**pic**

Compiles descriptions of pictures embedded within troff or TeX input files into commands understood by TeX or **troff**

**pic2graph**

Converts a PIC diagram into a cropped image

**post-grohtml**

Translates the output of GNU **troff** to HTML

**preconv**

Converts encoding of input files to something GNU **troff** understands

**pre-grohtml**

Translates the output of GNU **troff** to HTML

**refer**

Copies the contents of a file to the standard output, except that lines between *.[* and *.]* are interpreted as citations, and lines between *.R1* and *.R2* are interpreted as commands for how citations are to be processed

**roff2dvi**

Transforms roff files into DVI format

**roff2html**

Transforms roff files into HTML format

**roff2pdf**

Transforms roff files into PDFs

**roff2ps**

Transforms roff files into ps files

**roff2text**

Transforms roff files into text files

**roff2x**

Transforms roff files into other formats

**soelim**

Reads files and replaces lines of the form *.so file* by the contents of the mentioned *file*

**tbl**

Compiles descriptions of tables embedded within troff input files into commands that are understood by **troff**

**tfmtodit**

Creates a font file for use with **groff -Tdvi**

**troff**

Is highly compatible with Unix **troff**; it should usually be invoked using the **groff** command, which will also run preprocessors and post-processors in the appropriate order and with the appropriate options

Last updated on

## 6.55. GRUB-2.02

The GRUB package contains the GRand Unified Bootloader.

**Approximate build time:**0.8 SBU

**Required disk space:**144 MB

### 6.55.1. Installation of GRUB

Prepare GRUB for compilation:

    ./configure --prefix=/usr          \
                --sbindir=/sbin        \
                --sysconfdir=/etc      \
                --disable-efiemu       \
                --disable-werror

**The meaning of the new configure options:**
*`--disable-werror`*
This allows the build to complete with warnings introduced by more recent Flex versions.
*`--disable-efiemu`*
This option minimizes what is built by disabling a feature and testing programs not needed for LFS.

Compile the package:

    make

This package does not come with a test suite.

Install the package:

    make install

Using GRUB to make your LFS system bootable will be discussed in [Section 8.4, “Using GRUB to Set Up the Boot Process”](Linux%20From%20Scratch.html#ch-bootable-grub).

### 6.55.2. Contents of GRUB

**Installed programs:**grub-bios-setup, grub-editenv, grub-file, grub-fstest, grub-glue-efi, grub-install, grub-kbdcomp, grub-macbless, grub-menulst2cfg, grub-mkconfig, grub-mkimage, grub-mklayout, grub-mknetdir, grub-mkpasswd-pbkdf2, grub-mkrelpath, grub-mkrescue, grub-mkstandalone, grub-ofpathname, grub-probe, grub-reboot, grub-render-label, grub-script-check, grub-set-default, grub-sparc64-setup, and grub-syslinux2cfg

**Installed directories:**/usr/lib/grub, /etc/grub.d, /usr/share/grub, and boot/grub (when grub-install is first run)

#### Short Descriptions

**grub-bios-setup**

Is a helper program for grub-install

**grub-editenv**

A tool to edit the environment block

**grub-file**

Checks if FILE is of the specified type.

**grub-fstest**

Tool to debug the filesystem driver

**grub-glue-efi**

Processes ia32 and amd64 EFI images and glues them according to Apple format.

**grub-install**

Install GRUB on your drive

**grub-kbdcomp**

Script that converts an xkb layout into one recognized by GRUB

**grub-macbless**

Mac-style bless on HFS or HFS+ files

**grub-menulst2cfg**

Converts a GRUB Legacy `menu.lst` into a `grub.cfg` for use with GRUB 2

**grub-mkconfig**

Generate a grub config file

**grub-mkimage**

Make a bootable image of GRUB

**grub-mklayout**

Generates a GRUB keyboard layout file

**grub-mknetdir**

Prepares a GRUB netboot directory

**grub-mkpasswd-pbkdf2**

Generates an encrypted PBKDF2 password for use in the boot menu

**grub-mkrelpath**

Makes a system pathname relative to its root

**grub-mkrescue**

Make a bootable image of GRUB suitable for a floppy disk or CDROM/DVD

**grub-mkstandalone**

Generates a standalone image

**grub-ofpathname**

Is a helper program that prints the path of a GRUB device

**grub-probe**

Probe device information for a given path or device

**grub-reboot**

Sets the default boot entry for GRUB for the next boot only

**grub-render-label**

Render Apple .disk_label for Apple Macs

**grub-script-check**

Checks GRUB configuration script for syntax errors

**grub-set-default**

Sets the default boot entry for GRUB

**grub-sparc64-setup**

Is a helper program for grub-setup

**grub-syslinux2cfg**

Transform a syslinux config file into grub.cfg format

Last updated on

## 6.56. Less-487

The Less package contains a text file viewer.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**3.5 MB

### 6.56.1. Installation of Less

Prepare Less for compilation:

    ./configure --prefix=/usr --sysconfdir=/etc

**The meaning of the configure options:**
*`--sysconfdir=/etc`*
This option tells the programs created by the package to look in `/etc` for the configuration files.

Compile the package:

    make

This package does not come with a test suite.

Install the package:

    make install

### 6.56.2. Contents of Less

**Installed programs:**less, lessecho, and lesskey

#### Short Descriptions

**less**

A file viewer or pager; it displays the contents of the given file, letting the user scroll, find strings, and jump to marks

**lessecho**

Needed to expand meta-characters, such as *** and *?*, in filenames on Unix systems

**lesskey**

Used to specify the key bindings for **less**

Last updated on

## 6.57. Gzip-1.8

The Gzip package contains programs for compressing and decompressing files.

**Approximate build time:**0.1 SBU

**Required disk space:**19 MB

### 6.57.1. Installation of Gzip

Prepare Gzip for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Two tests are known to fail in the LFS environment: help-version and zmore.

Install the package:

    make install

Move a program that needs to be on the root filesystem:

    mv -v /usr/bin/gzip /bin

### 6.57.2. Contents of Gzip

**Installed programs:**gunzip, gzexe, gzip, uncompress (hard link with gunzip), zcat, zcmp, zdiff, zegrep, zfgrep, zforce, zgrep, zless, zmore, and znew

#### Short Descriptions

**gunzip**

Decompresses gzipped files

**gzexe**

Creates self-decompressing executable files

**gzip**

Compresses the given files using Lempel-Ziv (LZ77) coding

**uncompress**

Decompresses compressed files

**zcat**

Decompresses the given gzipped files to standard output

**zcmp**

Runs **cmp** on gzipped files

**zdiff**

Runs **diff** on gzipped files

**zegrep**

Runs **egrep** on gzipped files

**zfgrep**

Runs **fgrep** on gzipped files

**zforce**

Forces a `.gz` extension on all given files that are gzipped files, so that **gzip** will not compress them again; this can be useful when file names were truncated during a file transfer

**zgrep**

Runs **grep** on gzipped files

**zless**

Runs **less** on gzipped files

**zmore**

Runs **more** on gzipped files

**znew**

Re-compresses files from **compress** format to **gzip** format—`.Z` to `.gz`

Last updated on

## 6.58. IPRoute2-4.12.0

The IPRoute2 package contains programs for basic and advanced IPV4-based networking.

**Approximate build time:**0.2 SBU

**Required disk space:**12 MB

### 6.58.1. Installation of IPRoute2

The **arpd** program included in this package will not be built since it is dependent on Berkeley DB, which is not installed in LFS. However, documentation files and a directory for **arpd** will still be installed. Prevent this by running the commands below. If the **arpd** binary is needed, instructions for compiling Berkeley DB can be found in the BLFS Book at [http://www.linuxfromscratch.org/blfs/view/8.1/server/databases.html#db](http://www.linuxfromscratch.org/blfs/view/8.1/server/databases.html#db).

    sed -i /ARPD/d Makefile
    sed -i 's/arpd.8//' man/man8/Makefile
    rm -v doc/arpd.sgml

It is also necessary to disable building one module that requires [http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/iptables.html](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/iptables.html).

    sed -i 's/m_ipt.o//' tc/Makefile

Compile the package:

    make

This package does not have a working test suite.

Install the package:

    make DOCDIR=/usr/share/doc/iproute2-4.12.0 install

### 6.58.2. Contents of IPRoute2

**Installed programs:**bridge, ctstat (link to lnstat), genl, ifcfg, ifstat, ip, lnstat, nstat, routef, routel, rtacct, rtmon, rtpr, rtstat (link to lnstat), ss, and tc

**Installed directories:**/etc/iproute2, /usr/lib/tc, and /usr/share/doc/iproute2-4.12.0,

#### Short Descriptions

**bridge**

Configures network bridges

**ctstat**

Connection status utility

**genl**

Generic netlink utility frontend

**ifcfg**

A shell script wrapper for the **ip** command [Note that it requires the **arping** and **rdisk** programs from the iputils package found at [http://www.skbuff.net/iputils/](http://www.skbuff.net/iputils/).]

**ifstat**

Shows the interface statistics, including the amount of transmitted and received packets by interface

**ip**

The main executable. It has several different functions:

**ip link *`<device>`*** allows users to look at the state of devices and to make changes

**ip addr** allows users to look at addresses and their properties, add new addresses, and delete old ones

**ip neighbor** allows users to look at neighbor bindings and their properties, add new neighbor entries, and delete old ones

**ip rule** allows users to look at the routing policies and change them

**ip route** allows users to look at the routing table and change routing table rules

**ip tunnel** allows users to look at the IP tunnels and their properties, and change them

**ip maddr** allows users to look at the multicast addresses and their properties, and change them

**ip mroute** allows users to set, change, or delete the multicast routing

**ip monitor** allows users to continuously monitor the state of devices, addresses and routes

**lnstat**

Provides Linux network statistics; it is a generalized and more feature-complete replacement for the old **rtstat** program

**nstat**

Shows network statistics

**routef**

A component of **ip route**. This is for flushing the routing tables

**routel**

A component of **ip route**. This is for listing the routing tables

**rtacct**

Displays the contents of `/proc/net/rt_acct`

**rtmon**

Route monitoring utility

**rtpr**

Converts the output of **ip -o** back into a readable form

**rtstat**

Route status utility

**ss**

Similar to the **netstat** command; shows active connections

**tc**

Traffic Controlling Executable; this is for Quality Of Service (QOS) and Class Of Service (COS) implementations

**tc qdisc** allows users to setup the queueing discipline

**tc class** allows users to setup classes based on the queuing discipline scheduling

**tc estimator** allows users to estimate the network flow into a network

**tc filter** allows users to setup the QOS/COS packet filtering

**tc policy** allows users to setup the QOS/COS policies

Last updated on

## 6.59. Kbd-2.0.4

The Kbd package contains key-table files, console fonts, and keyboard utilities.

**Approximate build time:**0.1 SBU

**Required disk space:**29 MB

### 6.59.1. Installation of Kbd

The behaviour of the Backspace and Delete keys is not consistent across the keymaps in the Kbd package. The following patch fixes this issue for i386 keymaps:

    patch -Np1 -i ../kbd-2.0.4-backspace-1.patch

After patching, the Backspace key generates the character with code 127, and the Delete key generates a well-known escape sequence.

Remove the redundant **resizecons** program (it requires the defunct svgalib to provide the video mode files - for normal use **setfont** sizes the console appropriately) together with its manpage.

    sed -i 's/\(RESIZECONS_PROGS=\)yes/\1no/g' configure
    sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in

Prepare Kbd for compilation:

    PKG_CONFIG_PATH=/tools/lib/pkgconfig ./configure --prefix=/usr --disable-vlock

**The meaning of the configure options:**
*`--disable-vlock`*
This option prevents the vlock utility from being built, as it requires the PAM library, which isn't available in the chroot environment.

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### Note

For some languages (e.g., Belarusian) the Kbd package doesn't provide a useful keymap where the stock “by” keymap assumes the ISO-8859-5 encoding, and the CP1251 keymap is normally used. Users of such languages have to download working keymaps separately.

If desired, install the documentation:

    mkdir -v       /usr/share/doc/kbd-2.0.4
    cp -R -v docs/doc/* /usr/share/doc/kbd-2.0.4

### 6.59.2. Contents of Kbd

**Installed programs:**chvt, deallocvt, dumpkeys, fgconsole, getkeycodes, kbdinfo, kbd_mode, kbdrate, loadkeys, loadunimap, mapscrn, openvt, psfaddtable (link to psfxtable), psfgettable (link to psfxtable), psfstriptable (link to psfxtable), psfxtable, setfont, setkeycodes, setleds, setmetamode, setvtrgb, showconsolefont, showkey, unicode_start, and unicode_stop

**Installed directories:**/usr/share/consolefonts, /usr/share/consoletrans, /usr/share/keymaps, /usr/share/doc/kbd-2.0.4, and /usr/share/unimaps

#### Short Descriptions

**chvt**

Changes the foreground virtual terminal

**deallocvt**

Deallocates unused virtual terminals

**dumpkeys**

Dumps the keyboard translation tables

**fgconsole**

Prints the number of the active virtual terminal

**getkeycodes**

Prints the kernel scancode-to-keycode mapping table

**kbdinfo**

Obtains information about the status of a console

**kbd_mode**

Reports or sets the keyboard mode

**kbdrate**

Sets the keyboard repeat and delay rates

**loadkeys**

Loads the keyboard translation tables

**loadunimap**

Loads the kernel unicode-to-font mapping table

**mapscrn**

An obsolete program that used to load a user-defined output character mapping table into the console driver; this is now done by **setfont**

**openvt**

Starts a program on a new virtual terminal (VT)

**psfaddtable**

Adds a Unicode character table to a console font

**psfgettable**

Extracts the embedded Unicode character table from a console font

**psfstriptable**

Removes the embedded Unicode character table from a console font

**psfxtable**

Handles Unicode character tables for console fonts

**setfont**

Changes the Enhanced Graphic Adapter (EGA) and Video Graphics Array (VGA) fonts on the console

**setkeycodes**

Loads kernel scancode-to-keycode mapping table entries; this is useful if there are unusual keys on the keyboard

**setleds**

Sets the keyboard flags and Light Emitting Diodes (LEDs)

**setmetamode**

Defines the keyboard meta-key handling

**setvtrgb**

Sets the console color map in all virtual terminals

**showconsolefont**

Shows the current EGA/VGA console screen font

**showkey**

Reports the scancodes, keycodes, and ASCII codes of the keys pressed on the keyboard

**unicode_start**

Puts the keyboard and console in UNICODE mode [Don't use this program unless your keymap file is in the ISO-8859-1 encoding. For other encodings, this utility produces incorrect results.]

**unicode_stop**

Reverts keyboard and console from UNICODE mode

Last updated on

## 6.60. Libpipeline-1.4.2

The Libpipeline package contains a library for manipulating pipelines of subprocesses in a flexible and convenient way.

**Approximate build time:**0.1 SBU

**Required disk space:**8.0 MB

### 6.60.1. Installation of Libpipeline

Prepare Libpipeline for compilation:

    PKG_CONFIG_PATH=/tools/lib/pkgconfig ./configure --prefix=/usr

**The meaning of the configure options:**
*`PKG_CONFIG_PATH`*
Use pkg-config to obtain the location of the test library metadata built in [Section 5.14, “Check-0.11.0”](Linux%20From%20Scratch.html#ch-tools-check).

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.60.2. Contents of Libpipeline

**Installed library:**libpipeline.so

#### Short Descriptions

`libpipeline`

This library is used to safely construct pipelines between subprocesses

Last updated on

## 6.61. Make-4.2.1

The Make package contains a program for compiling packages.

**Approximate build time:**0.6 SBU

**Required disk space:**12.6 MB

### 6.61.1. Installation of Make

Prepare Make for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

The test suite needs to know where supporting perl files are located. We use an envronment variable to accomplish this. To test the results, issue:

    make PERL5LIB=$PWD/tests/ check

Install the package:

    make install

### 6.61.2. Contents of Make

**Installed program:**make

#### Short Descriptions

**make**

Automatically determines which pieces of a package need to be (re)compiled and then issues the relevant commands

Last updated on

## 6.62. Patch-2.7.5

The Patch package contains a program for modifying or creating files by applying a “patch” file typically created by the **diff** program.

**Approximate build time:**0.2 SBU

**Required disk space:**11 MB

### 6.62.1. Installation of Patch

Prepare Patch for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.62.2. Contents of Patch

**Installed program:**patch

#### Short Descriptions

**patch**

Modifies files according to a patch file [A patch file is normally a difference listing created with the **diff** program. By applying these differences to the original files, **patch** creates the patched versions.]

Last updated on

## 6.63. Sysklogd-1.5.1

The Sysklogd package contains programs for logging system messages, such as those given by the kernel when unusual things happen.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**0.7 MB

### 6.63.1. Installation of Sysklogd

First, fix problems that causes a segmentation fault under some conditions in klogd and fix an obsolete program construct:

    sed -i '/Error loading kernel symbols/{n;n;d}' ksym_mod.c
    sed -i 's/union wait/int/' syslogd.c

Compile the package:

    make

This package does not come with a test suite.

Install the package:

    make BINDIR=/sbin install

### 6.63.2. Configuring Sysklogd

Create a new `/etc/syslog.conf` file by running the following:

    cat > /etc/syslog.conf << "EOF"
    # Begin /etc/syslog.conf

    auth,authpriv.* -/var/log/auth.log
    *.*;auth,authpriv.none -/var/log/sys.log
    daemon.* -/var/log/daemon.log
    kern.* -/var/log/kern.log
    mail.* -/var/log/mail.log
    user.* -/var/log/user.log
    *.emerg *

    # End /etc/syslog.conf
    EOF

### 6.63.3. Contents of Sysklogd

**Installed programs:**klogd and syslogd

#### Short Descriptions

**klogd**

A system daemon for intercepting and logging kernel messages

**syslogd**

Logs the messages that system programs offer for logging [Every logged message contains at least a date stamp and a hostname, and normally the program's name too, but that depends on how trusting the logging daemon is told to be.]

Last updated on

## 6.64. Sysvinit-2.88dsf

The Sysvinit package contains programs for controlling the startup, running, and shutdown of the system.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**1.1 MB

### 6.64.1. Installation of Sysvinit

First, apply a patch that removes several programs installed by other packages, clarifies a message, and fixes a compiler warning:

    patch -Np1 -i ../sysvinit-2.88dsf-consolidated-1.patch

Compile the package:

    make -C src

This package does not come with a test suite.

Install the package:

    make -C src install

### 6.64.2. Contents of Sysvinit

**Installed programs:**bootlogd, fstab-decode, halt, init, killall5, poweroff (link to halt), reboot (link to halt), runlevel, shutdown, and telinit (link to init)

#### Short Descriptions

**bootlogd**

Logs boot messages to a log file

**fstab-decode**

Run a command with fstab-encoded arguments

**halt**

Normally invokes **shutdown** with the *`-h`* option, except when already in run-level 0, then it tells the kernel to halt the system; it notes in the file `/var/log/wtmp` that the system is being brought down

**init**

The first process to be started when the kernel has initialized the hardware which takes over the boot process and starts all the proceses specified in its configuration file

**killall5**

Sends a signal to all processes, except the processes in its own session so it will not kill its parent shell

**poweroff**

Tells the kernel to halt the system and switch off the computer (see **halt**)

**reboot**

Tells the kernel to reboot the system (see **halt**)

**runlevel**

Reports the previous and the current run-level, as noted in the last run-level record in `/var/run/utmp`

**shutdown**

Brings the system down in a secure way, signaling all processes and notifying all logged-in users

**telinit**

Tells **init** which run-level to change to

Last updated on

## 6.65. Eudev-3.2.2

The Eudev package contains programs for dynamic creation of device nodes.

**Approximate build time:**0.2 SBU

**Required disk space:**78 MB

### 6.65.1. Installation of Eudev

First, fix a test script:

    sed -r -i 's|/usr(/bin/test)|\1|' test/udev-test.pl

Next, remove an unneeded line that causes a build failure:

    sed -i '/keyboard_lookup_key/d' src/udev/udev-builtin-keyboard.c

Next, add a workaround to prevent the /tools directory from being hard coded into Eudev binary files library locations:

    cat > config.cache << "EOF"
    HAVE_BLKID=1
    BLKID_LIBS="-lblkid"
    BLKID_CFLAGS="-I/tools/include"
    EOF

Prepare Eudev for compilation:

    ./configure --prefix=/usr           \
                --bindir=/sbin          \
                --sbindir=/sbin         \
                --libdir=/usr/lib       \
                --sysconfdir=/etc       \
                --libexecdir=/lib       \
                --with-rootprefix=      \
                --with-rootlibdir=/lib  \
                --enable-manpages       \
                --disable-static        \
                --config-cache

Compile the package:

    LIBRARY_PATH=/tools/lib make

Create some directories now that are needed for tests, but will also be used as a part of installation:

    mkdir -pv /lib/udev/rules.d
    mkdir -pv /etc/udev/rules.d

To test the results, issue:

    make LD_LIBRARY_PATH=/tools/lib check

Install the package:

    make LD_LIBRARY_PATH=/tools/lib install

Install some custom rules and support files useful in an LFS environment:

    tar -xvf ../udev-lfs-20140408.tar.bz2
    make -f udev-lfs-20140408/Makefile.lfs install

### 6.65.2. Configuring Eudev

Information about hardware devices is maintained in the `/etc/udev/hwdb.d` and `/usr/lib/udev/hwdb.d` directories. Eudev needs that information to be compiled into a binary database `/etc/udev/hwdb.bin`. Create the initial database:

    LD_LIBRARY_PATH=/tools/lib udevadm hwdb --update

This command needs to be run each time the hardware information is updated.

### 6.65.3. Contents of Eudev

**Installed programs:**udevadm and udevd

**Installed libraries:**libudev.so

**Installed directories:**/etc/udev, /lib/udev, and /usr/share/doc/udev-20140408

#### Short Descriptions

**udevadm**

Generic udev administration tool: controls the udevd daemon, provides info from the Udev database, monitors uevents, waits for uevents to finish, tests Udev configuration, and triggers uevents for a given device

**udevd**

A daemon that listens for uevents on the netlink socket, creates devices and runs the configured external programs in response to these uevents

`libudev`

A library interface to udev device information

`/etc/udev`

Contains Udev configuration files, device permissions, and rules for device naming

Last updated on

## 6.66. Util-linux-2.30.1

The Util-linux package contains miscellaneous utility programs. Among them are utilities for handling file systems, consoles, partitions, and messages.

**Approximate build time:**1.0 SBU

**Required disk space:**181 MB

### 6.66.1. FHS compliance notes

The FHS recommends using the `/var/lib/hwclock` directory instead of the usual `/etc` directory as the location for the `adjtime` file. First create a directory to enable storage for the **hwclock** program:

    mkdir -pv /var/lib/hwclock

### 6.66.2. Installation of Util-linux

Prepare Util-linux for compilation:

    ./configure ADJTIME_PATH=/var/lib/hwclock/adjtime   \
                --docdir=/usr/share/doc/util-linux-2.30.1 \
                --disable-chfn-chsh  \
                --disable-login      \
                --disable-nologin    \
                --disable-su         \
                --disable-setpriv    \
                --disable-runuser    \
                --disable-pylibmount \
                --disable-static     \
                --without-python     \
                --without-systemd    \
                --without-systemdsystemunitdir

The --disable and --without options prevent warnings about building components that require packages not in LFS or are inconsistent with programs installed by other packages.

Compile the package:

    make

If desired, run the test suite as a non-root user:

### Warning

Running the test suite as the root user can be harmful to your system. To run it, the CONFIG_SCSI_DEBUG option for the kernel must be available in the currently running system, and must be built as a module. Building it into the kernel will prevent booting. For complete coverage, other BLFS packages must be installed. If desired, this test can be run after rebooting into the completed LFS system and running:

    bash tests/run.sh --srcdir=$PWD --builddir=$PWD

    chown -Rv nobody .
    su nobody -s /bin/bash -c "PATH=$PATH make -k check"

### Note

One test, fincore/count, may fail in the initial chroot environment but will pass if the test is rerun after the LFS system is complete.

Install the package:

    make install

### 6.66.3. Contents of Util-linux

**Installed programs:**addpart, agetty, blkdiscard, blkid, blockdev, cal, cfdisk, chcpu, chrt, col, colcrt, colrm, column, ctrlaltdel, delpart, dmesg, eject, fallocate, fdformat, fdisk, findfs, findmnt, flock, fsck, fsck.cramfs, fsck.minix, fsfreeze, fstrim, getopt, hexdump, hwclock, i386, ionice, ipcmk, ipcrm, ipcs, isosize, kill, last, lastb (link to last), ldattach, linux32, linux64, logger, look, losetup, lsblk, lscpu, lsipc, lslocks, lslogins, mcookie, mesg, mkfs, mkfs.bfs, mkfs.cramfs, mkfs.minix, mkswap, more, mount, mountpoint, namei, nsenter, partx, pg, pivot_root, prlimit, raw, readprofile, rename, renice, resizepart, rev, rtcwake, script, scriptreplay, setarch, setsid, setterm, sfdisk, sulogin, swaplabel, swapoff (link to swapon), swapon, switch_root, tailf, taskset, ul, umount, uname26, unshare, utmpdump, uuidd, uuidgen, wall, wdctl, whereis, wipefs, x86_64, and zramctl

**Installed libraries:**libblkid.so, libfdisk.so, libmount.so, libsmartcols.so, and libuuid.so

**Installed directories:**/usr/include/blkid, /usr/include/libfdisk, /usr/include/libmount, /usr/include/libsmartcols, /usr/include/uuid, /usr/share/doc/util-linux-2.30.1, and /var/lib/hwclock

#### Short Descriptions

**addpart**

Informs the Linux kernel of new partitions

**agetty**

Opens a tty port, prompts for a login name, and then invokes the **login** program

**blkdiscard**

Discards sectors on a device

**blkid**

A command line utility to locate and print block device attributes

**blockdev**

Allows users to call block device ioctls from the command line

**cal**

Displays a simple calendar

**cfdisk**

Manipulates the partition table of the given device

**chcpu**

Modifies the state of CPUs

**chrt**

Manipulates real-time attributes of a process

**col**

Filters out reverse line feeds

**colcrt**

Filters **nroff** output for terminals that lack some capabilities, such as overstriking and half-lines

**colrm**

Filters out the given columns

**column**

Formats a given file into multiple columns

**ctrlaltdel**

Sets the function of the Ctrl+Alt+Del key combination to a hard or a soft reset

**delpart**

Asks the Linux kernel to remove a partition

**dmesg**

Dumps the kernel boot messages

**eject**

Ejects removable media

**fallocate**

Preallocates space to a file

**fdformat**

Low-level formats a floppy disk

**fdisk**

Manipulates the partition table of the given device

**findfs**

Finds a file system by label or Universally Unique Identifier (UUID)

**findmnt**

Is a command line interface to the libmount library for work with mountinfo, fstab and mtab files

**flock**

Acquires a file lock and then executes a command with the lock held

**fsck**

Is used to check, and optionally repair, file systems

**fsck.cramfs**

Performs a consistency check on the Cramfs file system on the given device

**fsck.minix**

Performs a consistency check on the Minix file system on the given device

**fsfreeze**

Is a very simple wrapper around FIFREEZE/FITHAW ioctl kernel driver operations

**fstrim**

Discards unused blocks on a mounted filesystem

**getopt**

Parses options in the given command line

**hexdump**

Dumps the given file in hexadecimal or in another given format

**hwclock**

Reads or sets the system's hardware clock, also called the Real-Time Clock (RTC) or Basic Input-Output System (BIOS) clock

**i386**

A symbolic link to setarch

**ionice**

Gets or sets the io scheduling class and priority for a program

**ipcmk**

Creates various IPC resources

**ipcrm**

Removes the given Inter-Process Communication (IPC) resource

**ipcs**

Provides IPC status information

**isosize**

Reports the size of an iso9660 file system

**kill**

Sends signals to processes

**last**

Shows which users last logged in (and out), searching back through the `/var/log/wtmp` file; it also shows system boots, shutdowns, and run-level changes

**lastb**

Shows the failed login attempts, as logged in `/var/log/btmp`

**ldattach**

Attaches a line discipline to a serial line

**linux32**

A symbolic link to setarch

**linux64**

A symbolic link to setarch

**logger**

Enters the given message into the system log

**look**

Displays lines that begin with the given string

**losetup**

Sets up and controls loop devices

**lsblk**

Lists information about all or selected block devices in a tree-like format

**lscpu**

Prints CPU architecture information

**lsipc**

Prints information on IPC facilities currently employed in the system

**lslocks**

Lists local system locks

**lslogins**

Lists information about users, groups and system accounts

**mcookie**

Generates magic cookies (128-bit random hexadecimal numbers) for **xauth**

**mesg**

Controls whether other users can send messages to the current user's terminal

**mkfs**

Builds a file system on a device (usually a hard disk partition)

**mkfs.bfs**

Creates a Santa Cruz Operations (SCO) bfs file system

**mkfs.cramfs**

Creates a cramfs file system

**mkfs.minix**

Creates a Minix file system

**mkswap**

Initializes the given device or file to be used as a swap area

**more**

A filter for paging through text one screen at a time

**mount**

Attaches the file system on the given device to a specified directory in the file-system tree

**mountpoint**

Checks if the directory is a mountpoint

**namei**

Shows the symbolic links in the given pathnames

**nsenter**

Runs a program with namespaces of other processes

**partx**

Tells the kernel about the presence and numbering of on-disk partitions

**pg**

Displays a text file one screen full at a time

**pivot_root**

Makes the given file system the new root file system of the current process

**prlimit**

Get and set a process' resource limits

**raw**

Bind a Linux raw character device to a block device

**readprofile**

Reads kernel profiling information

**rename**

Renames the given files, replacing a given string with another

**renice**

Alters the priority of running processes

**resizepart**

Asks the Linux kernel to resize a partition

**rev**

Reverses the lines of a given file

**rtcwake**

Used to enter a system sleep state until specified wakeup time

**script**

Makes a typescript of a terminal session

**scriptreplay**

Plays back typescripts using timing information

**setarch**

Changes reported architecture in a new program environment and sets personality flags

**setsid**

Runs the given program in a new session

**setterm**

Sets terminal attributes

**sfdisk**

A disk partition table manipulator

**sulogin**

Allows `root` to log in; it is normally invoked by **init** when the system goes into single user mode

**swaplabel**

Allows to change swaparea UUID and label

**swapoff**

Disables devices and files for paging and swapping

**swapon**

Enables devices and files for paging and swapping and lists the devices and files currently in use

**switch_root**

Switches to another filesystem as the root of the mount tree

**tailf**

Tracks the growth of a log file; displays the last 10 lines of a log file, then continues displaying any new entries in the log file as they are created

**taskset**

Retrieves or sets a process' CPU affinity

**ul**

A filter for translating underscores into escape sequences indicating underlining for the terminal in use

**umount**

Disconnects a file system from the system's file tree

**uname26**

A symbolic link to setarch

**unshare**

Runs a program with some namespaces unshared from parent

**utmpdump**

Displays the content of the given login file in a more user-friendly format

**uuidd**

A daemon used by the UUID library to generate time-based UUIDs in a secure and guranteed-unique fashion

**uuidgen**

Creates new UUIDs. Each new UUID can reasonably be considered unique among all UUIDs created, on the local system and on other systems, in the past and in the future

**wall**

Displays the contents of a file or, by default, its standard input, on the terminals of all currently logged in users

**wdctl**

Shows hardware watchdog status

**whereis**

Reports the location of the binary, source, and man page for the given command

**wipefs**

Wipes a filesystem signature from a device

**x86_64**

A symbolic link to setarch

**zramctl**

A program to set up and control zram (compressed ram disk) devices

`libblkid`

Contains routines for device identification and token extraction

`libfdisk`

Contains routines for manipulating partition tables

`libmount`

Contains routines for block device mounting and unmounting

`libsmartcols`

Contains routines for aiding screen output in tabular form

`libuuid`

Contains routines for generating unique identifiers for objects that may be accessible beyond the local system

Last updated on

## 6.67. Man-DB-2.7.6.1

The Man-DB package contains programs for finding and viewing man pages.

**Approximate build time:**0.4 SBU

**Required disk space:**30 MB

### 6.67.1. Installation of Man-DB

Prepare Man-DB for compilation:

    ./configure --prefix=/usr                        \
                --docdir=/usr/share/doc/man-db-2.7.6.1 \
                --sysconfdir=/etc                    \
                --disable-setuid                     \
                --enable-cache-owner=bin             \
                --with-browser=/usr/bin/lynx         \
                --with-vgrind=/usr/bin/vgrind        \
                --with-grap=/usr/bin/grap            \
                --with-systemdtmpfilesdir=

**The meaning of the configure options:**
*`--disable-setuid`*
This disables making the **man** program setuid to user `man`.
*`--enable-cache-owner=bin`*
This makes the system-wide cache files be owned by user bin.
*`--with-...`*
These three parameters are used to set some default programs. **lynx** is a text-based web browser (see BLFS for installation instructions), **vgrind** converts program sources to Groff input, and **grap** is useful for typesetting graphs in Groff documents. The **vgrind** and **grap** programs are not normally needed for viewing manual pages. They are not part of LFS or BLFS, but you should be able to install them yourself after finishing LFS if you wish to do so.

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

### 6.67.2. Non-English Manual Pages in LFS

The following table shows the character set that Man-DB assumes manual pages installed under `/usr/share/man/<ll>` will be encoded with. In addition to this, Man-DB correctly determines if manual pages installed in that directory are UTF-8 encoded.

**Table 6.1. Expected character encoding of legacy 8-bit manual pages**

Language (code)EncodingLanguage (code)EncodingDanish (da)ISO-8859-1Croatian (hr)ISO-8859-2German (de)ISO-8859-1Hungarian (hu)ISO-8859-2English (en)ISO-8859-1Japanese (ja)EUC-JPSpanish (es)ISO-8859-1Korean (ko)EUC-KREstonian (et)ISO-8859-1Lithuanian (lt)ISO-8859-13Finnish (fi)ISO-8859-1Latvian (lv)ISO-8859-13French (fr)ISO-8859-1Macedonian (mk)ISO-8859-5Irish (ga)ISO-8859-1Polish (pl)ISO-8859-2Galician (gl)ISO-8859-1Romanian (ro)ISO-8859-2Indonesian (id)ISO-8859-1Russian (ru)KOI8-RIcelandic (is)ISO-8859-1Slovak (sk)ISO-8859-2Italian (it)ISO-8859-1Slovenian (sl)ISO-8859-2Norwegian Bokmal (nb)ISO-8859-1Serbian Latin (sr@latin)ISO-8859-2Dutch (nl)ISO-8859-1Serbian (sr)ISO-8859-5Norwegian Nynorsk (nn)ISO-8859-1Turkish (tr)ISO-8859-9Norwegian (no)ISO-8859-1Ukrainian (uk)KOI8-UPortuguese (pt)ISO-8859-1Vietnamese (vi)TCVN5712-1Swedish (sv)ISO-8859-1Simplified Chinese (zh_CN)GBKBelarusian (be)CP1251Simplified Chinese, Singapore (zh_SG)GBKBulgarian (bg)CP1251Traditional Chinese, Hong Kong (zh_HK)BIG5HKSCSCzech (cs)ISO-8859-2Traditional Chinese (zh_TW)BIG5Greek (el)ISO-8859-7

### Note

Manual pages in languages not in the list are not supported.

### 6.67.3. Contents of Man-DB

**Installed programs:**accessdb, apropos (link to whatis), catman, lexgrog, man, mandb, manpath, and whatis

**Installed libraries:**libman.so and libmandb.so

**Installed directories:**/usr/lib/man-db, /usr/lib/tmpfiles.d, /usr/libexec/man-db, and /usr/share/doc/man-db-2.7.6.1

#### Short Descriptions

**accessdb**

Dumps the **whatis** database contents in human-readable form

**apropos**

Searches the **whatis** database and displays the short descriptions of system commands that contain a given string

**catman**

Creates or updates the pre-formatted manual pages

**lexgrog**

Displays one-line summary information about a given manual page

**man**

Formats and displays the requested manual page

**mandb**

Creates or updates the **whatis** database

**manpath**

Displays the contents of $MANPATH or (if $MANPATH is not set) a suitable search path based on the settings in man.conf and the user's environment

**whatis**

Searches the **whatis** database and displays the short descriptions of system commands that contain the given keyword as a separate word

`libman`

Contains run-time support for **man**

`libmandb`

Contains run-time support for **man**

Last updated on

## 6.68. Tar-1.29

The Tar package contains an archiving program.

**Approximate build time:**2.6 SBU

**Required disk space:**39 MB

### 6.68.1. Installation of Tar

Prepare Tar for compilation:

    FORCE_UNSAFE_CONFIGURE=1  \
    ./configure --prefix=/usr \
                --bindir=/bin

**The meaning of the configure options:**
`FORCE_UNSAFE_CONFIGURE=1`
This forces the test for `mknod` to be run as root. It is generally considered dangerous to run this test as the root user, but as it is being run on a system that has only been partially built, overriding it is OK.

Compile the package:

    make

To test the results (about 1 SBU), issue:

    make check

Install the package:

    make install
    make -C doc install-html docdir=/usr/share/doc/tar-1.29

### 6.68.2. Contents of Tar

**Installed programs:**tar

**Installed directory:**/usr/share/doc/tar-1.29

#### Short Descriptions

**tar**

Creates, extracts files from, and lists the contents of archives, also known as tarballs

Last updated on

## 6.69. Texinfo-6.4

The Texinfo package contains programs for reading, writing, and converting info pages.

**Approximate build time:**1.1 SBU

**Required disk space:**128 MB

### 6.69.1. Installation of Texinfo

Prepare Texinfo for compilation:

    ./configure --prefix=/usr --disable-static

**The meaning of the configure options:**
*`--disable-static`*
In this case, the top-level configure script will complain that this is an unrecognized option, but the configure script for XSParagraph recognizes it and uses it to disable installing a static `XSParagraph.a` to `/usr/lib/texinfo`.

Compile the package:

    make

To test the results, issue:

    make check

Install the package:

    make install

Optionally, install the components belonging in a TeX installation:

    make TEXMF=/usr/share/texmf install-tex

**The meaning of the make parameter:**
*`TEXMF=/usr/share/texmf`*
The `TEXMF` makefile variable holds the location of the root of the TeX tree if, for example, a TeX package will be installed later.

The Info documentation system uses a plain text file to hold its list of menu entries. The file is located at `/usr/share/info/dir`. Unfortunately, due to occasional problems in the Makefiles of various packages, it can sometimes get out of sync with the info pages installed on the system. If the `/usr/share/info/dir` file ever needs to be recreated, the following optional commands will accomplish the task:

    pushd /usr/share/info
    rm -v dir
    for f in *
      do install-info $f dir 2>/dev/null
    done
    popd

### 6.69.2. Contents of Texinfo

**Installed programs:**info, install-info, makeinfo (link to texi2any), pdftexi2dvi, pod2texi, texi2any, texi2dvi, texi2pdf, and texindex

**Installed library:**XSParagraph.so

**Installed directories:**/usr/share/texinfo and /usr/lib/texinfo

#### Short Descriptions

**info**

Used to read info pages which are similar to man pages, but often go much deeper than just explaining all the available command line options [For example, compare **man bison** and **info bison**.]

**install-info**

Used to install info pages; it updates entries in the **info** index file

**makeinfo**

Translates the given Texinfo source documents into info pages, plain text, or HTML

**pdftexi2dvi**

Used to format the given Texinfo document into a Portable Document Format (PDF) file

**pod2texi**

Converts Pod to Texinfo format

**texi2any**

Translate Texinfo source documentation to various other formats

**texi2dvi**

Used to format the given Texinfo document into a device-independent file that can be printed

**texi2pdf**

Used to format the given Texinfo document into a Portable Document Format (PDF) file

**texindex**

Used to sort Texinfo index files

Last updated on

## 6.70. Vim-8.0.586

The Vim package contains a powerful text editor.

**Approximate build time:**1.1 SBU

**Required disk space:**128 MB

### Alternatives to Vim

If you prefer another editor—such as Emacs, Joe, or Nano—please refer to [http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/editors.html](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/editors.html) for suggested installation instructions.

### 6.70.1. Installation of Vim

First, change the default location of the `vimrc` configuration file to `/etc`:

    echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h

Disable a test that fails:

    sed -i '/call/{s/split/xsplit/;s/303/492/}' src/testdir/test_recover.vim

Prepare Vim for compilation:

    ./configure --prefix=/usr

Compile the package:

    make

To test the results, issue:

    make -j1 test &> vim-test.log

However, this test suite outputs a lot of binary data to the screen, which can cause issues with the settings of the current terminal. This can be resolved by redirecting the output to a log file. A successful test will result in the words "ALL DONE" at completion.

Install the package:

    make install

Many users are used to using **vi** instead of **vim**. To allow execution of **vim** when users habitually enter **vi**, create a symlink for both the binary and the man page in the provided languages:

    ln -sv vim /usr/bin/vi
    for L in  /usr/share/man/{,*/}man1/vim.1; do
        ln -sv vim.1 $(dirname $L)/vi.1
    done

By default, Vim's documentation is installed in `/usr/share/vim`. The following symlink allows the documentation to be accessed via `/usr/share/doc/vim-8.0.586`, making it consistent with the location of documentation for other packages:

    ln -sv ../vim/vim80/doc /usr/share/doc/vim-8.0.586

If an X Window System is going to be installed on the LFS system, it may be necessary to recompile Vim after installing X. Vim comes with a GUI version of the editor that requires X and some additional libraries to be installed. For more information on this process, refer to the Vim documentation and the Vim installation page in the BLFS book at [http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/vim.html](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/vim.html).

### 6.70.2. Configuring Vim

By default, **vim** runs in vi-incompatible mode. This may be new to users who have used other editors in the past. The “nocompatible” setting is included below to highlight the fact that a new behavior is being used. It also reminds those who would change to “compatible” mode that it should be the first setting in the configuration file. This is necessary because it changes other settings, and overrides must come after this setting. Create a default **vim** configuration file by running the following:

    cat > /etc/vimrc << "EOF"
    " Begin /etc/vimrc

    set nocompatible
    set backspace=2
    set mouse=r
    syntax on
    if (&term == "xterm") || (&term == "putty")
      set background=dark
    endif


    " End /etc/vimrc
    EOF

    touch ~/.vimrc

The *`set nocompatible`* setting makes **vim** behave in a more useful way (the default) than the vi-compatible manner. Remove the “no” to keep the old **vi** behavior. The *`set backspace=2`* setting allows backspacing over line breaks, autoindents, and the start of insert. The *`syntax on`* parameter enables vim's syntax highlighting. The *`set mouse=r`* setting enables proper pasting of text with the mouse when working in chroot or over a remote connection. Finally, the *if* statement with the *`set background=dark`* setting corrects **vim**'s guess about the background color of some terminal emulators. This gives the highlighting a better color scheme for use on the black background of these programs.

Creating an empty `~/.vimrc` prevents vim from overriding settings in `/etc/vimrc` by using `/usr/share/vim/vim80/defaults.vim`.

Documentation for other available options can be obtained by running the following command:

    vim -c ':options'

### Note

By default, Vim only installs spell files for the English language. To install spell files for your preferred language, download the `*.spl` and optionally, the `*.sug` files for your language and character encoding from[ftp://ftp.vim.org/pub/vim/runtime/spell/](ftp://ftp.vim.org/pub/vim/runtime/spell/) and save them to `/usr/share/vim/vim80/spell/`.

To use these spell files, some configuration in `/etc/vimrc` is needed, e.g.:

    set spelllang=en,ru
    set spell

For more information, see the appropriate README file located at the URL above.

### 6.70.3. Contents of Vim

**Installed programs:**ex (link to vim), rview (link to vim), rvim (link to vim), vi (link to vim), view (link to vim), vim, vimdiff (link to vim), vimtutor, and xxd

**Installed directory:**/usr/share/vim

#### Short Descriptions

**ex**

Starts **vim** in ex mode

**rview**

Is a restricted version of **view**; no shell commands can be started and **view** cannot be suspended

**rvim**

Is a restricted version of **vim**; no shell commands can be started and **vim** cannot be suspended

**vi**

Link to **vim**

**view**

Starts **vim** in read-only mode

**vim**

Is the editor

**vimdiff**

Edits two or three versions of a file with **vim** and show differences

**vimtutor**

Teaches the basic keys and commands of **vim**

**xxd**

Creates a hex dump of the given file; it can also do the reverse, so it can be used for binary patching

Last updated on

## 6.71. About Debugging Symbols

Most programs and libraries are, by default, compiled with debugging symbols included (with **gcc**'s *`-g`* option). This means that when debugging a program or library that was compiled with debugging information included, the debugger can provide not only memory addresses, but also the names of the routines and variables.

However, the inclusion of these debugging symbols enlarges a program or library significantly. The following is an example of the amount of space these symbols occupy:

-
A **bash** binary with debugging symbols: 1200 KB

-
A **bash** binary without debugging symbols: 480 KB

-
Glibc and GCC files (`/lib` and `/usr/lib`) with debugging symbols: 87 MB

-
Glibc and GCC files without debugging symbols: 16 MB

Sizes may vary depending on which compiler and C library were used, but when comparing programs with and without debugging symbols, the difference will usually be a factor between two and five.

Because most users will never use a debugger on their system software, a lot of disk space can be regained by removing these symbols. The next section shows how to strip all debugging symbols from the programs and libraries.

## 6.72. Stripping Again

This section is optional. If the intended user is not a programmer and does not plan to do any debugging on the system software, the system size can be decreased by about 90 MB by removing the debugging symbols from binaries and libraries. This causes no inconvenience other than not being able to debug the software fully anymore.

Most people who use the commands mentioned below do not experience any difficulties. However, it is easy to make a typo and render the new system unusable, so before running the **strip** commands, it is a good idea to make a backup of the LFS system in its current state.

First place the debugging symbols for selected libraries in separate files. This debugging information is needed if running regression tests that use [valgrind](http://www.linuxfromscratch.org/blfs/view/8.1//general/valgrind.html) or [gdb](http://www.linuxfromscratch.org/blfs/view/8.1//general/gdb.html) later in BLFS.

    save_lib="ld-2.26.so libc-2.26.so libpthread-2.26.so libthread_db-1.0.so"

    cd /lib

    for LIB in $save_lib; do
        objcopy --only-keep-debug $LIB $LIB.dbg
        strip --strip-unneeded $LIB
        objcopy --add-gnu-debuglink=$LIB.dbg $LIB
    done

    save_usrlib="libquadmath.so.0.0.0 libstdc++.so.6.0.24
                 libmpx.so.2.0.1 libmpxwrappers.so.2.0.1 libitm.so.1.0.0
                 libcilkrts.so.5.0.0 libatomic.so.1.2.0"

    cd /usr/lib

    for LIB in $save_usrlib; do
        objcopy --only-keep-debug $LIB $LIB.dbg
        strip --strip-unneeded $LIB
        objcopy --add-gnu-debuglink=$LIB.dbg $LIB
    done

    unset LIB save_lib save_usrlib

Before performing the stripping, take special care to ensure that none of the binaries that are about to be stripped are running. If unsure whether the user entered chroot with the command given in [Section 6.4, “Entering the Chroot Environment,”](Linux%20From%20Scratch.html#ch-system-chroot) first exit from chroot:

    logout

Then reenter it with:

    chroot $LFS /tools/bin/env -i            \
        HOME=/root TERM=$TERM PS1='\u:\w\$ ' \
        PATH=/bin:/usr/bin:/sbin:/usr/sbin   \
        /tools/bin/bash --login

Now the binaries and libraries can be safely stripped:

    /tools/bin/find /usr/lib -type f -name \*.a \
       -exec /tools/bin/strip --strip-debug {} ';'

    /tools/bin/find /lib /usr/lib -type f \( -name \*.so* -a ! -name \*dbg \) \
       -exec /tools/bin/strip --strip-unneeded {} ';'

    /tools/bin/find /{bin,sbin} /usr/{bin,sbin,libexec} -type f \
        -exec /tools/bin/strip --strip-all {} ';'

A large number of files will be reported as having their file format not recognized. These warnings can be safely ignored. These warnings indicate that those files are scripts instead of binaries.

## 6.73. Cleaning Up

Finally, clean up some extra files left around from running tests:

    rm -rf /tmp/*

From now on, when reentering the chroot environment after exiting, use the following modified chroot command:

    chroot "$LFS" /usr/bin/env -i              \
        HOME=/root TERM="$TERM" PS1='\u:\w\$ ' \
        PATH=/bin:/usr/bin:/sbin:/usr/sbin     \
        /bin/bash --login

The reason for this is that the programs in `/tools` are no longer needed. Since they are no longer needed you can delete the `/tools` directory if so desired.

### Note

Removing `/tools` will also remove the temporary copies of Tcl, Expect, and DejaGNU which were used for running the toolchain tests. If you need these programs later on, they will need to be recompiled and re-installed. The BLFS book has instructions for this (see [http://www.linuxfromscratch.org/blfs/](http://www.linuxfromscratch.org/blfs/)).

If the virtual kernel file systems have been unmounted, either manually or through a reboot, ensure that the virtual kernel file systems are mounted when reentering the chroot. This process was explained in [Section 6.2.2, “Mounting and Populating /dev”](Linux%20From%20Scratch.html#ch-system-bindmount) and [Section 6.2.3, “Mounting Virtual Kernel File Systems”](Linux%20From%20Scratch.html#ch-system-kernfsmount).

Finally, there were several static libraries that were not suppressed earlier in the chapter in order to satisfy the regression tests in several packages. These libraries are from binutils, bzip2, e2fsprogs, flex, libtool, and zlib. If desired, remove them now:

    rm -f /usr/lib/lib{bfd,opcodes}.a
    rm -f /usr/lib/libbz2.a
    rm -f /usr/lib/lib{com_err,e2p,ext2fs,ss}.a
    rm -f /usr/lib/libltdl.a
    rm -f /usr/lib/libfl.a
    rm -f /usr/lib/libfl_pic.a
    rm -f /usr/lib/libz.a
