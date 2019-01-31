# TP4

## I. Mise en place du lab

### 2. Création des VMs

La désactivation de "SELinux", l'installation des paquets réseau et la désactivation de la carte "NAT" sont déjà décrites dans le sujet.
Les cartes réseau réliants des VMS ne permettant pas de communication avec le PC client utilisé pour le TP la carte "NAT" du le VM routeur restera active pour permettre une connexion SSH aux trois machines.

Définition des IPs statique:

* Pour la carte réseau de la VM routeur le réseau 1

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
> *édit*: Bon méthode un peu plus propre un fichier "/etc/systemd/system/tp.service" qui renvoit sur un fichier /root/tp.sh
> ```bash
> sysctl -w net.ipv4.conf.all.forwarding=1
> sysctl -w net.ipv4.conf.eth0.forwarding=0
> ```
> Puis
> ```bash
> systemctl enable tp
> ```
>
> *édit2*: bon mon idée avait pour moi une certaine logique mais ne fonctionne pas donc j'ai trouvé une autre solution bien plus propre (BAC +5 recherhce Google)
> Ajouter ```net.ipv4.ip_forward = 1``` au fichier /etc/sysctl.conf
> Cette fois ça fonctionne !
>
> *édit3*: En fait avant la méthode de l'édit2 j'étais parti sur un script service en utilisant systemctl mais j'ai pas réussi et je viens juste de comprendre pourquoi, en pensant à faire un script pour l'automatisation de l'installation de docker et kubernetes sur centos je me suis dit que j'avais dû oublier quelque chose et c'était le ```#!/bin/bash``` à mon avis. Je viens d'ouvrir vscode en deux secondes juste pour écrire ça mais je ferai le script kube plus tard si j'ai un peu de temps et on pourra en parler.

Ensuite on désactive le pare-feu

```bash
systemctl disable firewalld && systemctl stop firewalld
```

On vérifie les routes

```bash
[root@router1 ~]# ip route show
default via 192.168.20.1 dev eth0 proto dhcp metric 100 
10.1.0.0/24 dev eth1 proto kernel scope link src 10.1.0.254 metric 101 
10.2.0.0/24 dev eth2 proto kernel scope link src 10.2.0.254 metric 102
```

Maintenant il faut définir les routes pour le client et le serveur.
