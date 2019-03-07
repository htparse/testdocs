##Synchronize Hostname, MTA Name and Reverse DNS Entry
Some mail servers reject emails if the sender does not have a matching hostname, MTA name (MTA = SMTP mail server) and/or reverse DNS entry. This description serves to describe this relationship.

##Preliminary Considerations
The **hostname** is simply the description of the server and assists the administrator with the management and recognition of their server. The hostname requires neither www, mail or smtp to start, nor does it have to have anything to do with the domains which are served by the server. The hostname should however be resolvable to the IP address by DNS.

The **MTA name/HELO hostname** is the name which the mail server forwards to external mail servers, e.g. with the SMTP protocol. This MTA name does not have to have anything to do with the domains for the emails it forwards either. However, as protection against spammers, many mail servers check the validity and consistency of MTA names (e.g. by reverse DNS queries), so as to guarantee a smooth exchange of email, even if the MTA name is reverse and forward resolvable.

The **Reverse DNS Entry** on the other hand is the "name" of an IP address. This entry helps traceroutes, for example, to recognize gateways and hosts - and often checks the given MTA name during email exchanges. Each IP address only has one name (even if a lot of different domains are hosted on it) and should also be forward resolvable via A record.

##Choice of Names
As described, these three names are tied to one another, the same name should be signed in all three cases. However, what should the name be?

A small server which only has to administer its own domain, homepage and email could be called "www.toms-little-wonderworld.de" or "server.toms-little-wonderworld.de".

But what if the server has to administer different domains?

The MTA name does not need to have anything to do with the domains it administers. Large providers with many thousands of domains can also only give each of their mail servers one MTA name each. As each IP address only has one reverse DNS entry, a more neutral name should appear as the last host during traceroutes.

Here the Hetzner predefined reverse DNS entry "xxx-xxx-xxx-xxx.clients.your-server.de" could be used as host and MTA name. Similarly, one of the domains could be used e.g."server3.super-duper-hoster.de". This looks more professional and does not run the risk of being rejected as an IP address from a dynamically assigned dialup address area.

##Define Entries
We select "server3.super-duper-hoster.de" as our name. Now the name needs to be entered into the different areas:

###Hostname
This name is defined in the zone file of the domain as the A record (the IP address is just an example):

`server3		IN	A	213.133.99.99`

###Reverse DNS Entry
The reverse DNS entry is individually set for each IP address via Hetzner Registration Robot:

Menu item "Server" -> click on the server -> "IPs" -> click on the text field at right next to the desired IP address

**Warning:** Reverse DNS entries have a TTL of 24 hours. Therefore changes to the entries are taken over immediately, where possible external DNS servers may still supply the old entries from their cache until expiry of TTL.

###MTA Name
Each mail server configures this in different places, in our example, the MTA name would be "server3.super-duper-hoster.de".

###MX Entries
Receipt of email should, of course, be possible. Therefore this entry is added in to the zone file of the domain "super-duper-hoster.de":

`@		IN	MX 10	server3`

This entry is added in to the other domains:

`@		IN	MX 10	server3.super-duper-hoster.de.`

As you can see, the MX entry does not need to be called mail... or smtp..., these entries are actually only useful for customers to configure their email programs. The entries in the email program do not need to match the MX entry in the domain either. So, the MX entry name can be the same as our example "server3.super-duper-hoster.de", but customers enter "mail.bigcompany.de" , for example, in their email programs. To define this, the following entries need to be set up:

###Web Server, FTP Server, Mail Callup and Dispatch by the Work Stations
In domain "super-duper-hoster.de" as well as in the other domains:

```
www			IN	A	213.133.99.99
ftp			IN	A	213.133.99.99
smtp			IN	A	213.133.99.99
pop3			IN	A	213.133.99.99
mail			IN	A	213.133.99.99
```

However, it is worth considering using CNAMEs instead of A Records. If the IP address changes, then only the A entry for "server3.super-duper-hoster.de" needs to be changed instead of manually changing all zone files. The entries would then look like this:

```
www			IN	CNAME	server3.super-duper-hoster.de.
ftp			IN	CNAME	server3.super-duper-hoster.de.
smtp			IN	CNAME	server3.super-duper-hoster.de.
pop3			IN	CNAME	server3.super-duper-hoster.de.
mail			IN	CNAME	server3.super-duper-hoster.de.
```

The web server software naturally also has to be informed that it should react to client domains via its configuration file.

##Conclusion
The complete zone file for super-duper-hoster.de could like this:

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
 server3			IN	A	213.133.99.99

 www			IN	CNAME	server3
 ftp			IN	CNAME	server3
 smtp			IN	CNAME	server3
 pop3			IN	CNAME	server3
 mail			IN	CNAME	server3

 @			IN	MX 10	server3
```
 
The zone file for a customer domain would look like this:

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

The reverse DNS entry in the Hetzner Registration Robot would then be called

`server3.super-duper-hoster.de`

##Testing Functions
The DNS report at www.dnsstuff.com has proven itself to be useful. Here you can very systematically look for errors in DNS configuration and progressively achieve error-free and compatible configuration.

##How do I Assign My IP Address Several Reverse DNS Entries?
This is not possible. If you take a look a the way DNS and Reverse DNS work, it is clear why this is the case:

**DNS Resolution (forward):**

`www.bigcompany.de		-->	213.133.99.99`

Several host names can refer "forward" to the same IP address, enabling a web server to be addressed using different domains:

```
www.bigcompany.de		-->	213.133.99.99
www.evenbiggercompany.de	-->	213.133.99.99
www.enormouscompany.de	 	-->	213.133.99.99
```

**Reverse DNS Resolution (backward):**

`213.133.99.99			-->	server3.super-duper-hoster.de`

Here one IP address always returns one hostname only, that which has been specified in the Hetzner Robot. A reverse DNS entry does not allow conclusions to be drawn about which domains are served by this server.

The PTR entries relevant for reverse DNS can only be created on the name servers for the IP address via the Hetzner Registration Robot. PTR entries created in own domains are not considered.
