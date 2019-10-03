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
 

