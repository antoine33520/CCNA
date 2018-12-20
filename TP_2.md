# TP 2

## I. EXPLORATION LOCAL EN SOLO

### 1) Affichage d'informations sur la pile TCP/IP locale


### Avec Ubuntu:

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

![](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/ip_gui.png?raw=true "Configuration réseau avec Ubuntu en graphique")



### Avec Windows:

```	
•     Win + R
•     Cmd
•     Ipconfig
```

![](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/adresseswindows.png?raw=true "Résultat Ipconfig Windows")

#### Affichez votre gateway
•	Trouvez sur internet une commande pour connaître l'adresse IP de la passerelle de votre carte WiFi

![](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/passerelle.png?raw=true "Adresse de la Passerelle par Ipconfig")


#### En graphique (GUI : Graphical User Interface)
En utilisant l'interface graphique de votre OS :
Trouvez comment afficher les informations sur une carte IP (change selon l'OS)
•	là aussi, cherchez sur internet
•	trouvez l'IP, la MAC et la gateway pour l'interface WiFi de votre PC

```
Paramètres -> Réseau et Internet -> Wi-Fi -> Propriétés du matériel
```

![](https://raw.githubusercontent.com/antoine33520/CCNA/master/TP2/detailsipgui.png?raw=true "Résultat graphique des paramètres réseau avec Windows")


### Questions
•	à quoi sert la gateway dans le réseau d'Ingésup ?
    ->	A faire passerelle entre le réseau interne c’est-à-dire les élèves et professeurs avec l’outil internet.



### II. Modifications des informations

#### A. Modification d'adresse IP - pt. 1

Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

 •	*Calculez la première et la dernière IP du réseau :* 

**Adresse IPV4 de l’ordinateur** : 10.33.0.218
**Binaire** : 00001010.00100001.00000000.11011010
**Sous-réseau** : 11111111.11111111.11111100.00000000
**Adresse network** : 10.33.0.0
**Adresse Broadcast** : 10.33.3.255    


• 	Changez l'adresse IP de votre carte WiFi sur Windows pour une autre (mais toujours dans le même réseau)   

```
	> Panneau de configuration
	> Réseau et Internet
	> Centre Réseau et partage
	> Connexions : Wi-Fi (Nom_Du_Wifi)
	> Propriétés
	> Protocole Internet Version 4 (TCP/IPV4)
	> Propritétés
	> Utiliser l’adresse IP suivante :
```
![](https://github.com/antoine33520/CCNA/blob/master/TP2/changement-ip-windows.png?raw=true "Champ de saisie des paramètres réseau")



Antoine a choisi l'adresse 10.33.3.241 au hasard en respectant la plage d'ip utilisable sur le réseau d'Ynov:
Adresse Réseau : 10.33.0.0
Masque du Réseau : 255.255.252.0 (/22)
Adresses utilisables : 10.33.0.1 ->  10.33.3.254 (il faut enlever ensuite une adresse pour la passerelle, ici 10.33.3.253 ainsi que d'autres adresses réservées que l'on ne connait pas actuellement)


#### B. `nmap`

Utilisez nmap pour scanner le réseau de votre carte WiFi et trouver une adresse **IP libre** :    


Aujourd'hui avec nmap nous n'avons trouvé aucune ip utilisée (nmap étant visiblement bloqué par le réseau Ynov, voir photo).
Par chance l'IP choisie par Antoine à la question précédente était visiblement inutilisée, j'ai utiliser la passerelle trouvée dans les questions précédentes (10.33.3.253).

En utilisant une machine virtuelle Ubuntu sur hyper-v, une version modifiée de linux avec WSL de Windows et PowerShell

![](https://github.com/antoine33520/CCNA/blob/master/TP2/resultatnmap.png?raw=true?raw=true "Résultats nmap avec différents sytèmes")

### C. Modification d'adresse IP - pt. 2

•	Modifiez de nouveau votre adresse IP vers une adresse IP que vous savez libre grâce à **nmap**

![](https://github.com/antoine33520/CCNA/blob/master/TP2/reseau_nmap.png?raw=true "Plusieurs résultats avec un nmap ayant fonctionné")

Grace à **nmap** on trouve une plage de 10 adresses IP libre.



## II. EXPLORATION LOCAL EN DUO

### Avec 3 PCs:
Il nous manque les screens et le code utilisé lors de nôtre TP avec 3 PCs avec des hyperviseurs (logiciel permettant la virtualisation de machines) différents : Virtualbox, VMware workstation et Hyper-V contenant chacun une machine virtuelle Ubuntu. 
* Les VMs étaient en accès par pont au port RJ45 de chaque PC
* Chaque PC était relié au switch via un câble réseau
* Un des PCs était également connecté au Wifi d'Ynov qui donnait cet accès à sa VM via une carte virtuelle faisant un pont entre la carte réseau Wifi et la Vm afin de partager l'accès à Internet
* La VM de cette ordinateur recevait donc la connexion au réseau Wifi et partagait cette accès via la carte ethernet aux autres machines en passant par le switch jusqu'aux VMs connectées par pont à la carte réseau filaire des deux autres PCs
* Les trois VMs avaient donc accès à Internet

Ensuite nous avons établi un netcat qui étaient fonctionnel, le seul problème était que nous n'avons pas réussi à utiliser netcat entre les 3 VMs mais uniquement entre 2 VMs.

Nous avons ensuite reconfigurer nos parefeu avec la commande iptables d'Ubuntu grâce à internet car les commandes sont compliquées.


```
antoinethys@at-laptop:~$ curl https://google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```

### 5. Petit chat privé ?

![](https://github.com/antoine33520/CCNA/blob/master/TP2/chat-incroyable.png?raw=true)


### 6. Wireshark

