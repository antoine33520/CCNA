# TP5

## I. Préparation du lab

### 1. Préparation VMs

On fait correspondre la machines à la configuration demandée en paramétrant une nouvelle carte "host-only" permettant la connexion SSH.
Ensuite on clone le patron pour avoir 3 VMs et on les joute dans gns3 en précisant qu'il y a 2 interfaces réseau.

### 2. Préparation Routeurs Cisco

On importe l'image qui va servire pour mettre en place les 2 routeurs.
Le réseau a utilisé pour les routeurs peut être 10.5.3.0/30 pour utilisé les ip 10.5.12.1 et 10.5.12.2 pour les deux routeurs.

Récapitulation des IPs:

| Machines       |    net1    |    net2    |    net3   |
|----------------|:----------:|:----------:|:---------:|
| client1.tp5.b1 |      X     |  10.5.2.10 |     X     |
| client2.tp5.b1 |      X     |  10.5.2.11 |     X     |
| router1.tp5.b1 | 10.5.1.254 |      X     | 10.5.12.1 |
| router2.tp5.b1 |      X     | 10.5.2.254 | 10.5.12.2 |
| server1.tp5.b1 |  10.5.1.10 |      X     |     X     |

## II. Lancement et configuration du lab

### Checklist IP VMs

La désactivation de SELinux, les installation et la désactivation de la carte ont été fait dans lors de la préparation des VMs.
Il faut ensuite définir les IPs statiques, les noms de domaines et avoir une connexion ssh fonctionnelle, pour ce il faut suivre la procédure indiquée. comme dans les TPs précédents.

### Checklist IP Routeurs

Pour la définition des IPs statiques et des noms de domaines des routeurs `router1.tp5.b1` et `router2.tp5.b1` il faut également suivre la procédure indiquée.

*Hier je n'ai pas fait mon push après mon commit et en faisant une mise à jour pour simplifier mon ordi s'est éteint et comme je voulais KDE je me suis dit que j'avais rien de très important sur mon dualboot donc petite réinstallation aujourd'hui1 et je viens juste de voir qu'il me manquait mon joli passage bien détaillé jusqu'à la fin de la checklist des routes. Mais saches que j'avais fait un bel effort mais comme j'ai bien avancé je vais passer un peu rapidement sur cette partie du rendu pour avoir le temps d'avancer la pratique*

Pour le `router1`:

```cisco
(config-if)# ip address 10.5.1.254 255.255.255.0
```

```cisco
(config-if)# hostname router1.tp5.b1
```

### Checklist routes

- [ ] `router1`:

  ```cisco
  (config)# ip route 10.5.2.0 255.255.255.0 10.5.12.2
  ```
 
 - [ ] `server1`:
 
  - Routes

    ```bash
    [root@server1 ~]# ip route add 10.2.0.0/24 via 10.2.0.254 dev eth0
    ```
    Temporaire

    ```bash
    [root@server1 ~]# nano /etc/sysconfig/network-scripts/route-eth0

    10.2.0.0/24 via 10.2.0.254 dev eth0
    ```
    Définitif
  
  -`Hosts`
    
    ```bash
    [root@server1 ~]# nano /etc/hosts
    
    10.5.2.10 client1 client1.tp5.b1
    10.5.2.11 client2 client2.tp5.b1
    ```

Depuis `client2`:

```bash
[root@client2 ~]# ping client1.tp5.b1 -c 2 && echo &&  ping server1.tp5.b1 -c 2
PING client1 (10.5.2.10) 56(84) bytes of data.
64 bytes from client1 (10.5.2.10): icmp_seq=1 ttl=64 time=4.83 ms
64 bytes from client1 (10.5.2.10): icmp_seq=2 ttl=64 time=6.53 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.830/5.683/6.537/0.856 ms

PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=33.7 ms
64 bytes from server1 (10.5.1.10): icmp_seq=2 ttl=62 time=27.9 ms

--- server1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 27.911/30.842/33.774/2.936 ms
```

## III. DHCP
### 1. Mise en place du serveur DHCP
#### 1. Renommer la machine

```bash
[root@client2 ~]# hostname dhcp-net2.tp5.b1
```

```bash
[root@client2 ~]# nano /etc/hostname
hostname dhcp-net2.tp5.b1
```

#### 2. Installer le serveur DHCP
La carte NAT (eth2) est déjà présente pour la réinstallation des VMs sur eve-ng il suffit donc de l'activer `ifup eth2`

```bash
[root@client2 ~]# ip a show dev eth2
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:00:00:05:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.17/24 brd 192.168.20.255 scope global noprefixroute dynamic eth2
       valid_lft 7143sec preferred_lft 7143sec
    inet6 fe80::250:ff:fe00:502/64 scope link 
       valid_lft forever preferred_lft forever
```

Ensuite il faut installer les paquets pour le serveur dhccp:

```bash
[root@dhcp-net2 ~]# yum install -y dhcp
```

#### 5. Démarrer le serveur DHCP