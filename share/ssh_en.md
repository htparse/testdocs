---
path: "/community/share/ssh_en"
date: "2019-02-05"
title: "SSH"
short_description: "Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua."
tags: ["Test", "Hetzner Offical"]
views: "100"
author: "Hetzner Online"
author_img: ""
author_description: "At vero eos et accusam <a class='link-single-internal' href='#'>Link</a> et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur  <a class='link-single-external' href='#'>Github</a>  sadipscing elitr, sed diam nonumy eirmod tempor"
header_img: ""     
---

## What is SSH?

SSH is, simply put, a program that allows you to open a terminal on a remote computer. If you use Windows, it is similar to when you open a DOS window on your computer, even though you use a Linux operating system when you are working on the server. SSH encrypts the entire communication.

## How do I get an SSH server?

An SSH server is already included and active on our  [standard images](https://wiki.hetzner.de/index.php/Standardimages/en "Standardimages/en"). You don't have to do anything else. If you installed the operating system yourself, you'll need to install the openssh packet (it might be under a similar name, like openssh-server). If you use a firewall, you need to open port 22 (ssh).

## How do I get an SSH client?

Since Windows does not come with an SSH client, we recommend downloading PuTTY from  [greenend.org.uk](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Download putty.exe and save the program somewhere easily accessible, like your desktop. Then open the program and enter your server name. Then select “open”. When the program starts for the first time, there will be a warning which you can acknowledge by selecting “yes”. Then enter your user name and password and you can work on the server.

With Linux/Unix SSH is generally already provided or can be easily installed by using your distribution's package manager.

## How do I use my SSH client to connect to a server?

If you are using PuTTY, enter your DNS server name or its IP address. If necessary, select SSH and Port 22 and click on “connect”. Then you will be asked to enter your user name (usually “root”) and password. Once you enter these correctly, you will be logged in to the system.

If you use a command-line system (such as Linux or DOS), you need to enter a command like:

`ssh <your IP address>`

(Please replace <your IP address> with the IP address of your server)

## How do I ensure that I am connecting to the right server?

The first time you connect to a server a message prompts you to examine the so-called fingerprint of the server and to confirm it. The fingerprint is a condensed version of the public key of the server.

```
The authenticity of host 'example.org (192.0.2.42)' can't be established.
RSA key fingerprint is SHA256:DlxqI8BctJqAgyCfyExywbm9a7qdL7nqfMKgoQuGp5w..
Are you sure you want to continue connecting (yes/no)?
```
Depending on which key is used for the connection, the output will look different. In addition to RSA, DSA, ECDSA and ED25519 are all common types of keys, though DSA should no longer be used and by default is no longer used as of OpenSSH 7.

With the automatic installation via the  [Robot](https://wiki.hetzner.de/index.php/Robot/en "Robot/en")  the fingerprints are displayed and transmitted additionally by email. When activating the  [Rescue system](https://wiki.hetzner.de/index.php/Hetzner_Rescue-System/en "Hetzner Rescue-System/en")  these fingerprints are also displayed in the  [Robot](https://wiki.hetzner.de/index.php/Robot/en "Robot/en").

If the following warning appears when reconnecting, it 
should be taken seriously:

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
Here a possible man-in-the-middle attacks is being warned of. If the server is not in the Rescue System, the key itself has not been changed, or there is no other reason why the key could have changed, then the connection should be interrupted and a  [KVM Console](https://wiki.hetzner.de/index.php/KVM-Console/en "KVM-Console/en")  remote console ordered to check the condition of the server.

## How do I create a new SSH host key?

With an automatic installation via the  [Robot](https://wiki.hetzner.de/index.php/Robot/en "Robot/en")  or via the  [installimage](https://wiki.hetzner.de/index.php/Installimage/en "Installimage/en")  in the Rescue System all host keys are automatically regenerated. To replace a key in an installed system, "ssh-keygen" is used. A list of all available Keys (ssh_host*) can be found under  **/etc/ssh/**

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
For example, to renew the ED25519 key type the following command:

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
