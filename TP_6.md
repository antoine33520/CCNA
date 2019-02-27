# TP6

## Tableau des Informations IP et OSPF

### Réseaux IP et aires OSPF

| Réseaux          | `area 0` | `area 1` | `area 2` | Commentaire                |
| ---------------- | -------- | -------- | -------- | -------------------------- |
| `10.6.100.0/30`  | X        | -        | -        | Liaison entre `r1` et `r2` |
| `10.6.100.4/30`  | X        | -        | -        | Liaison entre `r1` et `r4` |
| `10.6.100.8/30`  | X        | -        | -        | Liaison entre `r2` et `r3` |
| `10.6.100.12/30` | X        | -        | -        | Liaison entre `r3` et `r4` |
| `10.6.101.0/30`  | -        | X        | -        | Liaison entre `r3` et `r5` |
| `10.6.201.0/24`  | -        | X        | -        | Réseau des clients         |
| `10.6.202.0/24`  | -        | -        | X        | Réseau des serveurs        |

### Adressage IP de chacune des machines

| Machines         | `10.6.100.0/30` | `10.6.100.4/30` | `10.6.100.8/30` | `10.6.100.12/30` | `10.6.101.0/30` | `10.6.201.0/24` | `10.6.202.0/24` |
| ---------------- | --------------- | --------------- | --------------- | ---------------- | --------------- | --------------- | --------------- |
| `r1.tp6.b1`      | `10.6.100.1`    | `10.6.100.5`    | -               | -                | -               | -               | `10.6.202.254`  |
| `r2.tp6.b1`      | `10.6.100.2`    | -               | `10.6.100.9`    | -                | -               | -               | -               |
| `r3.tp6.b1`      | -               | -               | `10.6.100.10`   | `10.6.100.14`    | `10.6.101.1`    | -               | -               |
| `r4.tp6.b1`      | -               | `10.6.100.6`    | -               | `10.6.100.13`    | -               | -               | -               |
| `r5.tp6.b1`      | -               | -               | -               | -                | `10.6.101.2`    | `10.6.201.254`  | -               |
| `client1.tp6.b1` | -               | -               | -               | -                | -               | `10.6.201.10`   | -               |
| `client2.tp6.b1` | -               | -               | -               | -                | -               | `10.6.201.11`   | -               |
| `server1.tp6.b1` | -               | -               | -               | -                | -               | -               | `10.6.202.10`   |

## 2. Mise en place du lab

### Checklist IP Routeurs

Pour r1.tp6.b1, r2.tp6.b1, r3.tp6.b1, r4.tp6.b1 et r5.tp6.b1 la définition des IPs statiques et des noms de domaine se fait avec la même procédure que pour le TP précédent en utilisant les valeurs rappelées dans la section [Tableau des Informations IP et OSPF](./TP_6.md#tableau-des-informations-ip-et-ospf).

### Checklist VMs

Après avoir cloné la `VM patron` 4 fois pour correspondre aux demande du cahier des charges du TP, enuite on utilise la procédure habituelle pour:

* Définir les IPs statiques
* Définir les noms de domaine
* Remplir le fichier `hosts`

avec les valeurs de la section [Tableau des Informations IP et OSPF](./TP_6.md#tableau-des-informations-ip-et-ospf).

### Vérifier que tout ça fonctionne

Maintenant en faisant plusieurs `ping` on voit que le réseau est fonctionnel:

* `server1` <==> `server2` <==> `r1`
* `client1` <==> `client2` <==> `r2`
* `r2` <== `r1` ==> `r4`
* `r1` <== `r2` ==> `r3`
* `r2` <== `r3` ==> `r4`
* `r3` <== `r4` ==> `r1`

### Configuration de OSPF

*Pour cette partie on va utiliser le `r1.tp6.b1` comme exemple*

#### Activation de OSPF

Pour l'activation de OSPF une fois en mode de configuration avec une seule commande on active `OSPF` et dans le mode de configuration de cette fonction.

```cisco
r1.tp6.b1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
r1.tp6.b1(config)#router os
r1.tp6.b1(config)#router ospf 1
r1.tp6.b1(config-router)#
```

#### `router-id`

Ensuite on va configurer une `router-id` différent pour chaque routeur:

* router1 : 1.1.1.1
* router2 : 2.2.2.2
* router3 : 3.3.3.3
* router4 : 4.4.4.4
* router5 : 5.5.5.5

```cisco
r1.tp6.b1(config-router)#router-id 1.1.1.1
```

#### Configuration des routes à partager

On va indiqué aux routeurs de partager tout les réseau auxquels il sont directement connecté en précisant l'`AREA`.

* `router1.tp6.b1`
  * network 10.6.100.0 0.0.0.3 area 0
  * network 10.6.100.4 0.0.0.3 area 0
  * network 10.6.202.0 0.0.0.255 area 2
* `router2.tp6.b1`
  * network 10.6.100.0 0.0.0.3 area 0
  * network 10.6.100.8 0.0.0.3 area 0
* `router3.tp6.b1`
  * network 10.6.100.8 0.0.0.3 area 0
  * network 10.6.100.12 0.0.0.3 area 0
  * network 10.6.101.0 0.0.0.3 area 1
* `router4.tp6.b1`
  * network 10.6.100.4 0.0.0.3 area 0
  * network 10.6.100.12 0.0.0.3 area 0
* `router5.tp6.b1`
  * network 10.6.101.0 0.0.0.3 area 1
  * network 10.6.201.0 0.0.0.255 area 1

En gardant l'exemple de `r1.tp6.b1`:

```cisco
r1.tp6.b1(config-router)#network 10.6.100.0 0.0.0.3 area 0
r1.tp6.b1(config-router)#network 10.6.100.4 0.0.0.3 area 0
r1.tp6.b1(config-router)#network 10.6.202.0 0.0.0.255 area 2
```

#### Une vérification pour être sûr

On va utiliser une nouvelle fois `router1.tp6.b1`

##### On vérifie les routes enregistrées sur le routeur:

```cisco
r1.tp6.b1#sh ip ro ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
O        10.6.100.8/30 [110/20] via 10.6.100.2, 00:12:40, Ethernet1/2
O        10.6.100.12/30 [110/20] via 10.6.100.6, 00:09:01, Ethernet1/4
O        10.6.101.0/30 [110/30] via 10.6.100.6, 00:07:49, Ethernet1/4
                       [110/30] via 10.6.100.2, 00:07:49, Ethernet1/2
```

On peut voir avec le résultat de la commande les différentes routes `OSPF`, via quelle interface elles passent et dans le cas de `10.6.101.0/30` on voit qu'il est possible de passer par `r2.tp6.b1` ou par `r4.tp6.b1`.

##### Ensuite que le protocol OSPF est bien actif

```cisco
r1.tp6.b1#sh ip pro
*** IP Routing is NSF aware ***

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.6.100.0 0.0.0.3 area 0
    10.6.100.4 0.0.0.3 area 0
    10.6.202.0 0.0.0.255 area 2
  Routing Information Sources:
    Gateway         Distance      Last Update
    3.3.3.3              110      00:12:35
    2.2.2.2              110      00:17:27
  Distance: (default is 110)
```

Ici on voit les routes partagées et deux des routeurs OSPF du réseau.

##### Les voisins

```cisco
r1.tp6.b1#sh ip os ne  

Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           1   FULL/BDR        00:00:33    10.6.100.6      Ethernet1/4
2.2.2.2           1   FULL/BDR        00:00:36    10.6.100.2      Ethernet1/2
```

Là on a les `ID` des deux routeurs voisins, `r4.tp6.b1` et `r2.tp6.b1`.

##### Les pings

* `r1` vers `r5`

```cisco
r1.tp6.b1#ping 10.6.101.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.6.101.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 36/39/40 ms
```

* `client1` vers `server1`

```bash
[root@client1 ~]# ping -c 2 server1
PING server1 (10.6.202.10) 56(84) bytes of data.
64 bytes from server1 (10.6.202.10): icmp_seq=1 ttl=60 time=61.8 ms
64 bytes from server1 (10.6.202.10): icmp_seq=2 ttl=60 time=60.6 ms

--- server1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 60.607/61.238/61.870/0.678 ms
```

* On refait la même chose mais avec un `traceroute` sur `r1` (*Commande lente et non visuellement optimisée*)

```cisco
r1.tp6.b1#traceroute 10.6.101.2
Type escape sequence to abort.
Tracing the route to 10.6.101.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.6.100.2 16 msec
    10.6.100.6 16 msec
    10.6.100.2 4 msec
  2 10.6.100.14 16 msec
    10.6.100.10 16 msec
    10.6.100.14 16 msec
  3 10.6.101.2 20 msec 40 msec *
```

* On refait la même chose mais avec un `traceroute` sur `client1`

```bash
[root@client1 ~]# traceroute server1
traceroute to server1 (10.6.202.10), 30 hops max, 60 byte packets
 1  gateway (10.6.201.254)  18.575 ms  18.488 ms  31.109 ms
 2  10.6.101.1 (10.6.101.1)  43.974 ms  49.978 ms  49.938 ms
 3  10.6.100.9 (10.6.100.9)  56.803 ms  58.413 ms  77.469 ms
 4  10.6.100.1 (10.6.100.1)  85.206 ms  105.641 ms  111.506 ms
 5  server1 (10.6.202.10)  133.442 ms !X  133.822 ms !X  139.991 ms !X
```

## Lab 3 : Let's end this properly

<!-- ## __Informations__

- Ce TP a entièrement été réalisé sur [EVE-NG Community](https://www.eve-ng.net/) et la connexion `telnet` pour l'accès console aux routeurs et aux machines sont faites avec `"EVE-NG Intergration (Linux client side)"` utilisant le protocol `telnet` mais permetant aussi les protocols `vnc` et `rdp`.
- La virtualisation des trois VMs Centos a été faite avec `QEMU` intégré à `EVE-NG`, (par défaut la version 2.4.0).
- EVE-NG a été installé via son image iso sur une machine virtuelle hébergée `Proxmox`.
- Le routeur servant pour le réseau NAT et un pfsense hébergé également sur le `Proxmox`.
- Le serveur DNS est sur un réseau différent que celui attribué par le serveur DHCP mais le routage est bien éffectué entre les deux réseau via un tunnel IPsec.
- Topology du réseau sur EVE-NG [ici](./TP6/topology_eve.png).
- Auteur: Antoine THYS B1B Ynov Informatique -->