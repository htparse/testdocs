---
path: "/community/share/failover-script"
date: "2019-03-07"
title: "Failover Script"
short_description: "With this script it is possible to move the failover IP."
tags: ["Test", "Hetzner Official"]
views: "100"
author: "Hetzner Online"
author_img: ""
author_description: ""
header_img: ""     
---

#FreeBSD Installieren

##Einleitung
Das FreeBSD Rescuesystem wird regelmaessig aktualisiert und stellt bsdinstall zur einfachen Installation bereit.

##Rescue-System starten
Um FreeBSD auf Ihrem Server zu installieren, starten Sie bitte wie unter FreeBSD Rescue-System beschrieben das Rescue-System und melden Sie sich per SSH bei Ihrem Server an. Achten Sie vor der Installation darauf, dass Sie ein aktuelles Backup Ihres Servers vorliegen haben.

##Automatische Installation
Um die Installation durchführen zu können, geben Sie bitte folgenden Befehl ein:

`# bsdinstallimage`

Die Installationsroutine basiert auf dem offiziellen "bsdinstall" von FreeBSD. Wählen Sie zur Installation die gewünschten Optionen wie Version, Architektur und Pakete aus. Die Quelldateien werden vom offiziellen FreeBSD-Mirror geladen. Nach der Installation können Sie nun Ihren Server neustarten:

`# reboot`

Wenige Minuten später sollte Ihr Server per SSH wieder erreichbar sein.

##Manuelle Installation
Es kann auch eine Installation mit dem original Installer von FreeBSD durchgeführt werden. Dazu muss folgender Befehl aufgerufen werden

`bsdinstall`

**Hinweis zu den CX Modellen:** Es muss folgende Zeile zu /boot/loader.conf hinzugefügt werden, da FreeBSD sonst nicht bootet:

`beastie_disable="YES"`

**Hinweis zu den EX41/EX51 und PX61 Modellen:** Es kann folgende Zeile zu /boot/loader.conf hinzugefügt werden, um die ACPI Debugmeldungen zu deaktivieren:

`debug.acpi.disabled="thermal"`

##Netzkonfiguration
Bitte beachten Sie dass Sie die Server ihres eigenen Subnetzes nicht direkt erreichen können, sondern die Pakete immer über das Gateway schicken müssen.

###IPv4
Der entsprechende Routing-Eintrag kann im /etc/rc.conf definiert werden:

```
ifconfig_re0="inet <Haupt-IP>/32"
gateway_if="re0"
gateway_ip="<IP des Gateways>"
static_routes="gateway default"
route_gateway="-host $gateway_ip -interface $gateway_if"
route_default="default $gateway_ip"
```

***Wichtig:*** bei dieser Konfiguration darf "defaultrouter" nicht verwendet werden!


###IPv6
Für die Konfiguration von IPv6 genügt es das default Gateway fe80::1 explizit anzugeben.

```
ipv6_default_interface="re0"
ifconfig_re0_ipv6="2a01:4f8:XX:YY::1:1/64"
# set a static local interface-route
ipv6_defaultrouter="fe80::1%re0"
```

##Zusätzliche IP-Adressen einrichten
Die Einrichtung von zusätzlichen IP-Adressen oder eines Subnetzes unter FreeBSD erschöpft sich in Alias-Einträgen in der ***/etc/rc.conf***. Dabei wird die jeweils erste IP-Adresse für jedes neue Subnetz (d.h. auch wenn die Zusatz-IP in einem anderem Subnetz als die Haupt-IP ist) mit der richtigen Netzmaske eingetragen, die weiteren im gleich Subnetz mit /32 (255.255.255.255)

```
ifconfig_<interface>_alias0="inet <ipadresse1> netmask 255.255.255.248"
ifconfig_<interface>_alias1="inet <ipadresse2> netmask 255.255.255.255"
usw.
```
