# TP2 : Network low-level, Switching


## I. Simplest setup

- 🌞 mettre en place la topologie ci-dessus

- 🌞 faire communiquer les deux PCs
    - avec un ping qui fonctionne
        - déterminer le protocole utilisé par ping à l'aide de Wireshark  
    - analyser les échanges ARP
        - utiliser Wireshark et mettre en évidence l'échange ARP entre les deux machines (ARP Request et ARP Reply)
        - corréler avec les tables ARP des différentes machines

- 🌞 récapituler toutes les étapes (dans le compte-rendu, à l'écrit) quand PC1 exécute ping PC2 pour la première fois

    - échanges ARP
    - échange ping

- 🌞 expliquer...

    - pourquoi le switch n'a pas besoin d'IP
    - pourquoi les machines ont besoin d'une IP pour pouvoir se ping

Le ping utilise le protocole ICMP.  
Quand PC1 ping PC2 pour la 1ère fois, il y a tout d'abord un échange ARP entre les 2 machines. PC1 envoie un request qui demande à qui appartient l'IP qu'il ping et PC2 répond avec un reply qui indique son adresse MAC à PC1.  
Les adresses IP et MAC des machines sont alors renseignés dans leur table ARP respective.  
Ensuite les pings se font via ICMP donc, avec des échanges request et reply entre les 2 machines.

Les switch n'ont pas besoin d'IP car ils utilisent les adresses MAC pour transferer les trames et ils ne sont jamais les destinataires finaux.  
Les PCs au contraire on besoin d'une adresse IP pour pouvoir se faire adresser des messages/trames.

## II. More switches
 

### ToDo :

- 🌞 mettre en place la topologie ci-dessus

- 🌞 faire communiquer les trois PCs

    - avec des ping qui fonctionnent

- 🌞 analyser la table MAC d'un switch

    - show mac address-table
    - comprendre/expliquer chaque ligne

```
IOU1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6800    DYNAMIC     Et0/1
   1    0050.7966.6801    DYNAMIC     Et0/2
   1    0050.7966.6802    DYNAMIC     Et0/3
   1    aabb.cc00.0210    DYNAMIC     Et0/2
   1    aabb.cc00.0310    DYNAMIC     Et0/3
   1    aabb.cc00.0320    DYNAMIC     Et0/2
Total Mac Addresses for this criterion: 6

```

Chaque ligne correspond à l'association d'une adresse MAC et du port utilisé pour joindre cette même adresse. Les 3 premières lignes sont les adresses MAC des PCs, les 3 dernières correspondent à une interface du switch 2 et deux interfaces du switch 3.



### Mise en évidence du Spanning Tree Protocol

- 🌞 déterminer les informations STP

    - à l'aide des commandes dédiées au protocole
    - qui est le root bridge, quels sont les ports désactivés, etc.

```
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Altn BLK 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr
Et1/3               Desg FWD 100       128.8    Shr
Et2/0               Desg FWD 100       128.9    Shr
Et2/1               Desg FWD 100       128.10   Shr
Et2/2               Desg FWD 100       128.11   Shr
Et2/3               Desg FWD 100       128.12   Shr
Et3/0               Desg FWD 100       128.13   Shr
Et3/1               Desg FWD 100       128.14   Shr
Et3/2               Desg FWD 100       128.15   Shr
Et3/3               Desg FWD 100       128.16   Shr
```

Ici on voit bien que le port Et0/2 du switch IOU3 est bloqué (BLK). On voit aussi que le port Et0/1 est dédiée au root bridge (de toute façons c'est le seul ouvert). En faisant la même commande sur IOU1, on voit bien que c'est lui le root bridge.

```
IOU1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root     <=============== Juste là
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

....
```

-  🌞 faire un schéma en représentant les informations STP

    - rôle des switches (qui est le root bridge)
    - rôle de chacun des ports


```
                        +-----+
                        | PC2 |
                        +--+--+
                           |
                           |
                     RB+---+---+FWD
                   +---+  SW2  +----+
                   |   +-------+    |
                   |                |
                FWD|                |BLK
+-----+        +---+---+ FWD    +---+---+        +-----+
| PC1 +--------+  SW1  +--------+  SW3  +--------+ PC3 |
+-----+        +-------+      RB+-------+        +-----+
                RootBr
```

- 🌞 confirmer les informations STP

    - effectuer un ping d'une machine à une autre
    - vérifier que les trames passent bien par le chemin attendu (Wireshark)

En faisant un ping du PC3 vers le PC2, les trames devraient passer par le SW1 car il est le root bridge et que le port Et0/2 est bloqué, bien que cela semble "plus court" de passer directement au SW2. 

```
No.     Time           Source                Destination           Protocol Length Info
     34 40.571743      10.2.1.3              10.2.1.2              ICMP     98     Echo (ping) request  id=0xe567, seq=1/256, ttl=64 (reply in 41)

Frame 34: 98 bytes on wire (784 bits), 98 bytes captured (784 bits) on interface 0
Ethernet II, Src: Private_66:68:02 (00:50:79:66:68:02), Dst: Private_66:68:01 (00:50:79:66:68:01)
Internet Protocol Version 4, Src: 10.2.1.3, Dst: 10.2.1.2
Internet Control Message Protocol

No.     Time           Source                Destination           Protocol Length Info
     41 40.573675      10.2.1.2              10.2.1.3              ICMP     98     Echo (ping) reply    id=0xe567, seq=1/256, ttl=64 (request in 34)

Frame 41: 98 bytes on wire (784 bits), 98 bytes captured (784 bits) on interface 0
Ethernet II, Src: Private_66:68:01 (00:50:79:66:68:01), Dst: Private_66:68:02 (00:50:79:66:68:02)
Internet Protocol Version 4, Src: 10.2.1.2, Dst: 10.2.1.3
Internet Control Message Protocol
```

- 🌞 ainsi, déterminer quel lien a été désactivé par STP

Du coup c'est le lien entre SW2 et SW3.

- 🌞 faire un schéma qui explique le trajet d'une requête ARP lorsque PC1 ping PC3, et de sa réponse

    - représenter TOUTES les trames ARP (n'oubliez pas les broadcasts)

```
                        +-----+
                        | PC2 |
                        +--+--+
              broadcast    |
              who has ..   |
                           |
                       +---+---+
                ^  +---+  SW2  +----+
                |  |   +-------+    |
                |  |                |
                   |                |
+-----+        +---+---+        +---+---+        +-----+
| PC1 +--------+  SW1  +--------+  SW3  +--------+ PC3 |
+-----+        +-------+        +-------+        +-----+
Envoie   -->     Who has   -->   Who has   -->   Moi ! et ma
un ping       IP 10.2.1.3         ...            MAC c'est ...
              en broadcast       broadcast       reply  |
                                                        |
    Et voila      <--               <--            <-----
```


### Reconfigurer STP

- 🌞 changer la priorité d'un switch qui n'est pas le root bridge

- 🌞 vérifier les changements

    - avec des commandes sur les switches
    - 🐙 capturer les échanges qui suivent une reconfiguration STP avec Wireshark

On a fait la manip sur le SW3 :
```
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    4097
             Address     aabb.cc00.0300
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
```
Il est bien devenu le root bridge.


## Isolation 

### 1. Simple

- 🌞 mettre en place la topologie ci-dessus avec des VLANs

    - voir les commandes dédiées à la manipulation de VLANs

- 🌞 faire communiquer les PCs deux à deux

    - vérifier que PC2 ne peut joindre que PC3
    - vérifier que PC1 ne peut joindre personne alors qu'il est dans le même réseau (sad)

Config pour le switch :
```
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#vlan 10
IOU1(config-vlan)#name vlan10
IOU1(config-vlan)#exit
IOU1(config)#interface Ethernet 0/1
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 10
```
La même chose pour la vlan 20 mais cette fois ci sur les port Et0/2 et Et0/3.

Config pour les PCs :
```
Rien haha, il faut juste leur donner des IP.
```
Ainsi le PC1 connecté au SW1 sur le port Et0/1 ne peut pas communiquer avec les autres.

### 2. Avec trunk

- 🌞 mettre en place la topologie ci-dessus (en utilisant des VLAN, vous aurez besoin de la notion de trunk)

```
IOU1#conf t
IOU1(config)#interface Ethernet 0/0
IOU1(config-if)#switchport trunk encapsulation dot1q
IOU1(config-if)#switchport mode trunk
IOU1(config-if)#switchport trunk allowed vlan 10,20
```

Même chose pour IOU2.

- 🌞 faire communiquer les PCs deux à deux

    - vérifier que PC1 ne peut joindre que PC3
    - vérifier que PC4 ne peut joindre que PC2

- 🌞 mettre en évidence l'utilisation des VLANs avec Wireshark

Sur Wireshark on peut identifier l'ID du VLAN pour lequel la trame est destinée, ici un ping.
```
802.1Q Virtual LAN, PRI: 0, DEI: 0, ID: 10
    000. .... .... .... = Priority: Best Effort (default) (0)
    ...0 .... .... .... = DEI: Ineligible
    .... 0000 0000 1010 = ID: 10    <---- Juste là
    Type: IPv4 (0x0800)
```

## IV. Need perfs

- 🌞 mettre en place la topologie ci-dessus

    - configurer LACP entre SW1 et SW2

    - utiliser Wireshark pour mettre en évidence l'utilisation de trames LACP

    - vérifier avec un show interface po1 que la bande passante a bien été doublée

Pour mettre en place LACP sur IOU1 :
```
IOU1# conf t
IOU1(config)# interface range ethernet 0/0 , ethernet 0/3
IOU1(config-if-range)# channel-group 1 mode active
IOU1(config-if-range)# channel-protocol lacp
```
Même chose pour IOU2.

Sur Wireshark, on peut voir que les switchs communiquent avec le protocole LACP. Les paquets ont alors l'ID du VLAN natif, ici 1.

```
No.     Time           Source                Destination           Protocol Length Info
      4 2.555166       aa:bb:cc:00:01:00     Slow-Protocols        LACP     128    v1 ACTOR aa:bb:cc:80:01:00 P: 1 K: 1 **DCSG*A PARTNER aa:bb:cc:80:02:00 P: 1 K: 1 **DCSG*A

Frame 4: 128 bytes on wire (1024 bits), 128 bytes captured (1024 bits) on interface 0
Ethernet II, Src: aa:bb:cc:00:01:00 (aa:bb:cc:00:01:00), Dst: Slow-Protocols (01:80:c2:00:00:02)
802.1Q Virtual LAN, PRI: 0, DEI: 0, ID: 1
Slow Protocols
Link Aggregation Control Protocol
```


En tapant la commande `show interfaces po1` et `show interface Et0/0` (Et0/0 fait partie du port-channel), on peut voir sur la 3ème ligne que la bande passante à était doublée (BW est passé de 10k pour Et0/0 à 20k pour po1) :

```
Port-channel1 is up, line protocol is up (connected)
  Hardware is EtherChannel, address is aabb.cc00.0200 (bia aabb.cc00.0200)
  MTU 1500 bytes, BW 20000 Kbit/sec, DLY 1000 usec,   <---- Hop juste ici
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Auto-duplex, Auto-speed, media type is unknown
  input flow-control is off, output flow-control is unsupported
  Members in this channel: Et0/0 Et0/3
  ARP type: ARPA, ARP Timeout 04:00:00
....
```

    

