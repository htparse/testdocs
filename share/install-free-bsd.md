---
path: "/community/share/install-free-bsd"
date: "2019-02-06"
title: "How to get started with FreeBSD"
short_description: "The FreeBSD Rescue System is updated regularly and provides bsdinstall for easy installation."
tags: ["getting started", "Hetzner Offical"]
views: "100"
author: "Hetzner Online"
author_img: "https://avatars3.githubusercontent.com/u/30047064?s=200&v=4"
author_description: "Web hoster & data center operator | For more than 20 years we've made a name for ourself with our amazingly low prices, quality hardware, efficient data centers, and knowledgable customer support."
header_img: "https://community-cdn-digitalocean-com.global.ssl.fastly.net/assets/tutorials/images/large/get_started_freeBSD.png?1540922872"     
---


##Introduction
The FreeBSD Rescue System is updated regularly and provides bsdinstall for easy installation.

##Starting the Rescue System
To install FreeBSD on your server, you first need to start the FreeBSD Rescue System. Log in to your server via SSH. For reasons of security, it is recommended that you have a backup of your server before proceeding.

## Automatic Installation
The installation can be started by using the following command:

`# bsdinstallimage`

The script is based on the official freebsd "bsdinstall". In the dialogs you can select options like version, architecture and packets. The source file are downloaded from the official freebsd mirror. After the installation is complete, you can restart your server:

`# reboot`

A few minutes later your server will once again be reachable via SSH.

##Manual Installation
An installation can also be performed by using the original installer of FreeBSD. The following command can be used for this:

`bsdinstall`

**A note about the CX models:** The following line must be added to /boot/loader.conf, as FreeBSD will otherwise not boot:

`beastie_disable="YES"`

**A note regarding the EX41/EX51 and PX61 models:** The following line can be added to /boot/loader.conf to disable the ACPI debug messages on the console:

`debug.acpi.disabled="thermal"`

To work around clock skew issues on virtual servers (like CX models), it is also recommended to set

`kern.timecounter.hardware=i8254`

in /etc/sysctl.conf

##Network Configuration
Please note that it is not possible to reach servers on the same subnet directly. Instead, you will have to send the packets to your gateway.

###IPv4
The special routing paramters can be specified in /etc/rc.conf:

```
ifconfig_re0="inet <insert main ip>/32"
gateway_if="re0"
gateway_ip="<insert gateway ip>"
static_routes="gateway default"
route_gateway="-host $gateway_ip -interface $gateway_if"
route_default="default $gateway_ip"
```

**Notice:** do not use the "defaultrouter" keyword with this configuration!

###IPv6

The default IPv6 gateway fe80::1 can be defined in /etc/rc.conf:

```
ipv6_default_interface="re0"
ifconfig_re0_ipv6="2a01:4f8:XX:YY::1:1/64"
# set a static local interface-route
ipv6_defaultrouter="fe80::1%re0"
```

##Configure additional IP addresses
The configuration of an additional IP addresses or subnets in FreeBSD is achieved by adding the alias entries in **/etc/rc.conf**. For each additional subnet (or if the additional IP is on a different subnet than the main IP), the correct netmask must be used on the first IP address for that subnet. All subsequent IP addresses should be added as /32's (255.255.255.255).

```
ifconfig_<interface>_alias0="inet <ipadresse1> netmask 255.255.255.248"
ifconfig_<interface>_alias1="inet <ipadresse2> netmask 255.255.255.255"
etc.
```
