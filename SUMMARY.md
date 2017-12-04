# Summary

* [Contents](README.md)
* [Preface](chapter0.md)
  * [Foreword](chapter0.md#1)
  * [Audience](chapter0.md#2)
  * [LFS Target Architectures](chapter0.md#3)
  * [LFS and Standards](chapter0.md#4)
  * [Rationale for Packages in the Book](chapter0.md#5)
  * [Prerequisites](chapter0.md#6)
  * [Typography](chapter0.md#7)
  * [Structure](chapter0.md#8)
  * [Errata](chapter0.md#9)
* [Introduction](chapter1.md)
  * [How to Build a LFS SYSTEM]()
  * [What's new since the last release]()
  * [Changelog]()
  * [Resources]()
  * [Help]()
* [Preparing for the Build](chapter2.md)
  * [Preparing the Host System](chapter2.md)
    * [Introduction](chapter2.md#Introduction)
    * [Host System Requirements]()
    * [Building LFS in Stages](chapter2.md#BuildingLFSinStages)
    * [Creating a New Partition]()
    * [Creating a File System on the Partition]()
    * [Setting The $LFS Variable]()
    * [Mounting the New Partition]()
  * [Packages and Patches](chapter3.md)
    * [Introduction]()
    * [All Packages]()
    * [Needed Patches]()
  * [Final Preparations](chapter4.md)
    * [Introduction]()
    * [Creating the $LFS/tools Directory]()
    * [Adding the LFS User]()
    * [Setting Up the Environment]()
    * [About SBUs]()
    * [About the Test Suites]()
  * [Constructing a Temporary System](chapter5.md)
    * [Introduction]()
    * [Toolchain Technical Notes]()
    * [General Compilation Instructions]()
    * [Binutils-2.29 - Pass 1]()
    * [GCC-7.2.0 - Pass 1]()
    * [Linux-4.12.7 API Headers]()
    * [Glibc-2.26]()
    * [Libstdc++-7.2.0]()
    * [Binutils-2.29 - Pass 2]()
    * [GCC-7.2.0 - Pass 2]()
    * [Tcl-core-8.6.7]()
    * [Expect-5.45]()
    * [DejaGNU-1.6]()
    * [Check-0.11.0]()
    * [Ncurses-6.0]()
    * [Bash-4.4]()
    * [Bison-3.0.4]()
    * [Bzip2-1.0.6]()
    * [Coreutils-8.27]()
    * [Diffutils-3.6]()
    * [File-5.31]()
    * [Findutils-4.6.0]()
    * [Gawk-4.1.4]()
    * [Gettext-0.19.8.1]()
    * [Grep-3.1]()
    * [Gzip-1.8]()
    * [M4-1.4.18]()
    * [Make-4.2.1]()
    * [Patch-2.7.5]()
    * [Perl-5.26.0]()
    * [Sed-4.4]()
    * [Tar-1.29]()
    * [Texinfo-6.4]()
    * [Util-linux-2.30.1]()
    * [Xz-5.2.3]()
    * [Stripping]()
    * [Changing Ownership]()
* [Building the LFS System](chapter6.md)
  * [Installing Basic System Software](chapter6.md)
    * [Introduction]()
    * [Preparing Virtual Kernel File Systems]()
    * [Package Management]()
    * [Entering the Chroot Environment]()
    * [Creating Directories]()
    * [Creating Essential Files and Symlinks]()
    * [Linux-4.12.7 API Headers]()
    * [Man-pages-4.12]()
    * [Glibc-2.26]()
    * [Adjusting the Toolchain]()
    * [Zlib-1.2.11]()
    * [File-5.31]()
    * [Readline-7.0]()
    * [M4-1.4.18]()
    * [Bc-1.07.1]()
    * [Binutils-2.29]()
    * [GMP-6.1.2]()
    * [MPFR-3.1.5]()
    * [MPC-1.0.3]()
    * [GCC-7.2.0]()
    * [Bzip2-1.0.6]()
    * [Pkg-config-0.29.2]()
    * [Ncurses-6.0]()
    * [Attr-2.4.47]()
    * [Acl-2.2.52]()
    * [Libcap-2.25]()
    * [Sed-4.4]()
    * [Shadow-4.5]()
    * [Psmisc-23.1]()
    * [Iana-Etc-2.30]()
    * [Bison-3.0.4]()
    * [Flex-2.6.4]()
    * [Grep-3.1]()
    * [Bash-4.4]()
    * [Libtool-2.4.6]()
    * [GDBM-1.13]()
    * [Gperf-3.1]()
    * [Expat-2.2.3]()
    * [Inetutils-1.9.4]()
    * [Perl-5.26.0]()
    * [XML::Parser-2.44]()
    * [Intltool-0.51.0]()
    * [Autoconf-2.69]()
    * [Automake-1.15.1]()
    * [Xz-5.2.3]()
    * [Kmod-24]()
    * [Gettext-0.19.8.1]()
    * [Procps-ng-3.3.12]()
    * [E2fsprogs-1.43.5]()
    * [Coreutils-8.27]()
    * [Diffutils-3.6]()
    * [Gawk-4.1.4]()
    * [Findutils-4.6.0]()
    * [Groff-1.22.3]()
    * [GRUB-2.02]()
    * [Less-487]()
    * [Gzip-1.8]()
    * [IPRoute2-4.12.0]()
    * [Kbd-2.0.4]()
    * [Libpipeline-1.4.2]()
    * [Make-4.2.1]()
    * [Patch-2.7.5]()
    * [Sysklogd-1.5.1]()
    * [Sysvinit-2.88dsf]()
    * [Eudev-3.2.2]()
    * [Util-linux-2.30.1]()
    * [Man-DB-2.7.6.1]()
    * [Tar-1.29]()
    * [Texinfo-6.4]()
    * [Vim-8.0.586]()
    * [About Debugging Symbols]()
    * [Stripping Again]()
    * [Cleaning Up]()
  * [System Configuration](chapter7.md)
    * [Introduction]()
    * [LFS-Bootscripts-20170626]()
    * [Overview of Device and Module Handling]()
    * [Managing Devices]()
    * [General Network Configuration]()
    * [System V Bootscript Usage and Configuration]()
    * [The Bash Shell Startup Files]()
    * [Creating the /etc/inputrc File]()
    * [Creating the /etc/shells File]()
  * [Making the LFS System Bootable](chapter8.md)
    * [Introduction]()
    * [Creating the /etc/fstab File]()
    * [Linux-4.12.7]()
    * [Using GRUB to Set Up the Boot Process]()
  * [The End](chapter9.md)
    * [The End]()
    * [Get Counted]()
    * [Rebooting the System]()
    * [What Now?]()
* [Appendices](chapter10.md)
  * [A. Acronyms and Terms]()
  * [B. Acknowledgments]()
  * [C. Dependencies]()
  * [D. Boot and sysconfig scripts version-20170626]()
    * [/etc/rc.d/init.d/rc]()
    * [/lib/lsb/init-functions]()
    * [/etc/rc.d/init.d/mountvirtfs]()
    * [/etc/rc.d/init.d/modules]()
    * [/etc/rc.d/init.d/udev]()
    * [/etc/rc.d/init.d/swap]()
    * [/etc/rc.d/init.d/setclock]()
    * [/etc/rc.d/init.d/checkfs]()
    * [/etc/rc.d/init.d/mountfs]()
    * [/etc/rc.d/init.d/udev\_retry]()
    * [/etc/rc.d/init.d/cleanfs]()
    * [/etc/rc.d/init.d/console]()
    * [/etc/rc.d/init.d/localnet]()
    * [/etc/rc.d/init.d/sysctl]()
    * [/etc/rc.d/init.d/sysklogd]()
    * [/etc/rc.d/init.d/network]()
    * [/etc/rc.d/init.d/sendsignals]()
    * [/etc/rc.d/init.d/reboot]()
    * [/etc/rc.d/init.d/halt]()
    * [/etc/rc.d/init.d/template]()
    * [/etc/sysconfig/modules]()
    * [/etc/sysconfig/createfiles]()
    * [/etc/sysconfig/udev-retry]()
    * [/sbin/ifup]()
    * [/sbin/ifdown]()
    * [/lib/services/ipv4-static]()
    * [/lib/services/ipv4-static-route]()
  * [E. Udev configuration rules]()
    * [55-lfs.rules]()
  * [F. LFS Licenses]()
    * [Creative Commons License]()
    * [The MIT License]()