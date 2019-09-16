##Hostname, MTA-Name und Reverse DNS Eintrag synchronisieren
Einige Mailserver lehnen den Empfang von Mails ab, wenn beim Absender Hostname, MTA-Name (MTA = SMTP Mailserver) und/oder der Reverse DNS Eintrag nicht übereinstimmen. Diese Beschreibung soll die Zusammenhänge erklären.

##Vorüberlegungen
Der **Hostname** ist lediglich die Bezeichnung des Servers, er soll dem Administrator helfen, seine Server zu verwalten bzw. wiederzuerkennen. Der Hostname braucht weder mit www, mail, oder smtp zu beginnen, noch muss er mit den Domains, die vom Server bedient werden, zu tun haben. Der Hostname sollte aber über DNS zur IP-Adresse auflösbar sein.

Der **MTA-Name/HELO-Hostname** ist die Bezeichnung, die der Mailserver z.B. beim SMTP-Protokoll an den fremden Mailserver übermittelt. Auch dieser MTA-Name muss mit den Domains der Mails, die er dann überträgt nichts zu tun haben. Doch um sich vor Spammern zu schützen, prüfen viele Mailserver die MTA-Namen auf Gültigkeit und Konsistenz (z.B. durch Reverse DNS Abfragen), sodass reibungsloser Mailaustausch nur gewährleistet ist, wenn auch der MTA-Name vorwärts und rückwärts auflösbar ist.

Der **Reverse DNS Eintrag** wiederum ist der "Name" einer IP-Adresse. Dieser Eintrag dient z.B. bei Traceroutes der Erkennung der Gateways und der Hosts - und eben oft beim Mailaustausch zur Überprüfung des angegebenen MTA-Namens. Jede IP-Adresse hat nur einen Namen (auch wenn über sie viele verschiedene Domains gehostet werden) und sollte auch über einen A-Record "vorwärts" auflösbar sein.

##Namenswahl
Diese drei Namen sind also wie beschrieben aneinander gebunden, es sollte in allen drei Fällen der gleiche Name vergeben werden. Doch wie soll der Name denn nun lauten?

Ein kleiner Server, der nur die eigene Domain, Homepage und Mail verwalten soll, kann z.B. "www.toms-kleine-wunderwelt.de" oder "server.toms-kleine-wunderwelt.de" lauten.

Doch was, wenn der Server verschiedene Domains verwalten soll?

Der MTA-Name muss nichts mit den Domains, die er verwaltet, zu tun haben. Große Provider mit vielen tausend Domains können jedem Ihrer Mailserver auch nur jeweils einen MTA-Namen geben. Und da ja jede IP-Adresse nur einen Reverse DNS Eintrag hat, sollte bei Traceroutes ein neutraler Name als letzter Host erscheinen.

Hier könnte man den von Hetzner vordefinierten Reverse DNS Eintrag "xxx-xxx-xxx-xxx.clients.your-server.de" als Host- und MTA-Name übernehmen. Genauso kann aber eine der Domain verwendet werden, z.B. "server3.super-duper-hoster.de". Dies sieht wesentlich professioneller aus und läuft auch nicht Gefahr, als IP-Adresse aus einem dynamisch vergebenen Einwahl-Adressbereich abgelehnt zu werden.

##Einträge definieren
Wir wählen "server3.super-duper-hoster.de" als unseren Namen. Jetzt muss der Name an den verschiedenen Stellen eingetragen werden:

###Hostname
Dieser Name wird in der Zonendatei der Domain als A-Record definiert (die IP-Adresse ist nur ein Beispiel):

`server3		IN	A	213.133.99.99`

###Reverse DNS Eintrag
Der Reverse DNS Eintrag wird über den Hetzner Registration Robot für jede IP-Adresse individuell festgelegt:

Menüpunkt "Server" -> Klick auf den Server -> "IPs" -> Klick auf das Textfeld rechts neben der gewünschten IP-Adresse

**Achtung:** Reverse DNS Einträge haben eine TTL von 24 Stunden. Daher werden Änderungen an den Einträgen zwar sofort übernommen, fremde DNS-Server liefern aber womöglich noch bis zum Ablauf der TTL die alten Einträge aus deren Cache aus.

###MTA-Name
Das wird bei jedem Mailserver an unterschiedlichen Stellen konfiguriert, in unserem Beispiel würde der MTA-Name aber "server3.super-duper-hoster.de" lauten.

###MX-Einträge
Mailempfang soll natürlich möglich sein. Dazu wird in der Zonendatei der Domain "super-duper-hoster.de" dieser Eintrag eingefügt:

`@		IN	MX 10	server3`

In den anderen Domains fügt man diesen Eintrag ein:

`@		IN	MX 10	server3.super-duper-hoster.de.`

Der MX-Eintrag muss wie man sieht nicht mail... oder smtp... lauten, diese Einträge sind eigentlich nur für die Konfiguration der Mailprogramme bei den Kunden sinnvoll. Die Einträge im Mailprogramm müssen mit dem MX-Eintrag in der Domain auch nicht übereinstimmen. So kann der MX-Eintrag wie im Beispiel "server3.super-duper-hoster.de" lauten, die Kunden tragen in Ihren Mailprogrammen aber beispielsweise "mail.grossefirma.de" ein. Um dies zu definieren, muss man noch die folgenden Einträge anlegen:

###Webserver, FTP-Server, Mailabruf und -versand von den Arbeitsplätzen
In der Domain "super-duper-hoster.de" sowie in den anderen Domains:

```
www			IN	A	213.133.99.99
ftp			IN	A	213.133.99.99
smtp			IN	A	213.133.99.99
pop3			IN	A	213.133.99.99
mail			IN	A	213.133.99.99
```

Überlegenswert wäre allerdings die Vewendung von CNAME's statt A-Records. Ändert sich die IP-Adresse, dann müssen nicht alle Zonendateien manuell geändert werden, sondern lediglich der A-Eintrag für "server3.super-duper-hoster.de". Die Einträge würden dann so aussehen:

```
www			IN	CNAME	server3.super-duper-hoster.de.
ftp			IN	CNAME	server3.super-duper-hoster.de.
smtp			IN	CNAME	server3.super-duper-hoster.de.
pop3			IN	CNAME	server3.super-duper-hoster.de.
mail			IN	CNAME	server3.super-duper-hoster.de.
```

Der Webserversoftware muss natürlich über deren Konfigurationsdatei auch noch mitgeteilt werden, dass sie auf die Kundendomains reagieren soll.

##Zusammenfassung
Die komplette Zonendatei für super-duper-hoster.de könnte so aussehen:

```
 @ IN SOA ns1.first-ns.de. postmaster.robot.first-ns.de. (
 	2000091604   ; Serial
 	14400        ; Refresh
 	1800         ; Retry
 	604800       ; Expire
 	86400  )     ; Minimum

 @			IN	NS	ns1.first-ns.de.
 @			IN	NS	robotns2.second-ns.de.

 localhost		IN	A	127.0.0.1
 @			IN	A	213.133.99.99
 server3		IN	A	213.133.99.99

 www			IN	CNAME	server3
 ftp			IN	CNAME	server3
 smtp			IN	CNAME	server3
 pop3			IN	CNAME	server3
 mail			IN	CNAME	server3

 @			IN	MX 10	server3
```
 
Die Zonendatei einer der Kundendomains würde so aussehen:

```
 @ IN SOA ns1.first-ns.de. postmaster.robot.first-ns.de. (
 	2000091604   ; Serial
 	14400        ; Refresh
 	1800         ; Retry
 	604800       ; Expire
 	86400  )     ; Minimum

 @			IN	NS	ns1.first-ns.de.
 @			IN	NS	robotns2.second-ns.de.

 localhost		IN	A	127.0.0.1

 www			IN	CNAME	server3.super-duper-hoster.de.
 ftp			IN	CNAME	server3.super-duper-hoster.de.
 smtp			IN	CNAME	server3.super-duper-hoster.de.
 pop3			IN	CNAME	server3.super-duper-hoster.de.
 mail			IN	CNAME	server3.super-duper-hoster.de.

 @			IN	MX 10	server3.super-duper-hoster.de.
```

Der Reverse DNS Eintrag im Hetzner Registration Robot müsste dann

`server3.super-duper-hoster.de`

lauten.

##Funktionstests
Bestens bewährt hat sich der DNS Report auf www.dnsstuff.com. Hier kann man sehr systematisch nach Fehlern in der DNS-Konfiguration suchen und nach und nach zu einer fehlerfreien und kompatiblen Konfiguration gelangen.

##Wie weise ich meinen IP-Adressen mehrere Reverse DNS Einträge zu?
Das funktioniert nicht. Wenn man sich die Funktionsweise von DNS und Reverse DNS ansieht, wird auch klar warum:

**DNS Auflösung (vorwärts):**

`www.grossefirma.de		-->	213.133.99.99`

Auch mehrere Hostnamen können "vorwärts" auf die gleiche IP-Adresse verweisen, dadurch kann ein Webserver unter verschiedenen Domains angesprochen werden:

```
www.grossefirma.de		-->	213.133.99.99
www.nocheinegrossefirma.de	-->	213.133.99.99
www.riesengrossefirma.de	-->	213.133.99.99
```

**Reverse DNS Auflösung (rückwärts):**

`213.133.99.99			-->	server3.super-duper-hoster.de`

Hier liefert eine IP-Adresse immer nur einen Hostnamen zurück, und zwar den, der über den Robot von Hetzner definiert wurde. Rückschlüsse darüber, welche Domains dieser Server ausliefert, lässt ein Reverse DNS Eintrag nicht zu.

Die für Reverse DNS relevanten PTR-Einträge kann man nur über den Hetzner Registration Robot an den für die IP-Adressen zuständigen Nameservern anlegen, in der eigenen Domain angelegte PTR-Einträge werden nicht berücksichtigt.
