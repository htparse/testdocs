---
path: "/community/share/3ware-hardware-RAID-controller_de"
date: "2019-02-15"
title: "3ware hardware RAID controller_de"
short_description: "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua."
tags: ["Test", "Hetzner Offical"]
views: "100"
author: "Hetzner Online"
author_img: ""
author_description: "At vero eos et accusam <a class='link-single-internal' href='#'>Link</a> et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur  <a class='link-single-external' href='#'>Github</a>  sadipscing elitr, sed diam nonumy eirmod tempor"
header_img: ""     
---

##3ware Hardware-RAID-Controller
Das von 3ware entwickelte Programm zur Administration des RAID-Controllers finden Sie in unserem Download-Bereich unter:

`http://download.hetzner.de/tools/3ware/`

* Die Zugangsdaten zu diesem Bereich haben Sie bereits mit der Fertigstellungsmail Ihres Root Servers erhalten.
* Bitte achten Sie beim Download des Programms darauf, dass Sie die zu Ihrem Betriebssystem passende Architektur wählen.
* Für ein 32-Bit Linux Betriebssystem laden Sie bitte folgendes Archiv herunter:

`tw_cli-linux-x86-9.5.0.tgz`

* Für ein 64-Bit Linux Betriebssystem wählen Sie das Archiv:

`tw_cli-linux-x86_64-9.5.0.tgz`

* Nach dem Download entpacken Sie das Archiv in ein Verzeichnis Ihrer Wahl.

Nähere Informationen zur Anwendung des Programmes finden Sie ebenfalls in unserem Download-Bereich unter:

`http://download.hetzner.de/tools/3ware/tools/CLI/tw_cli.8.html`

##Wie kann man den Status eines Hardware-RAID auslesen?
Um den Status der 3ware RAID-Controller auszulesen muss das Commandline-Tool tw_cli von 3ware installiert sein. Im Rescue-System ist dies bereits der Fall.

Die Nummer des RAID-Controllers erhält man mittels tw_cli show. Entsprechend ist c0 in den Beispielen durch diese Angabe zu ersetzen.

Den Zustand des RAIDs erhält man mit folgenden Befehl:

`tw_cli /c0 show`

Ausgabe Beispiel RAID-1:

```
Unit  UnitType  Status         %RCmpl  %V/I/M  Stripe  Size(GB)  Cache  AVrfy
 ------------------------------------------------------------------------------
 u0    RAID-1    OK             -       -       -       698.637   ON     -

 Port   Status           Unit   Size        Blocks        Serial
 ---------------------------------------------------------------
 p0     OK               u0     698.63 GB   1465149168    S13UJ1CQ704597
 p1     OK               u0     698.63 GB   1465149168    S13UJ1BQ708871
```
 
##Wie kann man ein Hardware-RAID anlegen?
Egal ob man den Modus eines bestehenden RAIDs wechseln möchte oder ein neues RAID (nach Einbau eines Controllers) anlegen möchte muss man als erstes die beiden Festplatten aus Ihren "Units" löschen:

`tw_cli maint deleteunit c0 u0`

`tw_cli maint deleteunit c0 u1`

RAID-0 erstellen:

`tw_cli maint createunit c0 rraid0 p0:1`

RAID-1 erstellen (empfohlene Konfiguration bei zwei Festplatten):

`tw_cli maint createunit c0 rraid1 p0:1`

RAID-5 erstellen (mit drei Festplatten):

`tw_cli maint createunit c0 rraid5 p0:1:2`

RAID-5 erstellen (mit vier Festplatten und Startvolume von 200GB):

`root@rescue ~ # tw_cli`

`//rescue> /c0 add type=raid5 disk=0:1:2:3 v0=200`

##Können die Festplatten als JBOD genutzt werden?
Der 3ware 9650SE Controller kann die Festplatten auch direkt als JBOD durchreichen. LSI/3ware empfiehlt jedoch die Festplatten stattdesssen als Single-Disk zu konfigurieren.

```
tw_cli /c0 show exportjbod
/c0 JBOD Export Policy = off
```
```
tw_cli /c0 set exportjbod=on
Enabling JBOD Export Policy on /c0... Done.
```

##Wie startet man mit tw_cli einen REBUILD bei einem RAID das DEGRADED ist?
Zuerst prüft man den Status des RAID Controllers (diese weiter oben). Dort sieht man dann in etwa folgendes:

Ausgabe Beispiel RAID-1:

```
Unit  UnitType  Status         %RCmpl  %V/I/M  Stripe  Size(GB)  Cache  AVrfy
 ------------------------------------------------------------------------------
 u0    RAID-1    DEGRADED       -       -       -       698.637   ON     -

 Port   Status           Unit   Size        Blocks        Serial
 ---------------------------------------------------------------
 p0     DEGRADED         u0     698.63 GB   1465149168    S13UJ1KS210609
 p1     OK               u0     698.63 GB   1465149168    S13UJ1NQ600102
```
 
Man muss nun zuerst die DEGRADED Festplatte aus dem Array löschen und den Controller neu einscannen:

`tw_cli maint remove c0 p0`

`tw_cli maint rescan c0`

Jetzt wird der Rebuild des Arrays gestartet:

`tw_cli maint rebuild c0 u0 p0`

Sobald dies fertig ist sollte der Rebuild anlaufen, man kann den Status wieder auslesen:

`tw_cli /c0 show rebuild`

Ausgabe nach gestartetem Rebuild:

```
Unit  UnitType  Status         %RCmpl  %V/I/M  Stripe  Size(GB)  Cache  AVrfy
 ------------------------------------------------------------------------------
 u0    RAID-1    REBUILDING     5       -       -       698.637   ON     -

 Port   Status           Unit   Size        Blocks        Serial
 ---------------------------------------------------------------
 p0     DEGRADED         u0     698.63 GB   1465149168    S13UJ1KS210609
 p1     OK               u0     698.63 GB   1465149168    S13UJ1NQ600102
```

Falls der Rebuild aufgrund eines ECC Errors unterbrochen wird, kann man den Rebuild erzwingen. Dies ist aber nicht zu empfehlen:

`tw_cli /c0/u0 start rebuild ignoreECC`
