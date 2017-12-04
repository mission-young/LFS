
## Chapter 7. System Configuration

## 7.1. Introduction

Booting a Linux system involves several tasks. The process must mount both virtual and real file systems, initialize devices, activate swap, check file systems for integrity, mount any swap partitions or files, set the system clock, bring up networking, start any daemons required by the system, and accomplish any other custom tasks needed by the user. This process must be organized to ensure the tasks are performed in the correct order but, at the same time, be executed as fast as possible.

### 7.1.1. System V

System V is the classic boot process that has been used in Unix and Unix-like systems such as Linux since about 1983. It consists of a small program, **init**, that sets up basic programs such as **login** (via getty) and runs a script. This script, usually named **rc**, controls the execution of a set of additional scripts that perform the tasks required to initialize the system.

The **init** program is controlled by the `/etc/inittab` file and is organized into run levels that can be run by the user:

0 — halt
1 — Single user mode
2 — Multiuser, without networking
3 — Full multiuser mode
4 — User definable
5 — Full multiuser mode with display manager
6 — reboot

The usual default run level is 3 or 5.

#### Advantages

-
Established, well understood system.

-
Easy to customize.

#### Disadvantages

-
Slower to boot. A medium speed base LFS system takes 8-12 seconds where the boot time is measured from the first kernel message to the login prompt. Network connectivity is typically established about 2 seconds after the login prompt.

-
Serial processing of boot tasks. This is related to the previous point. A delay in any process such as a file system check, will delay the entire boot process.

-
Does not directly support advanced features like control groups (cgroups), and per-user fair share scheduling.

-
Adding scripts requires manual, static sequencing decisions.

## 7.2. LFS-Bootscripts-20170626

The LFS-Bootscripts package contains a set of scripts to start/stop the LFS system at bootup/shutdown. The configuration files and procedures needed to customize the boot process are described in the following sections.

**Approximate build time:**less than 0.1 SBU

**Required disk space:**244 KB

### 7.2.1. Installation of LFS-Bootscripts

Install the package:

    make install

### 7.2.2. Contents of LFS-Bootscripts

**Installed scripts:**checkfs, cleanfs, console, functions, halt, ifdown, ifup, localnet, modules, mountfs, mountvirtfs, network, rc, reboot, sendsignals, setclock, ipv4-static, swap, sysctl, sysklogd, template, udev, and udev_retry

**Installed directories:**/etc/rc.d, /etc/init.d (symbolic link), /etc/sysconfig, /lib/services, /lib/lsb (symbolic link)

#### Short Descriptions

**checkfs**

Checks the integrity of the file systems before they are mounted (with the exception of journal and network based file systems)

**cleanfs**

Removes files that should not be preserved between reboots, such as those in `/var/run/` and `/var/lock/`; it re-creates`/var/run/utmp` and removes the possibly present `/etc/nologin`, `/fastboot`, and `/forcefsck` files

**console**

Loads the correct keymap table for the desired keyboard layout; it also sets the screen font

**functions**

Contains common functions, such as error and status checking, that are used by several bootscripts

**halt**

Halts the system

**ifdown**

Stops a network device

**ifup**

Initializes a network device

**localnet**

Sets up the system's hostname and local loopback device

**modules**

Loads kernel modules listed in `/etc/sysconfig/modules`, using arguments that are also given there

**mountfs**

Mounts all file systems, except ones that are marked *noauto* or are network based

**mountvirtfs**

Mounts virtual kernel file systems, such as `proc`

**network**

Sets up network interfaces, such as network cards, and sets up the default gateway (where applicable)

**rc**

The master run-level control script; it is responsible for running all the other bootscripts one-by-one, in a sequence determined by the name of the symbolic links being processed

**reboot**

Reboots the system

**sendsignals**

Makes sure every process is terminated before the system reboots or halts

**setclock**

Resets the kernel clock to local time in case the hardware clock is not set to UTC time

**ipv4-static**

Provides the functionality needed to assign a static Internet Protocol (IP) address to a network interface

**swap**

Enables and disables swap files and partitions

**sysctl**

Loads system configuration values from `/etc/sysctl.conf`, if that file exists, into the running kernel

**sysklogd**

Starts and stops the system and kernel log daemons

**template**

A template to create custom bootscripts for other daemons

**udev**

Prepares the `/dev` directory and starts Udev

**udev_retry**

Retries failed udev uevents, and copies generated rules files from `/run/udev` to `/etc/udev/rules.d` if required

Last updated on

## 7.3. Overview of Device and Module Handling

In [Chapter 6](Linux%20From%20Scratch.html#chapter-building-system), we installed the Udev package when eudev was built. Before we go into the details regarding how this works, a brief history of previous methods of handling devices is in order.

Linux systems in general traditionally used a static device creation method, whereby a great many device nodes were created under `/dev`(sometimes literally thousands of nodes), regardless of whether the corresponding hardware devices actually existed. This was typically done via a **MAKEDEV** script, which contains a number of calls to the **mknod** program with the relevant major and minor device numbers for every possible device that might exist in the world.

Using the Udev method, only those devices which are detected by the kernel get device nodes created for them. Because these device nodes will be created each time the system boots, they will be stored on a `devtmpfs` file system (a virtual file system that resides entirely in system memory). Device nodes do not require much space, so the memory that is used is negligible.

### 7.3.1. History

In February 2000, a new filesystem called `devfs` was merged into the 2.3.46 kernel and was made available during the 2.4 series of stable kernels. Although it was present in the kernel source itself, this method of creating devices dynamically never received overwhelming support from the core kernel developers.

The main problem with the approach adopted by `devfs` was the way it handled device detection, creation, and naming. The latter issue, that of device node naming, was perhaps the most critical. It is generally accepted that if device names are allowed to be configurable, then the device naming policy should be up to a system administrator, not imposed on them by any particular developer(s). The `devfs` file system also suffered from race conditions that were inherent in its design and could not be fixed without a substantial revision to the kernel. It was marked as deprecated for a long period – due to a lack of maintenance – and was finally removed from the kernel in June, 2006.

With the development of the unstable 2.5 kernel tree, later released as the 2.6 series of stable kernels, a new virtual filesystem called `sysfs`came to be. The job of `sysfs` is to export a view of the system's hardware configuration to userspace processes. With this userspace-visible representation, the possibility of developing a userspace replacement for `devfs` became much more realistic.

### 7.3.2. Udev Implementation

#### 7.3.2.1. Sysfs

The `sysfs` filesystem was mentioned briefly above. One may wonder how `sysfs` knows about the devices present on a system and what device numbers should be used for them. Drivers that have been compiled into the kernel directly register their objects with a `sysfs` (devtmpfs internally) as they are detected by the kernel. For drivers compiled as modules, this registration will happen when the module is loaded. Once the `sysfs` filesystem is mounted (on /sys), data which the drivers register with `sysfs` are available to userspace processes and to udevd for processing (including modifications to device nodes).

#### 7.3.2.2. Device Node Creation

Device files are created by the kernel by the `devtmpfs` filesystem. Any driver that wishes to register a device node will go through the `devtmpfs`(via the driver core) to do it. When a `devtmpfs` instance is mounted on `/dev`, the device node will initially be created with a fixed name, permissions, and owner.

A short time later, the kernel will send a uevent to **udevd**. Based on the rules specified in the files within the `/etc/udev/rules.d`, `/lib/udev/rules.d`, and `/run/udev/rules.d` directories, **udevd** will create additional symlinks to the device node, or change its permissions, owner, or group, or modify the internal **udevd** database entry (name) for that object.

The rules in these three directories are numbered and all three directories are merged together. If **udevd** can't find a rule for the device it is creating, it will leave the permissions and ownership at whatever `devtmpfs` used initially.

#### 7.3.2.3. Module Loading

Device drivers compiled as modules may have aliases built into them. Aliases are visible in the output of the **modinfo** program and are usually related to the bus-specific identifiers of devices supported by a module. For example, the *snd-fm801* driver supports PCI devices with vendor ID 0x1319 and device ID 0x0801, and has an alias of “pci:v00001319d00000801sv*sd*bc04sc01i*”. For most devices, the bus driver exports the alias of the driver that would handle the device via `sysfs`. E.g., the `/sys/bus/pci/devices/0000:00:0d.0/modalias` file might contain the string “pci:v00001319d00000801sv00001319sd00001319bc04sc01i00”. The default rules provided with Udev will cause **udevd** to call out to **/sbin/modprobe** with the contents of the `MODALIAS` uevent environment variable (which should be the same as the contents of the `modalias` file in sysfs), thus loading all modules whose aliases match this string after wildcard expansion.

In this example, this means that, in addition to *snd-fm801*, the obsolete (and unwanted) *forte* driver will be loaded if it is available. See below for ways in which the loading of unwanted drivers can be prevented.

The kernel itself is also able to load modules for network protocols, filesystems and NLS support on demand.

#### 7.3.2.4. Handling Hotpluggable/Dynamic Devices

When you plug in a device, such as a Universal Serial Bus (USB) MP3 player, the kernel recognizes that the device is now connected and generates a uevent. This uevent is then handled by **udevd** as described above.

### 7.3.3. Problems with Loading Modules and Creating Devices

There are a few possible problems when it comes to automatically creating device nodes.

#### 7.3.3.1. A kernel module is not loaded automatically

Udev will only load a module if it has a bus-specific alias and the bus driver properly exports the necessary aliases to `sysfs`. In other cases, one should arrange module loading by other means. With Linux-4.12.7, Udev is known to load properly-written drivers for INPUT, IDE, PCI, USB, SCSI, SERIO, and FireWire devices.

To determine if the device driver you require has the necessary support for Udev, run **modinfo** with the module name as the argument. Now try locating the device directory under `/sys/bus` and check whether there is a `modalias` file there.

If the `modalias` file exists in `sysfs`, the driver supports the device and can talk to it directly, but doesn't have the alias, it is a bug in the driver. Load the driver without the help from Udev and expect the issue to be fixed later.

If there is no `modalias` file in the relevant directory under `/sys/bus`, this means that the kernel developers have not yet added modalias support to this bus type. With Linux-4.12.7, this is the case with ISA busses. Expect this issue to be fixed in later kernel versions.

Udev is not intended to load “wrapper” drivers such as *snd-pcm-oss* and non-hardware drivers such as *loop* at all.

#### 7.3.3.2. A kernel module is not loaded automatically, and Udev is not intended to load it

If the “wrapper” module only enhances the functionality provided by some other module (e.g., *snd-pcm-oss* enhances the functionality of *snd-pcm* by making the sound cards available to OSS applications), configure **modprobe** to load the wrapper after Udev loads the wrapped module. To do this, add a “softdep” line in any `/etc/modprobe.d/*`<filename>`*.conf` file. For example:

    softdep snd-pcm post: snd-pcm-oss

Note that the “softdep” command also allows `pre:` dependencies, or a mixture of both `pre:` and `post:`. See the `modprobe.d(5)` manual page for more information on “softdep” syntax and capabilities.

If the module in question is not a wrapper and is useful by itself, configure the **modules** bootscript to load this module on system boot. To do this, add the module name to the `/etc/sysconfig/modules` file on a separate line. This works for wrapper modules too, but is suboptimal in that case.

#### 7.3.3.3. Udev loads some unwanted module

Either don't build the module, or blacklist it in a `/etc/modprobe.d/blacklist.conf` file as done with the *forte* module in the example below:

    blacklist forte

Blacklisted modules can still be loaded manually with the explicit **modprobe** command.

#### 7.3.3.4. Udev creates a device incorrectly, or makes a wrong symlink

This usually happens if a rule unexpectedly matches a device. For example, a poorly-written rule can match both a SCSI disk (as desired) and the corresponding SCSI generic device (incorrectly) by vendor. Find the offending rule and make it more specific, with the help of the **udevadm info** command.

#### 7.3.3.5. Udev rule works unreliably

This may be another manifestation of the previous problem. If not, and your rule uses `sysfs` attributes, it may be a kernel timing issue, to be fixed in later kernels. For now, you can work around it by creating a rule that waits for the used `sysfs` attribute and appending it to the `/etc/udev/rules.d/10-wait_for_sysfs.rules` file (create this file if it does not exist). Please notify the LFS Development list if you do so and it helps.

#### 7.3.3.6. Udev does not create a device

Further text assumes that the driver is built statically into the kernel or already loaded as a module, and that you have already checked that Udev doesn't create a misnamed device.

Udev has no information needed to create a device node if a kernel driver does not export its data to `sysfs`. This is most common with third party drivers from outside the kernel tree. Create a static device node in `/lib/udev/devices` with the appropriate major/minor numbers (see the file `devices.txt` inside the kernel documentation or the documentation provided by the third party driver vendor). The static device node will be copied to `/dev` by **udev**.

#### 7.3.3.7. Device naming order changes randomly after rebooting

This is due to the fact that Udev, by design, handles uevents and loads modules in parallel, and thus in an unpredictable order. This will never be “fixed”. You should not rely upon the kernel device names being stable. Instead, create your own rules that make symlinks with stable names based on some stable attributes of the device, such as a serial number or the output of various *_id utilities installed by Udev. See [Section 7.4, “Managing Devices”](Linux%20From%20Scratch.html#ch-scripts-symlinks) and [Section 7.5, “General Network Configuration”](Linux%20From%20Scratch.html#ch-scripts-network) for examples.

### 7.3.4. Useful Reading

Additional helpful documentation is available at the following sites:

-
A Userspace Implementation of `devfs` [http://www.kroah.com/linux/talks/ols_2003_udev_paper/Reprint-Kroah-Hartman-OLS2003.pdf](http://www.kroah.com/linux/talks/ols_2003_udev_paper/Reprint-Kroah-Hartman-OLS2003.pdf)

-
The `sysfs` Filesystem [http://www.kernel.org/pub/linux/kernel/people/mochel/doc/papers/ols-2005/mochel.pdf](http://www.kernel.org/pub/linux/kernel/people/mochel/doc/papers/ols-2005/mochel.pdf)

## 7.4. Managing Devices

### 7.4.1. Network Devices

Udev, by default, names network devices according to Firmware/BIOS data or physical characteristics like the bus, slot, or MAC address. The purpose of this naming convention is to ensure that network devices are named consistently and not based on the time the network card was discovered. For example, on a computer having two network cards made by Intel and Realtek, the network card manufactured by Intel may become eth0 and the Realtek card becomes eth1. In some cases, after a reboot the cards get renumbered the other way around.

In the new naming scheme, typical network device names would then be something like enp5s0 or wlp3s0. If this naming convention is not desired, the traditional naming scheme or a custom scheme can be implemented.

#### 7.4.1.1. Disabling Persistent Naming on the Kernel Command Line

The traditional naming scheme using eth0, eth1, etc can be restored by adding **`net.ifnames=0`** on the kernel command line. This is most appropriate for those systems that have only one ethernet device of the same type. Laptops often have multiple ethernet connections that are named eth0 and wlan0 and are also candidates for this method. The command line is passed in the GRUB configuration file. See [Section 8.4.4, “Creating the GRUB Configuration File”](Linux%20From%20Scratch.html#grub-cfg).

#### 7.4.1.2. Creating Custom Udev Rules

The naming scheme can be customized by creating custom Udev rules. A script has been included that generates the initial rules. Generate these rules by running:

    bash /lib/udev/init-net-rules.sh

Now, inspect the `/etc/udev/rules.d/70-persistent-net.rules` file, to find out which name was assigned to which network device:

    cat /etc/udev/rules.d/70-persistent-net.rules

### Note

In some cases such as when MAC addresses have been assigned to a network card manually or in a virtual environment such as Qemu or Xen, the network rules file may not have been generated because addresses are not consistently assigned. In these cases, this method cannot be used.

The file begins with a comment block followed by two lines for each NIC. The first line for each NIC is a commented description showing its hardware IDs (e.g. its PCI vendor and device IDs, if it's a PCI card), along with its driver in parentheses, if the driver can be found. Neither the hardware ID nor the driver is used to determine which name to give an interface; this information is only for reference. The second line is the Udev rule that matches this NIC and actually assigns it a name.

All Udev rules are made up of several keys, separated by commas and optional whitespace. This rule's keys and an explanation of each of them are as follows:

-
`SUBSYSTEM=="net"` - This tells Udev to ignore devices that are not network cards.

-
`ACTION=="add"` - This tells Udev to ignore this rule for a uevent that isn't an add ("remove" and "change" uevents also happen, but don't need to rename network interfaces).

-
`DRIVERS=="?*"` - This exists so that Udev will ignore VLAN or bridge sub-interfaces (because these sub-interfaces do not have drivers). These sub-interfaces are skipped because the name that would be assigned would collide with their parent devices.

-
`ATTR{address}` - The value of this key is the NIC's MAC address.

-
`ATTR{type}=="1"` - This ensures the rule only matches the primary interface in the case of certain wireless drivers, which create multiple virtual interfaces. The secondary interfaces are skipped for the same reason that VLAN and bridge sub-interfaces are skipped: there would be a name collision otherwise.

-
`NAME` - The value of this key is the name that Udev will assign to this interface.

The value of `NAME` is the important part. Make sure you know which name has been assigned to each of your network cards before proceeding, and be sure to use that `NAME` value when creating your configuration files below.

### 7.4.2. CD-ROM symlinks

Some software that you may want to install later (e.g., various media players) expect the `/dev/cdrom` and `/dev/dvd` symlinks to exist, and to point to a CD-ROM or DVD-ROM device. Also, it may be convenient to put references to those symlinks into `/etc/fstab`. Udev comes with a script that will generate rules files to create these symlinks for you, depending on the capabilities of each device, but you need to decide which of two modes of operation you wish to have the script use.

First, the script can operate in “by-path” mode (used by default for USB and FireWire devices), where the rules it creates depend on the physical path to the CD or DVD device. Second, it can operate in “by-id” mode (default for IDE and SCSI devices), where the rules it creates depend on identification strings stored in the CD or DVD device itself. The path is determined by Udev's **path_id** script, and the identification strings are read from the hardware by its **ata_id** or **scsi_id** programs, depending on which type of device you have.

There are advantages to each approach; the correct approach to use will depend on what kinds of device changes may happen. If you expect the physical path to the device (that is, the ports and/or slots that it plugs into) to change, for example because you plan on moving the drive to a different IDE port or a different USB connector, then you should use the “by-id” mode. On the other hand, if you expect the device's identification to change, for example because it may die, and you would replace it with a different device with the same capabilities and which is plugged into the same connectors, then you should use the “by-path” mode.

If either type of change is possible with your drive, then choose a mode based on the type of change you expect to happen more often.

### Important

External devices (for example, a USB-connected CD drive) should not use by-path persistence, because each time the device is plugged into a new external port, its physical path will change. All externally-connected devices will have this problem if you write Udev rules to recognize them by their physical path; the problem is not limited to CD and DVD drives.

If you wish to see the values that the Udev scripts will use, then for the appropriate CD-ROM device, find the corresponding directory under `/sys` (e.g., this can be `/sys/block/hdd`) and run a command similar to the following:

    udevadm test /sys/block/hdd

Look at the lines containing the output of various *_id programs. The “by-id” mode will use the ID_SERIAL value if it exists and is not empty, otherwise it will use a combination of ID_MODEL and ID_REVISION. The “by-path” mode will use the ID_PATH value.

If the default mode is not suitable for your situation, then the following modification can be made to the `/etc/udev/rules.d/83-cdrom-symlinks.rules`file, as follows (where *`mode`* is one of “by-id” or “by-path”):

    sed -i -e 's/"write_cd_rules"/"write_cd_rules *mode*"/' \
        /etc/udev/rules.d/83-cdrom-symlinks.rules

Note that it is not necessary to create the rules files or symlinks at this time, because you have bind-mounted the host's `/dev` directory into the LFS system, and we assume the symlinks exist on the host. The rules and symlinks will be created the first time you boot your LFS system.

However, if you have multiple CD-ROM devices, then the symlinks generated at that time may point to different devices than they point to on your host, because devices are not discovered in a predictable order. The assignments created when you first boot the LFS system will be stable, so this is only an issue if you need the symlinks on both systems to point to the same device. If you need that, then inspect (and possibly edit) the generated `/etc/udev/rules.d/70-persistent-cd.rules` file after booting, to make sure the assigned symlinks match what you need.

### 7.4.3. Dealing with duplicate devices

As explained in [Section 7.3, “Overview of Device and Module Handling”](Linux%20From%20Scratch.html#ch-scripts-udev), the order in which devices with the same function appear in `/dev` is essentially random. E.g., if you have a USB web camera and a TV tuner, sometimes `/dev/video0` refers to the camera and `/dev/video1` refers to the tuner, and sometimes after a reboot the order changes to the opposite one. For all classes of hardware except sound cards and network cards, this is fixable by creating Udev rules for custom persistent symlinks. The case of network cards is covered separately in [Section 7.5, “General Network Configuration”](Linux%20From%20Scratch.html#ch-scripts-network), and sound card configuration can be found in [BLFS](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/devices.html).

For each of your devices that is likely to have this problem (even if the problem doesn't exist in your current Linux distribution), find the corresponding directory under `/sys/class` or `/sys/block`. For video devices, this may be `/sys/class/video4linux/video*`X`*`. Figure out the attributes that identify the device uniquely (usually, vendor and product IDs and/or serial numbers work):

    udevadm info -a -p /sys/class/video4linux/video0

Then write rules that create the symlinks, e.g.:

    cat > /etc/udev/rules.d/83-duplicate_devs.rules << "EOF"

    # Persistent symlinks for webcam and tuner
    KERNEL=="video*", ATTRS{idProduct}=="1910", ATTRS{idVendor}=="0d81", \
        SYMLINK+="webcam"
    KERNEL=="video*", ATTRS{device}=="0x036f", ATTRS{vendor}=="0x109e", \
        SYMLINK+="tvtuner"

    EOF

The result is that `/dev/video0` and `/dev/video1` devices still refer randomly to the tuner and the web camera (and thus should never be used directly), but there are symlinks `/dev/tvtuner` and `/dev/webcam` that always point to the correct device.

## 7.5. General Network Configuration

### 7.5.1. Creating Network Interface Configuration Files

Which interfaces are brought up and down by the network script usually depends on the files in `/etc/sysconfig/`. This directory should contain a file for each interface to be configured, such as `ifconfig.xyz`, where “xyz” should describe the network card. The interface name (e.g. eth0) is usually appropriate. Inside this file are attributes to this interface, such as its IP address(es), subnet masks, and so forth. It is necessary that the stem of the filename be *ifconfig*.

### Note

If the procedure in the previous section was not used, Udev will assign network card interface names based on system physical characteristics such as enp2s1. If you are not sure what your interface name is, you can always run **ip link** or **ls /sys/class/net** after you have booted your system.

The following command creates a sample file for the *eth0* device with a static IP address:

    cd /etc/sysconfig/
    cat > ifconfig.eth0 << "EOF"
    ONBOOT=yes
    IFACE=eth0
    SERVICE=ipv4-static
    IP=192.168.1.2
    GATEWAY=192.168.1.1
    PREFIX=24
    BROADCAST=192.168.1.255
    EOF

The values of these variables must be changed in every file to match the proper setup.

If the `ONBOOT` variable is set to “yes” the System V network script will bring up the Network Interface Card (NIC) during booting of the system. If set to anything but “yes” the NIC will be ignored by the network script and not be automatically brought up. The interface can be manually started or stopped with the **ifup** and **ifdown** commands.

The `IFACE` variable defines the interface name, for example, eth0. It is required for all network device configuration files.

The `SERVICE` variable defines the method used for obtaining the IP address. The LFS-Bootscripts package has a modular IP assignment format, and creating additional files in the `/lib/services/` directory allows other IP assignment methods. This is commonly used for Dynamic Host Configuration Protocol (DHCP), which is addressed in the BLFS book.

The `GATEWAY` variable should contain the default gateway IP address, if one is present. If not, then comment out the variable entirely.

The `PREFIX` variable contains the number of bits used in the subnet. Each octet in an IP address is 8 bits. If the subnet's netmask is 255.255.255.0, then it is using the first three octets (24 bits) to specify the network number. If the netmask is 255.255.255.240, it would be using the first 28 bits. Prefixes longer than 24 bits are commonly used by DSL and cable-based Internet Service Providers (ISPs). In this example (PREFIX=24), the netmask is 255.255.255.0. Adjust the `PREFIX` variable according to your specific subnet. If omitted, the PREFIX defaults to 24.

For more information see the **ifup** man page.

### 7.5.2. Creating the /etc/resolv.conf File

The system will need some means of obtaining Domain Name Service (DNS) name resolution to resolve Internet domain names to IP addresses, and vice versa. This is best achieved by placing the IP address of the DNS server, available from the ISP or network administrator, into `/etc/resolv.conf`. Create the file by running the following:

    cat > /etc/resolv.conf << "EOF"
    # Begin /etc/resolv.conf

    domain *<Your Domain Name>*
    nameserver *<IP address of your primary nameserver>*
    nameserver *<IP address of your secondary nameserver>*

    # End /etc/resolv.conf
    EOF

The `domain` statement can be omitted or replaced with a `search` statement. See the man page for resolv.conf for more details.

Replace *`<IP address of the nameserver>`* with the IP address of the DNS most appropriate for the setup. There will often be more than one entry (requirements demand secondary servers for fallback capability). If you only need or want one DNS server, remove the second *nameserver*line from the file. The IP address may also be a router on the local network.

### Note

The Google Public IPv4 DNS addresses are 8.8.8.8 and 8.8.4.4.

### 7.5.3. Configuring the system hostname

During the boot process, the file `/etc/hostname` is used for establishing the system's hostname.

Create the `/etc/hostname` file and enter a hostname by running:

    echo "*<lfs>*" > /etc/hostname

*`<lfs>`* needs to be replaced with the name given to the computer. Do not enter the Fully Qualified Domain Name (FQDN) here. That information is put in the `/etc/hosts` file.

### 7.5.4. Customizing the /etc/hosts File

Decide on the IP address, fully-qualified domain name (FQDN), and possible aliases for use in the `/etc/hosts` file. The syntax is:

    IP_address myhost.example.org aliases

Unless the computer is to be visible to the Internet (i.e., there is a registered domain and a valid block of assigned IP addresses—most users do not have this), make sure that the IP address is in the private network IP address range. Valid ranges are:

    Private Network Address Range      Normal Prefix
    10.0.0.1 - 10.255.255.254           8
    172.x.0.1 - 172.x.255.254           16
    192.168.y.1 - 192.168.y.254         24

x can be any number in the range 16-31. y can be any number in the range 0-255.

A valid private IP address could be 192.168.1.1. A valid FQDN for this IP could be lfs.example.org.

Even if not using a network card, a valid FQDN is still required. This is necessary for certain programs to operate correctly.

Create the `/etc/hosts` file by running:

    cat > /etc/hosts << "EOF"
    # Begin /etc/hosts

    127.0.0.1 localhost
    127.0.1.1 *<FQDN>**<HOSTNAME>**<192.168.1.1>**<FQDN>**<HOSTNAME>**[alias1] [alias2 ...]*
    ::1       localhost ip6-localhost ip6-loopback
    ff02::1   ip6-allnodes
    ff02::2   ip6-allrouters

    # End /etc/hosts
    EOF

The *`<192.168.1.1>`*, *`<FQDN>`*, and *`<HOSTNAME>`* values need to be changed for specific uses or requirements (if assigned an IP address by a network/system administrator and the machine will be connected to an existing network). The optional alias name(s) can be omitted.

## 7.6. System V Bootscript Usage and Configuration

### 7.6.1. How Do the System V Bootscripts Work?

Linux uses a special booting facility named SysVinit that is based on a concept of *run-levels*. It can be quite different from one system to another, so it cannot be assumed that because things worked in one particular Linux distribution, they should work the same in LFS too. LFS has its own way of doing things, but it respects generally accepted standards.

SysVinit (which will be referred to as “init” from now on) works using a run-levels scheme. There are seven (numbered 0 to 6) run-levels (actually, there are more run-levels, but they are for special cases and are generally not used. See `init(8)` for more details), and each one of those corresponds to the actions the computer is supposed to perform when it starts up. The default run-level is 3. Here are the descriptions of the different run-levels as they are implemented:

0: halt the computer
1: single-user mode
2: multi-user mode without networking
3: multi-user mode with networking
4: reserved for customization, otherwise does the same as 3
5: same as 4, it is usually used for GUI login (like X's **xdm** or KDE's **kdm**)
6: reboot the computer

### 7.6.2. Configuring Sysvinit

During the kernel initialization, the first program that is run is either specified on the command line or, by default **init**. This program reads the initialization file `/etc/inittab`. Create this file with:

    cat > /etc/inittab << "EOF"
    # Begin /etc/inittab

    id:3:initdefault:

    si::sysinit:/etc/rc.d/init.d/rc S

    l0:0:wait:/etc/rc.d/init.d/rc 0
    l1:S1:wait:/etc/rc.d/init.d/rc 1
    l2:2:wait:/etc/rc.d/init.d/rc 2
    l3:3:wait:/etc/rc.d/init.d/rc 3
    l4:4:wait:/etc/rc.d/init.d/rc 4
    l5:5:wait:/etc/rc.d/init.d/rc 5
    l6:6:wait:/etc/rc.d/init.d/rc 6

    ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now

    su:S016:once:/sbin/sulogin

    1:2345:respawn:/sbin/agetty --noclear tty1 9600
    2:2345:respawn:/sbin/agetty tty2 9600
    3:2345:respawn:/sbin/agetty tty3 9600
    4:2345:respawn:/sbin/agetty tty4 9600
    5:2345:respawn:/sbin/agetty tty5 9600
    6:2345:respawn:/sbin/agetty tty6 9600

    # End /etc/inittab
    EOF

An explanation of this initialization file is in the man page for *inittab*. For LFS, the key command that is run is **rc**. The initialization file above will instruct **rc** to run all the scripts starting with an S in the `/etc/rc.d/rcS.d` directory followed by all the scripts starting with an S in the `/etc/rc.d/rc?.d` directory where the question mark is specified by the initdefault value.

As a convenience, the **rc** script reads a library of functions in `/lib/lsb/init-functions`. This library also reads an optional configuration file, `/etc/sysconfig/rc.site`. Any of the system configuration file parameters described in subsequent sections can be alternatively placed in this file allowing consolidation of all system parameters in this one file.

As a debugging convenience, the functions script also logs all output to `/run/var/bootlog`. Since the `/run` directory is a tmpfs, this file is not persistent across boots, however it is appended to the more permanent file `/var/log/boot.log` at the end of the boot process.

#### 7.6.2.1. Changing Run Levels

Changing run-levels is done with **init *`<runlevel>`***, where *`<runlevel>`* is the target run-level. For example, to reboot the computer, a user could issue the **init 6** command, which is an alias for the **reboot** command. Likewise, **init 0** is an alias for the **halt** command.

There are a number of directories under `/etc/rc.d` that look like `rc?.d` (where ? is the number of the run-level) and `rcsysinit.d`, all containing a number of symbolic links. Some begin with a *K*, the others begin with an *S*, and all of them have two numbers following the initial letter. The K means to stop (kill) a service and the S means to start a service. The numbers determine the order in which the scripts are run, from 00 to 99—the lower the number the earlier it gets executed. When **init** switches to another run-level, the appropriate services are either started or stopped, depending on the runlevel chosen.

The real scripts are in `/etc/rc.d/init.d`. They do the actual work, and the symlinks all point to them. K links and S links point to the same script in `/etc/rc.d/init.d`. This is because the scripts can be called with different parameters like *`start`*, *`stop`*, *`restart`*, *`reload`*, and *`status`*. When a K link is encountered, the appropriate script is run with the *`stop`* argument. When an S link is encountered, the appropriate script is run with the *`start`* argument.

There is one exception to this explanation. Links that start with an *S* in the `rc0.d` and `rc6.d` directories will not cause anything to be started. They will be called with the parameter *`stop`* to stop something. The logic behind this is that when a user is going to reboot or halt the system, nothing needs to be started. The system only needs to be stopped.

These are descriptions of what the arguments make the scripts do:

*`start`*
The service is started.
*`stop`*
The service is stopped.
*`restart`*
The service is stopped and then started again.
*`reload`*
The configuration of the service is updated. This is used after the configuration file of a service was modified, when the service does not need to be restarted.
*`status`*
Tells if the service is running and with which PIDs.

Feel free to modify the way the boot process works (after all, it is your own LFS system). The files given here are an example of how it can be done.

### 7.6.3. Udev Bootscripts

The `/etc/rc.d/init.d/udev` initscript starts **udevd**, triggers any "coldplug" devices that have already been created by the kernel and waits for any rules to complete. The script also unsets the uevent handler from the default of `/sbin/hotplug` . This is done because the kernel no longer needs to call out to an external binary. Instead **udevd** will listen on a netlink socket for uevents that the kernel raises.

The **/etc/rc.d/init.d/udev_retry** initscript takes care of re-triggering events for subsystems whose rules may rely on filesystems that are not mounted until the **mountfs** script is run (in particular, `/usr` and `/var` may cause this). This script runs after the **mountfs** script, so those rules (if re-triggered) should succeed the second time around. It is configured from the `/etc/sysconfig/udev_retry` file; any words in this file other than comments are considered subsystem names to trigger at retry time. To find the subsystem of a device, use **udevadm info --attribute-walk <device>**where <device> is an absolute path in /dev or /sys such as /dev/sr0 or /sys/class/rtc.

#### 7.6.3.1. Module Loading

Device drivers compiled as modules may have aliases built into them. Aliases are visible in the output of the **modinfo** program and are usually related to the bus-specific identifiers of devices supported by a module. For example, the *snd-fm801* driver supports PCI devices with vendor ID 0x1319 and device ID 0x0801, and has an alias of “pci:v00001319d00000801sv*sd*bc04sc01i*”. For most devices, the bus driver exports the alias of the driver that would handle the device via `sysfs`. E.g., the `/sys/bus/pci/devices/0000:00:0d.0/modalias` file might contain the string “pci:v00001319d00000801sv00001319sd00001319bc04sc01i00”. The default rules provided with Udev will cause **udevd** to call out to **/sbin/modprobe** with the contents of the `MODALIAS` uevent environment variable (which should be the same as the contents of the `modalias` file in sysfs), thus loading all modules whose aliases match this string after wildcard expansion.

In this example, this means that, in addition to *snd-fm801*, the obsolete (and unwanted) *forte* driver will be loaded if it is available. See below for ways in which the loading of unwanted drivers can be prevented.

The kernel itself is also able to load modules for network protocols, filesystems and NLS support on demand.

#### 7.6.3.2. Handling Hotpluggable/Dynamic Devices

When you plug in a device, such as a Universal Serial Bus (USB) MP3 player, the kernel recognizes that the device is now connected and generates a uevent. This uevent is then handled by **udevd** as described above.

### 7.6.4. Configuring the System Clock

The **setclock** script reads the time from the hardware clock, also known as the BIOS or the Complementary Metal Oxide Semiconductor (CMOS) clock. If the hardware clock is set to UTC, this script will convert the hardware clock's time to the local time using the `/etc/localtime`file (which tells the **hwclock** program which timezone the user is in). There is no way to detect whether or not the hardware clock is set to UTC, so this needs to be configured manually.

The **setclock** is run via udev when the kernel detects the hardware capability upon boot. It can also be run manually with the stop parameter to store the system time to the CMOS clock.

If you cannot remember whether or not the hardware clock is set to UTC, find out by running the **`hwclock --localtime --show`** command. This will display what the current time is according to the hardware clock. If this time matches whatever your watch says, then the hardware clock is set to local time. If the output from **hwclock** is not local time, chances are it is set to UTC time. Verify this by adding or subtracting the proper amount of hours for the timezone to the time shown by **hwclock**. For example, if you are currently in the MST timezone, which is also known as GMT -0700, add seven hours to the local time.

Change the value of the `UTC` variable below to a value of *`0`* (zero) if the hardware clock is *not* set to UTC time.

Create a new file `/etc/sysconfig/clock` by running the following:

    cat > /etc/sysconfig/clock << "EOF"
    # Begin /etc/sysconfig/clock

    UTC=1

    # Set this to any options you might need to give to hwclock,
    # such as machine hardware clock type for Alphas.
    CLOCKPARAMS=

    # End /etc/sysconfig/clock
    EOF

A good hint explaining how to deal with time on LFS is available at [http://www.linuxfromscratch.org/hints/downloads/files/time.txt](http://www.linuxfromscratch.org/hints/downloads/files/time.txt). It explains issues such as time zones, UTC, and the `TZ`environment variable.

### Note

The CLOCKPARAMS and UTC paramaters may be alternatively set in the `/etc/sysconfig/rc.site` file.

### 7.6.5. Configuring the Linux Console

This section discusses how to configure the **console** bootscript that sets up the keyboard map, console font and console kernel log level. If non-ASCII characters (e.g., the copyright sign, the British pound sign and Euro symbol) will not be used and the keyboard is a U.S. one, much of this section can be skipped. Without the configuration file, (or equivalent settings in `rc.site`), the **console** bootscript will do nothing.

The **console** script reads the `/etc/sysconfig/console` file for configuration information. Decide which keymap and screen font will be used. Various language-specific HOWTOs can also help with this, see [http://www.tldp.org/HOWTO/HOWTO-INDEX/other-lang.html](http://www.tldp.org/HOWTO/HOWTO-INDEX/other-lang.html). If still in doubt, look in the `/usr/share/keymaps` and `/usr/share/consolefonts` directories for valid keymaps and screen fonts. Read `loadkeys(1)` and `setfont(8)` manual pages to determine the correct arguments for these programs.

The `/etc/sysconfig/console` file should contain lines of the form: VARIABLE="value". The following variables are recognized:

LOGLEVEL
This variable specifies the log level for kernel messages sent to the console as set by **dmesg**. Valid levels are from "1" (no messages) to "8". The default level is "7".
KEYMAP
This variable specifies the arguments for the **loadkeys** program, typically, the name of keymap to load, e.g., “it”. If this variable is not set, the bootscript will not run the **loadkeys** program, and the default kernel keymap will be used. Note that a few keymaps have multiple versions with the same name (cz and its variants in qwerty/ and qwertz/, es in olpc/ and qwerty/, and trf in fgGIod/ and qwerty/). In these cases the parent directory should also be specified (e.g. qwerty/es) to ensure the proper keymap is loaded.
KEYMAP_CORRECTIONS
This (rarely used) variable specifies the arguments for the second call to the **loadkeys** program. This is useful if the stock keymap is not completely satisfactory and a small adjustment has to be made. E.g., to include the Euro sign into a keymap that normally doesn't have it, set this variable to “euro2”.
FONT
This variable specifies the arguments for the **setfont** program. Typically, this includes the font name, “-m”, and the name of the application character map to load. E.g., in order to load the “lat1-16” font together with the “8859-1” application character map (as it is appropriate in the USA), set this variable to “lat1-16 -m 8859-1”. In UTF-8 mode, the kernel uses the application character map for conversion of composed 8-bit key codes in the keymap to UTF-8, and thus the argument of the "-m" parameter should be set to the encoding of the composed key codes in the keymap.
UNICODE
Set this variable to “1”, “yes” or “true” in order to put the console into UTF-8 mode. This is useful in UTF-8 based locales and harmful otherwise.
LEGACY_CHARSET
For many keyboard layouts, there is no stock Unicode keymap in the Kbd package. The **console** bootscript will convert an available keymap to UTF-8 on the fly if this variable is set to the encoding of the available non-UTF-8 keymap.

Some examples:

-
For a non-Unicode setup, only the KEYMAP and FONT variables are generally needed. E.g., for a Polish setup, one would use:

    cat > /etc/sysconfig/console << "EOF"
    # Begin /etc/sysconfig/console

    KEYMAP="pl2"
    FONT="lat2a-16 -m 8859-2"

    # End /etc/sysconfig/console
    EOF

-
As mentioned above, it is sometimes necessary to adjust a stock keymap slightly. The following example adds the Euro symbol to the German keymap:

    cat > /etc/sysconfig/console << "EOF"
    # Begin /etc/sysconfig/console

    KEYMAP="de-latin1"
    KEYMAP_CORRECTIONS="euro2"
    FONT="lat0-16 -m 8859-15"

    # End /etc/sysconfig/console
    EOF

-
The following is a Unicode-enabled example for Bulgarian, where a stock UTF-8 keymap exists:

    cat > /etc/sysconfig/console << "EOF"
    # Begin /etc/sysconfig/console

    UNICODE="1"
    KEYMAP="bg_bds-utf8"
    FONT="LatArCyrHeb-16"

    # End /etc/sysconfig/console
    EOF

-
Due to the use of a 512-glyph LatArCyrHeb-16 font in the previous example, bright colors are no longer available on the Linux console unless a framebuffer is used. If one wants to have bright colors without framebuffer and can live without characters not belonging to his language, it is still possible to use a language-specific 256-glyph font, as illustrated below:

    cat > /etc/sysconfig/console << "EOF"
    # Begin /etc/sysconfig/console

    UNICODE="1"
    KEYMAP="bg_bds-utf8"
    FONT="cyr-sun16"

    # End /etc/sysconfig/console
    EOF

-
The following example illustrates keymap autoconversion from ISO-8859-15 to UTF-8 and enabling dead keys in Unicode mode:

    cat > /etc/sysconfig/console << "EOF"
    # Begin /etc/sysconfig/console

    UNICODE="1"
    KEYMAP="de-latin1"
    KEYMAP_CORRECTIONS="euro2"
    LEGACY_CHARSET="iso-8859-15"
    FONT="LatArCyrHeb-16 -m 8859-15"

    # End /etc/sysconfig/console
    EOF

-
Some keymaps have dead keys (i.e., keys that don't produce a character by themselves, but put an accent on the character produced by the next key) or define composition rules (such as: “press Ctrl+. A E to get Æ” in the default keymap). Linux-4.12.7 interprets dead keys and composition rules in the keymap correctly only when the source characters to be composed together are not multibyte. This deficiency doesn't affect keymaps for European languages, because there accents are added to unaccented ASCII characters, or two ASCII characters are composed together. However, in UTF-8 mode it is a problem, e.g., for the Greek language, where one sometimes needs to put an accent on the letter “alpha”. The solution is either to avoid the use of UTF-8, or to install the X window system that doesn't have this limitation in its input handling.

-
For Chinese, Japanese, Korean and some other languages, the Linux console cannot be configured to display the needed characters. Users who need such languages should install the X Window System, fonts that cover the necessary character ranges, and the proper input method (e.g., SCIM, it supports a wide variety of languages).

### Note

The `/etc/sysconfig/console` file only controls the Linux text console localization. It has nothing to do with setting the proper keyboard layout and terminal fonts in the X Window System, with ssh sessions or with a serial console. In such situations, limitations mentioned in the last two list items above do not apply.

### 7.6.6. Creating Files at Boot

At times, it is desired to create files at boot time. For instance, the `/tmp/.ICE-unix` directory may be desired. This can be done by creating an entry in the `/etc/sysconfig/createfiles` configuration script. The format of this file is embedded in the comments of the default configuration file.

### 7.6.7. Configuring the sysklogd Script

The `sysklogd` script invokes the **syslogd** program as a part of System V initialization. The *`-m 0`* option turns off the periodic timestamp mark that **syslogd** writes to the log files every 20 minutes by default. If you want to turn on this periodic timestamp mark, edit `/etc/sysconfig/rc.site` and define the variable SYSKLOGD_PARMS to the desired value. For instance, to remove all parameters, set the variable to a null value:

    SYSKLOGD_PARMS=


See **`man syslogd`** for more options.

### 7.6.8. The rc.site File

The optional `/etc/sysconfig/rc.site` file contains settings that are automatically set for each SystemV boot script. It can alternatively set the values specified in the `hostname`, `console`, and `clock` files in the `/etc/sysconfig/` directory. If the associated variables are present in both these separate files and `rc.site`, the values in the script specific files have precedence.

`rc.site` also contains parameters that can customize other aspects of the boot process. Setting the IPROMPT variable will enable selective running of bootscripts. Other options are described in the file comments. The default version of the file is as follows:

    # rc.site
    # Optional parameters for boot scripts.

    # Distro Information
    # These values, if specified here, override the defaults
    #DISTRO="Linux From Scratch" # The distro name
    #DISTRO_CONTACT="lfs-dev@linuxfromscratch.org" # Bug report address
    #DISTRO_MINI="LFS" # Short name used in filenames for distro config

    # Define custom colors used in messages printed to the screen

    # Please consult `man console_codes` for more information
    # under the "ECMA-48 Set Graphics Rendition" section
    #
    # Warning: when switching from a 8bit to a 9bit font,
    # the linux console will reinterpret the bold (1;) to
    # the top 256 glyphs of the 9bit font.  This does
    # not affect framebuffer consoles

    # These values, if specified here, override the defaults
    #BRACKET="\\033[1;34m" # Blue
    #FAILURE="\\033[1;31m" # Red
    #INFO="\\033[1;36m"    # Cyan
    #NORMAL="\\033[0;39m"  # Grey
    #SUCCESS="\\033[1;32m" # Green
    #WARNING="\\033[1;33m" # Yellow

    # Use a colored prefix
    # These values, if specified here, override the defaults
    #BMPREFIX="     "
    #SUCCESS_PREFIX="${SUCCESS}  *  ${NORMAL}"
    #FAILURE_PREFIX="${FAILURE}*****${NORMAL}"
    #WARNING_PREFIX="${WARNING} *** ${NORMAL}"

    # Manually seet the right edge of message output (characters)
    # Useful when resetting console font during boot to override
    # automatic screen width detection
    #COLUMNS=120

    # Interactive startup
    #IPROMPT="yes" # Whether to display the interactive boot prompt
    #itime="3"    # The amount of time (in seconds) to display the prompt

    # The total length of the distro welcome string, without escape codes
    #wlen=$(echo "Welcome to ${DISTRO}" | wc -c )
    #welcome_message="Welcome to ${INFO}${DISTRO}${NORMAL}"

    # The total length of the interactive string, without escape codes
    #ilen=$(echo "Press 'I' to enter interactive startup" | wc -c )
    #i_message="Press '${FAILURE}I${NORMAL}' to enter interactive startup"

    # Set scripts to skip the file system check on reboot
    #FASTBOOT=yes

    # Skip reading from the console
    #HEADLESS=yes

    # Write out fsck progress if yes
    #VERBOSE_FSCK=no

    # Speed up boot without waiting for settle in udev
    #OMIT_UDEV_SETTLE=y

    # Speed up boot without waiting for settle in udev_retry
    #OMIT_UDEV_RETRY_SETTLE=yes

    # Skip cleaning /tmp if yes
    #SKIPTMPCLEAN=no

    # For setclock
    #UTC=1
    #CLOCKPARAMS=

    # For consolelog (Note that the default, 7=debug, is noisy)
    #LOGLEVEL=7

    # For network
    #HOSTNAME=mylfs

    # Delay between TERM and KILL signals at shutdown
    #KILLDELAY=3

    # Optional sysklogd parameters
    #SYSKLOGD_PARMS="-m 0"

    # Console parameters
    #UNICODE=1
    #KEYMAP="de-latin1"
    #KEYMAP_CORRECTIONS="euro2"
    #FONT="lat0-16 -m 8859-15"
    #LEGACY_CHARSET=



#### 7.6.8.1. Customizing the Boot and Shutdown Scripts

The LFS boot scripts boot and shut down a system in a fairly efficient manner, but there are a few tweaks that you can make in the rc.site file to improve speed even more and to adjust messages according to your preferences. To do this, adjust the settings in the `/etc/sysconfig/rc.site` file above.

-
During the boot script `udev`, there is a call to **udev settle** that requires some time to complete. This time may or may not be required depending on devices present in the system. If you only have simple partitions and a single ethernet card, the boot process will probably not need to wait for this command. To skip it, set the variable OMIT_UDEV_SETTLE=y.

-
The boot script `udev_retry` also runs **udev settle** by default. This command is only needed by default if the `/var` directory is separately mounted. This is because the clock needs the file `/var/lib/hwclock/adjtime`. Other customizations may also need to wait for udev to complete, but in many installations it is not needed. Skip the command by setting the variable OMIT_UDEV_RETRY_SETTLE=y.

-
By default, the file system checks are silent. This can appear to be a delay during the bootup process. To turn on the **fsck** output, set the variable VERBOSE_FSCK=y.

-
When rebooting, you may want to skip the filesystem check, **fsck**, completely. To do this, either create the file `/fastboot` or reboot the system with the command **/sbin/shutdown -f -r now**. On the other hand, you can force all file systems to be checked by creating `/forcefsck` or running **shutdown** with the *`-F`* parameter instead of *`-f`*.

Setting the variable FASTBOOT=y will disable **fsck** during the boot process until it is removed. This is not recommended on a permanent basis.

-
Normally, all files in the `/tmp` directory are deleted at boot time. Depending on the number of files or directories present, this can cause a noticeable delay in the boot process. To skip removing these files set the variable SKIPTMPCLEAN=y.

-
During shutdown, the **init** program sends a TERM signal to each program it has started (e.g. agetty), waits for a set time (default 3 seconds), and sends each process a KILL signal and waits again. This process is repeated in the **sendsignals** script for any processes that are not shut down by their own scripts. The delay for **init** can be set by passing a parameter. For example to remove the delay in **init**, pass the -t0 parameter when shutting down or rebooting (e.g. **/sbin/shutdown -t0 -r now**). The delay for the **sendsignals** script can be skipped by setting the parameter KILLDELAY=0.

## 7.7. The Bash Shell Startup Files

The shell program **/bin/bash** (hereafter referred to as “the shell”) uses a collection of startup files to help create an environment to run in. Each file has a specific use and may affect login and interactive environments differently. The files in the `/etc` directory provide global settings. If an equivalent file exists in the home directory, it may override the global settings.

An interactive login shell is started after a successful login, using **/bin/login**, by reading the `/etc/passwd` file. An interactive non-login shell is started at the command-line (e.g., `[prompt]$`**/bin/bash**). A non-interactive shell is usually present when a shell script is running. It is non-interactive because it is processing a script and not waiting for user input between commands.

For more information, see **info bash** under the *Bash Startup Files and Interactive Shells* section.

The files `/etc/profile` and `~/.bash_profile` are read when the shell is invoked as an interactive login shell.

The base `/etc/profile` below sets some environment variables necessary for native language support. Setting them properly results in:

-
The output of programs translated into the native language

-
Correct classification of characters into letters, digits and other classes. This is necessary for **bash** to properly accept non-ASCII characters in command lines in non-English locales

-
The correct alphabetical sorting order for the country

-
Appropriate default paper size

-
Correct formatting of monetary, time, and date values

Replace *`<ll>`* below with the two-letter code for the desired language (e.g., “en”) and *`<CC>`* with the two-letter code for the appropriate country (e.g., “GB”). *`<charmap>`* should be replaced with the canonical charmap for your chosen locale. Optional modifiers such as “@euro” may also be present.

The list of all locales supported by Glibc can be obtained by running the following command:

    locale -a

Charmaps can have a number of aliases, e.g., “ISO-8859-1” is also referred to as “iso8859-1” and “iso88591”. Some applications cannot handle the various synonyms correctly (e.g., require that “UTF-8” is written as “UTF-8”, not “utf8”), so it is safest in most cases to choose the canonical name for a particular locale. To determine the canonical name, run the following command, where *`<locale name>`* is the output given by **locale -a** for your preferred locale (“en_GB.iso88591” in our example).

    LC_ALL=*<locale name>* locale charmap

For the “en_GB.iso88591” locale, the above command will print:

    ISO-8859-1

This results in a final locale setting of “en_GB.ISO-8859-1”. It is important that the locale found using the heuristic above is tested prior to it being added to the Bash startup files:

    LC_ALL=<locale name> locale language
    LC_ALL=<locale name> locale charmap
    LC_ALL=<locale name> locale int_curr_symbol
    LC_ALL=<locale name> locale int_prefix

The above commands should print the language name, the character encoding used by the locale, the local currency, and the prefix to dial before the telephone number in order to get into the country. If any of the commands above fail with a message similar to the one shown below, this means that your locale was either not installed in Chapter 6 or is not supported by the default installation of Glibc.

    locale: Cannot set LC_* to default locale: No such file or directory

If this happens, you should either install the desired locale using the **localedef** command, or consider choosing a different locale. Further instructions assume that there are no such error messages from Glibc.

Some packages beyond LFS may also lack support for your chosen locale. One example is the X library (part of the X Window System), which outputs the following error message if the locale does not exactly match one of the character map names in its internal files:

    Warning: locale not supported by Xlib, locale set to C

In several cases Xlib expects that the character map will be listed in uppercase notation with canonical dashes. For instance, "ISO-8859-1" rather than "iso88591". It is also possible to find an appropriate specification by removing the charmap part of the locale specification. This can be checked by running the **locale charmap** command in both locales. For example, one would have to change "de_DE.ISO-8859-15@euro" to "de_DE@euro" in order to get this locale recognized by Xlib.

Other packages can also function incorrectly (but may not necessarily display any error messages) if the locale name does not meet their expectations. In those cases, investigating how other Linux distributions support your locale might provide some useful information.

Once the proper locale settings have been determined, create the `/etc/profile` file:

    cat > /etc/profile << "EOF"
    # Begin /etc/profile

    export LANG=*<ll>_<CC>.<charmap><@modifiers>*

    # End /etc/profile
    EOF

The “C” (default) and “en_US” (the recommended one for United States English users) locales are different. “C” uses the US-ASCII 7-bit character set, and treats bytes with the high bit set as invalid characters. That's why, e.g., the **ls** command substitutes them with question marks in that locale. Also, an attempt to send mail with such characters from Mutt or Pine results in non-RFC-conforming messages being sent (the charset in the outgoing mail is indicated as “unknown 8-bit”). So you can use the “C” locale only if you are sure that you will never need 8-bit characters.

UTF-8 based locales are not supported well by some programs. Work is in progress to document and, if possible, fix such problems, see [http://www.linuxfromscratch.org/blfs/view/8.1/introduction/locale-issues.html](http://www.linuxfromscratch.org/blfs/view/8.1/introduction/locale-issues.html).

## 7.8. Creating the /etc/inputrc File

The `inputrc` file is the configuration file for Readline library, which provides editing capabilities while the user is entering a line from the terminal. It works by tranlating keyboard inputs into specific actions. Readline is used by Bash and most other shells as well as many other applications.

Most people do not need user-specific functionality so the command below creates a global `/etc/inputrc` used by everyone who logs in. If you later decide you need to override the defaults on a per-user basis, you can create a `.inputrc` file in the user's home directory with the modified mappings.

For more information on how to edit the `inputrc` file, see **info bash** under the *Readline Init File* section. **info readline** is also a good source of information.

Below is a generic global `inputrc` along with comments to explain what the various options do. Note that comments cannot be on the same line as commands. Create the file using the following command:

    cat > /etc/inputrc << "EOF"
    # Begin /etc/inputrc
    # Modified by Chris Lynn <roryo@roryo.dynup.net>

    # Allow the command prompt to wrap to the next line
    set horizontal-scroll-mode Off

    # Enable 8bit input
    set meta-flag On
    set input-meta On

    # Turns off 8th bit stripping
    set convert-meta Off

    # Keep the 8th bit for display
    set output-meta On

    # none, visible or audible
    set bell-style none

    # All of the following map the escape sequence of the value
    # contained in the 1st argument to the readline specific functions
    "\eOd": backward-word
    "\eOc": forward-word

    # for linux console
    "\e[1~": beginning-of-line
    "\e[4~": end-of-line
    "\e[5~": beginning-of-history
    "\e[6~": end-of-history
    "\e[3~": delete-char
    "\e[2~": quoted-insert

    # for xterm
    "\eOH": beginning-of-line
    "\eOF": end-of-line

    # for Konsole
    "\e[H": beginning-of-line
    "\e[F": end-of-line

    # End /etc/inputrc
    EOF

## 7.9. Creating the /etc/shells File

The `shells` file contains a list of login shells on the system. Applications use this file to determine whether a shell is valid. For each shell a single line should be present, consisting of the shell's path, relative to the root of the directory structure (/).

For example, this file is consulted by **chsh** to determine whether an unprivileged user may change the login shell for her own account. If the command name is not listed, the user will be denied of change.

It is a requirement for applications such as GDM which does not populate the face browser if it can't find `/etc/shells`, or FTP daemons which traditionally disallow access to users with shells not included in this file.

    cat > /etc/shells << "EOF"
    # Begin /etc/shells

    /bin/sh
    /bin/bash

    # End /etc/shells
    EOF
