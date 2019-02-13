#Haupt-IP-Adresse
##IPv4
###Root-Server
Die Hauptadresse eines Root-Servers liegt in der Regel in einem /26 oder /27-Netz. Um die (versehentliche) Übernahme fremder IP-Adresse zu verhindern, ist ausschließlich die Kommunikation mit der Gateway-Adresse möglich.

Um auch Server im gleichen Netzsegment ansprechen zu können, wird daher in der Standardinstallation eine Point-to-Point Setup eingerichtet, die alle Pakete an das Gateway leitet.

Bei der Konfiguration via DHCP kann diese spezielle Konfiguration nicht übermittelt werden, so daß hier eine normale Konfiguration (kein /32-Netz) erfolgt. Dies ist in der Regel unproblematisch, außer es müssen IPs im gleichen Subnetz erreicht werden. Ohne die unten stehende Konfiguration können diese Hosts nicht erreicht werden.

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR=<MAC-Adresse>
ONBOOT=yes
BOOTPROTO=static
IPADDR=<IP-Adresse>
NETMASK=255.255.255.255
SCOPE="peer <gateway IP>"
# ev. zusätzliche IPv6-Einträge
```

Die default route wird separat angelegt.

```
##/ etc/sysconfig/network-scripts/route-eth0
ADDRESS0=0.0.0.0
NETMASK0=0.0.0.0
GATEWAY0=<gateway IP>
```

**Mögliche Fehlerquellen**

Wenn der Server nach Setzen der oben genannten Netzwerkeinstellungen nicht mehr erreichbar ist, ist zu prüfen, ob in der /etc/sysconfig/network die Variable "GATEWAYDEV" gesetzt ist. Dies kann eine mögliche Ursache für eine Nichterreichbarkeit sein. Das Problem äußert sich dadurch, dass nach Neuladen der Netzwerkeinstellungen die Fehlermeldung "RTNETLINK answers: file exists" erscheint.

###vServer
Bei vServern erfolgt die Konfiguration in der Standardinstallation ohne spezielle Konfiguration und entspricht der, die auch per DHCP bezogen werden kann. Server im gleichen Subnetz können ohne weitere Anpassungen direkt erreicht werden.

##IPv6
###Root-Server / CX-vServer
Jeder Server erhält ein /64 IPv6-Netz. Im Gegensatz zur IPv4 Konfiguration ist für IPv6 keine spezielle "pointopoint" Konfiguration nötig.

Beispiel:

* Adressblock: 2a01:4f8:61:20e1::1 bis 2a01:4f8:61:20e1:ffff:ffff:ffff:ffff
* Davon verwenden wir die erste Adresse 2a01:4f8:61:20e1::2
* Gateway: fe80::1

Aktivieren Sie IPv6 an Ihrem Server indem Sie folgende Zeilen der Datei **/etc/sysconfig/network-scripts/ifcfg-eth0** hinzufügen

```
IPV6INIT=yes
IPV6ADDR=<Ihre IPv6-Adresse>/<Prefix>
IPV6_DEFAULTGW=fe80::1
IPV6_DEFAULTDEV=eth0
```

**Optional:** Hinzufügen weiterer IPv6-Adressen zum Interface In der Datei **/etc/sysconfig/network-scripts/ifcfg-eth0** ergänzen Sie bitte folgende Zeile:

`IPV6ADDR_SECONDARIES=<Ihre IPv6-Adresse>/<Prefix>`

Beachten Sie, dass Sie hier beliebig viele, durch Leerzeichen getrennte IPv6-Adressen angeben können.

###vServer (VQ/VX-Modellreihe)
Bei vServer liegt anders als bei Root-Servern das Gateway innerhalb des zugeteilten /64 Subnetzes.

Beispiel:

* Adressblock: 2a01:4f8:61:20e1::2 bis 2a01:4f8:61:20e1:ffff:ffff:ffff:ffff
* Gateway: 2a01:4f8:61:20e1::1

```
#/etc/sysconfig/network-scripts/ifcfg-eth0
IPV6INIT=yes
IPV6ADDR=<IPv6-Adresse>/64
IPV6_DEFAULTGW=<Gateway-IP>
IPV6_DEFAULTDEV=eth0
```

#Zusätzliche IP-Adressen (Host)
##Einrichten einer zusätzlichen Einzel-IPv4-Adressen
Die IP-Addressen können temporär auf zwei verschiedene Arten nutzbar gemacht werden:

1. ifconfig eth0:1 192.0.2.10 netmask 255.255.255.255 oder
2. ip addr add 192.0.2.10/32 dev eth0

###CentOS
Eine dauerhafte Konfiguration ist standardmäßig nur über Alias-Interfaces (eth0:1, eth0:2 usw.) möglich. Dazu muß pro IP-Adresse eine Datei angelegt werden.

```
/etc/sysconfig/network-scripts/ifcfg-eth0:1
/etc/sysconfig/network-scripts/ifcfg-eth0:2
```

Diese Dateien müssen jeweils folgende Informationen enthalten:

```
DEVICE=eth0:1
BOOTPROTO=none
ONBOOT=yes
IPADDR=<IP-Adresse>
NETMASK=255.255.255.255
```

Anschließend muss ein "service network restart" ausgelöst oder der Server neu gestartet werden ("reboot").

**HINWEIS:** Für die Nutzung der IP-Adressen in virtuellen Maschinen ist eine andere Konfiguration notwendig!

###Fedora
Für eine dauerhafte Konfiguration der Adressen können diese in der entsprechenden Konfigurationsdatei hinzugefügt werden:

```
# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
...
IPADDR=192.0.2.1
NETMASK=255.255.255.240
IPADDR0=192.0.2.10 # Zusatz-IP
PREFIX0=28
IPADDR1=192.0.2.11
PREFIX1=28
IPADDR2=...
```

##Einrichten eines zusätzlichen IPv4-Subnetzes
Subnetze werden auf die Haupt-IP eines Servers geroutet. In der Regel wird die erste (Netz-IP) und die letzte (Broadcast) nicht verwendet, netto bleiben damit sechs Adressen übrig.

Nennen wir sie hier im Folgenden

```
aaa.aaa.aaa.aaa (Netz-IP)
bbb.bbb.bbb.bbb
ccc.ccc.ccc.ccc
ddd.ddd.ddd.ddd
eee.eee.eee.eee
fff.fff.fff.fff
ggg.ggg.ggg.ggg
hhh.hhh.hhh.hhh (Broadcast)
```

Die IPs "b" bis "g" können wie bei Einzel-IPs nutzbar gemacht werden. Alternativ können Sie eine Datei anlegen

*/etc/sysconfig/network-scripts/ifcfg-eth0-range0*

```
IPADDR_START=<Ihre-Netzwerk-Adresse + 1>
IPADDR_END=<Ihre Netzwerk-Adresse + 6>
BROADCAST=<Ihre Netzwerk-Adresse + 7>
CLONENUM_START=0
NETMASK=255.255.255.248
```

Starten Sie den Netzwerk-Service mit Hilfe von "service network restart" neu.

##Einrichten eines zusätzlichen IP-Subnetzes für Virtualisierung
Für die Nutzung der IP-Adressen in virtuellen Maschinen ist eine andere Konfiguration notwendig. Es gibt mehrere Möglichkeiten ein Subnetz zu konfigurieren. Die einfachste Variante besteht aus der Einrichtung einer zusätzlichen Bridge, welche mit einer IP-Adresse aus dem Subnetz konfiguriert wird. Diese dient als Standardgateway für die virtuellen Maschinen.

Voraussetzung sind die installierten bridge-utils:

`yum install bridge-utils`

***/etc/sysconfig/network-scripts/ifcfg-br0***

```
DEVICE=br0
ONBOOT=yes
TYPE=Bridge
BOOTPROTO=none
IPADDR=bbb.bbb.bbb.bbb
NETMASK=255.255.255.248 # entsprechend anpassen. Dies ist für eine /29 Subnetz
STP=off
DELAY=0
```

#Zusätzliche IP-Adressen (Virtualisierung)
Bei Einsatz von Virtualisierung werden die zusätzliche IP-Adressen durch die Gast-Systeme genutzt. Damit diese im Internet erreichbar sind, muß im Hostsystem eine entsprechende Konfiguration entsprechend angepasst werden, um die Pakete weiterzuleiten. Dabei gibt es für zusätzliche Einzel-IPs zwei Möglichkeiten: Routed und Bridged.

##Routed (brouter)
Bei einer Routed-Konfiguration werden die Pakete geroutet. Dafür muß zusätzlich zu eth0 eine Bridge mit nahezu gleicher Konfiguration (ohne Gateway) wie eth0 angelegt werden.

Host:

```
# /etc/sysconfig/network-scripts/ifcfg-eth0 (Hetzner Standardinstallation)
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
IPADDR=<Haupt-IP>
IPV6INIT=yes
IPV6ADDR=2a01:4f8:XXX:YYYY::2/128
IPV6_DEFAULTGW=fe80::1
IPV6_DEFAULTDEV=eth0
NETMASK=255.255.255.255
SCOPE="peer <Default-GW>"
```

```
# /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
TYPE="Bridge"
ONBOOT=yes
BOOTPROTO=none
IPADDR=<Haupt-IP>
NETMASK=255.255.255.255
IPV6INIT=yes
IPV6ADDR=2a01:4f8:XXX:YYYY::2/64
STP=off
DELAY=0
```

Die Konfiguration von eth0 für IPv4 unverändert, sofern es sich um eine Standardinstallation via installimage/Robot handelt (Das Default-Gateway wird dabei in der Datei *route-eth0* eingerichtet.). Für IPv6 wird das Prefix von /64 auf /128 reduziert. Das Setzen der Host-Routen für die Zusatz-IPv4 Adressen geschieht über eine weitere Konfigurationsdatei:

```
# /etc/sysconfig/network-scripts/route-br0
ADDRESS0=<Zusatz-IP>
NETMASK0=255.255.255.255
```

Weitere Routen können analog über ADDRESS1, NETMASK1, ADDRESS2, NETMASK2, usw. hinzugefügt werden. Für IPv6 ist keine weitere Konfiguration nötig.

Gast:

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
IPADDR=<Zusatz-IP>
NETMASK=255.255.255.255
SCOPE="peer <Haupt-IP>"
IPV6INIT=yes
IPV6ADDR=2a01:4f8:XXX:YYYY::4/64
IPV6_DEFAULTGW=2a01:4f8:XXX:YYYY::2
```

##Bridged
Bei einer Bridge-Konfiguration werden die Pakete direkt zugestellt. Das Gast-System verhält sich so, als ob es eigenständig wäre. Da die MAC-Adressen des Gasts dadurch nach außen sichtbar werden, muß für jede IP-Adressen über den Hetzner Robot eine virtuelle MAC beantragt und der Netzwerkkarte des Gasts zugewiesen werden.

``` 
# /etc/sysconfig/network-scripts/ifcfg-eth0
# device: eth0
DEVICE=eth0
BOOTPROTO=static
HWADDR=<MAC der physischen Netzwerkkarte>
ONBOOT=yes
BRIDGE=br0
```

```
# /etc/sysconfig/network-scripts/ifcfg-br0 (pointopoint, Hetzner Standard)
DEVICE=br0
TYPE="Bridge"
BOOTPROTO=static
IPADDR=<Haupt-IP>
NETMASK=255.255.255.255
SCOPE="peer <Gateway der Haupt-IP>"
ONBOOT=yes
DELAY=0
```

Bei der Hetzner-Standardinstallaton wird die Default-Route über die zusätzliche Konfigurationsdatei *route-eth0* gesetzt. Diese einfach in *route-br0* umbenennen.

**HINWEIS:** Bei dieser Konfiguration ist die Nutzung von IPv6 nur eingeschränkt möglich. Das IPv6 Netz kann über den Hetzner Robot entweder auf das Hostsystem oder auf EINE Zusatz-IP geroutet werden. (genauer: auf die IPv6-Link-Local-Adresse, die sich aus der MAC-Adresse ergibt)
