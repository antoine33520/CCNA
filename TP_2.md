# TP 2

## I. EXPLORATION LOCAL EN SOLO

### 1) Affichage d'informations sur la pile TCP/IP locale

### Avec Ubuntu

#### En ligne de commande

    Il faut utiliser la commande "ip a" ou anciennement "ifconfig"

    Carte Wifi :
        Nom : wlp2s0
        Adresse MAC : 7c:b0:c2:df:05:61
        Adresse IP : 10.33.2.150
        Adresse Réseau : 10.33.0.0
        Gateway : 10.33.3.253 (Nom DNS : _gateway)

    Carte Ethernet :
        Nom : enp3s0f1
        Adresse MAC : 70:4d:7b:35:a4:8f
        Adresse IP : Pas d'adresse car la carte n'est pas

#### En graphique

!["Configuration réseau avec Ubuntu en graphique"](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/ip_gui.png?raw=true)

### Sur Windows

``` cmd
•     Win + R
•     Cmd
•     ipconfig
```

!["Résultat Ipconfig Windows"](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/adresseswindows.png?raw=true)

#### Affichez votre gateway

•Trouvez sur internet une commande pour connaître l'adresse IP de la passerelle de votre carte WiFi

!["Adresse de la Passerelle par Ipconfig](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/passerelle.png?raw=true)

#### En graphique (GUI : Graphical User Interface)

En utilisant l'interface graphique de votre OS :

* Trouvez comment afficher les informations sur une carte IP (change selon l'OS)
* trouvez l'IP, la MAC et la gateway pour l'interface WiFi de votre PC

``` _
Paramètres -> Réseau et Internet -> Wi-Fi -> Propriétés du matériel
```

![Résultat graphique des paramètres réseau avec Windows](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/detailsipgui.png?raw=true)

### Questions

A quoi sert la gateway dans le réseau d'Ingésup ?

* A faire passerelle entre le réseau interne c’est-à-dire les élèves et professeurs avec l’outil internet.

### II. Modifications des informations

#### A. Modification d'adresse IP - pt. 1

Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

* Calculez la première et la dernière IP du réseau :

**Adresse IPV4 de l’ordinateur** : 10.33.0.218
**Binaire** : 00001010.00100001.00000000.11011010
**Sous-réseau** : 11111111.11111111.11111100.00000000
**Adresse network** : 10.33.0.0
**Adresse Broadcast** : 10.33.3.255

• Changez l'adresse IP de votre carte WiFi sur Windows pour une autre (mais toujours dans le même réseau)

``` _
* Panneau de configuration
* Réseau et Internet
* Centre Réseau et partage
* Connexions : Wi-Fi (Nom_Du_Wifi)
* Propriétés
* Protocole Internet Version 4 (TCP/IPV4)
* Propritétés
* Utiliser l’adresse IP suivante :
```

![Champ de saisie des paramètres réseau](<https://github.com/antoine33520/CCNA/blob/master/TP2/changement-ip-windows.png?raw=true>)

Antoine a choisi l'adresse 10.33.3.241 au hasard en respectant la plage d'ip utilisable sur le réseau d'Ynov:
Adresse Réseau : 10.33.0.0
Masque du Réseau : 255.255.252.0 (/22)
Adresses utilisables : 10.33.0.1 ->  10.33.3.254 (il faut enlever ensuite une adresse pour la passerelle, ici 10.33.3.253 ainsi que d'autres adresses réservées que l'on ne connait pas actuellement)

#### B. `nmap`

Utilisez nmap pour scanner le réseau de votre carte WiFi et trouver une adresse **IP libre** :

Aujourd'hui avec nmap nous n'avons trouvé aucune ip utilisée (nmap étant visiblement bloqué par le réseau Ynov, voir photo).
Par chance l'IP choisie par Antoine à la question précédente était visiblement inutilisée, j'ai utiliser la passerelle trouvée dans les questions précédentes (10.33.3.253).

En utilisant une machine virtuelle Ubuntu sur hyper-v, une version modifiée de linux avec WSL de Windows et PowerShell

![Résultats nmap avec différents sytèmes](<https://github.com/antoine33520/CCNA/blob/master/TP2/resultatnmap.png?raw=true?raw=true> )

### C. Modification d'adresse IP - pt. 2

* Modifiez de nouveau votre adresse IP vers une adresse IP que vous savez libre grâce à **nmap**

![Plusieurs résultats avec un nmap ayant fonctionné](https://github.com/antoine33520/CCNA/blob/master/TP2/reseau_nmap.png?raw=true )

Grace à **nmap** on trouve une plage de 10 adresses IP libre.

## II. EXPLORATION LOCAL EN DUO

### Avec 3 PCs

Il nous manque les screens et le code utilisé lors de nôtre TP avec 3 PCs avec des hyperviseurs (logiciel permettant la virtualisation de machines) différents : Virtualbox, VMware workstation et Hyper-V contenant chacun une machine virtuelle Ubuntu.

* Les VMs étaient en accès par pont au port RJ45 de chaque PC
* Chaque PC était relié au switch via un câble réseau
* Un des PCs était également connecté au Wifi d'Ynov qui donnait cet accès à sa VM via une carte virtuelle faisant un pont entre la carte réseau Wifi et la Vm afin de partager l'accès à Internet
* La VM de cette ordinateur recevait donc la connexion au réseau Wifi et partagait cette accès via la carte ethernet aux autres machines en passant par le switch jusqu'aux VMs connectées par pont à la carte réseau filaire des deux autres PCs
* Les trois VMs avaient donc accès à Internet

Ensuite nous avons établi un netcat qui étaient fonctionnel, le seul problème était que nous n'avons pas réussi à utiliser netcat entre les 3 VMs mais uniquement entre 2 VMs.

Nous avons ensuite reconfigurer nos parefeu avec la commande iptables d'Ubuntu grâce à internet car les commandes sont compliquées.

``` bash
antoinethys@at-laptop:~$ curl https://google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```

### 5. Petit chat privé

![screen chat netcat](https://github.com/antoine33520/CCNA/blob/master/TP2/chat-incroyable.png?raw=true)

* Dans un premier temps il faut installer l'utilitaire pour utiliser le routeur "netcat-7.70-1.x86_64.rpm".
* Le fichier n'étant pas éxecutable sur Ubuntu . Il faut donc le transformer grâce à l'utilitaire alien.
* Le processus d'installation est donc possible pour le fichier.
* Il suffit ensuite de taper comme si au-dessus "nc" suivi d'une adresse IP et d'un port dans notre cas le "8888".

### 6. Wireshark

Voici des screens de wireshark lors de plusieurs utilisations du réseau:

#### Pendant un ping

![screen wireshark netcat pendant ping](https://github.com/antoine33520/CCNA/blob/master/TP2/ping.png?raw=true)

#### Pendant un netcat

![screen wireshark netcat pendant envoi d'un message](https://github.com/antoine33520/CCNA/blob/master/TP2/wireshark_ncat.png?raw=true)

#### Pendant que le PC1 sert du PC2 comme gateway (lors de l'ouverture du site de Wireshark)

![screen wireshark netcat gateway PC1 PC2](https://github.com/antoine33520/CCNA/blob/master/TP2/pcgateway.png?raw=true)

### 7. Firewall

Pour cette expérience, le port 1222 a été choisi.

• Configurer le firewall de votre OS pour accepter le ping :

![screen exeption firewall + ping](https://github.com/antoine33520/CCNA/blob/master/TP2/regle_firewall_ping.PNG?raw=true)

• Communication avec netcat par groupe de deux avec le firewall activé sur le PC serveur :

![screen exeption firewall + chat netcat](https://github.com/antoine33520/CCNA/blob/master/TP2/regle_firewall_netcat.png?raw=true)

## III. Manipulations d'autres outils/protocoles côté client

### 1. DHCP

• Afficher l'adresse IP du serveur DHCP du réseau WiFi :

#### Avec Windows

``` cmd
•     Win + R
•     Cmd
•     Ipconfig -all

```

Ce qui nous donne l'adresse DHCP suivante

![screen adresse IP DHCP](https://github.com/antoine33520/CCNA/blob/master/TP2/adresseDHCP.png?raw=true)

• Trouver la date d'expiration de votre bail DHCP :

On utilise les mêmes lignes de commande que précédemment pour trouver la date d'expiration du bail. On trouve ainsi :

![screen Bail DHCP](https://github.com/antoine33520/CCNA/blob/master/TP2/bail.png?raw=true)

• Demandez une nouvelle adresse IP (en ligne de commande):

#### Toujours avec Windows

``` cmd
•    Win + R
•    Cmd
•    ipconfig /release
```

(Cette commande envoie un message DHCPRELEASE au serveur DHCP pour libérer la configuration DHCP actuelle et annuler la configuration d'adresse IP de toutes les cartes (si aucune carte n'est spécifié) ou d'une carte spécifique si le paramètre Carte est inclus.)

``` cmd
•    ipconfig /renew
```

(Cette commande renouvelle la configuration DHCP de toutes les cartes. C’est à dire la connexion au serveur DHCP du provider et donc qu'il y a de grandes chances d'avoir une nouvelle adresse IP.)

### 2. DNS

• Trouver l'adresse IP du serveur DNS que connaît votre ordinateur :

#### Windows

En utilisant à nouveau la commande ipconfig -all, on obtiens les IP des serveurs DNS suivants :

![screen adresse IP serveur DNS](https://github.com/antoine33520/CCNA/blob/master/TP2/dns.png?raw=true)

• En utilisant l'outil nslookup, faites un lookup :

Pour Google.com :

![screen nslookup avec Google](https://github.com/antoine33520/CCNA/blob/master/TP2/nslookupgoogle.png?raw=true)

Pour Ynov.com :

![sreen nslookup avec Ynov.com](https://github.com/antoine33520/CCNA/blob/master/TP2/nslookupynov.png?raw=true)

*Nslookup permet de voir l'adresse IP correspondant à un nom de domaine quelconque. Chaque IP associée à un nom de domaine est donc unique et propre à celui-ci.*

• Faites un reverse lookup :

Pour l'adresse 78.78.21.21 :

![screen reverse lookup pour 78.78.21.21](https://github.com/antoine33520/CCNA/blob/master/TP2/nslookup78.png?raw=true)

Pour l'adresse 92.16.54.88 :

![sreen reverse lookup pour 92.16.54.88](https://github.com/antoine33520/CCNA/blob/master/TP2/nslookup92.png?raw=true)

*Le reverse lookup retourne le nom de l'hôte associé à une adresse IP. Par exemple pour les utilisateurs sur mobiles, le reverse lookup permet d'obtenir le nom du fournisseur ainsi que le pays et la ville du dit utilisateur.*