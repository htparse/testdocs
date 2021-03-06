---
path: "/community/share/ssh_de"
date: "2019-02-15"
title: "SSH"
short_description: "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua."
tags: ["Test", "Hetzner Offical"]
views: "100"
author: "Hetzner Online"
author_img: "https://avatars3.githubusercontent.com/u/30047064?s=200&v=4"
author_description: "At vero eos et accusam <a class='link-single-internal' href='#'>Link</a> et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur  <a class='link-single-external' href='#'>Github</a>  sadipscing elitr, sed diam nonumy eirmod tempor"
header_img: "https://community-cdn-digitalocean-com.global.ssl.fastly.net/assets/tutorials/images/large/Database-Mostov_v4.1_twitter-_-facebook.png?1550071669"     
---

## Was ist SSH?

SSH ist, einfach ausgedrückt, ein Programm, das Ihnen erlaubt ein Terminal auf einem anderen Rechner als dem Ihren zu öffnen. Wenn Sie unter Windows arbeiten, ist es so ähnlich, als ob Sie ein DOS Fenster auf Ihrem Rechner öffnen, obwohl Sie das Linux Betriebssystem benutzen, sobald Sie sich auf dem Server befinden. Dabei wird die gesammte Kommunikation verschlüsselt.

## Wie erhalte ich einen SSH Server?

In den  [Standardimages](https://wiki.hetzner.de/index.php/Standardimages "Standardimages")  ist bereits ein SSH Server enthalten und aktiv. Sie müssen hier nichts weiter tun. Falls Sie das Betriebssystem selbst installiert haben, installieren Sie das openssh Paket nach (Eventuell unter anderem Namen wie openssh-server). Falls Sie eine Firewall betreiben, müssen Sie den Port 22 (ssh) freigeben.

## Wie erhalte ich einen SSH Client?

Da Windows keinen integrierten SSH Client mitliefert, empfehlen wir Ihnen putty, welches Sie bei  [greenend.org.uk](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)downloaden können.

Laden Sie sich putty.exe herunter und speichern Sie das Programm beispielsweise auf Ihrem Desktop. Öffnen Sie nun das Programm und geben Sie den Servernamen ein. Wählen Sie nun "Open" aus. Beim ersten Start erscheint eine Warnmeldung welche Sie mit "Yes" quittieren können. Danach geben Sie Ihren Benutzernamen sowie Passwort ein und können auf dem Server arbeiten.

Unter Linux / Unix ist SSH in der Regel vorhanden bzw. kann auf unkompliziertem Weg mittels der Paket-Verwaltung Ihrer Distribution installiert werden.

## Wie verwende ich meinen SSH Client, um eine Verbindung zum Server herzustellen?

Wenn Sie ein Putty verwenden, geben Sie Ihren DNS-Servernamen oder dessen IP-Adresse ein. Wählen Sie (falls notwendig) SSH unnd Port 22 aus und klicken Sie auf Verbinden. Danach werden Sie zur Eingabe von Benutzername (in der Regel 'root') und Passwort aufgefordert. Wenn Sie diese korrekt eingegeben haben, werden Sie ins System eingeloggt.

Wenn Sie ein Kommandozeilen-System (wie Linux oder DOS) benutzen, müssen Sie einen Befehl eingeben, wie:

`ssh <Ihre IP-Adresse>`


## Wie stelle ich sicher, daß ich mich mit dem richtigen Server verbinde?

Beim erstmaligen Verbinden mit einem Server erscheint eine Anzeige, die den Benutzer auffordert den sogenannten Fingerprint (Fingerabdruck) des Servers zu prüfen und zu bestätigen. Der Fingerprint ist ein verkürzte Darstellung des öffentlichen Schlüssels des Servers.

```
The authenticity of host 'example.org (192.0.2.42)' can't be established.
RSA key fingerprint is SHA256:DlxqI8BctJqAgyCfyExywbm9a7qdL7nqfMKgoQuGp5w..
Are you sure you want to continue connecting (yes/no)?
```

Je nachdem welcher Schlüssel für die Verbindung verwendet wird, ändert sich die Ausgabe. Neben RSA sind DSA, ECDSA und ED25519 gängige Schlüsseltypen, wobei DSA nicht mehr verwendet werden sollte und ab OpenSSH 7 standardmäßig nicht mehr verwendet wird.

Bei der automatischen Installation via Hetzner  [Robot](https://wiki.hetzner.de/index.php/Robot "Robot")  werden die Fingerprints angezeigt und zusätzlich per E-Mail übermittelt. Auch bei der Aktivierung des  [Rescue-Systems](https://wiki.hetzner.de/index.php/Hetzner_Rescue-System "Hetzner Rescue-System")  werden dessen Fingerprints im Hetzner  [Robot](https://wiki.hetzner.de/index.php/Robot "Robot")angezeigt.

Sollte beim erneuten Verbinden folgende Warnung erscheinen, sollte man diese Ernst nehmen:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:DlxqI8BctJqAgyCfyExywbm9a7qdL7nqfMKgoQuGp5w.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in /home/user/.ssh/known_hosts:1
Password authentication is disabled to avoid man-in-the-middle attacks.
Keyboard-interactive authentication is disabled to avoid man-in-the-middle
attacks.
Permission denied (publickey,password). 
```

Hier wird vor einem möglichen Man-in-the-Middle Angriff gewarnt. Falls der Server sich nicht z.b. im Rescue System befindet, der Key selbst geändert wurde oder es sonst einen bekannten Grund gibt, warum sich der Key geändert hat, sollte man die Verbindung unterbrechen und den Zustand des Server auf anderem Weg (z.b. via KVM-Konsole) überprüfen.

## Wie erzeuge ich einen neuen SSH Host-Key?

Bei der automatischen Installation via Hetzner  [Robot](https://wiki.hetzner.de/index.php/Robot "Robot")  bzw. via  [installimage](https://wiki.hetzner.de/index.php/Installimage "Installimage")  im Rescue-System werden in der Regel alle Host-Schlüssel neu generiert. Um in einem installierten System einen Schlüssel auszutauschen, wird "ssh-keygen" verwendet. Eine Liste aller vorhanden Keys (ssh_host*) findet man unter  _/etc/ssh/_

```
# ls -l /etc/ssh
total 280
-rw-r--r-- 1 root root 242091 Oct  3  2014 moduli
-rw-r--r-- 1 root root   1689 Oct 17  2014 ssh_config
-rw-r--r-- 1 root root   2530 Dec 30 10:51 sshd_config
-rw------- 1 root root    668 Dec 30 10:44 ssh_host_dsa_key
-rw-r--r-- 1 root root    622 Dec 30 10:44 ssh_host_dsa_key.pub
-rw------- 1 root root    227 Dec 30 10:44 ssh_host_ecdsa_key
-rw-r--r-- 1 root root    194 Dec 30 10:44 ssh_host_ecdsa_key.pub
-rw------- 1 root root    432 Dec 30 10:44 ssh_host_ed25519_key
-rw-r--r-- 1 root root    114 Dec 30 10:44 ssh_host_ed25519_key.pub
-rw------- 1 root root   1675 Dec 30 10:44 ssh_host_rsa_key
-rw-r--r-- 1 root root    414 Dec 30 10:44 ssh_host_rsa_key.pub
```

Um z.b. den ED25519 Key zu erneuern lautet das Kommando:

``` 
# ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N 
Generating public/private ed25519 key pair.
/etc/ssh/ssh_host_ed25519_key already exists.
Overwrite (y/n)? y
Your identification has been saved in /etc/ssh/ssh_host_ed25519_key.
Your public key has been saved in /etc/ssh/ssh_host_ed25519_key.pub.
The key fingerprint is:
d5:1d:28:01:f7:c5:0f:fb:7b:43:07:08:1f:93:1c:c6 root@host
The key's randomart image is:
+--[ED25519 256]--+
|        ..o+o=o  |
|         .o+Eoo. |
|          .+o+.+ |
|         .  o o .|
|        S      o |
|               .o|
|              . o|
|               o.|
|                o|
+-----------------+
```
