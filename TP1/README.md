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

Ce sont donc bien les DNS renseignés sur la machine qui sont utilisés. Dans "Answer Section", on peut remarquer que plusieurs adresses IP ce qui correspond à une répartition de charges serveur chez Reddit.  

- 🌞 afficher l'état actuel du firewall

```
[hbooex@TP-reseau ~]$ sudo firewall-cmd --list-all
[sudo] password for hbooex: 
public (active)
  target: default
  icmp-block-inversion: no 
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
Le firewall filtre donc les interfaces enp0s3 et enp0s8, seul le port 22 est autorisé pour "écouter" des demandes de connexions ssh externes.   
   - 🐙 sous CentOS8, ce n'est plus iptables qui est utilisé pour manipuler le filtrage réseau mais nftables. Jouez un peu avec nft et affichez les "vraies" règles firewall (firewalld, manipulé avec firewall-cmd n'est qu'une surcouche à nft)

   ```
   [hbooex@TP-reseau ~]$ sudo nft list ruleset
   table ip filter {
        chain INPUT {
                type filter hook input priority 0; policy accept;  
        }

        chain FORWARD {
                type filter hook forward priority 0; policy accept;
        }

        chain OUTPUT {
                type filter hook output priority 0; policy accept; 
        }
   }
                  
...
                         
      chain nat_POST_public_deny {
        }

        chain nat_POST_public_allow {
        }

        chain nat_POST_public_post {
        }
   }
   ```
   
## II. Edit configuration

### 1. Configuration cartes réseau

- 🌞 modifier la configuration de la carte réseau privée

   - modifier la configuration de la carte réseau privée pour avoir une nouvelle IP statique définie par vos soins


- ajouter une nouvelle carte réseau dans un DEUXIEME réseau privé UNIQUEMENT privé

   - 🌞 dans la VM définir une IP statique pour cette nouvelle carte


- vérifier vos changements

   - afficher les nouvelles cartes/IP
```
[hbooex@TP-reseau ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:17:69:88 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85218sec preferred_lft 85218sec
    inet6 fe80::18f2:de7e:b56:a0e5/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:94:f3:90 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.10/24 brd 192.168.100.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe94:f390/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4d:6d:82 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.10/24 brd 192.168.200.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4d:6d82/64 scope link
       valid_lft forever preferred_lft forever

[hbooex@TP-reseau ~]$ nmcli d
DEVICE  TYPE      STATE      CONNECTION 
enp0s3  ethernet  connected  enp0s3
enp0s8  ethernet  connected  enp0s8
enp0s9  ethernet  connected  enp0s9
lo      loopback  unmanaged  --
```
   - vérifier les nouvelles tables ARP/de routage
```
[hbooex@TP-reseau ~]$ ip r s
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
192.168.100.0/24 dev enp0s8 proto kernel scope link src 192.168.100.10 metric 103
192.168.200.0/24 dev enp0s9 proto kernel scope link src 192.168.200.10 metric 102

[hbooex@TP-reseau ~]$ arp -e
Address                  HWtype  HWaddress           Flags Mask            Iface
_gateway                 ether   52:54:00:12:35:02   C                     enp0s3
192.168.100.1            ether   0a:00:27:00:00:16   C                     enp0s8
```

L'interface enp0s8 apparaît sur notre table ARP, contrairement à l'interface enp0s9, car c'est l'interface que j'utilise pour me connecter en ssh.

### 2. Serveur SSH

- 🌞 modifier la configuration du système pour que le serveur SSH tourne sur le port 2222  

Il suffit de modifier le fichier `/etc/ssh/sshd_config` pour changer de port.

   - adapter la configuration du firewall (fermer l'ancien port, ouvrir le nouveau)
```
[hbooex@TP-reseau ~]$ sudo firewall-cmd --permanent --add-port=2222/tcp
success
[hbooex@TP-reseau ~]$ sudo firewall-cmd --permanent --remove-port=22/tcp
success
[hbooex@TP-reseau ~]$ sudo firewall-cmd --reload
success
[hbooex@TP-reseau ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8 enp0s9
  services: cockpit dhcpv6-client ssh
  ports: 2222/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[hbooex@TP-reseau ~]$ sudo ss -4tln
State                     Recv-Q                     Send-Q                                           Local Address:Port                                           Peer Address:Port
LISTEN                    0                          128                                                    0.0.0.0:2222                                                0.0.0.0:*
```

- 🌞 analyser les trames de connexion au serveur SSH

   - intercepter avec Wireshark et/ou tcpdump le trafic entre le client SSH et le serveur SSH
   détailler l'établissement de la connexion

   - doivent figurer au moins : échanges ARP, 3-way handshake TCP

- une fois la connexion établie, choisir une trame du trafic SSH et détailler son contenu


```
[hbooex@clone1 ~]$ sudo tcpdump -i enp0s8
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
16:45:38.294796 IP 192.168.100.20.44946 > clone1.TP-reseau-1.EtherNet/IP-1: Flags [S], seq 193922150, win 29200, options [mss 1460,sackOK,TS val 3839309149 ecr 0,nop,wscale 7], length 0
16:45:38.294906 ARP, Request who-has 192.168.100.20 tell clone1.TP-reseau-1, length 28
16:45:38.296563 ARP, Reply 192.168.100.20 is-at 08:00:27:a9:3e:e7 (oui Unknown), length 46
16:45:38.296573 IP clone1.TP-reseau-1.EtherNet/IP-1 > 192.168.100.20.44946: Flags [S.], seq 2807030402, ack 193922151, win 28960, options [mss 1460,sackOK,TS val 599196557 ecr 3839309149,nop,wscale 
7], length 0
16:45:38.297053 IP 192.168.100.20.44946 > clone1.TP-reseau-1.EtherNet/IP-1: Flags [.], ack 1, win 229, options [nop,nop,TS val 3839309152 ecr 599196557], length 0
16:45:38.330171 IP 192.168.100.20.44946 > clone1.TP-reseau-1.EtherNet/IP-1: Flags [P.], seq 1:22, ack 1, win 229, options [nop,nop,TS val 3839309185 ecr 599196557], length 21

...

^C
39 packets captured
39 packets received by filter
0 packets dropped by kernel

```

On distingue bien l'échange ARP entre les 2 machines. Je pense avoir repèrer les échanges TCP syn, syn-ack et ack (respectivement flag [S], [S.] et [.]) mais j'ai du mal à détailler le contenu d'une trame SSH.  

## III. Routage simple




## IV. Autres applications et métrologie

### 1. Commandes

- jouer avec iftop

   - expliquer son utilisation et imaginer un cas où iftop peut être utile

Cette commande nous permet de monitorer en temps réel notre utilisation de la bande passante sur une interface.

### 2. Cockpit

- 🌞 mettre en place cockpit sur la VM1

   - trouver (à l'aide d'une commande shell) sur quel port (TCP ou UDP) écoute Cockpit
   - vérifier que le port est ouvert dans le firewall

```
[hbooex@clone1 ~]$ sudo ss -ltu
Netid   State     Recv-Q    Send-Q            Local Address:Port                  Peer Address:Port    
udp     UNCONN    0         0              10.0.2.15%enp0s3:bootpc                     0.0.0.0:*       
tcp     LISTEN    0         128                     0.0.0.0:EtherNet/IP-1              0.0.0.0:*       
tcp     LISTEN    0         128                           *:websm                            *:*       
tcp     LISTEN    0         128                        [::]:EtherNet/IP-1                 [::]:*       
[hbooex@clone1 ~]$ sudo ss -ltun
Netid    State      Recv-Q     Send-Q               Local Address:Port           Peer Address:Port     
udp      UNCONN     0          0                 10.0.2.15%enp0s3:68                  0.0.0.0:*        
tcp      LISTEN     0          128                        0.0.0.0:2222                0.0.0.0:*        
tcp      LISTEN     0          128                              *:9090                      *:*        
tcp      LISTEN     0          128                           [::]:2222                   [::]:*        
```

- 🌞 explorer Cockpit, plus spécifiquement ce qui est en rapport avec le réseau  

On peut se connecter à l'interface web de cockpit à l'adresse du clone1 sur le port 9090. Cela nous donne accès à tout un tas d'infos pour administrer notre serveur.

### 3. Netdata