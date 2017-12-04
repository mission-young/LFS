## Chapter 3. Packages and Patches

## 3.1. Introduction

This chapter includes a list of packages that need to be downloaded in order to build a basic Linux system. The listed version numbers correspond to versions of the software that are known to work, and this book is based on their use. We highly recommend against using newer versions because the build commands for one version may not work with a newer version. The newest package versions may also have problems that require work-arounds. These work-arounds will be developed and stabilized in the development version of the book.

Download locations may not always be accessible. If a download location has changed since this book was published, Google ([http://www.google.com/](http://www.google.com/)) provides a useful search engine for most packages. If this search is unsuccessful, try one of the alternative means of downloading discussed at [http://www.linuxfromscratch.org/lfs/packages.html#packages](http://www.linuxfromscratch.org/lfs/packages.html#packages).

Downloaded packages and patches will need to be stored somewhere that is conveniently available throughout the entire build. A working directory is also required to unpack the sources and build them. `$LFS/sources` can be used both as the place to store the tarballs and patches and as a working directory. By using this directory, the required elements will be located on the LFS partition and will be available during all stages of the building process.

To create this directory, execute the following command, as user `root`, before starting the download session:

    mkdir -v $LFS/sources

Make this directory writable and sticky. “Sticky” means that even if multiple users have write permission on a directory, only the owner of a file can delete the file within a sticky directory. The following command will enable the write and sticky modes:

    chmod -v a+wt $LFS/sources

An easy way to download all of the packages and patches is by using [wget-list](http://www.linuxfromscratch.org/lfs/downloads/stable/wget-list) as an input to **wget**. For example:

    wget --input-file=wget-list --continue --directory-prefix=$LFS/sources

Additionally, starting with LFS-7.0, there is a separate file, [md5sums](http://www.linuxfromscratch.org/lfs/downloads/stable/md5sums), which can be used to verify that all the correct packages are available before proceeding. Place that file in `$LFS/sources` and run:

    pushd $LFS/sources
    md5sum -c md5sums
    popd

## 3.2. All Packages

Download or otherwise obtain the following packages:

Acl (2.2.52) - 380 KB:
Download: [http://download.savannah.gnu.org/releases/acl/acl-2.2.52.src.tar.gz](http://download.savannah.gnu.org/releases/acl/acl-2.2.52.src.tar.gz)

MD5 sum: `a61415312426e9c2212bd7dc7929abda`
Attr (2.4.47) - 336 KB:
Home page: [http://savannah.nongnu.org/projects/attr](http://savannah.nongnu.org/projects/attr)

Download: [http://download.savannah.gnu.org/releases/attr/attr-2.4.47.src.tar.gz](http://download.savannah.gnu.org/releases/attr/attr-2.4.47.src.tar.gz)

MD5 sum: `84f58dec00b60f2dc8fd1c9709291cc7`
Autoconf (2.69) - 1,186 KB:
Home page: [http://www.gnu.org/software/autoconf/](http://www.gnu.org/software/autoconf/)

Download: [http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.xz](http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.xz)

MD5 sum: `50f97f4159805e374639a73e2636f22e`
Automake (1.15.1) - 1,475 KB:
Home page: [http://www.gnu.org/software/automake/](http://www.gnu.org/software/automake/)

Download: [http://ftp.gnu.org/gnu/automake/automake-1.15.1.tar.xz](http://ftp.gnu.org/gnu/automake/automake-1.15.1.tar.xz)

MD5 sum: `24cd3501b6ad8cd4d7e2546f07e8b4d4`
Bash (4.4) - 9,158 KB:
Home page: [http://www.gnu.org/software/bash/](http://www.gnu.org/software/bash/)

Download: [http://ftp.gnu.org/gnu/bash/bash-4.4.tar.gz](http://ftp.gnu.org/gnu/bash/bash-4.4.tar.gz)

MD5 sum: `148888a7c95ac23705559b6f477dfe25`
Bc (1.07.1) - 411 KB:
Home page: [http://www.gnu.org/software/bc/](http://www.gnu.org/software/bc/)

Download: [http://ftp.gnu.org/gnu/bc/bc-1.07.1.tar.gz](http://ftp.gnu.org/gnu/bc/bc-1.07.1.tar.gz)

MD5 sum: `cda93857418655ea43590736fc3ca9fc`
Binutils (2.29) - 28,392 KB:
Home page: [http://www.gnu.org/software/binutils/](http://www.gnu.org/software/binutils/)

Download: [http://ftp.gnu.org/gnu/binutils/binutils-2.29.tar.bz2](http://ftp.gnu.org/gnu/binutils/binutils-2.29.tar.bz2)

MD5 sum: `23733a26c8276edbb1168c9bee60e40e`
Bison (3.0.4) - 1,928 KB:
Home page: [http://www.gnu.org/software/bison/](http://www.gnu.org/software/bison/)

Download: [http://ftp.gnu.org/gnu/bison/bison-3.0.4.tar.xz](http://ftp.gnu.org/gnu/bison/bison-3.0.4.tar.xz)

MD5 sum: `c342201de104cc9ce0a21e0ad10d4021`
Bzip2 (1.0.6) - 764 KB:
Home page: [http://www.bzip.org/](http://www.bzip.org/)

Download: [http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz](http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz)

MD5 sum: `00b516f4704d4a7cb50a1d97e6e8e15b`
Check (0.11.0) - 736 KB:
Home page: [https://libcheck.github.io/check](https://libcheck.github.io/check)

Download: [https://github.com/libcheck/check/releases/download/0.11.0/check-0.11.0.tar.gz](https://github.com/libcheck/check/releases/download/0.11.0/check-0.11.0.tar.gz)

MD5 sum: `9b90522b31f5628c2e0f55dda348e558`
Coreutils (8.27) - 5,162 KB:
Home page: [http://www.gnu.org/software/coreutils/](http://www.gnu.org/software/coreutils/)

Download: [http://ftp.gnu.org/gnu/coreutils/coreutils-8.27.tar.xz](http://ftp.gnu.org/gnu/coreutils/coreutils-8.27.tar.xz)

MD5 sum: `502795792c212932365e077946d353ae`
DejaGNU (1.6) - 512 KB:
Home page: [http://www.gnu.org/software/dejagnu/](http://www.gnu.org/software/dejagnu/)

Download: [http://ftp.gnu.org/gnu/dejagnu/dejagnu-1.6.tar.gz](http://ftp.gnu.org/gnu/dejagnu/dejagnu-1.6.tar.gz)

MD5 sum: `1fdc2eb0d592c4f89d82d24dfdf02f0b`
Diffutils (3.6) - 1,366 KB:
Home page: [http://www.gnu.org/software/diffutils/](http://www.gnu.org/software/diffutils/)

Download: [http://ftp.gnu.org/gnu/diffutils/diffutils-3.6.tar.xz](http://ftp.gnu.org/gnu/diffutils/diffutils-3.6.tar.xz)

MD5 sum: `07cf286672ced26fba54cd0313bdc071`
Eudev (3.2.2) - 1,780 KB:
Download: [http://dev.gentoo.org/~blueness/eudev/eudev-3.2.2.tar.gz](http://dev.gentoo.org/~blueness/eudev/eudev-3.2.2.tar.gz)

MD5 sum: `41e19b70462692fefd072a3f38818b6e`
E2fsprogs (1.43.5) - 7,425 KB:
Home page: [http://e2fsprogs.sourceforge.net/](http://e2fsprogs.sourceforge.net/)

Download: [http://downloads.sourceforge.net/project/e2fsprogs/e2fsprogs/v1.43.5/e2fsprogs-1.43.5.tar.gz](http://downloads.sourceforge.net/project/e2fsprogs/e2fsprogs/v1.43.5/e2fsprogs-1.43.5.tar.gz)

MD5 sum: `40aa1b7d7d6bd9c71db0fbf325a7c199`
Expat (2.2.3) - 426 KB:
Home page: [http://expat.sourceforge.net/](http://expat.sourceforge.net/)

Download: [http://prdownloads.sourceforge.net/expat/expat-2.2.3.tar.bz2](http://prdownloads.sourceforge.net/expat/expat-2.2.3.tar.bz2)

MD5 sum: `f053af63ef5f39bd9b78d01fbc203334`
Expect (5.45) - 614 KB:
Home page: [http://expect.sourceforge.net/](http://expect.sourceforge.net/)

Download: [http://prdownloads.sourceforge.net/expect/expect5.45.tar.gz](http://prdownloads.sourceforge.net/expect/expect5.45.tar.gz)

MD5 sum: `44e1a4f4c877e9ddc5a542dfa7ecc92b`
File (5.31) - 774 KB:
Home page: [http://www.darwinsys.com/file/](http://www.darwinsys.com/file/)

Download: [ftp://ftp.astron.com/pub/file/file-5.31.tar.gz](ftp://ftp.astron.com/pub/file/file-5.31.tar.gz)

MD5 sum: `319627d20c9658eae85b056115b8c90a`

### Note

File (5.31) may no longer be available at the listed location. The site administrators of the master download location occasionally remove older versions when new ones are released. An alternative download location that may have the correct version available can also be found at: [http://www.linuxfromscratch.org/lfs/download.html#ftp](http://www.linuxfromscratch.org/lfs/download.html#ftp).

Findutils (4.6.0) - 3,692 KB:
Home page: [http://www.gnu.org/software/findutils/](http://www.gnu.org/software/findutils/)

Download: [http://ftp.gnu.org/gnu/findutils/findutils-4.6.0.tar.gz](http://ftp.gnu.org/gnu/findutils/findutils-4.6.0.tar.gz)

MD5 sum: `9936aa8009438ce185bea2694a997fc1`
Flex (2.6.4) - 1,386 KB:
Home page: [http://flex.sourceforge.net](http://flex.sourceforge.net/)

Download: [https://github.com/westes/flex/releases/download/v2.6.4/flex-2.6.4.tar.gz](https://github.com/westes/flex/releases/download/v2.6.4/flex-2.6.4.tar.gz)

MD5 sum: `2882e3179748cc9f9c23ec593d6adc8d`
Gawk (4.1.4) - 2,313 KB:
Home page: [http://www.gnu.org/software/gawk/](http://www.gnu.org/software/gawk/)

Download: [http://ftp.gnu.org/gnu/gawk/gawk-4.1.4.tar.xz](http://ftp.gnu.org/gnu/gawk/gawk-4.1.4.tar.xz)

MD5 sum: `4e7dbc81163e60fd4f0b52496e7542c9`
GCC (7.2.0) - 60,853 KB:
Home page: [http://gcc.gnu.org/](http://gcc.gnu.org/)

Download: [http://ftp.gnu.org/gnu/gcc/gcc-7.2.0/gcc-7.2.0.tar.xz](http://ftp.gnu.org/gnu/gcc/gcc-7.2.0/gcc-7.2.0.tar.xz)

MD5 sum: `ff370482573133a7fcdd96cd2f552292`
GDBM (1.13) - 872 KB:
Home page: [http://www.gnu.org/software/gdbm/](http://www.gnu.org/software/gdbm/)

Download: [http://ftp.gnu.org/gnu/gdbm/gdbm-1.13.tar.gz](http://ftp.gnu.org/gnu/gdbm/gdbm-1.13.tar.gz)

MD5 sum: `8929dcda2a8de3fd2367bdbf66769376`
Gettext (0.19.8.1) - 7,041 KB:
Home page: [http://www.gnu.org/software/gettext/](http://www.gnu.org/software/gettext/)

Download: [http://ftp.gnu.org/gnu/gettext/gettext-0.19.8.1.tar.xz](http://ftp.gnu.org/gnu/gettext/gettext-0.19.8.1.tar.xz)

MD5 sum: `df3f5690eaa30fd228537b00cb7b7590`
Glibc (2.26) - 14,339 KB:
Home page: [http://www.gnu.org/software/libc/](http://www.gnu.org/software/libc/)

Download: [http://ftp.gnu.org/gnu/glibc/glibc-2.26.tar.xz](http://ftp.gnu.org/gnu/glibc/glibc-2.26.tar.xz)

MD5 sum: `102f637c3812f81111f48f2427611be1`

### Note

This version of glibc addresses a security issue not yet in the latest stable release.

GMP (6.1.2) - 1,901 KB:
Home page: [http://www.gnu.org/software/gmp/](http://www.gnu.org/software/gmp/)

Download: [http://ftp.gnu.org/gnu/gmp/gmp-6.1.2.tar.xz](http://ftp.gnu.org/gnu/gmp/gmp-6.1.2.tar.xz)

MD5 sum: `f58fa8001d60c4c77595fbbb62b63c1d`
Gperf (3.1) - 1,188 KB:
Home page: [http://www.gnu.org/software/gperf/](http://www.gnu.org/software/gperf/)

Download: [http://ftp.gnu.org/gnu/gperf/gperf-3.1.tar.gz](http://ftp.gnu.org/gnu/gperf/gperf-3.1.tar.gz)

MD5 sum: `9e251c0a618ad0824b51117d5d9db87e`
Grep (3.1) - 1,339 KB:
Home page: [http://www.gnu.org/software/grep/](http://www.gnu.org/software/grep/)

Download: [http://ftp.gnu.org/gnu/grep/grep-3.1.tar.xz](http://ftp.gnu.org/gnu/grep/grep-3.1.tar.xz)

MD5 sum: `feca7b3e7c7f4aab2b42ecbfc513b070`
Groff (1.22.3) - 4,091 KB:
Home page: [http://www.gnu.org/software/groff/](http://www.gnu.org/software/groff/)

Download: [http://ftp.gnu.org/gnu/groff/groff-1.22.3.tar.gz](http://ftp.gnu.org/gnu/groff/groff-1.22.3.tar.gz)

MD5 sum: `cc825fa64bc7306a885f2fb2268d3ec5`
GRUB (2.02) - 5,970 KB:
Home page: [http://www.gnu.org/software/grub/](http://www.gnu.org/software/grub/)

Download: [http://ftp.gnu.org/gnu/grub/grub-2.02.tar.xz](http://ftp.gnu.org/gnu/grub/grub-2.02.tar.xz)

MD5 sum: `8a4a2a95aac551fb0fba860ceabfa1d3`
Gzip (1.8) - 712 KB:
Home page: [http://www.gnu.org/software/gzip/](http://www.gnu.org/software/gzip/)

Download: [http://ftp.gnu.org/gnu/gzip/gzip-1.8.tar.xz](http://ftp.gnu.org/gnu/gzip/gzip-1.8.tar.xz)

MD5 sum: `f7caabb65cddc1a4165b398009bd05b9`
Iana-Etc (2.30) - 201 KB:
Home page: [http://freecode.com/projects/iana-etc](http://freecode.com/projects/iana-etc)

Download: [http://anduin.linuxfromscratch.org/LFS/iana-etc-2.30.tar.bz2](http://anduin.linuxfromscratch.org/LFS/iana-etc-2.30.tar.bz2)

MD5 sum: `3ba3afb1d1b261383d247f46cb135ee8`
Inetutils (1.9.4) - 1,333 KB:
Home page: [http://www.gnu.org/software/inetutils/](http://www.gnu.org/software/inetutils/)

Download: [http://ftp.gnu.org/gnu/inetutils/inetutils-1.9.4.tar.xz](http://ftp.gnu.org/gnu/inetutils/inetutils-1.9.4.tar.xz)

MD5 sum: `87fef1fa3f603aef11c41dcc097af75e`
Intltool (0.51.0) - 159 KB:
Home page: [http://freedesktop.org/wiki/Software/intltool](http://freedesktop.org/wiki/Software/intltool)

Download: [http://launchpad.net/intltool/trunk/0.51.0/+download/intltool-0.51.0.tar.gz](http://launchpad.net/intltool/trunk/0.51.0/+download/intltool-0.51.0.tar.gz)

MD5 sum: `12e517cac2b57a0121cda351570f1e63`
IPRoute2 (4.12.0) - 647 KB:
Home page: [https://www.kernel.org/pub/linux/utils/net/iproute2/](https://www.kernel.org/pub/linux/utils/net/iproute2/)

Download: [https://www.kernel.org/pub/linux/utils/net/iproute2/iproute2-4.12.0.tar.xz](https://www.kernel.org/pub/linux/utils/net/iproute2/iproute2-4.12.0.tar.xz)

MD5 sum: `e6fecdf46a1542a26044e756fbbabe3b`
Kbd (2.0.4) - 1,008 KB:
Home page: [http://ftp.altlinux.org/pub/people/legion/kbd](http://ftp.altlinux.org/pub/people/legion/kbd)

Download: [https://www.kernel.org/pub/linux/utils/kbd/kbd-2.0.4.tar.xz](https://www.kernel.org/pub/linux/utils/kbd/kbd-2.0.4.tar.xz)

MD5 sum: `c1635a5a83b63aca7f97a3eab39ebaa6`
Kmod (24) - 525 KB:
Download: [https://www.kernel.org/pub/linux/utils/kernel/kmod/kmod-24.tar.xz](https://www.kernel.org/pub/linux/utils/kernel/kmod/kmod-24.tar.xz)

MD5 sum: `08297dfb6f2b3f625f928ca3278528af`
Less (487) - 312 KB:
Home page: [http://www.greenwoodsoftware.com/less/](http://www.greenwoodsoftware.com/less/)

Download: [http://www.greenwoodsoftware.com/less/less-487.tar.gz](http://www.greenwoodsoftware.com/less/less-487.tar.gz)

MD5 sum: `dcc8bf183a83b362d37fe9ef8df1fb60`
LFS-Bootscripts (20170626) - 32 KB:
Download: [http://www.linuxfromscratch.org/lfs/downloads/8.1/lfs-bootscripts-20170626.tar.bz2](http://www.linuxfromscratch.org/lfs/downloads/8.1/lfs-bootscripts-20170626.tar.bz2)

MD5 sum: `d4992527d67f28e2d0c12e3495422eab`
Libcap (2.25) - 64 KB:
Home page: [https://sites.google.com/site/fullycapable/](https://sites.google.com/site/fullycapable/)

Download: [https://www.kernel.org/pub/linux/libs/security/linux-privs/libcap2/libcap-2.25.tar.xz](https://www.kernel.org/pub/linux/libs/security/linux-privs/libcap2/libcap-2.25.tar.xz)

MD5 sum: `6666b839e5d46c2ad33fc8aa2ceb5f77`
Libpipeline (1.4.2) - 808 KB:
Home page: [http://libpipeline.nongnu.org/](http://libpipeline.nongnu.org/)

Download: [http://download.savannah.gnu.org/releases/libpipeline/libpipeline-1.4.2.tar.gz](http://download.savannah.gnu.org/releases/libpipeline/libpipeline-1.4.2.tar.gz)

MD5 sum: `d5c80387eb9c9e5d089da2a06e8a6b12`
Libtool (2.4.6) - 951 KB:
Home page: [http://www.gnu.org/software/libtool/](http://www.gnu.org/software/libtool/)

Download: [http://ftp.gnu.org/gnu/libtool/libtool-2.4.6.tar.xz](http://ftp.gnu.org/gnu/libtool/libtool-2.4.6.tar.xz)

MD5 sum: `1bfb9b923f2c1339b4d2ce1807064aa5`
Linux (4.12.7) - 96,865 KB:
Home page: [http://www.kernel.org/](http://www.kernel.org/)

Download: [https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.12.7.tar.xz](https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.12.7.tar.xz)

MD5 sum: `245d1b4dc6e82669aac2c9e6a2dd82fe`

### Note

The Linux kernel is updated relatively often, many times due to discoveries of security vulnerabilities. The latest available 4.12.x kernel version should be used, unless the errata page says otherwise.

For users with limited speed or expensive bandwidth who wish to update the Linux kernel, a baseline version of the package and patches can be downloaded separately. This may save some time or cost for a subsequent patch level upgrade within a minor release.

M4 (1.4.18) - 1,180 KB:
Home page: [http://www.gnu.org/software/m4/](http://www.gnu.org/software/m4/)

Download: [http://ftp.gnu.org/gnu/m4/m4-1.4.18.tar.xz](http://ftp.gnu.org/gnu/m4/m4-1.4.18.tar.xz)

MD5 sum: `730bb15d96fffe47e148d1e09235af82`
Make (4.2.1) - 1,375 KB:
Home page: [http://www.gnu.org/software/make/](http://www.gnu.org/software/make/)

Download: [http://ftp.gnu.org/gnu/make/make-4.2.1.tar.bz2](http://ftp.gnu.org/gnu/make/make-4.2.1.tar.bz2)

MD5 sum: `15b012617e7c44c0ed482721629577ac`
Man-DB (2.7.6.1) - 1,506 KB:
Home page: [http://www.nongnu.org/man-db/](http://www.nongnu.org/man-db/)

Download: [http://download.savannah.gnu.org/releases/man-db/man-db-2.7.6.1.tar.xz](http://download.savannah.gnu.org/releases/man-db/man-db-2.7.6.1.tar.xz)

MD5 sum: `2948d49d0ed7265f60f83aa4a9ac9268`
Man-pages (4.12) - 1,552 KB:
Home page: [http://www.kernel.org/doc/man-pages/](http://www.kernel.org/doc/man-pages/)

Download: [https://www.kernel.org/pub/linux/docs/man-pages/man-pages-4.12.tar.xz](https://www.kernel.org/pub/linux/docs/man-pages/man-pages-4.12.tar.xz)

MD5 sum: `a87cdf43ddc1844e7edc8950a28a51f0`
MPC (1.0.3) - 655 KB:
Home page: [http://www.multiprecision.org/](http://www.multiprecision.org/)

Download: [http://www.multiprecision.org/mpc/download/mpc-1.0.3.tar.gz](http://www.multiprecision.org/mpc/download/mpc-1.0.3.tar.gz)

MD5 sum: `d6a1d5f8ddea3abd2cc3e98f58352d26`
MPFR (3.1.5) - 1,101 KB:
Home page: [http://www.mpfr.org/](http://www.mpfr.org/)

Download: [http://www.mpfr.org/mpfr-3.1.5/mpfr-3.1.5.tar.xz](http://www.mpfr.org/mpfr-3.1.5/mpfr-3.1.5.tar.xz)

MD5 sum: `c4ac246cf9795a4491e7766002cd528f`
Ncurses (6.0) - 3,059 KB:
Home page: [http://www.gnu.org/software/ncurses/](http://www.gnu.org/software/ncurses/)

Download: [http://ftp.gnu.org/gnu//ncurses/ncurses-6.0.tar.gz](http://ftp.gnu.org/gnu//ncurses/ncurses-6.0.tar.gz)

MD5 sum: `ee13d052e1ead260d7c28071f46eefb1`
Patch (2.7.5) - 711 KB:
Home page: [http://savannah.gnu.org/projects/patch/](http://savannah.gnu.org/projects/patch/)

Download: [http://ftp.gnu.org/gnu/patch/patch-2.7.5.tar.xz](http://ftp.gnu.org/gnu/patch/patch-2.7.5.tar.xz)

MD5 sum: `e3da7940431633fb65a01b91d3b7a27a`
Perl (5.26.0) - 11,682 KB:
Home page: [http://www.perl.org/](http://www.perl.org/)

Download: [http://www.cpan.org/src/5.0/perl-5.26.0.tar.xz](http://www.cpan.org/src/5.0/perl-5.26.0.tar.xz)

MD5 sum: `8c6995718e4cb62188f0d5e3488cd91f`
Pkg-config (0.29.2) - 1,970 KB:
Home page: [http://www.freedesktop.org/wiki/Software/pkg-config](http://www.freedesktop.org/wiki/Software/pkg-config)

Download: [https://pkg-config.freedesktop.org/releases/pkg-config-0.29.2.tar.gz](https://pkg-config.freedesktop.org/releases/pkg-config-0.29.2.tar.gz)

MD5 sum: `f6e931e319531b736fadc017f470e68a`
Procps (3.3.12) - 826 KB:
Home page: [http://sourceforge.net/projects/procps-ng](http://sourceforge.net/projects/procps-ng)

Download: [http://sourceforge.net/projects/procps-ng/files/Production/procps-ng-3.3.12.tar.xz](http://sourceforge.net/projects/procps-ng/files/Production/procps-ng-3.3.12.tar.xz)

MD5 sum: `957e42e8b193490b2111252e4a2b443c`
Psmisc (23.1) - 290 KB:
Home page: [http://psmisc.sourceforge.net/](http://psmisc.sourceforge.net/)

Download: [https://sourceforge.net/projects/psmisc/files/psmisc/psmisc-23.1.tar.xz](https://sourceforge.net/projects/psmisc/files/psmisc/psmisc-23.1.tar.xz)

MD5 sum: `bbba1f701c02fb50d59540d1ff90d8d1`
Readline (7.0) - 2,842 KB:
Home page: [http://cnswww.cns.cwru.edu/php/chet/readline/rltop.html](http://cnswww.cns.cwru.edu/php/chet/readline/rltop.html)

Download: [http://ftp.gnu.org/gnu/readline/readline-7.0.tar.gz](http://ftp.gnu.org/gnu/readline/readline-7.0.tar.gz)

MD5 sum: `205b03a87fc83dab653b628c59b9fc91`
Sed (4.4) - 1,154 KB:
Home page: [http://www.gnu.org/software/sed/](http://www.gnu.org/software/sed/)

Download: [http://ftp.gnu.org/gnu/sed/sed-4.4.tar.xz](http://ftp.gnu.org/gnu/sed/sed-4.4.tar.xz)

MD5 sum: `e0c583d4c380059abd818cd540fe6938`
Shadow (4.5) - 1,589 KB:
Download: [https://github.com/shadow-maint/shadow/releases/download/4.5/shadow-4.5.tar.xz](https://github.com/shadow-maint/shadow/releases/download/4.5/shadow-4.5.tar.xz)

MD5 sum: `c350da50c2120de6bb29177699d89fe3`
Sysklogd (1.5.1) - 88 KB:
Home page: [http://www.infodrom.org/projects/sysklogd/](http://www.infodrom.org/projects/sysklogd/)

Download: [http://www.infodrom.org/projects/sysklogd/download/sysklogd-1.5.1.tar.gz](http://www.infodrom.org/projects/sysklogd/download/sysklogd-1.5.1.tar.gz)

MD5 sum: `c70599ab0d037fde724f7210c2c8d7f8`
Sysvinit (2.88dsf) - 108 KB:
Home page: [http://savannah.nongnu.org/projects/sysvinit](http://savannah.nongnu.org/projects/sysvinit)

Download: [http://download.savannah.gnu.org/releases/sysvinit/sysvinit-2.88dsf.tar.bz2](http://download.savannah.gnu.org/releases/sysvinit/sysvinit-2.88dsf.tar.bz2)

MD5 sum: `6eda8a97b86e0a6f59dabbf25202aa6f`
Tar (1.29) - 1,950 KB:
Home page: [http://www.gnu.org/software/tar/](http://www.gnu.org/software/tar/)

Download: [http://ftp.gnu.org/gnu/tar/tar-1.29.tar.xz](http://ftp.gnu.org/gnu/tar/tar-1.29.tar.xz)

MD5 sum: `a1802fec550baaeecff6c381629653ef`
Tcl (8.6.7) - 5,738 KB:
Home page: [http://tcl.sourceforge.net/](http://tcl.sourceforge.net/)

Download: [http://sourceforge.net/projects/tcl/files/Tcl/8.6.7/tcl-core8.6.7-src.tar.gz](http://sourceforge.net/projects/tcl/files/Tcl/8.6.7/tcl-core8.6.7-src.tar.gz)

MD5 sum: `3f723d62c2e074bdbb2ddf330b5a71e1`
Texinfo (6.4) - 4,393 KB:
Home page: [http://www.gnu.org/software/texinfo/](http://www.gnu.org/software/texinfo/)

Download: [http://ftp.gnu.org/gnu/texinfo/texinfo-6.4.tar.xz](http://ftp.gnu.org/gnu/texinfo/texinfo-6.4.tar.xz)

MD5 sum: `2a676c8339efe6ddea0f1cb52e626d15`
Time Zone Data (2017b) - 317 KB:
Home page: [http://www.iana.org/time-zones](http://www.iana.org/time-zones)

Download: [http://www.iana.org/time-zones/repository/releases/tzdata2017b.tar.gz](http://www.iana.org/time-zones/repository/releases/tzdata2017b.tar.gz)

MD5 sum: `50dc0dc50c68644c1f70804f2e7a1625`
Udev-lfs Tarball (udev-lfs-20140408) - 11 KB:
Download: [http://anduin.linuxfromscratch.org/LFS/udev-lfs-20140408.tar.bz2](http://anduin.linuxfromscratch.org/LFS/udev-lfs-20140408.tar.bz2)

MD5 sum: `c2d6b127f89261513b23b6d458484099`
Util-linux (2.30.1) - 4,360 KB:
Home page: [http://freecode.com/projects/util-linux](http://freecode.com/projects/util-linux)

Download: [https://www.kernel.org/pub/linux/utils/util-linux/v2.30/util-linux-2.30.1.tar.xz](https://www.kernel.org/pub/linux/utils/util-linux/v2.30/util-linux-2.30.1.tar.xz)

MD5 sum: `5e5ec141e775efe36f640e62f3f8cd0d`
Vim (8.0.586) - 10,613 KB:
Home page: [http://www.vim.org](http://www.vim.org/)

Download: [ftp://ftp.vim.org/pub/vim/unix/vim-8.0.586.tar.bz2](ftp://ftp.vim.org/pub/vim/unix/vim-8.0.586.tar.bz2)

MD5 sum: `b35e794140c196ff59b492b56c1e73db`
XML::Parser (2.44) - 232 KB:
Home page: [https://github.com/chorny/XML-Parser](https://github.com/chorny/XML-Parser)

Download: [http://cpan.metacpan.org/authors/id/T/TO/TODDR/XML-Parser-2.44.tar.gz](http://cpan.metacpan.org/authors/id/T/TO/TODDR/XML-Parser-2.44.tar.gz)

MD5 sum: `af4813fe3952362451201ced6fbce379`
Xz Utils (5.2.3) - 1009 KB:
Home page: [http://tukaani.org/xz](http://tukaani.org/xz)

Download: [http://tukaani.org/xz/xz-5.2.3.tar.xz](http://tukaani.org/xz/xz-5.2.3.tar.xz)

MD5 sum: `60fb79cab777e3f71ca43d298adacbd5`
Zlib (1.2.11) - 457 KB:
Home page: [http://www.zlib.net/](http://www.zlib.net/)

Download: [http://zlib.net/zlib-1.2.11.tar.xz](http://zlib.net/zlib-1.2.11.tar.xz)

MD5 sum: `85adef240c5f370b308da8c938951a68`

Total size of these packages: about 324 MB

## 3.3. Needed Patches

In addition to the packages, several patches are also required. These patches correct any mistakes in the packages that should be fixed by the maintainer. The patches also make small modifications to make the packages easier to work with. The following patches will be needed to build an LFS system:

Bash Upstream Fixes Patch - 17 KB:
Download: [http://www.linuxfromscratch.org/patches/lfs/8.1/bash-4.4-upstream_fixes-1.patch](http://www.linuxfromscratch.org/patches/lfs/8.1/bash-4.4-upstream_fixes-1.patch)

MD5 sum: `e3d5bf23a4e5628680893d46e6ff286e`
Bzip2 Documentation Patch - 1.6 KB:
Download: [http://www.linuxfromscratch.org/patches/lfs/8.1/bzip2-1.0.6-install_docs-1.patch](http://www.linuxfromscratch.org/patches/lfs/8.1/bzip2-1.0.6-install_docs-1.patch)

MD5 sum: `6a5ac7e89b791aae556de0f745916f7f`
Coreutils Internationalization Fixes Patch - 168 KB:
Download: [http://www.linuxfromscratch.org/patches/lfs/8.1/coreutils-8.27-i18n-1.patch](http://www.linuxfromscratch.org/patches/lfs/8.1/coreutils-8.27-i18n-1.patch)

MD5 sum: `a9404fb575dfd5514f3c8f4120f9ca7d`
Glibc FHS Patch - 2.8 KB:
Download: [http://www.linuxfromscratch.org/patches/lfs/8.1/glibc-2.26-fhs-1.patch](http://www.linuxfromscratch.org/patches/lfs/8.1/glibc-2.26-fhs-1.patch)

MD5 sum: `9a5997c3452909b1769918c759eff8a2`
Kbd Backspace/Delete Fix Patch - 12 KB:
Download: [http://www.linuxfromscratch.org/patches/lfs/8.1/kbd-2.0.4-backspace-1.patch](http://www.linuxfromscratch.org/patches/lfs/8.1/kbd-2.0.4-backspace-1.patch)

MD5 sum: `f75cca16a38da6caa7d52151f7136895`
Sysvinit Consolidated Patch - 3.9 KB:
Download: [http://www.linuxfromscratch.org/patches/lfs/8.1/sysvinit-2.88dsf-consolidated-1.patch](http://www.linuxfromscratch.org/patches/lfs/8.1/sysvinit-2.88dsf-consolidated-1.patch)

MD5 sum: `0b7b5ea568a878fdcc4057b2bf36e5cb`

Total size of these patches: about 205.3 KB

In addition to the above required patches, there exist a number of optional patches created by the LFS community. These optional patches solve minor problems or enable functionality that is not enabled by default. Feel free to peruse the patches database located at [http://www.linuxfromscratch.org/patches/downloads/](http://www.linuxfromscratch.org/patches/downloads/) and acquire any additional patches to suit your system needs.
