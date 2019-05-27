##Gentoo Installation
This guide is partly based on the Gentoo Handbook: <http://www.gentoo.org/doc/en/handbook/>

##Booting the Rescue System
To install Gentoo on dedicated or virtual server, the server must be booted into the Rescue System. This can be done via Robot.

##Preparing the drives
After the server has successfully booted into the Rescue System, the drive(s) must be partitioned.
Once the partitions are created and formated, they can be mounted using /mnt/, as described in the Gentoo Handbook (Chapter 4).

##Installing Gentoo
From this point on the Handbook on installing Gentoo can be followed.

Tip: The network card configuration can be regulated at Hetzner through DHCP. Just follow the simple instructions in the guide for DHCP and install DHCPCD.

**Important tip:** so you can log in after rebooting the system: you must make sure **BEFORE** the first reboot that sshd is starting up at boot time, as you can otherwise lock yourself out right at the beginning. The following can be added to make sshd a default runlevel:

`rc-update add sshd default`

`nano -w /etc/ssh/sshd_config (set configuration)`

Since after the first reboot there are no users (they haven't been created yet) you should enable root login in sshd_config before rebooting.

Tip: To see if SSH works you can change the port (the Rescue System works with port 22) and start the server with */etc/init.d/sshd* start to test the login (with the new root password).

After rebooting you will have a clean Gentoo system on the server.

##Providing Gentoo with packages and updates
Hetzner operates an internal mirror server that provides the portage tree and distfiles, but no stage3 tarballs. These must be obtained from one of the public mirrors.


**Distribution sources (including the Stage3 Tarballs)**

* Mirror list: <http://www.gentoo.org/main/de/mirrors2.xml>
The tarballs can be found under releases/x86/autobuilds/current-stage3/ (replace x86with amd64 for 64bit systems, rather than 32bit)

**Portage Tree**

To use the mirror for both the distfiles and the portage tree, this can be selected via the mirrorselect tool. Mir information on this can be found in the Gentoo Handbook
<http://www.gentoo.org/doc/de/handbook/handbook-x86.xml?part=1&chap=6#doc_chap1>

Alternatively this can be manually specified in /etc/make.conf:

```
/etc/make.conf:
GENTOO_MIRRORS="ftp://mirror.hetzner.de/gentoo/"
SYNC="rsync://mirror.hetzner.de/gentoo/portage"
```
