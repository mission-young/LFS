
## Chapter 9. The End

## 9.1. The End

Well done! The new LFS system is installed! We wish you much success with your shiny new custom-built Linux system.

It may be a good idea to create an `/etc/lfs-release` file. By having this file, it is very easy for you (and for us if you need to ask for help at some point) to find out which LFS version is installed on the system. Create this file by running:

    echo 8.1 > /etc/lfs-release

It is also a good idea to create a file to show the status of your new system with respect to the Linux Standards Base (LSB). To create this file, run:

    cat > /etc/lsb-release << "EOF"
    DISTRIB_ID="Linux From Scratch"
    DISTRIB_RELEASE="8.1"
    DISTRIB_CODENAME="<your name here>"
    DISTRIB_DESCRIPTION="Linux From Scratch"
    EOF

Be sure to put some sort of customization for the field 'DISTRIB_CODENAME' to make the system uniquely yours.

## 9.2. Get Counted

Now that you have finished the book, do you want to be counted as an LFS user? Head over to [http://www.linuxfromscratch.org/cgi-bin/lfscounter.php](http://www.linuxfromscratch.org/cgi-bin/lfscounter.php) and register as an LFS user by entering your name and the first LFS version you have used.

Let's reboot into LFS now.

## 9.3. Rebooting the System

Now that all of the software has been installed, it is time to reboot your computer. However, you should be aware of a few things. The system you have created in this book is quite minimal, and most likely will not have the functionality you would need to be able to continue forward. By installing a few extra packages from the BLFS book while still in our current chroot environment, you can leave yourself in a much better position to continue on once you reboot into your new LFS installation. Here are some suggestions:

-
A text mode browser such as [Lynx](http://www.linuxfromscratch.org/blfs/view/8.1/basicnet/lynx.html) will allow you to easily view the BLFS book in one virtual terminal, while building packages in another.

-
The [GPM](http://www.linuxfromscratch.org/blfs/view/8.1/general/gpm.html) package will allow you to perform copy/paste actions in your virtual terminals.

-
If you are in a situation where static IP configuration does not meet your networking requirements, installing a package such as [dhcpcd](http://www.linuxfromscratch.org/blfs/view/8.1/basicnet/dhcpcd.html)or the client portion of [dhcp](http://www.linuxfromscratch.org/blfs/view/8.1/basicnet/dhcp.html) may be useful.

-
Installing [sudo](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/sudo.html) may be useful for building packages as a non-root user and easily installing the resulting packages in your new system.

-
If you want to access your new system from a remote system within a comfortable GUI environment, install [openssh](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/openssh.html) and its prerequisite, [openssl](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/openssl.html).

-
To make fetching files over the internet easier, install [wget](http://www.linuxfromscratch.org/blfs/view/8.1/basicnet/wget.html).

-
If one or more of your disk drives have a GUID partition table (GPT), either [gptfdisk](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/gptfdisk.html) or [parted](http://www.linuxfromscratch.org/blfs/view/8.1/postlfs/parted.html) will be useful.

-
Finally, a review of the following configuration files is also appropriate at this point.

-
/etc/bashrc

-
/etc/dircolors

-
/etc/fstab

-
/etc/hosts

-
/etc/inputrc

-
/etc/profile

-
/etc/resolv.conf

-
/etc/vimrc

-
/root/.bash_profile

-
/root/.bashrc

-
/etc/sysconfig/network

-
/etc/sysconfig/ifconfig.eth0

Now that we have said that, let's move on to booting our shiny new LFS installation for the first time! First exit from the chroot environment:

    logout

Then unmount the virtual file systems:

    umount -v $LFS/dev/pts
    umount -v $LFS/dev
    umount -v $LFS/run
    umount -v $LFS/proc
    umount -v $LFS/sys

Unmount the LFS file system itself:

    umount -v $LFS

If multiple partitions were created, unmount the other partitions before unmounting the main one, like this:

    umount -v $LFS/usr
    umount -v $LFS/home
    umount -v $LFS

Now, reboot the system with:

    shutdown -r now

Assuming the GRUB boot loader was set up as outlined earlier, the menu is set to boot *LFS 8.1* automatically.

When the reboot is complete, the LFS system is ready for use and more software may be added to suit your needs.

## 9.4. What Now?

Thank you for reading this LFS book. We hope that you have found this book helpful and have learned more about the system creation process.

Now that the LFS system is installed, you may be wondering “What next?” To answer that question, we have compiled a list of resources for you.

-
Maintenance

Bugs and security notices are reported regularly for all software. Since an LFS system is compiled from source, it is up to you to keep abreast of such reports. There are several online resources that track such reports, some of which are shown below:

-
[CERT](http://www.cert.org/) (Computer Emergency Response Team)

CERT has a mailing list that publishes security alerts concerning various operating systems and applications. Subscription information is available at [http://www.us-cert.gov/cas/signup.html](http://www.us-cert.gov/cas/signup.html).

-
Bugtraq

Bugtraq is a full-disclosure computer security mailing list. It publishes newly discovered security issues, and occasionally potential fixes for them. Subscription information is available at [http://www.securityfocus.com/archive](http://www.securityfocus.com/archive).

-
Beyond Linux From Scratch

The Beyond Linux From Scratch book covers installation procedures for a wide range of software beyond the scope of the LFS Book. The BLFS project is located at [http://www.linuxfromscratch.org/blfs/](http://www.linuxfromscratch.org/blfs/).

-
LFS Hints

The LFS Hints are a collection of educational documents submitted by volunteers in the LFS community. The hints are available at [http://www.linuxfromscratch.org/hints/list.html](http://www.linuxfromscratch.org/hints/list.html).

-
Mailing lists

There are several LFS mailing lists you may subscribe to if you are in need of help, want to stay current with the latest developments, want to contribute to the project, and more. See [Chapter 1 - Mailing Lists](Linux%20From%20Scratch.html#ch-intro-maillists) for more information.

-
The Linux Documentation Project

The goal of The Linux Documentation Project (TLDP) is to collaborate on all of the issues of Linux documentation. The TLDP features a large collection of HOWTOs, guides, and man pages. It is located at [http://www.tldp.org/](http://www.tldp.org/).
