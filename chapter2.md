# Part II. Preparing for the Build

## Chapter 2. Preparing the Host System

## 2.1. Introduction <span id="Introduction"></span>

In this chapter, the host tools needed for building LFS are checked and, if necessary, installed. Then a partition which will host the LFS system is prepared. We will create the partition itself, create a file system on it, and mount it.

## 2.2. Host System Requirements 

Your host system should have the following software with the minimum versions indicated. This should not be an issue for most modern Linux distributions. Also note that many distributions will place software headers into separate packages, often in the form of “<package-name>-devel” or “<package-name>-dev”. Be sure to install those if your distribution provides them.

Earlier versions of the listed software packages may work, but have not been tested.

- **Bash-3.2** (/bin/sh should be a symbolic or hard link to bash 

- **Binutils-2.17** (Versions greater than 2.29 are not recommended as they have not been tested)

- **Bison-2.3** (/usr/bin/yacc should be a link to bison or small script that executes bison)

- **Bzip2-1.0.4**

- **Coreutils-6.9**

- **Diffutils-2.8.1**

- **Findutils-4.2.31**

- **Gawk-4.0.1** (/usr/bin/awk should be a link to gawk)

- **GCC-4.7** including the C++ compiler, **g++** (Versions greater than 7.2.0 are not recommended as they have not been tested)

- **Glibc-2.11** (Versions greater than 2.26 are not recommended as they have not been tested)

- **Grep-2.5.1a**

- **Gzip-1.3.12**

- **Linux Kernel-3.2**

The reason for the kernel version requirement is that we specify that version when building glibc in Chapter 6 at the recommendation of the developers. It is also required by udev.

If the host kernel is earlier than 3.2 you will need to replace the kernel with a more up to date version. There are two ways you can go about this. First, see if your Linux vendor provides a 3.2 or later kernel package. If so, you may wish to install it. If your vendor doesn't offer an acceptable kernel package, or you would prefer not to install it, you can compile a kernel yourself. Instructions for compiling the kernel and configuring the boot loader (assuming the host uses GRUB) are located in [chapter8](chapter8.md).

- **M4-1.4.10**

- **Make-3.81**

- **Patch-2.5.4**

- **Perl-5.8.8**

- **Sed-4.1.5**

- **Tar-1.22**

- **Texinfo-4.7**

- **Xz-5.0.0**

### Important

Note that the symlinks mentioned above are required to build an LFS system using the instructions contained within this book. Symlinks that point to other software (such as dash, mawk, etc.) may work, but are not tested or supported by the LFS development team, and may require either deviation from the instructions or additional patches to some packages.

To see whether your host system has all the appropriate versions, and the ability to compile programs, run the following:

    cat > version-check.sh << "EOF"
    #!/bin/bash
    # Simple script to list version numbers of critical development tools
    export LC_ALL=C
    bash --version | head -n1 | cut -d" " -f2-4
    MYSH=$(readlink -f /bin/sh)
    echo "/bin/sh -> $MYSH"
    echo $MYSH | grep -q bash || echo "ERROR: /bin/sh does not point to bash"
    unset MYSH

    echo -n "Binutils: "; ld --version | head -n1 | cut -d" " -f3-
    bison --version | head -n1

    if [ -h /usr/bin/yacc ]; then
      echo "/usr/bin/yacc -> `readlink -f /usr/bin/yacc`";
    elif [ -x /usr/bin/yacc ]; then
      echo yacc is `/usr/bin/yacc --version | head -n1`
    else
      echo "yacc not found"
    fi

    bzip2 --version 2>&1 < /dev/null | head -n1 | cut -d" " -f1,6-
    echo -n "Coreutils: "; chown --version | head -n1 | cut -d")" -f2
    diff --version | head -n1
    find --version | head -n1
    gawk --version | head -n1

    if [ -h /usr/bin/awk ]; then
      echo "/usr/bin/awk -> `readlink -f /usr/bin/awk`";
    elif [ -x /usr/bin/awk ]; then
      echo awk is `/usr/bin/awk --version | head -n1`
    else
      echo "awk not found"
    fi

    gcc --version | head -n1
    g++ --version | head -n1
    ldd --version | head -n1 | cut -d" " -f2-  # glibc version
    grep --version | head -n1
    gzip --version | head -n1
    cat /proc/version
    m4 --version | head -n1
    make --version | head -n1
    patch --version | head -n1
    echo Perl `perl -V:version`
    sed --version | head -n1
    tar --version | head -n1
    makeinfo --version | head -n1
    xz --version | head -n1

    echo 'int main(){}' > dummy.c && g++ -o dummy dummy.c
    if [ -x dummy ]
      then echo "g++ compilation OK";
      else echo "g++ compilation failed"; fi
    rm -f dummy.c dummy
    EOF

    bash version-check.sh

## 2.3. Building LFS in Stages <span id="BuildingLFSinStages"></span>

LFS is designed to be built in one session. That is, the instructions assume that the system will not be shut down during the process. That does not mean that the system has to be done in one sitting. The issue is that certain procedures have to be re-accomplished after a reboot if resuming LFS at different points.

### 2.3.1. Chapters 1-4

These chapters are accomplished on the host system. When restarting, be careful of the following:

- Procedures done as the root user after Section 2.4 need to have the LFS environment variable set *FOR THE ROOT USER*.

### 2.3.2. Chapter 5

- The /mnt/lfs partition must be mounted.

- *ALL* instructions in Chapter 5 must be done by user *lfs*. A **su - lfs** needs to be done before any task in Chapter 5.

- The procedures in [Section 5.3, “General Compilation Instructions”](Linux%20From%20Scratch.html#ch-tools-generalinstructions) are critical. If there is any doubt about installing a package, ensure any previously expanded tarballs are removed, re-extract the package files, and complete all instructions in that section.

### 2.3.3. Chapters 6-8

- The /mnt/lfs partition must be mounted.

- When entering chroot, the LFS environment variable must be set for root. The LFS variable is not used otherwise.

- The virtual file systems must be mounted. This can be done before or after entering chroot by changing to a host virtual terminal and, as root, running the commands in [Section 6.2.2, “Mounting and Populating /dev”](Linux%20From%20Scratch.html#ch-system-bindmount) and [Section 6.2.3, “Mounting Virtual Kernel File Systems”](Linux%20From%20Scratch.html#ch-system-kernfsmount).

## 2.4. Creating a New Partition

Like most other operating systems, LFS is usually installed on a dedicated partition. The recommended approach to building an LFS system is to use an available empty partition or, if you have enough unpartitioned space, to create one.

A minimal system requires a partition of around 6 gigabytes (GB). This is enough to store all the source tarballs and compile the packages. However, if the LFS system is intended to be the primary Linux system, additional software will probably be installed which will require additional space. A 20 GB partition is a reasonable size to provide for growth. The LFS system itself will not take up this much room. A large portion of this requirement is to provide sufficient free temporary storage as well as for adding additional capabilities after LFS is complete. Additionally, compiling packages can require a lot of disk space which will be reclaimed after the package is installed.

Because there is not always enough Random Access Memory (RAM) available for compilation processes, it is a good idea to use a small disk partition as `swap` space. This is used by the kernel to store seldom-used data and leave more memory available for active processes. The `swap`partition for an LFS system can be the same as the one used by the host system, in which case it is not necessary to create another one.

Start a disk partitioning program such as **cfdisk** or **fdisk** with a command line option naming the hard disk on which the new partition will be created—for example `/dev/sda` for the primary Integrated Drive Electronics (IDE) disk. Create a Linux native partition and a `swap` partition, if needed. Please refer to `cfdisk(8)` or `fdisk(8)` if you do not yet know how to use the programs.

### Note

For experienced users, other partitioning schemes are possible. The new LFS system can be on a software [RAID](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/raid.html) array or an [LVM](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/aboutlvm.html) logical volume. However, some of these options require an [initramfs](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/initramfs.html), which is an advanced topic. These partitioning methodologies are not recommended for first time LFS users.

Remember the designation of the new partition (e.g., `sda5`). This book will refer to this as the LFS partition. Also remember the designation of the `swap` partition. These names will be needed later for the `/etc/fstab` file.

### 2.4.1. Other Partition Issues

Requests for advice on system partitioning are often posted on the LFS mailing lists. This is a highly subjective topic. The default for most distributions is to use the entire drive with the exception of one small swap partition. This is not optimal for LFS for several reasons. It reduces flexibility, makes sharing of data across multiple distributions or LFS builds more difficult, makes backups more time consuming, and can waste disk space through inefficient allocation of file system structures.

#### 2.4.1.1. The Root Partition

A root LFS partition (not to be confused with the `/root` directory) of ten gigabytes is a good compromise for most systems. It provides enough space to build LFS and most of BLFS, but is small enough so that multiple partitions can be easily created for experimentation.

#### 2.4.1.2. The Swap Partition

Most distributions automatically create a swap partition. Generally the recommended size of the swap partition is about twice the amount of physical RAM, however this is rarely needed. If disk space is limited, hold the swap partition to two gigabytes and monitor the amount of disk swapping.

Swapping is never good. Generally you can tell if a system is swapping by just listening to disk activity and observing how the system reacts to commands. The first reaction to swapping should be to check for an unreasonable command such as trying to edit a five gigabyte file. If swapping becomes a normal occurrence, the best solution is to purchase more RAM for your system.

#### 2.4.1.3. The Grub Bios Partition

If the *boot disk* has been partitioned with a GUID Partition Table (GPT), then a small, typically 1 MB, partition must be created if it does not already exist. This partition is not formatted, but must be available for GRUB to use during installation of the boot loader. This partition will normally be labeled 'BIOS Boot' if using **fdisk** or have a code of *EF02* if using **gdisk**.

### Note

The Grub Bios partition must be on the drive that the BIOS uses to boot the system. This is not necessarily the same drive where the LFS root partition is located. Disks on a system may use different partition table types. The requirement for this partition depends only on the partition table type of the boot disk.

#### 2.4.1.4. Convenience Partitions

There are several other partitions that are not required, but should be considered when designing a disk layout. The following list is not comprehensive, but is meant as a guide.

- /boot – Highly recommended. Use this partition to store kernels and other booting information. To minimize potential boot problems with larger disks, make this the first physical partition on your first disk drive. A partition size of 100 megabytes is quite adequate.

- /home – Highly recommended. Share your home directory and user customization across multiple distributions or LFS builds. The size is generally fairly large and depends on available disk space.

- /usr – A separate /usr partition is generally used if providing a server for a thin client or diskless workstation. It is normally not needed for LFS. A size of five gigabytes will handle most installations.

- /opt – This directory is most useful for BLFS where multiple installations of large packages like Gnome or KDE can be installed without embedding the files in the /usr hierarchy. If used, 5 to 10 gigabytes is generally adequate.

- /tmp – A separate /tmp directory is rare, but useful if configuring a thin client. This partition, if used, will usually not need to exceed a couple of gigabytes.

- /usr/src – This partition is very useful for providing a location to store BLFS source files and share them across LFS builds. It can also be used as a location for building BLFS packages. A reasonably large partition of 30-50 gigabytes allows plenty of room.

Any separate partition that you want automatically mounted upon boot needs to be specified in the `/etc/fstab`. Details about how to specify partitions will be discussed in [Section 8.2, “Creating the /etc/fstab File”](Linux%20From%20Scratch.html#ch-bootable-fstab).

## 2.5. Creating a File System on the Partition

Now that a blank partition has been set up, the file system can be created. LFS can use any file system recognized by the Linux kernel, but the most common types are ext3 and ext4. The choice of file system can be complex and depends on the characteristics of the files and the size of the partition. For example:

ext2
is suitable for small partitions that are updated infrequently such as /boot.
ext3
is an upgrade to ext2 that includes a journal to help recover the partition's status in the case of an unclean shutdown. It is commonly used as a general purpose file system.
ext4
is the latest version of the ext file system family of partition types. It provides several new capabilities including nano-second timestamps, creation and use of very large files (16 TB), and speed improvements.

Other file systems, including FAT32, NTFS, ReiserFS, JFS, and XFS are useful for specialized purposes. More information about these file systems can be found at [http://en.wikipedia.org/wiki/Comparison_of_file_systems](http://en.wikipedia.org/wiki/Comparison_of_file_systems).

LFS assumes that the root file system (/) is of type ext4. To create an `ext4` file system on the LFS partition, run the following:

    mkfs -v -t ext4 /dev/*<xxx>*

If you are using an existing `swap` partition, there is no need to format it. If a new `swap` partition was created, it will need to be initialized with this command:

    mkswap /dev/*<yyy>*

Replace *`<yyy>`* with the name of the `swap` partition.

## 2.6. Setting The $LFS Variable

Throughout this book, the environment variable `LFS` will be used several times. You should ensure that this variable is always defined throughout the LFS build process. It should be set to the name of the directory where you will be building your LFS system - we will use`/mnt/lfs` as an example, but the directory choice is up to you. If you are building LFS on a separate partition, this directory will be the mount point for the partition. Choose a directory location and set the variable with the following command:

    export LFS=*/mnt/lfs*

Having this variable set is beneficial in that commands such as **mkdir -v $LFS/tools** can be typed literally. The shell will automatically replace “$LFS” with “/mnt/lfs” (or whatever the variable was set to) when it processes the command line.

### Caution

Do not forget to check that `LFS` is set whenever you leave and reenter the current working environment (such as when doing a **su** to `root` or another user). Check that the `LFS` variable is set up properly with:

    echo $LFS

Make sure the output shows the path to your LFS system's build location, which is `/mnt/lfs` if the provided example was followed. If the output is incorrect, use the command given earlier on this page to set `$LFS` to the correct directory name.

### Note

One way to ensure that the `LFS` variable is always set is to edit the `.bash_profile` file in both your personal home directory and in `/root/.bash_profile` and enter the export command above. In addition, the shell specified in the `/etc/passwd` file for all users that need the `LFS` variable needs to be bash to ensure that the `/root/.bash_profile` file is incorporated as a part of the login process.

## 2.7. Mounting the New Partition

Now that a file system has been created, the partition needs to be made accessible. In order to do this, the partition needs to be mounted at a chosen mount point. For the purposes of this book, it is assumed that the file system is mounted under the directory specified by the `LFS` environment variable as described in the previous section.

Create the mount point and mount the LFS file system by running:

    mkdir -pv $LFS
    mount -v -t ext4 /dev/*<xxx>* $LFS

Replace *`<xxx>`* with the designation of the LFS partition.

If using multiple partitions for LFS (e.g., one for `/` and another for `/usr`), mount them using:

    mkdir -pv $LFS
    mount -v -t ext4 /dev/*<xxx>* $LFS
    mkdir -v $LFS/usr
    mount -v -t ext4 /dev/*<yyy>* $LFS/usr

Replace *`<xxx>`* and *`<yyy>`* with the appropriate partition names.

Ensure that this new partition is not mounted with permissions that are too restrictive (such as the `nosuid` or `nodev` options). Run the **mount**command without any parameters to see what options are set for the mounted LFS partition. If `nosuid` and/or `nodev` are set, the partition will need to be remounted.

If you are using a `swap` partition, ensure that it is enabled using the **swapon** command:

    /sbin/swapon -v /dev/*<zzz>*

Replace *`<zzz>`* with the name of the `swap` partition.

Now that there is an established place to work, it is time to download the packages.
