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