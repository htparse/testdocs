---
path: "/community/share/failover-script"
date: "2019-02-05"
title: "Failover Script"
short_description: "With this script it is possible to move the failover IP."
tags: ["Test", "Hetzner Offical"]
views: "100"
author: "Hetzner Online"
author_img: ""
author_description: "At vero eos et accusam <a class='link-single-internal' href='#'>Link</a> et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur  <a class='link-single-external' href='#'>Github</a>  sadipscing elitr, sed diam nonumy eirmod tempor"
header_img: ""     
---

With this script it is possible to move the failover IP.

##Usage
`./script database get`

The current status gets shown.

`./script database set 188.10.11.113`

The failover IP gets routed to the IP 188.10.11.113.

##Example database server
/etc/failover/resources/database

```
resource database
ip 188.10.10.12
failover_ip 188.10.11.112 188.10.11.113
user MEINUSER
password MEINPASSWORT
script /etc/failover/script/startmysql
```

##The script

```
#!/bin/bash
#
# fail.sh v0.2
#
# (c) 2010 Digiconcept GmbH <www.digiconcept.net>
#
# This piece of scripting is free and may be redistributed and/or modified unter the terms of the GNU Gerneral Public License version 3
# This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
# <http://www.gnu.org/licenses/>.
# 
# Please report bugs to j.hauptmann@digiconcept.net
#

para=$1
parb=$2
parc=$3
pard=$4

# is curl installed?
curl=`whereis curl|awk '{ print $2 }'`
if [ $? -ne 0 ] ; then echo "Curl is not properly configured" ; exit 1 ; fi

# Where is the robot at?
uri="https://robot-ws.your-server.de/failover.yaml"

# check if first argument is an ip

echo $para | grep -E "\b(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\b" > /dev/null 2>&1
# if it is an ip, we do not need a config
if [ $? -eq 0 ]
        then
                if [ "$parb" == "set-and-exec" ] 
                        then 
                                echo "Option {set-and-exec} only available for configured services"
                                exit 1
                elif [ -z "$parb" ]
                        then
                                echo "$0 now needs one of the following options:"
                                echo "usage: $0 $para {get|set} <failover-IP>"
                                exit 1
                fi
                
                if [ -z "$parc"  -a "$parb" == "set" ] ; then echo "You have to specify the IP-address of the failover-server" ; exit 1 ; fi
                ip=$para
                stty -echo
                echo "username:"
                read user
                echo "password:"
                read pass
                stty echo
        else
# where will I find the config files

resourcedir="/etc/failover/resources"
resources=`cat $resourcedir/* | grep "resource" | awk '{ print $2 }'`

if [ $? -ne 0 ] 
        then 
                echo "There are no properly configured resources in $resourcedir"
        exit 1 
fi

# check if the service is properly configured in $resourcedir

if [ -z $para ]
        then
                echo "You have to specify a service or enter an IP"
                echo "configured services are:"
                echo $resources
        exit 1
fi


echo $resources | grep $para > /dev/null 

if [ $? -ne 0 ]
        then
                echo "$para not available"
                echo "configured services are:"
                echo $resources
                exit 1
fi

        ip=`cat $resourcedir/$para | grep "^ip" | awk '{ print $2 }'`
        if [ -z "$ip" ] ; then echo "There is no IP-address set for resource $para in $resourcedir/$para" ; exit 1 ; fi
        user=`cat $resourcedir/$para | grep "user" | awk '{ print $2 }'`
        pass=`cat $resourcedir/$para | grep "password" | awk '{ print $2 }'`
        if [ -z "$user" ] ; then echo "There is no username set for resource $para in $resourcedir/$para" ; exit 1 ; fi
        if [ -z "$pass" ] ; then echo "There is no password set for user $user in $resourcedir/$para" ; exit 1 ; fi

fi


case "$parb" in
  get)


        get=`$curl -s -u $user:$pass $uri/$ip`
        if echo "$get" | grep "^error" > /dev/null 2>&1
                then
                        echo "There is a communication error. Hetzner returned"
                        echo $get
                        exit 1
        elif [ -z "$get" ]
                then 
                        echo "There was no response from Hetzner"
                        exit 1
        elif echo "$get" | grep "ip" > /dev/null 2>&1 && echo "$get" | grep "server_ip" > /dev/null 2>&1 && echo "$get" | grep "active_server_ip" > /dev/null 2>&1
                then : 
                        else
                        echo "This script does not know how to handle Hetzners answer:"
                        echo $get
                exit 1
        fi

        yamlip=`echo "$get" | grep "^  ip" | awk '{ print $2 }'`
        yamlserverip=`echo "$get" | grep "^  server_ip" | awk '{ print $2 }'`
        yamlactiveip=`echo "$get" | grep "^  active_server_ip" | awk '{print $2 }'`

        echo "This is the active configuration of the service $para"
        echo "IP-address of the service: $yamlip"
        echo "failover-IP was ordered for: $yamlserverip"
        echo "currently the IP is routed to: $yamlactiveip"
        if [ "$ip" != "$para" ] ; then echo "the configured failover-server(s) is/(are): `cat $resourcedir/$para | grep "^failover_ip" | sed -e 's/failover_ip //g'`" ; fi
  ;;

  set)
      
  if [ -z $parc ]
        then
                echo "You have to specifiy the IP-address of the failover-server"
                echo "the configured failover-server(s) is/(are): `cat $resourcedir/$para | grep "^failover_ip" | sed -e 's/failover_ip //g'`"
                exit 1 
 fi
        

        echo "Waiting for a response..."
        set=`$curl -s -u $user:$pass $uri/$ip -d active_server_ip=$parc`
        
        if echo "$set" | grep "^error" > /dev/null 2>&1 
                then
                        if echo "$set" | grep "^  status: 409" > /dev/null 2>&1
                                then
                                        echo "Looks like you tried to route the failover-IP to the currently selected server"
                                        exit 1
                        elif echo "$set" | grep "^  status: 500" > /dev/null 2>&1
                                then
                                        echo "There seems to be an error with the rerouting request on Hetzners part"
                                        echo "Try later again"
                                        exit 1
                        elif echo "$set" | grep "^  status: 404" > /dev/null 2>&1
                                then
                                        echo "Hetzner returned Error 404"
                                        echo "Most likely the failover-IP for the service is faulty"
                                        exit 1
                        fi
                        echo "An unknown error occurred" 
                        exit 1
        
        elif echo "$set" | grep "^failover" > /dev/null 2>&1
                then
                        yamlip=`echo "$set" | grep "^  ip" | awk '{ print $2 }'`
                        yamlserverip=`echo "$set" | grep "^  server_ip" | awk '{ print $2 }'`
                        yamlactiveip=`echo "$set" | grep "^  active_server_ip" | awk '{ print $2 }'`

                        echo "This is the changed configuration of the service $para"
                        echo "IP-address of the service: $yamlip"
                        echo "failover-IP was ordered for: $yamlserverip"
                        echo "currently the IP is routed to: $yamlactiveip"
                        if [ "$ip" != "$para" ] ; then echo "the configured failover-server(s) is/(are): `cat $resourcedir/$para | grep "^failover_ip" | sed -e 's/failover_ip //g'`" ; fi
        fi

  ;;

  set-and-exec)

        script=`grep "script" $resourcedir/$para | awk '{ print $2 }'`
        

        if [ -z "$script" ]
                then 
                        echo "There is no script set in $resourcedir/$para"
                        exit 1
        elif [ -x $script ]
                then
                        ./$0 $para set $parc
                        if [ $? = 0 ] ; then $script ; else exit 1 ; fi
        elif [ -f $script ]
                then
                        echo "Script is not executable"
                        exit 1
        fi
  ;;
  *)
        if [ -n $parb ]
                then
                        echo "$parb is an unknown option"
        fi
        echo -e "$0 needs one of the following options"
        echo "usage: $0 <service> {get|set|set-and-exec} <failover-IP>"
        exit 1
  ;;

esac
```
