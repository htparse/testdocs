#Main IP address
##IPv4
###Dedicated Servers
The main IP address of a dedicated server usually comes from a /26 or /27 subnet. In order to prevent (accidental) adoption of foreign IP addresses, communication is only possible via the gateway address.

In order to communicate with servers in the same network segment, a point-to-point setup is configured within the default installation, which directs all packets to the gateway.

When configuring via DHCP this particular configuration cannot be transmitted, meaning a normal configuration (without a /32 subnet) applied. This is not a problem, unless IPs must be reached from the same subnet. In order to reach any server in the same subnet, you need to use /32 subnet in network configuration:

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR=<MAC Address>
ONBOOT=yes
BOOTPROTO=static
IPADDR=<IP Address>
NETMASK=255.255.255.255
SCOPE="peer <Gateway IP>"
# potential additional IPv6 entries
```

The default route is created separately:

```
##/ etc/sysconfig/network-scripts/route-eth0
ADDRESS0=0.0.0.0
NETMASK0=0.0.0.0
GATEWAY0=<Gateway IP>
```

**Possible sources of error**

If it is not possible to reach the server after configuring the above mentioned network settings, it is necessary to check whether the variable "GATEWAYDEV" has been set in the file /etc/sysconfig/network. This may be one reason for non-availability and is indicated by the error message "RTNETLINK answers: file exists" which appears after reloading the network settings.

###Virtual Servers
With virtual servers the configuration is done in the standard installation and does not include any special settings. It corresponds to the confoguration that can be achieved by using DHCP. Server on the same subnet can be reached without any further adjustments.

##IPv6
Dedicated Servers / CX Virtual Servers
Each server receives a /64 IPv6 subnet. Unlike the IPv4 configuration, a point-to-point setup is not necessary.

Example:

* Address block: 2a01:4f8:61:20e1::1 to 2a01:4f8:61:20e1:ffff:ffff:ffff:ffff
* Of which we use the first IP: 2a01:4f8:61:20e1::2
* Gateway: fe80::1

To enable IPv6 on your server add the following lines to the file **/etc/sysconfig/network-scripts/ifcfg-eth0**:

```
IPV6INIT=yes
IPV6ADDR=<IPv6 Address>/<Prefix>
IPV6_DEFAULTGW=fe80::1
IPV6_DEFAULTDEV=eth0
```

**Optional:** To add additional IPv6 addresses to the interface in the file **/etc/sysconfig/network-scripts/ifcfg-eth0** please add the following line:

`IPV6ADDR_SECONDARIES=<IPv6 Address>/<Prefix>`

Note that you can add as many IPv6 addresses as you want, separated by a space.

###Virtual Servers (VQ/VX Models)
With virtual servers the gateway of the IPv6 subnet is located within the allocated subnet.

Example:

* Address block: 2a01:4f8:61:20e1::2 to 2a01:4f8:61:20e1:ffff:ffff:ffff:ffff
* Gateway: 2a01:4f8:61:20e1::1

```
#/etc/sysconfig/network-scripts/ifcfg-eth0
IPV6INIT=yes
IPV6ADDR=<IPv6 Address>/64
IPV6_DEFAULTGW=<Gateway IP>
IPV6_DEFAULTDEV=eth0
```

#Additional IP addresses (Host)
##Setting up additional single IPv4 addresses
The IP addresses can be made temporarily available in two different ways:

1. ifconfig eth0:1 192.0.2.10 netmask 255.255.255.255 or
2. ip addr add 192.0.2.10/32 dev eth0

###CentOS
A permanent configuration is only possible by default via alias interfaces (eth0:1, eth0:2 etc.). A file needs to be created for each IP address:

```
/etc/sysconfig/network-scripts/ifcfg-eth0:1
/etc/sysconfig/network-scripts/ifcfg-eth0:2
```

These files must include the following information:

```
DEVICE=eth0:1
BOOTPROTO=none
ONBOOT=yes
IPADDR=<IP Address>
NETMASK=255.255.255.255
```

Finally, a "service network restart" needs to be initiated or the server needs to be restarted ("reboot").

**Please note:** A different configuration is needed for the use of IP addresses in virtual machines!

###Fedora
For a permanent configuration the IP addresses can be added in the corresponding configuration file:

```
# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
...
IPADDR=192.0.2.1
NETMASK=255.255.255.240
IPADDR0=192.0.2.10 # Additional IP
PREFIX0=28
IPADDR1=192.0.2.11
PREFIX1=28
IPADDR2=...
```

##Setting up additional IPv4 subnets
Subnets are routed on the main IP of a server. In general the first (Network IP) and the last (Broadcast IP) cannot be used. This leaves six usable addresses for a /29 subnet.

A /29 subnet consisting of 8 IP addresses will look like this:

```
aaa.aaa.aaa.aaa (Network IP)
bbb.bbb.bbb.bbb
ccc.ccc.ccc.ccc
ddd.ddd.ddd.ddd
eee.eee.eee.eee
fff.fff.fff.fff
ggg.ggg.ggg.ggg
hhh.hhh.hhh.hhh (Broadcast IP)
```

The IPs "b" to "g" can be configured and used as single IPs. Alternatively, a file can be created:

*/etc/sysconfig/network-scripts/ifcfg-eth0-range0*

```
IPADDR_START=<Your Network Address + 1>
IPADDR_END=<Your Network Address + 6>
BROADCAST=<Your Network Address + 7>
CLONENUM_START=0
NETMASK=255.255.255.248
```

Restart the service network using "service network restart".

##Setting up an additional IPv4 subnet for virtualization
A different configuration is needed for the use of IP addresses in virtual machines. There are many possible configurations. One of the more straightforward ones is to setup a bridge device using one IP address of the subnet which serves as default gateway for all machines connected to the subnet.

A prerequisite is that the bridge-utils are installed:

`yum install bridge-utils`

***/etc/sysconfig/network-scripts/ifcfg-br0***

```
DEVICE=br0
ONBOOT=yes
TYPE=Bridge
BOOTPROTO=none
IPADDR=bbb.bbb.bbb.bbb
NETMASK=255.255.255.248 # adjust this accordingly. This is for a /29 subnet
STP=off
DELAY=0
```

#Additional IP addresses (Virtualization)
With virtualization the additional IP addresses are used through the guest system. So that these can be reached via the Internet, configuration in the host system needs to be adjusted accordingly in order to forward the packets. There are two ways of doing this for additional single IPs: Routed and Bridged.

##Routed (brouter)
In a routed configuration the packets are routed. In addition to eth0 a bridge needs to be set up with almost the same configuration (without gateway) as eth0.

Host:

```
# /etc/sysconfig/network-scripts/ifcfg-eth0 (Hetzner Standard Installation)
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
IPADDR=<Main IP>
IPV6INIT=yes
IPV6ADDR=2a01:4f8:XXX:YYYY::2/128
IPV6_DEFAULTGW=fe80::1
IPV6_DEFAULTDEV=eth0
NETMASK=255.255.255.255
SCOPE="peer <Default Gateway>"
```

```
# /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
ONBOOT=yes
TYPE="Bridge"
BOOTPROTO=static
IPADDR=<Main IP>
NETMASK=255.255.255.255
IPV6INIT=yes
IPV6ADDR=2a01:4f8:XXX:YYYY::2/64
STP=off
DELAY=0
```

The configuration of eth0 for IPv4 remains unchanged if it is a standard installation via installimage/Robot (the default gateway is entered in the file *route-eth0*.) . For IPv6 the prefix is reduced from /64 to /128. The setting of the host routes for the additional IPv4 addresses is done via an additional configuration file:

```
# /etc/sysconfig/network-scripts/route-br0
ADDRESS0=<Additional IP>
NETMASK0=255.255.255.255
```

Further routes can be added in the same way via ADDRESS1, NETMASK1, ADDRESS2, NETMASK2, etc. For IPv6 no further configuration is required.

Guest:

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
IPADDR=<Addon IP>
NETMASK=255.255.255.255
SCOPE="peer <Main IP>"
IPV6INIT=yes
IPV6ADDR=2a01:4f8:XXX:YYYY::4/64
IPV6_DEFAULTGW=2a01:4f8:XXX:YYYY::2
```

##Bridged
In a bridged configuration, packets are sent directly. The guest system behaves as if independent. As this makes the MAC addresses of the guest system visible from the outside, a virtual MAC address needs to be requested for each single IP address via the Hetzner Robot and assigned to the guest NIC.

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
# device: eth0
DEVICE=eth0
BOOTPROTO=static
HWADDR=<MAC of the physical NIC>
ONBOOT=yes
BRIDGE=br0
```

```
# /etc/sysconfig/network-scripts/ifcfg-br0 (pointopoint, Hetzner Standard)
DEVICE=br0
TYPE="Bridge"
BOOTPROTO=static
IPADDR=<Main IP>
NETMASK=255.255.255.255
SCOPE="peer <Gateway of the main IP>"
ONBOOT=yes
DELAY=0
```
The default route is set up via the additional *route-eth0* configuration file. Simply rename it *route-br0*.

**Please note:** In this configuration the use of IPv6 is limited. The IPv6 subnet can be routed via the Robot to either the main IP address or ONE of the additional IP addresses. (or more precisely: to the IPv6 link local address, that is generated from the MAC address)
