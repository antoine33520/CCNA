# TP4

## I. Mise en place du lab

### 2. Création des VMs

La désactivation de "SELinux", l'installation des paquets réseau et la désactivation de la carte "NAT" sont déjà décrites dans le sujet.
Les cartes réseau réliants des VMS ne permettant pas de communication avec le PC client utilisé pour le TP la carte "NAT" du le VM routeur restera active pour permettre une connexion SSH aux trois machines.

Définition des IPs statique:

- Pour la carte réseau de la VM routeur le réseau 1

```bash
NAME=eth1
DEVICE=eth1

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.1.0.254
NETMASK=255.255.255.0
```

Il faut suivre la même procédure pour la deuxième interface ainsi que pour les autres VMs.

Pour la connexion ssh pour une utilisation par défaut il n'y a aucune manipulation à faire, le serveur est installé, démarré et le port par défaut (22) est ouvert sur le pare-feu.

Pour la VM routeur:

Le FQDN peut être modifié soit soit de façon temporaire avec la commande.

```bash
sudo hostname router1.tp4
```

ou de façon permanente

```bash
echo 'router1.tp4' | sudo tee /etc/hostname
```

C'est la même procédure pour les autres machines.

La modification du fichier "/etc/hosts" permet d'indiquer à une machine le hostname et le FQDN d'une autre machine en fonction de son IP.

Pour la routeur il faut rajouter les lignes:

```bash
10.1.0.10 client1 client1.tp4
10.2.0.10 server1 server1.tp4
```

Il faut effectuer la même opération sur les autres machines en utilisant les informations correspondantes.

### 3. Mise en place du routage statique

Pour activer la fonction de routage sur la machine routeur il faut utiliser la commande

```bash
sysctl -w net.ipv4.conf.all.forwarding=1
```

> Pour parrer à la désactivation du routage lors du redémarrage de la machine j'ai inclu la commande dans le fichier ".bashrc" (la méthode trés trés sale) de l'utilisateur root (le seul utilisateur que j'ai et ça evite le mot de passe sudo).
>
> _édit_: Bon méthode un peu plus propre un fichier "/etc/systemd/system/tp.service" qui renvoit sur un fichier /root/tp.sh
>
> ```bash
> sysctl -w net.ipv4.conf.all.forwarding=1
> sysctl -w net.ipv4.conf.eth0.forwarding=0
> ```
>
> Puis
>
> ```bash
> systemctl enable tp
> ```
>
> _édit2_: bon mon idée avait pour moi une certaine logique mais ne fonctionne pas donc j'ai trouvé une autre solution bien plus propre (BAC +5 recherhce Google)
> Ajouter `net.ipv4.ip_forward = 1` au fichier /etc/sysctl.conf
> Cette fois ça fonctionne !
>
> _édit3_: En fait avant la méthode de l'édit2 j'étais parti sur un script service en utilisant systemctl mais j'ai pas réussi et je viens juste de comprendre pourquoi, en pensant à faire un script pour l'automatisation de l'installation de docker et kubernetes sur centos je me suis dit que j'avais dû oublier quelque chose et c'était le `#!/bin/bash` à mon avis. Je viens d'ouvrir vscode en deux secondes juste pour écrire ça mais je ferai le script kube plus tard si j'ai un peu de temps et on pourra en parler.

Ensuite on désactive le pare-feu

```bash
systemctl disable firewalld && systemctl stop firewalld
```

Vérification des routes

```bash
[root@router1 ~]# ip route show
default via 192.168.20.1 dev eth0 proto dhcp metric 100
10.1.0.0/24 dev eth1 proto kernel scope link src 10.1.0.254 metric 101
10.2.0.0/24 dev eth2 proto kernel scope link src 10.2.0.254 metric 102
```

Maintenant il faut définir les routes pour le client et le serveur:

- La vesrion temporaire a déjà été vu dans le précédent avec ip route add
- Pour une version permanentre de la commande il faut éditer le fichier `/etc/sysconfig/network-scripts/route-<LOCAL_INTERFACE_NAME>` et ajouter `<NETWORK_ADDRESS> via <IP_GATEWAY> dev <LOCAL_INTERFACE_NAME>`

Maintenant les VMs se ping.

Résultats des `traceroute`

- client1:

```bash
[root@client1 ~]# traceroute 10.2.0.10
traceroute to 10.2.0.10 (10.2.0.10), 30 hops max, 60 byte packets
 1  router1 (10.1.0.254)  0.179 ms  0.147 ms  0.125 ms
 2  server1 (10.2.0.10)  0.540 ms !X  0.511 ms !X  0.497 ms !X
```

- server1:

```bash
[root@server1 ~]# traceroute 10.1.0.10
traceroute to 10.1.0.10 (10.1.0.10), 30 hops max, 60 byte packets
 1  router1 (10.2.0.254)  0.213 ms  0.170 ms  0.105 ms
 2  client1 (10.1.0.10)  0.304 ms !X  0.260 ms !X  0.190 ms !X
```

## II. Spéléologie réseau

### 1. ARP

> Pour cette partie j'ai eu de gros problèmes surement dû à mes cartes réseau qui étaient Open vSwitch et non des cartes KVM standard, je pouvais pas trop me permettre de redémarrer le serveur mais bon j'ai changé le type de carte puis `apt update && apt dist-upgrade` et un petit `reboot` et visiblement problème résolu. Je ne sais pas exactement d'où vient le problème mais si j'ai le temps la piste des cartes OVS est a creusé un peu plus que ce que j'ai fait mais en effet dans certains cas en fonction du paramétrage elles peuvent faire automatiquement un filtrage de certains paquets.

#### A .Manip 1

##### Après un `ip neigh flush all`

Pour le client1:

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 REACHABLE
```

Pour le server1:

```bash
[root@server1 ~]# ip neigh show
10.2.0.254 dev eth1 lladdr 56:35:7f:db:96:6a DELAY
```

Ces lignes correspondent à l'adresse IP et MAC de la Gateway, comme il s'agit de la passerelle par défaut la table arp garde toujours cette adresse à jour.

##### Après un ping

> Léo concrètement j'ai un gros problème dont je cherche la source, j'ai pensé que c'était mes cartes réseaux comme je te l'ai dit mais visible ça ne vient pas de là parce que même après le changement de carte toujours le même problème, pour le `ping` pas de problème mais l'arp le passe pas même en désactivant le pare-feu de proxmox. J'ai donc tenté avec la commande `arping` et dans mon désespoire j'ai aussi tenté avec une machine en écoute et l'autre en demande mais toujours rien sur l'autre vm dans la table, `arping` et je précise aussi que `arping` passe aussi du client1 ou server1 vers la carte du routeur dans le réseau opposé. Le routeur lui a bien le client1 et le server1 dans sa table \
>
> _édit_: Bon c'est résolu mais j'ai pas trop compris comment mais l'arp passe après 3 jours a chercher une solution \
>
> _édit 2_: au moment où j'ai fait le premier édit j'étais heureux mais en fait ça fonctionnait... Donc après avoir fouillé un peu (Internet m'a pas vraiment aidé) j'ai modifier à la main des variables dans `sysctl` et donc après du `sysctl -w "net.ipv4.conf.all.proxy_arp"=1` et du `sysctl -w "net.ipv4.conf.all.arp_accept"=1`

Client1:

```bash
[root@server1 ~]# ping -c 2 client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.388 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=0.701 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.388/0.544/0.701/0.158 ms
```

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 DELAY
10.2.0.10 dev eth1 lladdr 6e:5d:c1:82:8a:05 STALE
```

Server1

```bash
[root@server1 ~]# ping -c 2 client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.388 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=0.701 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.388/0.544/0.701/0.158 ms
```

```bash
[root@server1 ~]# ip neigh show
10.1.0.10 dev eth1 lladdr 56:35:7f:db:96:6a STALE
10.2.0.254 dev eth1 lladdr 56:35:7f:db:96:6a DELAY
```

Lors du ping la machine expéditrice a besoin de l'adresse mac de la machine destinataire, cette première machine envoie une requête en broadcast pour savoir quelle est l'adresse MAC du la machine ayant l'ip demandée.

_Info_: _Il ne faut pas prendr en compte la carte eth0 sur le routeur, il s'agit de la carte NAT qui est acctivée pour l'accès SSH_

#### B. Manip 2

```bash
ip neigh flush all
```

La table ARP du routeur juste après le flush.

```bash
[root@router1 ~]# ip neigh show all
192.168.20.1 dev eth0 lladdr c2:59:6b:bf:53:4e REACHABLE
```

Ensuite après le ping du client1 vers server1

```bash
[root@router1 ~]# ip neigh show all
192.168.20.1 dev eth0 lladdr c2:59:6b:bf:53:4e REACHABLE
10.1.0.10 dev eth1 lladdr 42:c1:ee:c4:c5:ab REACHABLE
10.2.0.10 dev eth2 lladdr fe:02:46:4f:31:bd REACHABLE
```

Le router1 fait un pont entre les deux réseaux donc le client1 a dû passer par le router1 pour envoyé son ping puis de même pour le retour. Le router1 a donc fait des demandes arp pour savoir quelles machines avaient les IPs concernées.

#### C. Manip 3

On flush les 3 machines.

La table ARP

```bash
root@sys:~# ip neigh show
91.121.118.254 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
91.121.118.250 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
192.168.20.1 dev vmbr1 lladdr c2:59:6b:bf:53:4e STALE
fe80::2ff:ffff:feff:fffe dev vmbr0 lladdr 00:ff:ff:ff:ff:fe router REACHABLE
fe80::418:1 dev vmbr0 lladdr 00:00:5e:00:02:01 router STALE
fe80::2ff:ffff:feff:fffd dev vmbr0 lladdr 00:ff:ff:ff:ff:fd router REACHABLE
```

Après avoir vidé la table :

```bash
91.121.118.254 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
192.168.20.12 dev vmbr1 lladdr f6:9b:f9:f3:e3:ff DELAY
91.121.118.250 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
fe80::2ff:ffff:feff:fffe dev vmbr0 lladdr 00:ff:ff:ff:ff:fe router STALE
fe80::418:1 dev vmbr0 lladdr 00:00:5e:00:02:01 router STALE
fe80::2ff:ffff:feff:fffd dev vmbr0 lladdr 00:ff:ff:ff:ff:fd router STALE
```

Le résultat est sensiblement le même sachant que l'ip "192.168.20.1" est la Gateway
pour toutes les VMs Ayant accès au NAT. Et 192.168.20.12 est l'IP de la carte NAT du router1 que j'ai utilisé pour la connexion SSH au Promox.
91.121.118.254 est l'IP de Gateway fournie par So You Sart (OVH) et je ne sais pas à quoi correspond la 91.121.118.250, surement une redondance.
A cause mes problèmes ARP l'hyperviseur lui même n'a plus d'IP sur les cartes utilisées pour le TP et n'a donc plus accès qu'au réseau utilisé en local pour les autres VM et les cartes NAT, "192.168.20.0/24".

#### D. Manip 4

La table ARP:

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 DELAY
```

```bash
[root@client1 ~]# ifup eth0
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/2)
```

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 REACHABLE
192.168.20.1 dev eth0 lladdr c2:59:6b:bf:53:4e REACHABLE
```

Maintenant nous avons l'adresse IP et MAC de la passerelle utilisée pour l'accès au Internet (un petit PfSense).

### 2. Wireshark

#### A. Interception d'ARP et `ping`

On voit bien qu'ici notre client1 ne communique avec le protocol ARP (niveau 2) qu'avec le router1 et pas directement avec le server1.
Ceci est dû au fonctionnement du protocol arp, sauf dans certains précis l'ARP est envoyé en boradcast or le client1 et le server1 ne sont pas sur le même réseau donc n'ont pas la même adresse de broadcast et au final pas d'arp entre les deux machines. Pour régler celà la communication passe par le router qui lui est sur les deux réseaux.
