---
path: "/community/share/3ware hardware RAID controller"
date: "2019-02-05"
title: "3ware hardware RAID controller"
short_description: "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua."
tags: ["Test", "Hetzner Offical"]
views: "100"
author: "Hetzner Online"
author_img: ""
author_description: "At vero eos et accusam <a class='link-single-internal' href='#'>Link</a> et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur  <a class='link-single-external' href='#'>Github</a>  sadipscing elitr, sed diam nonumy eirmod tempor"
header_img: ""     
---

##3ware hardware RAID controller
The program developed by 3ware for the administration of the RAID Controller can be found in our download area under:

`http://download.hetzner.de/tools/3ware/`

* The access data for this area can be found in the order completion email for your dedicated server.
* When downloading the program, please make sure you select the architecture suitable for your operating system.
* For a 32-bit Linux operating system please download the following archive:

`tw_cli-linux-x86-9.5.0.tgz`

* For a 64-bit Linux operating system please select archive:

`tw_cli-linux-x86_64-9.5.0.tgz`

* After downloading, extract the archive into a directory of your choice.

Further information on how to use the program may be found in our Download area under:

`http://download.hetzner.de/tools/3ware/tools/CLI/tw_cli.8.html`

##How to read the status of the hardware-based RAID
To read the status of the LSI RAID Controller, the 3ware tw_cli command-line tool needs to be installed. This is already pre-installed in the Rescue System.

The number of the RAID controller can be obtained by tw_cli show. Replace c0 with the appropriate number in the examples below.

The status of the RAID may be obtained by using the following command:

`tw_cli /c0 show`

Example RAID 1:

```
Unit  UnitType  Status         %RCmpl  %V/I/M  Stripe  Size(GB)  Cache  AVrfy
 ------------------------------------------------------------------------------
 u0    RAID-1    OK             -       -       -       698.637   ON     -

 Port   Status           Unit   Size        Blocks        Serial
 ---------------------------------------------------------------
 p0     OK               u0     698.63 GB   1465149168    S13UJ1CQ704597
 p1     OK               u0     698.63 GB   1465149168    S13UJ1BQ708871
```

##How to set up a hardware-based RAID
Regardless of whether changes to the mode of an existing RAID are required or if a new RAID (after installing a controller) is to be set up, the first thing to do is to delete the drives from their "units":

`tw_cli maint deleteunit c0 u0`

`tw_cli maint deleteunit c0 u1`

Setup RAID 0:

`tw_cli maint createunit c0 rraid0 p0:1`

Setup RAID 1 (the recommended configuration with two drives):

`tw_cli maint createunit c0 rraid1 p0:1`

Setup RAID 5 (with three drives):

`tw_cli maint createunit c0 rraid5 p0:1:2`

Setup RAID 5 (with four drives and a starting size of 200GB):

`root@rescue ~ # tw_cli`

`//rescue> /c0 add type=raid5 disk=0:1:2:3 v0=200`

##How to use the drives as JBOD
The 3ware 9650SE controller can configure the drives as JBOD. However, LSI/3ware recommends that the drives instead be configured as single drives.

```
tw_cli /c0 show exportjbod
/c0 JBOD Export Policy = off
```

```
tw_cli /c0 set exportjbod=on
Enabling JBOD Export Policy on /c0... Done.
```

##How to start a REBUILD using tw_cli with a DEGRADED RAID
First, check the status of the RAID controller (see above).

Example for RAID 1:

```
Unit  UnitType  Status         %RCmpl  %V/I/M  Stripe  Size(GB)  Cache  AVrfy
 ------------------------------------------------------------------------------
 u0    RAID-1    DEGRADED       -       -       -       698.637   ON     -

 Port   Status           Unit   Size        Blocks        Serial
 ---------------------------------------------------------------
 p0     DEGRADED         u0     698.63 GB   1465149168    S13UJ1KS210609
 p1     OK               u0     698.63 GB   1465149168    S13UJ1NQ600102
```

The DEGRADED drives need to be deleted from the array and the controller needs to be rescanned:

`tw_cli maint remove c0 p0`

`tw_cli maint rescan c0`

Now the rebuild of the array can be started:

`tw_cli maint rebuild c0 u0 p0`

As soon as this is done the rebuild will start and the status can be queried:

`tw_cli /c0 show rebuild`

Output when the rebuild has been started:

```
Unit  UnitType  Status         %RCmpl  %V/I/M  Stripe  Size(GB)  Cache  AVrfy
 ------------------------------------------------------------------------------
 u0    RAID-1    REBUILDING     5       -       -       698.637   ON     -

 Port   Status           Unit   Size        Blocks        Serial
 ---------------------------------------------------------------
 p0     DEGRADED         u0     698.63 GB   1465149168    S13UJ1KS210609
 p1     OK               u0     698.63 GB   1465149168    S13UJ1NQ600102
```

If the rebuild is interrupted due to ECC errors, you can force a rebuild. This is not recommended:

`tw_cli /c0/u0 start rebuild ignoreECC`
