##Haupt-IP-Adresse
###IPv4
####Root-Server
Die Hauptadresse eines Root-Servers liegt in der Regel in einem /26 oder /27-Netz. Um die (versehentliche) Übernahme fremder IP-Adresse zu verhindern, ist ausschließlich die Kommunikation mit der Gateway-Adresse möglich. Um auch Server im gleichen Netzsegment ansprechen zu können, wird in der Standardinstallation eine zusätzliche Route eingerichtet, die Pakete an das eigene Subnetz über das Gateway leitet.

Eine alternative Möglichkeit diese Konfiguration zu erreichen, besteht darin, eine Punkt-zu-Punkt-Verbindung zwischen Haupt-Adresse und Gateway zu konfigurieren. Dazu wird als Netzmaske `255.255.255.255 (/32)` genutzt; der Server geht so davon aus, sich alleine in seinem Ethernet-Segment zu befinden und stellt keine Pakete direkt zu. Damit das das Gateway erreicht werden kann, wird eine Host-Route benötigt. Dies wird bei Debian mit der Option `pointopoint <Gateway-IP>` in der Konfiguration realisiert.

```
## /etc/network/interfaces Beispiel Hetzner Rootserver
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# LAN-Schnittstelle
auto eth0
iface eth0 inet static
  # Haupt-IP-Adresse des Servers
  address 192.168.0.250
  # Netzmaske 255.255.255.255 (/32) unabhängig von der
  # realen Netzaufteilung (z.B. /27)
  netmask 255.255.255.255
  # Explizite Hostroute zum Gateway
  gateway 192.168.0.1
  pointopoint 192.168.0.1
```

Die in der Standardinstallation gesetzte zusätzliche Route ist dann nicht mehr nötig.

####vServer (CX-Modellreihe)
Da bei CX vServern die öffentliche IP per 1:1 NAT auf die interne IP umgesetzt wird. Erfolgt die Konfiguration in der Standardinstallation per DHCP. Eine statische Konfiguration ist möglich, aber nicht zu empfehlen, da eventuell zukünftigen Features eine Anpassung erfordern können.

###IPv6
Root-Server / CX-vServer
Im Prinzip gilt für IPv6 das gleiche wie im IPv4 Abschnitt erwähnt. Statt der einzelnen Haupt-IP bekommt man einen /64 Block.

Im Gegensatz zur IPv4 Konfiguration gibt es für IPv6 keinen "pointopoint" Eintrag.

Beispiel:

* Adressblock: `2a01:4f8:61:20e1::1` bis `2a01:4f8:61:20e1:ffff:ffff:ffff:ffff`
* Davon verwenden wir die erste Adresse `2a01:4f8:61:20e1::2`
* Gateway: `fe80::1`

```
## /etc/network/interfaces Beispiel Hetzner Rootserver
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# IPv6 LAN
auto eth0
iface eth0 inet6 static
  # Haupt-IPv6-Adresse des Servers
  address 2a01:4f8:61:20e1::2
  netmask 64
  gateway fe80::1
```

###IPv4 + IPv6
Es ist zu erwarten, daß IPv4 und IPv6 die kommenden Jahre parallel verwendet werden. Dazu werden einfach die beiden Konfigurationsdateien aneinandergefügt und die doppelten Einträge weggelassen.

####Root-Server / CX-vServer
```
## /etc/network/interfaces Beispiel Hetzner Rootserver
# Loopback-Adapter
auto lo
iface lo inet loopback
#
# LAN-Schnittstelle
auto eth0
iface eth0 inet static
  # Haupt-IP-Adresse des Servers
  address 192.168.0.250
  # Netzmaske 255.255.255.255 (/32) unabhängig von der
  # realen Netzaufteilung (z.B. /27)
  netmask 255.255.255.255
  # Explizite Hostroute zum Gateway
  gateway 192.168.0.1
  pointopoint 192.168.0.1
#
iface eth0 inet6 static
  # Haupt-IPv6-Adresse des Servers
  address 2a01:4f8:61:20e1::2
  netmask 64
  gateway fe80::1
```

##Zusätzliche IP-Adressen (Host)
Alle aktuellen Root-Server-Modelle erhalten auf Antrag kostenpflichtig maximal 6 zusätzliche einzelne IP-Adressen. Die Konfiguration erfolgt jedoch auf die gleiche Weise:

Um die zusätzlichen Adressen auf dem Server (keine Virtualisierung) zu nutzen, wird das Paket "iproute" mit dem Dienstprogramm "ip" benötigt. Konfigurationen mit Alias-Schnittstellen (eth0:1, eth0:2 etc.) sind veraltet und sollten keine Verwendung mehr finden. Um eine Adresse hinzuzufügen, genügt das folgende Kommando:

`ip addr add 10.4.2.1/32 dev eth0`

Der Befehl `ip addr` zeigt die momentan aktiven IP-Adressen an. Da das Subnetz dem Server exklusiv zur Verfügung steht, ist es auch hier sinnvoll, die Adressen mit der Präfixlänge `/32`, also der Subnetzmaske `255.255.255.255` hinzuzufügen.

###Konfiguration
In der **/etc/network/interfaces** werden unter dem entsprechenden Interface (hier "eth0") die folgenden beiden Zeilen eingefügt:

```
up ip addr add 10.4.2.1/32 dev eth0
down ip addr del 10.4.2.1/32 dev eth0
```

"up" und "down" erwarten einfach eine Zeile Shell-Code und könnnen für mehrere Adressen wiederholt vorkommen. Der Nachteil: sowohl Schnittstellenname als auch die einzustellende Adresse müssen jeweils zwei mal aufgeführt werden, bei einer größeren Anzahl Adressen wird die Konfiguration daher unübersichtlich und fehleranfällig; ändern sich die Daten, müssen alle Einträge angepasst werden.

###Alternative Konfiguration via addresses-Skript
**Hinweis:** Bei der nachfolgenden Anleitung wird Software von einem Drittanbieter (<http://www.wertarbyte.de>) installiert. Diese wird nicht von Hetzner Online unterstützt. Bei Fehlern oder Problemen wenden Sie sich bitte an den [Entwickler](http://www.wertarbyte.de/kontakt.shtml).

Das Skript befindet sich im Paket "ifupdown-scripts-wa", das jedoch nicht Teil der offiziellen Debian-Distribution ist; fügt man folgende Zeile zur APT-Konfiguration hinzu, reicht der Befehl `apt-get install ifupdown-scripts-wa` um das Skript korrekt zu installieren:

```
# /etc/apt/sources.list.d/wertarbyte.list
# Tartarus, ifupdown-scripts etc.
deb http://wertarbyte.de/apt/ ./
```

Die gesamte Installationsroutine lässt sich mit den folgenden Befehlen abkürzen:

```
wget -P/etc/apt/sources.list.d/ http://wertarbyte.de/apt/wertarbyte-apt.list
wget -O - http://wertarbyte.de/apt/software-key.gpg | apt-key add -
apt-get update
apt-get install ifupdown-scripts-wa
```

Wer das Skript nicht über das Paketsystem installieren möchte, kann es auch manuell herunterladen: <http://wertarbyte.de/debian/ifupdown/addresses>. Es wird im Verzeichnis **/etc/network/if-up.d/** abgelegt und zusätzlich nach **/etc/network/if-down.d/** verlinkt:

```
cd /etc/network/if-up.d/ && \
wget http://wertarbyte.de/debian/ifupdown/addresses && \
chmod +x addresses && \
cd ../if-down.d/ && \
ln -s ../if-up.d/addresses .
```

Die Installation über das Paketsystem wird jedoch empfohlen, da so stets die aktuelle Version des Skripts verfügbar ist.

Das Skript erweitert die Syntax der Konfigurationsdatei um eine neue Anweisung namens "addresses", mit der zusätzliche zu bindende IP-Adressen (mit der Netzmaske in /-Notation) angegeben werden können:

`addresses 10.4.2.1/32 10.4.2.2/32 10.4.2.3/32`

Fügt man diese Zeile zur Konfiguration der Schnittstelle "eth0" hinzu, so werden die Adressen beim Aktivieren der Schnittstelle hinzugefügt und bei deren Deaktivierung wieder entfernt.

Zusätzlich ist es möglich, mehrere Zeilen zu verwenden, um Adressen in Kategorien zu bündeln und die Konfiguration übersichtlicher zu gestalten:

```
addresses       10.4.2.1/32
addresses-https 10.4.2.2/32 10.4.2.3/32 # SSL-Websites
addresses-mail  10.4.2.4/32             # Mailserver
```

Das Skript erfasst sämtliche Anweisungen, die mit dem Schlüsselwort "addresses-" und einer frei wählbaren Bezeichnung beginnen. Eine Bezeichnung darf nicht doppelt verwendet werden, da ansonsten ifupdown einen Syntaxfehler anzeigt und die Konfiguration der Schnittstelle abbricht - unter Umständen ist der Server dadurch nicht mehr erreichbar.

Die via "ip addr" hinzugefügten IP-Adressen sind in der Ausgabe von "ifconfig" nicht sichtbar; um sie anzuzeigen, wird der Befehl "ip addr show" benötigt. Das addresses-Skript kann jedoch auch Alias-Geräte anlegen:

```
addresses 10.0.0.1/32 10.0.0.2/32 10.0.0.3/32
create_alias_devices yes
```

Mit dieser Konfiguration legt das Skript durchnumerierte eth0:X-Geräte an, die auch in "ifconfig" sichtbar sind.

Anstatt die Geräte nur zu numerieren kann man jedoch auch die Beschreibungen aus der Konfiguration verwenden:

```
addresses-https 10.0.0.1/32 10.0.0.3/32
addresses-vhost 10.0.0.2/32
label_addresses yes
```

Die Adressen werden daraufhin in der Ausgabe von "ip addr" mit den Beschriftungen "eth0:https" bzw. "eth0:vhost" versehen, die auch von "ifconfig" angezeigt werden.

##Zusätzliche IP-Adressen (Virtualisierung)
Bei Einsatz von Virtualisierung werden die zusätzliche IP-Adressen durch die Gast-Systeme genutzt. Damit diese im Internet erreichbar sind, muß im Hostsystem eine entsprechende Konfiguration entsprechend angepasst werden, um die Pakete weiterzuleiten. Dabei gibt es für zusätzliche Einzel-IPs zwei Möglichkeiten: Routed und Bridged.

###Routed (brouter)
Bei einer Routed-Konfiguration werden die Pakete geroutet. Dafür muß eine zusätzliche Bridge mit nahezu gleicher Konfiguration (ohne Gateway) wie eth0 angelegt werden.

```
auto eth0
iface eth0 inet static
   address (Haupt-IP)
   netmask 255.255.255.255
   pointopoint (Gateway-IP)
   gateway (Gateway-IP)
#
iface eth0 inet6 static
  address 2a01:4f8:XX:YY::2
  netmask 128
  gateway fe80::1
#
auto virbr1
iface virbr1 inet static
   address (Haupt-IP)
   netmask 255.255.255.255
   bridge_ports none
   bridge_stp off
   bridge_fd 0
   pre-up brctl addbr virbr1
   up ip route add (Zusatz-IP)/32 dev virbr1
   down ip route del (Zusatz-IP)/32 dev virbr1
 #
 iface virbr1 inet6 static
   address 2a01:4f8:XX:YY::2
   netmask 64
```

Für jede Zusatz-IP muß eine entsprechende Host-Route angelegt werden. Die Konfiguration von eth0 bleibt unverändert.

###Bridged
Bei einer Bridge-Konfiguration werden die Pakete direkt zugestellt. Das Gast-System verhält sich so, als ob es eigenständig wäre. Da die MAC-Adressen des Gasts dadurch nach außen sichtbar werden, muß für jede IP-Adressen über den Hetzner Robot eine virtuelle MAC beantragt und der Netzwerkkarte des Gasts zugewiesen werden. Die Bridge erhält 1:1 die Netz-Konfiguration von eth0

```
auto  br0
iface br0 inet static
 address (Haupt-IP)
 netmask (wie eth0, z.B: 255.255.255.254)
 gateway (wie bei Haupt-IP)
 bridge_ports eth0
 bridge_stp off
 bridge_fd 1
 bridge_hello 2
 bridge_maxage 12
```

Die Konfiguration von eth0 entfällt ersatzlos.
