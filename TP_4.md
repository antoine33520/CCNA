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

