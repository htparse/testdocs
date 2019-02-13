##Main IP Address
###IPv4
####Dedicated Servers
The main IP of a dedicated server is usually located in a /26 or /27 subnet. In order to prevent the accidental use of a foreign IP address, our infrastructure rejects any Ethernet packets that are not addressed to the gateway address. In order to reach a server in the same subnet, our standard images already have a static route in their network configuration. The static route forwards the entire traffic to the subnet via the gateway.

This is not the best solution as duplicate and inconsistent information appears in the routing table. A better way to reach a server in your subnet is to set the netmask to `255.255.255.255 (/32)`. The server assumes it is alone in this subnet and will not send any packets directly. However, an explicit host route to the gateway is now needed. This is very easy to do with Debian by adding the option `pointopoint 192.168.0.1` in the configuration. Please change `192.168.0.1` to the valid IP address of your gateway.

```
## /etc/network/interfaces example Hetzner root server
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# LAN interface
auto eth0
iface eth0 inet static
  # Main IP address of the server
  address 192.168.0.250
  # Netmask 255.255.255.255 (/32) independent from the
  # real subnet size (e.g. /27)
  netmask 255.255.255.255
  # explicit host route to the gateway
  gateway 192.168.0.1
  pointopoint 192.168.0.1
```

The additional route to the gateway is now no longer necessary.

####vServers (CX models)
The configuration of standard installations is done via DHCP since, with CX vServers, the public IP is assigned to the interal IP via 1:1 NAT. A static configuration is possible - but it is not recommended since future new features might require you to make adjustments.

###IPv6
####Dedicated Servers / CX vServers
In principle the above applies to IPv6 as well. But instead of a single main IP, you get a /64 block.

As opposed to the IPv4 configuration, there is no "pointopoint" setting in IPv6.

For example:

* Address block: `2a01:4f8:61:20e1::2` until `2a01:4f8:61:20e1:ffff:ffff:ffff:ffff`
* We use the first address from this: `2a01:4f8:61:20e1::2`
* Gateway: `fe80::1`

```
## /etc/network/interfaces example Hetzner root server
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# IPv6 LAN
auto eth0
iface eth0 inet6 static
  # Main IPv6 Address of the server
  address 2a01:4f8:61:20e1::2
  netmask 64
  gateway fe80::1
```

###IPv4 + IPv6
It is expected that over the next few years, IPv4 and IPv6 will be used in parallel. Both configuration files are simply joined together and duplicate entries omitted.

####Dedicated Servers

```
## /etc/network/interfaces example Hetzner root server
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# LAN interface
auto eth0
iface eth0 inet static
  # Main IP address of the server
  address 192.168.0.250
  # Netmask 255.255.255.255 (/32) independent from the
  # real subnet size (e.g. /27)
  netmask 255.255.255.255
  # explicit host route to the gateway
  gateway 192.168.0.1
  pointopoint 192.168.0.1
#
iface eth0 inet6 static
  # Main IPv6 Address of the server
  address 2a01:4f8:61:20e1::2
  netmask 64
  gateway fe80::1
```

####Virtual Servers

```
## /etc/network/interfaces Example Hetzner Virtual Server
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# LAN interface
auto eth0
iface eth0 inet static
  # Main IP address of the server
  address 192.168.0.250
  netmask 255.255.255.224
  gateway 192.168.0.1
#
# IPv6 LAN
iface eth0 inet6 static
  # Main IPv6 Address of the server
  address 2a01:4f8:61:20e1::2
  netmask 64
  gateway 2a01:4f8:61:20e1::1
```

##Additional IP Addresses (Host)
For our dedicated root servers, you can order up to 6 additional single IPs. The network configuration is similar in both cases.

In order to use the additional addresses on the server (no virtualization) the package "iproute" and service program "ip" are needed. Configuration with alias interfaces (such as eth0:1, eth0:2 etc.) are outdated and should no longer be used. To add an address please run:

`ip addr add 10.4.2.1/32 dev eth0`

The command `ip addr` shows the IP addresses which are currently active. As the server uses the entire subnet, it is also useful here to add the addresses with the prefix `/32`, which means the subnet mask is `255.255.255.255`

###Configuration
In **/etc/network/interfaces**, insert the following two lines in the appropriate interface (e.g. "eth0"):

```
up ip addr add 10.4.2.1/32 dev eth0
down ip addr del 10.4.2.1/32 dev eth0
```

"up" and "down" expect just one line of shell code and this can be repeated for several addresses. The disadvantage is that both the interface name and address must be listed twice. If many IPs are used, the configuration file becomes confusing and prone to errors. If the data is changed, all entries need to be adjusted.

###Alternative configuration via addresses script

**Attention:** The following instructions involve the installation of software by a third party <http://www.wertarbyte.de>. This is not supported by Hetzner Online. In the event of errors or problems, please contact the developer.

The script is in package "ifupdown-scripts-wa", which is not a part of the official Debian distribution. If the following line is added for APT configuration, the `apt-get install ifupdown-scripts-wa` command is sufficient in order to install the script correctly:

```
# /etc/apt/sources.list.d/wertarbyte.list
# Tartarus, ifupdown-scripts etc.
deb http://wertarbyte.de/apt/ ./
```

The complete installation routine can be shortened using the following commands:

```
wget -P/etc/apt/sources.list.d/ http://wertarbyte.de/apt/wertarbyte-apt.list
wget -O - http://wertarbyte.de/apt/software-key.gpg | apt-key add -
apt-get update
apt-get install ifupdown-scripts-wa
```

If you do not wish to install the script using the package system, it can also be downloaded manually: <http://wertarbyte.de/debian/ifupdown/addresses>. It is filed in the **/etc/network/if-up.d/** directory and also linked with **/etc/network/if-down.d/**:

```
cd /etc/network/if-up.d/ && \
wget http://wertarbyte.de/debian/ifupdown/addresses && \
chmod +x addresses && \
cd ../if-down.d/ && \
ln -s ../if-up.d/addresses .
```

Installation via packet system is recommended as the current version of the script is always available.

The script extends the syntax of the configuration file by adding a new command "addresses". This enables the specification of additional binding IP addresses (with the netmask in /-notation):

`addresses 10.4.2.1/32 10.4.2.2/32 10.4.2.3/32`

If this line is added to configure the "eth0" interface, addresses are added upon activating the interface and removed upon deactivation.

It is also possible to use several lines to bundle addresses into categories and to make configuration more transparent:

```
addresses       10.4.2.1/32
addresses-https 10.4.2.2/32 10.4.2.3/32 # SSL-Websites
addresses-mail  10.4.2.4/32             # Mailserver
```

The script captures various commands that start with the key word "addresses-" and a label of your choice. Labels should not be used twice, as otherwise a syntax error is shown for ifupdown and configuration of the interface is interrupted. This can result in the server not being reachable.

The IP addresses which have been added via "ip addr" are not visible in the output of "ifconfig" ; the command "ip addr show" is required to show these. However, the addresses script can also set up alias devices:

```
addresses 10.0.0.1/32 10.0.0.2/32 10.0.0.3/32
create_alias_devices yes
```

The script creates consecutively numbered eth0:X devices using this configuration, which are also visible in "ifconfig".

Instead of simply numbering the devices, it is also possible to use the labels from the configuration:

```
addresses-https 10.0.0.1/32 10.0.0.3/32
addresses-vhost 10.0.0.2/32
label_addresses yes
```

The addresses are subsequently labelled "eth0:https" or "eth0:vhost" in the output of "ip addr" and are also shown in "ifconfig".

##Additional IP Addresses (Virtualization)
With virtualization the additional IP addresses are used via the guest system. So that these can be reached via the Internet, configuration in the host system needs to be adjusted accordingly in order to forward the packets. There are two ways of doing this for additional single IPs: Routed and Bridged.

###Routed (brouter)
In this type of configuration, the packets are routed. This requires the setting up of an additional bridge with almost the same configuration (without gateway) as eth0.

Host:

```
auto eth0
iface eth0 inet static
   address (Main IP)
   netmask 255.255.255.255
   pointopoint (Gateway IP)
   gateway (Gateway IP)
#
iface eth0 inet6 static
  address 2a01:4f8:XX:YY::2
  netmask 128
  gateway fe80::1
#
auto virbr1
iface virbr1 inet static
   address (Main IP)
   netmask 255.255.255.255
   bridge_ports none
   bridge_stp off
   bridge_fd 0
   pre-up brctl addbr virbr1
   up ip route add (Additional IP)/32 dev virbr1
   down ip route del (Additional IP)/32 dev virbr1
 #
 iface virbr1 inet6 static
   address 2a01:4f8:XX:YY::2
   netmask 64
```

A corresponding host route needs to be created for each additional IP address. The eth0 configuration remains unchanged for IPv4. For IPv6 the prefix is reduced from /64 to /128.

Guest:

```
auto eth0
iface eth0 inet static
   address (Additional IP)
   netmask 255.255.255.255
   pointopoint (Main IP)
   gateway (Main IP)
#
iface eth0 inet6 static
  address 2a01:4f8:XX:YY::4
  netmask 64
  gateway 2a01:4f8:XX:YY::2
```

###Bridged
With a bridged configuration, packets are sent directly. The guest system behaves as if it is independent. As this makes the MAC addresses of the guest system visible from the outside, a virtual MAC address needs to be requested for each IP address via the Hetzner Robot and assigned to the guest network card. The bridge gets the same network configuration as eth0.

```
# remove or disable configuration for eth0
#auto eth0
#iface eth0 inet static
#
auto  br0
iface br0 inet static
 address (Main IP)
 netmask (like eth0, e.g: 255.255.255.254)
 gateway (same as that for the main IP)
 bridge_ports eth0
 bridge_stp off
 bridge_fd 1
 bridge_hello 2
 bridge_maxage 12
```
 
The configuration of eth0 is omitted without replacement.
