# TP1 : Back to basics

## I. Gather informations

- 🌞 récupérer une liste des cartes réseau avec leur nom, leur IP et leur adresse MAC 
``` 
[hbooex@hbooex ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:17:69:88 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85193sec preferred_lft 85193sec
    inet6 fe80::18f2:de7e:b56:a0e5/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:94:f3:90 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.103/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s8
       valid_lft 1194sec preferred_lft 1194sec
    inet6 fe80::76aa:f2e2:4a2c:3e0f/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

- 🌞 déterminer si les cartes réseaux ont récupéré une IP en DHCP ou non
    - OUI pour enp0s3
```
[hbooex@hbooex ~]$ sudo nmcli -f DHCP4 con show enp0s3
DHCP4.OPTION[1]:                        domain_name = auvence.co
DHCP4.OPTION[2]:                        domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8 8.8.4.4
DHCP4.OPTION[3]:                        expiry = 1569589331
DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
DHCP4.OPTION[5]:                        requested_broadcast_address = 1
DHCP4.OPTION[6]:                        requested_dhcp_server_identifier = 1
DHCP4.OPTION[7]:                        requested_domain_name = 1
DHCP4.OPTION[8]:                        requested_domain_name_servers = 1
DHCP4.OPTION[9]:                        requested_domain_search = 1
DHCP4.OPTION[10]:                       requested_host_name = 1
DHCP4.OPTION[11]:                       requested_interface_mtu = 1
DHCP4.OPTION[12]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[13]:                       requested_nis_domain = 1
DHCP4.OPTION[14]:                       requested_nis_servers = 1
DHCP4.OPTION[15]:                       requested_ntp_servers = 1
DHCP4.OPTION[16]:                       requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[17]:                       requested_routers = 1
DHCP4.OPTION[18]:                       requested_static_routes = 1
DHCP4.OPTION[19]:                       requested_subnet_mask = 1
DHCP4.OPTION[20]:                       requested_time_offset = 1
DHCP4.OPTION[21]:                       requested_wpad = 1
DHCP4.OPTION[22]:                       routers = 10.0.2.2
DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
```

- 🌞 afficher la table de routage de la machine et sa table ARP
```
[hbooex@hbooex ~]$ ip route
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.103 metric 101
``` 

La 1ère ligne correspond à une route vers le routeur qui nous connecte au WAN.  
La 2ème ligne correspond à une route vers le réseau 10.0.2.0/24, utilisée pour une connexion locale, la passerelle pour cette route est à l'IP 10.0.2.15 et cette IP est portée par l'interface enp0s3.  
La 3ème ligne correspond à une route vers le réseau 192.168.56.0/24, utilisée pour une connexion locale, la passerelle pour cette route est à l'IP 192.168.56.103 et cette IP est portée par l'interface enp0s8.

```
[hbooex@hbooex ~]$ arp -a
? (192.168.56.101) at 0a:00:27:00:00:2f [ether] on enp0s8
_gateway (10.0.2.2) at 52:54:00:12:35:02 [ether] on enp0s3
```

La 1ère ligne indique l'adresse IP de mon ordinateur sur le réseau 192.168.56.0/24 avec son adresse MAC correspondante.  
La 2ème ligne indique l'adresse IP de la passerelle du réseau 10.0.2.0/24, toujours associée à son adresse MAC.


- 🌞 récupérer la liste des ports en écoute (listening) sur la machine (TCP et UDP)

```
[hbooex@TP-reseau ~]$ sudo ss -tlpnu
Netid    State      Recv-Q     Send-Q                  Local Address:Port         Peer Address:Port
udp      UNCONN     0          0               192.168.56.103%enp0s8:68                0.0.0.0:*         users:(("NetworkManager",pid=779,fd=22))
udp      UNCONN     0          0                    10.0.2.15%enp0s3:68                0.0.0.0:*         users:(("NetworkManager",pid=779,fd=18))
tcp      LISTEN     0          128                           0.0.0.0:22                0.0.0.0:*         users:(("sshd",pid=800,fd=6))
tcp      LISTEN     0          128                              [::]:22                   [::]:*         users:(("sshd",pid=800,fd=8))
```
La seule application qui "écoute" sur la machine est le ssh sur le port 22.


- 🌞 récupérer la liste des DNS utilisés par la machine

```
[hbooex@TP-reseau ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search auvence.co centos
nameserver 10.33.10.20
nameserver 10.33.10.2
nameserver 8.8.8.8

[hbooex@TP-reseau ~]$ dig www.reddit.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-17.P2.el8_0.1 <<>> www.reddit.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1765
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 13, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.reddit.com.                        IN      A

;; ANSWER SECTION:
www.reddit.com.         132     IN      CNAME   reddit.map.fastly.net.
reddit.map.fastly.net.  29      IN      A       151.101.129.140
reddit.map.fastly.net.  29      IN      A       151.101.1.140
reddit.map.fastly.net.  29      IN      A       151.101.193.140
reddit.map.fastly.net.  29      IN      A       151.101.65.140
```

Ce sont donc bien les DNS renseignés sur la machine qui sont utilisés. Dans "Answer Section", répartition de charges