##Voraussetzungen für die Installation anderer Betriebssystem-Images
Bitte beachten Sie, dass Hetzner Online für eigene Image-Installationen keinen Support leisten bzw. keine Garantie auf Funktionalität des installierten Systems geben kann

* Das Image muss ein Debian, Ubuntu, OpenSUSE oder CentOS sein, um korrekte Netzwerkeinstellungen vornehmen zu können.
+ Das Image sollte eine aktuelle Version der jeweiligen Distribution sein. Die Installation einer Vorgängerversion ist in der Regel eine gewisse Zeit nach Erscheinen der aktuellen Version noch möglich, wird aber nicht aktiv gepflegt (z.B. CentOS 6.x statt CentOS 7.x oder Debian 7.0 statt 8.0).
* Das komplette Betriebssystem muss im gepackten .tar.gz Format auf einem Web-, NFS oder FTP-Server hinterlegt sein. Das Archiv sollte ohne "/dev", "/proc" und "/sys"-Ordner gepackt werden.
* Das Archiv muss die Distribution und die Version (an zweiter Stelle) im Dateinamen enthalten. (z.B. "Debian-70-image.tar.gz" oder "suse-121-backup.tar.gz")
* Der Bootloader "grub" muss installiert sein und die entsprechende Option in der Konfigurationsdatei angegeben werden.
* Es darf nur ein Kernel in "/boot" liegen
* Zur Signaturprüfung muss die Signatur (image.tar.gz.sig) und der entsprechende Public Key (public-key.asc) im gleichen Verzeichnis abgelegt sein

##Vorgehensweise
Einfach 'installimage' aufrufen und im Menu den Menupunkt "custom_images" auswählen. Dort gibt es eine Blanko-Konfiguration zur Auswahl die entsprechend den Wünschen geändert werden muss.

Eine Beispielkonfigurationsdatei für ein Image, das von einem unserer Backupserver aus aufgespielt werden soll ist unten im Anhang zu finden.

Hierbei ist zu beachten, dass Einstellungen wie IMAGEPATH etc. momentan noch ohne Parameter sind und diese angegeben werden müssen.

Die verfügbaren Festplatten werden automatisch erkannt und die erste und ggf. eine zweite Festplatte an die Variablen "DRIVE1" und "DRIVE2" angehängt.

Sollten alle oben genannten Voraussetzungen erfüllt sein sollte die Installation ohne Probleme durchlaufen und das System wie jedes andere Standardimage bootfähig installieren.

##Beispielkonfigurationsdatei

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
