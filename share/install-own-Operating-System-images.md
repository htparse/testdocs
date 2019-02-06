##Requirements for installing other Operating System images
Please be aware that Hetzner Online doesn't offer any support for the installation of custom images, and can't offer any warranty on the functionality of the installed system

* The image must be Debian, Ubuntu, OpenSUSE or CentOS to have the correct network settings to be properly configured.
* The image should be a current version of the distribution. Installing a previous version is usually still possible for a certain amount of time after a new version is released, but this is not actively maintained (e.g., CentOS 6.x instead of 7.x or Debian 7.0 instead of 8.0).
* The complete OS must be archived in a .tar.gz format and be placed on a Web, NFS or FTP server. The archive should not contain `/dev`, `/proc` or `/sys` folders.
* The archive must have the name of the distribution and the version (at the second position) within the file name (eg. `Debian-70-image.tar.gz` or `suse-121-backup.tar.gz`).
* The bootloader `grub` must be installed and selected in the configuration file.
* There should only be one kernel in `/boot`.
* For signature verification, the signature (`image.tar.gz.sig`) and the corresponding public key (`public-key.asc`) must be stored in the same directory.

##Procedure
Simply select `installimage` and in the following menu select `custom_images`. There you can choose the blank configuration, which can be configured to suit your requirements.

An example configuration of an image that can be installed from one of our backup servers can be found below.

Please keep in mind that settings like the IMAGEPATH etc. have no paramaters and must be filled in by you.

The available drives are automatically found and given the variables `DRIVE1` and possibly `DRIVE2` (if you have two drives).

If the requirements and procedures outlined above are met, then the installation should proceed without a problem.

##Example configuration file

```
## ===================================================
##  Hetzner Online GmbH - installimage - standardconfig
## ===================================================



## ====================
##  HARD DISK DRIVE(S):
## ====================


# Onboard: QEMU HARDDISK
DRIVE1 /dev/sda




## ============
##  BOOTLOADER:
## ============


## Do not change. This image does not include or support lilo (grub only)!:

BOOTLOADER grub


## ==========
##  HOSTNAME:
## ==========

## which hostname should be set?
##

HOSTNAME Debian-82-jessie-64-minimal



## ==========================
##  PARTITIONS / FILESYSTEMS:
## ==========================

## define your partitions and filesystems like this:
##
## PART  <mountpoint/lvm>  <filesystem/VG>  <size in MB>
##
## * <mountpoint/lvm> mountpoint for this filesystem  *OR*  keyword 'lvm'
##                    to use this PART as volume group (VG) for LVM
## * <filesystem/VG>  can be ext2, ext3, reiserfs, xfs, swap  *OR*  name
##                    of the LVM volume group (VG), if this PART is a VG
## * <size>           you can use the keyword 'all' to assign all the
##                    remaining space of the drive to the *last* partition.
##                    you can use M/G/T for unit specification in MIB/GIB/TIB
##
## notes:
##   - extended partitions are created automatically
##   - '/boot' cannot be on a xfs filesystem!
##   - '/boot' cannot be on LVM!
##   - when using software RAID 0, you need a '/boot' partition
##
## example without LVM (default):
## -> 4GB   swapspace
## -> 512MB /boot
## -> 10GB  /
## -> 5GB   /tmp
## -> all the rest to /home
#PART swap   swap      4096
#PART /boot  ext2       512
#PART /      reiserfs 10240
#PART /tmp   xfs       5120
#PART /home  ext3       all
#
##
## to activate LVM, you have to define volume groups and logical volumes
##
## example with LVM:
#
## normal filesystems and volume group definitions:
## -> 512MB boot  (not on lvm)
## -> all the rest for LVM VG 'vg0'
#PART /boot  ext3     512M
#PART lvm    vg0       all
#
## logical volume definitions:
#LV <VG> <name> <mount> <filesystem> <size>
#
#LV vg0   root   /        ext4         10G
#LV vg0   swap   swap     swap          4G
#LV vg0   tmp    /tmp     reiserfs      5G
#LV vg0   home   /home    xfs          20G
#
#
## your system has the following devices:
#
# Disk /dev/sda: 42 GB (=> 40 GiB)
#
## Based on your disks and which RAID level you will choose you have
## the following free space to allocate (in GiB):
# RAID  0: ~40
# RAID  1: ~40
#

PART / ext3 all



## ========================
##  OPERATING SYSTEM IMAGE:
## ========================

## full path to the operating system image
##   supported image sources:  local dir,  ftp,  http,  nfs
##   supported image types: tar, tar.gz, tar.bz, tar.bz2, tar.xz, tgz, tbz, txz
## examples:
#
# local: /path/to/image/filename.tar.gz
# ftp:   ftp://<user>:<password>@hostname/path/to/image/filename.tar.bz2
# http:  http://<user>:<password>@hostname/path/to/image/filename.tbz
# https: https://<user>:<password>@hostname/path/to/image/filename.tbz
# nfs:   hostname:/path/to/image/filename.tgz
#
# for validation of the image, place the detached gpg-signature
# and your public key in the same directory as your image file.
# naming examples:
#  signature:   filename.tar.bz2.sig
#  public key:  public-key.asc

IMAGE /root/.oldroot/nfs/install/../images/Debian-82-jessie-64-minimal.tar.gz
```
