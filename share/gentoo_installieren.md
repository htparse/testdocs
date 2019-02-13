##Gentoo Installation
Diese Anleitung basiert sich teilweise auf der Gentoo Handbuch: <http://www.gentoo.org/doc/de/handbook/>

##Rescue System booten
Um Gentoo auf einem dedizierten oder virtuellen Server zu installieren, muss man den Server im Rescue-System starten. Dies wird über den Robot gemacht.

##Platte vorbereiten
Nachdem der Server erfolgreich in das Rescue-System hochgefahren ist, muss die Platte partitioniert werden.
Sind die Partitionen erstellt und formatiert, können sie nach /mnt/ gemountet werden, wie in der Gentoo Handbuch beschrieben ist (Kapitel 4).

##Gentoo installieren
Ab diesem Punkt kann mit der Gentoo-Installation entsprechend der Handbuch verfahren werden.

Tipp: Die Netzwerkkarten-Konfiguration kann man bei Hetzner gerne über DHCP regeln lassen. Dazu in der Handbuch einfach dem Zweig für DHCP folgen und auch den DHCPCD installieren.

**Wichtiger Tipp:** damit man sich nach dem Reboot des Systems auch einloggen kann: Es muß unbedingt **VOR** dem ersten Neustart sichergestellt werden, dass sshd beim Booten mit hochfährt, damit man sich nicht gleich am Anfang elegant "aussperrt". Dazu fügt man sshd zum Default-Runlevel hinzu:

`rc-update add sshd default`

`nano -w /etc/ssh/sshd_config (Konfiguration einstellen)`

Da nach dem Reboot zunächst noch keine User angelegt sind, sollte man vorher Root-Login in der sshd_config erlauben. ;)

Tipp: Um zu sehen, ob SSH funktioniert, kann man den Port verändern (auf 22 arbeitet das Rescue-System), mit */etc/init.d/sshd* start den Server starten und sich testweise einloggen (mit dem neuen Root-Passwort).

Nach dem Reboot hat man dann ein jungfräuliches Gentoo-System auf der Maschine.

##Gentoo mit Paketen versorgen und updaten
Hetzner betreibt einen internen Mirror-Server, der den Portage Tree und die Distfiles, aber keine Stage3 Tarballs bereitstellt. Diese müssen von einem der öffentlichen Mirror bezogen werden.

**Distributionsquellen (inklusive der Stage3 Tarballs)**

* Liste der Mirror: <http://www.gentoo.org/main/de/mirrors2.xml>

Die Tarballs befinden sich unter releases/x86/autobuilds/current-stage3/ (statt x86 für 32bit, entsprechend amd64 für 64bit)

**Portage Tree**

Um den Mirror sowohl für die Distfiles als auch für den Portage-Tree zu verwenden, kann dieser über das mirrorselect Tool ausgewählt werden. Mehr Information dazu finden sich im Gentoo Handbuch: 
<http://www.gentoo.org/doc/de/handbook/handbook-x86.xml?part=1&chap=6#doc_chap1>

Alternativ kann dieser händisch in der /etc/make.conf angegeben werden:

```
/etc/make.conf:
GENTOO_MIRRORS="ftp://mirror.hetzner.de/gentoo/"
SYNC="rsync://mirror.hetzner.de/gentoo/portage"
```
