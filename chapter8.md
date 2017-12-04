
## Chapter 8. Making the LFS System Bootable

## 8.1. Introduction

It is time to make the LFS system bootable. This chapter discusses creating an `fstab` file, building a kernel for the new LFS system, and installing the GRUB boot loader so that the LFS system can be selected for booting at startup.

## 8.2. Creating the /etc/fstab File

The `/etc/fstab` file is used by some programs to determine where file systems are to be mounted by default, in which order, and which must be checked (for integrity errors) prior to mounting. Create a new file systems table like this:

    cat > /etc/fstab << "EOF"
    # Begin /etc/fstab

    # file system  mount-point  type     options             dump  fsck
    #                                                              order

    /dev/*<xxx>*     /            *<fff>*    defaults            1     1
    /dev/*<yyy>*     swap         swap     pri=1               0     0
    proc           /proc        proc     nosuid,noexec,nodev 0     0
    sysfs          /sys         sysfs    nosuid,noexec,nodev 0     0
    devpts         /dev/pts     devpts   gid=5,mode=620      0     0
    tmpfs          /run         tmpfs    defaults            0     0
    devtmpfs       /dev         devtmpfs mode=0755,nosuid    0     0

    # End /etc/fstab
    EOF

Replace *`<xxx>`*, *`<yyy>`*, and *`<fff>`* with the values appropriate for the system, for example, `sda2`, `sda5`, and `ext4`. For details on the six fields in this file, see **man 5 fstab**.

Filesystems with MS-DOS or Windows origin (i.e.: vfat, ntfs, smbfs, cifs, iso9660, udf) need the “iocharset” mount option in order for non-ASCII characters in file names to be interpreted properly. The value of this option should be the same as the character set of your locale, adjusted in such a way that the kernel understands it. This works if the relevant character set definition (found under File systems -> Native Language Support) has been compiled into the kernel or built as a module. The “codepage” option is also needed for vfat and smbfs filesystems. It should be set to the codepage number used under MS-DOS in your country. E.g., in order to mount USB flash drives, a ru_RU.KOI8-R user would need the following in the options portion of its mount line in `/etc/fstab`:

    noauto,user,quiet,showexec,iocharset=koi8r,codepage=866

The corresponding options fragment for ru_RU.UTF-8 users is:

    noauto,user,quiet,showexec,iocharset=utf8,codepage=866

### Note

In the latter case, the kernel emits the following message:

    FAT: utf8 is not a recommended IO charset for FAT filesystems,
        filesystem will be case sensitive!

This negative recommendation should be ignored, since all other values of the “iocharset” option result in wrong display of filenames in UTF-8 locales.

It is also possible to specify default codepage and iocharset values for some filesystems during kernel configuration. The relevant parameters are named “Default NLS Option” (`CONFIG_NLS_DEFAULT)`, “Default Remote NLS Option” (`CONFIG_SMB_NLS_DEFAULT`), “Default codepage for FAT”(`CONFIG_FAT_DEFAULT_CODEPAGE`), and “Default iocharset for FAT” (`CONFIG_FAT_DEFAULT_IOCHARSET`). There is no way to specify these settings for the ntfs filesystem at kernel compilation time.

It is possible to make the ext3 filesystem reliable across power failures for some hard disk types. To do this, add the `barrier=1` mount option to the appropriate entry in `/etc/fstab`. To check if the disk drive supports this option, run [hdparm](http://www.linuxfromscratch.org/blfs/view/8.1/general/hdparm.html) on the applicable disk drive. For example, if:

    hdparm -I /dev/sda | grep NCQ

returns non-empty output, the option is supported.

Note: Logical Volume Management (LVM) based partitions cannot use the `barrier` option.

## 8.3. Linux-4.12.7

The Linux package contains the Linux kernel.

**Approximate build time:**4.4 - 66.0 SBU (typically about 6 SBU)

**Required disk space:**960 - 4250 MB (typically about 1100 MB)

### 8.3.1. Installation of the kernel

Building the kernel involves a few steps—configuration, compilation, and installation. Read the `README` file in the kernel source tree for alternative methods to the way this book configures the kernel.

Prepare for compilation by running the following command:

    make mrproper

This ensures that the kernel tree is absolutely clean. The kernel team recommends that this command be issued prior to each kernel compilation. Do not rely on the source tree being clean after un-tarring.

Configure the kernel via a menu-driven interface. For general information on kernel configuration see [http://www.linuxfromscratch.org/hints/downloads/files/kernel-configuration.txt](http://www.linuxfromscratch.org/hints/downloads/files/kernel-configuration.txt). BLFS has some information regarding particular kernel configuration requirements of packages outside of LFS at[http://www.linuxfromscratch.org/blfs/view/8.1/longindex.html#kernel-config-index](http://www.linuxfromscratch.org/blfs/view/8.1/longindex.html#kernel-config-index). Additional information about configuring and building the kernel can be found at [http://www.kroah.com/lkn/](http://www.kroah.com/lkn/)

### Note

A good starting place for setting up the kernel configuration is to run **make defconfig**. This will set the base configuration to a good state that takes your current system architecture into account.

Be sure to enable or disable following features or the system might not work correctly or boot at all:

    Device Drivers  --->
      Generic Driver Options  --->
       [ ] Support for uevent helper [CONFIG_UEVENT_HELPER]
       [*] Maintain a devtmpfs filesystem to mount at /dev [CONFIG_DEVTMPFS]


There are several other options that may be desired depending on the requirements for the system. For a list of options needed for BLFS packages, see the [BLFS Index of Kernel Settings](http://www.linuxfromscratch.org/blfs/view/8.1/longindex.html#kernel-config-index) (http://www.linuxfromscratch.org/blfs/view/8.1/longindex.html#kernel-config-index).

### Note

If your host hardware is using UEFI, then the 'make defconfig' above should automatically add in some EFI-related kernel options.

In order to allow your LFS kernel to be booted from within your host's UEFI boot environment, your kernel must have this option selected:

    Processor type and features  --->
       [*]   EFI stub support  [CONFIG_EFI_STUB]


A fuller description of managing UEFI environments from within LFS is covered by the lfs-uefi.txt hint at [http://www.linuxfromscratch.org/hints/downloads/files/lfs-uefi.txt](http://www.linuxfromscratch.org/hints/downloads/files/lfs-uefi.txt).

**The rationale for the above configuration items:**
*`Support for uevent helper`*
Having this option set may interfere with device management when using Udev/Eudev.
*`Maintain a devtmpfs`*
This will create automated device nodes which are populated by the kernel, even without Udev running. Udev then runs on top of this, managing permissions and adding symlinks. This configuration item is required for all users of Udev/Eudev.

    make menuconfig

**The meaning of optional make environment variables:**
*`LANG=<host_LANG_value> LC_ALL=`*
This establishes the locale setting to the one used on the host. This may be needed for a proper menuconfig ncurses interface line drawing on a UTF-8 linux text console.

If used, be sure to replace *`<host_LANG_value>`* by the value of the `$LANG` variable from your host. You can alternatively use instead the host's value of `$LC_ALL` or `$LC_CTYPE`.

Alternatively, **make oldconfig** may be more appropriate in some situations. See the `README` file for more information.

If desired, skip kernel configuration by copying the kernel config file, `.config`, from the host system (assuming it is available) to the unpacked `linux-4.12.7` directory. However, we do not recommend this option. It is often better to explore all the configuration menus and create the kernel configuration from scratch.

Compile the kernel image and modules:

    make

If using kernel modules, module configuration in `/etc/modprobe.d` may be required. Information pertaining to modules and kernel configuration is located in [Section 7.3, “Overview of Device and Module Handling”](Linux%20From%20Scratch.html#ch-scripts-udev) and in the kernel documentation in the `linux-4.12.7/Documentation` directory. Also, `modprobe.d(5)` may be of interest.

Install the modules, if the kernel configuration uses them:

    make modules_install

After kernel compilation is complete, additional steps are required to complete the installation. Some files need to be copied to the `/boot`directory.

### Caution

If the host system has a separate /boot partition, the files copied below should go there. The easiest way to do that is to bind /boot on the host to /mnt/lfs/boot before proceeding. As the root user in the *host system*:

    mount --bind /boot /mnt/lfs/boot

The path to the kernel image may vary depending on the platform being used. The filename below can be changed to suit your taste, but the stem of the filename should be *vmlinuz* to be compatible with the automatic setup of the boot process described in the next section. The following command assumes an x86 architecture:

    cp -v arch/x86/boot/bzImage /boot/vmlinuz-4.12.7-lfs-8.1

`System.map` is a symbol file for the kernel. It maps the function entry points of every function in the kernel API, as well as the addresses of the kernel data structures for the running kernel. It is used as a resource when investigating kernel problems. Issue the following command to install the map file:

    cp -v System.map /boot/System.map-4.12.7

The kernel configuration file `.config` produced by the **make menuconfig** step above contains all the configuration selections for the kernel that was just compiled. It is a good idea to keep this file for future reference:

    cp -v .config /boot/config-4.12.7

Install the documentation for the Linux kernel:

    install -d /usr/share/doc/linux-4.12.7
    cp -r Documentation/* /usr/share/doc/linux-4.12.7

It is important to note that the files in the kernel source directory are not owned by *root*. Whenever a package is unpacked as user *root*(like we did inside chroot), the files have the user and group IDs of whatever they were on the packager's computer. This is usually not a problem for any other package to be installed because the source tree is removed after the installation. However, the Linux source tree is often retained for a long time. Because of this, there is a chance that whatever user ID the packager used will be assigned to somebody on the machine. That person would then have write access to the kernel source.

### Note

In many cases, the configuration of the kernel will need to be updated for packages that will be installed later in BLFS. Unlike other packages, it is not necessary to remove the kernel source tree after the newly built kernel is installed.

If the kernel source tree is going to be retained, run **chown -R 0:0** on the `linux-4.12.7` directory to ensure all files are owned by user *root*.

### Warning

Some kernel documentation recommends creating a symlink from `/usr/src/linux` pointing to the kernel source directory. This is specific to kernels prior to the 2.6 series and *must not* be created on an LFS system as it can cause problems for packages you may wish to build once your base LFS system is complete.

### Warning

The headers in the system's `include` directory (`/usr/include`) should *always* be the ones against which Glibc was compiled, that is, the sanitised headers installed in [Section 6.7, “Linux-4.12.7 API Headers”](Linux%20From%20Scratch.html#ch-system-linux-headers). Therefore, they should *never* be replaced by either the raw kernel headers or any other kernel sanitized headers.

### 8.3.2. Configuring Linux Module Load Order

Most of the time Linux modules are loaded automatically, but sometimes it needs some specific direction. The program that loads modules, **modprobe** or **insmod**, uses `/etc/modprobe.d/usb.conf` for this purpose. This file needs to be created so that if the USB drivers (ehci_hcd, ohci_hcd and uhci_hcd) have been built as modules, they will be loaded in the correct order; ehci_hcd needs to be loaded prior to ohci_hcd and uhci_hcd in order to avoid a warning being output at boot time.

Create a new file `/etc/modprobe.d/usb.conf` by running the following:

    install -v -m755 -d /etc/modprobe.d
    cat > /etc/modprobe.d/usb.conf << "EOF"
    # Begin /etc/modprobe.d/usb.conf

    install ohci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i ohci_hcd ; true
    install uhci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i uhci_hcd ; true

    # End /etc/modprobe.d/usb.conf
    EOF

### 8.3.3. Contents of Linux

**Installed files:**config-4.12.7, vmlinuz-4.12.7-lfs-8.1, and System.map-4.12.7

**Installed directories:**/lib/modules, /usr/share/doc/linux-4.12.7

#### Short Descriptions

`config-4.12.7`

Contains all the configuration selections for the kernel

`vmlinuz-4.12.7-lfs-8.1`

The engine of the Linux system. When turning on the computer, the kernel is the first part of the operating system that gets loaded. It detects and initializes all components of the computer's hardware, then makes these components available as a tree of files to the software and turns a single CPU into a multitasking machine capable of running scores of programs seemingly at the same time

`System.map-4.12.7`

A list of addresses and symbols; it maps the entry points and addresses of all the functions and data structures in the kernel

Last updated on

## 8.4. Using GRUB to Set Up the Boot Process

### 8.4.1. Introduction

### Warning

Configuring GRUB incorrectly can render your system inoperable without an alternate boot device such as a CD-ROM. This section is not required to boot your LFS system. You may just want to modify your current boot loader, e.g. Grub-Legacy, GRUB2, or LILO.

Ensure that an emergency boot disk is ready to “rescue” the computer if the computer becomes unusable (un-bootable). If you do not already have a boot device, you can create one. In order for the procedure below to work, you need to jump ahead to BLFS and install **`xorriso`** from the [libisoburn](http://www.linuxfromscratch.org/blfs/view/8.1/multimedia/libisoburn.html) package.

    cd /tmp
    grub-mkrescue --output=grub-img.iso
    xorriso -as cdrecord -v dev=/dev/cdrw blank=as_needed grub-img.iso

### Note

To boot LFS on host systems that have UEFI enabled, the kernel needs to have been built with the CONFIG_EFI_STUB capabality described in the previous section. However, LFS can be booted using GRUB2 without such an addition. To do this, the UEFI Mode and Secure Boot capabilities in the host system's BIOS need to be turned off. For details, see [the lfs-uefi.txt hint](http://www.linuxfromscratch.org/hints/downloads/files/lfs-uefi.txt) at http://www.linuxfromscratch.org/hints/downloads/files/lfs-uefi.txt.

### 8.4.2. GRUB Naming Conventions

GRUB uses its own naming structure for drives and partitions in the form of *(hdn,m)*, where *n* is the hard drive number and *m* is the partition number. The hard drive number starts from zero, but the partition number starts from one for normal partitions and five for extended partitions. Note that this is different from earlier versions where both numbers started from zero. For example, partition `sda1` is*(hd0,1)* to GRUB and `sdb3` is *(hd1,3)*. In contrast to Linux, GRUB does not consider CD-ROM drives to be hard drives. For example, if using a CD on `hdb` and a second hard drive on `hdc`, that second hard drive would still be *(hd1)*.

### 8.4.3. Setting Up the Configuration

GRUB works by writing data to the first physical track of the hard disk. This area is not part of any file system. The programs there access GRUB modules in the boot partition. The default location is /boot/grub/.

The location of the boot partition is a choice of the user that affects the configuration. One recommendation is to have a separate small (suggested size is 100 MB) partition just for boot information. That way each build, whether LFS or some commercial distro, can access the same boot files and access can be made from any booted system. If you choose to do this, you will need to mount the separate partition, move all files in the current `/boot` directory (e.g. the linux kernel you just built in the previous section) to the new partition. You will then need to unmount the partition and remount it as `/boot`. If you do this, be sure to update `/etc/fstab`.

Using the current lfs partition will also work, but configuration for multiple systems is more difficult.

Using the above information, determine the appropriate designator for the root partition (or boot partition, if a separate one is used). For the following example, it is assumed that the root (or separate boot) partition is `sda2`.

Install the GRUB files into `/boot/grub` and set up the boot track:

### Warning

The following command will overwrite the current boot loader. Do not run the command if this is not desired, for example, if using a third party boot manager to manage the Master Boot Record (MBR).

    grub-install /dev/sda

### 8.4.4. Creating the GRUB Configuration File

Generate `/boot/grub/grub.cfg`:

    cat > /boot/grub/grub.cfg << "EOF"
    # Begin /boot/grub/grub.cfg
    set default=0
    set timeout=5

    insmod ext2
    set root=(hd0,2)

    menuentry "GNU/Linux, Linux 4.12.7-lfs-8.1" {
            linux   /boot/vmlinuz-4.12.7-lfs-8.1 root=/dev/sda2 ro
    }
    EOF

### Note

From GRUB's perspective, the kernel files are relative to the partition used. If you used a separate /boot partition, remove /boot from the above *linux* line. You will also need to change the *set root* line to point to the boot partition.

GRUB is an extremely powerful program and it provides a tremendous number of options for booting from a wide variety of devices, operating systems, and partition types. There are also many options for customization such as graphical splash screens, playing sounds, mouse input, etc. The details of these options are beyond the scope of this introduction.

### Caution

There is a command, grub-mkconfig, that can write a configuration file automatically. It uses a set of scripts in /etc/grub.d/ and will destroy any customizations that you make. These scripts are designed primarily for non-source distributions and are not recommended for LFS. If you install a commercial Linux distribution, there is a good chance that this program will be run. Be sure to back up your grub.cfg file.

Last updated on
