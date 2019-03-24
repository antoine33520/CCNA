# Infrastructure finale

Ce fichier correspond à la mise en place de l'infrastructure finale pour le cours de B1.\
Les bases de l'infrastructure seront celles du [TP 6](TP_6.md) et la configuration ne sera pas entièrement détaillée à nouveau.

## Introduction

Après une bonne configuration personnalisée de gns3 on commence l'ajout et le placement du matériel.\
Ici les routeurs cisco __C3640__ sont remplacés par des __C7200__, derrière il y a des VMs __CentOS 7__ avec __OpenVSwitch__(OVS), des Switchs Cisco __vIOS__ et deux containers Docker __gns3-endhost__ à la demande si besoin lors des tests et des __VPCS__ de GNS3 qui sont des machines clientes minimalistes.

### Tableau des Informations IP et OSPF

En premier lui pour la mise en place d'__OSPF__ il faut le plan d'adressage IP de ces équipements.\

| __Routeurs__    | `10.6.100.0/30` | `10.6.100.4/30` | `10.6.100.8/30` | `10.6.100.12/30` | `10.6.100.16/30` | `10.6.100.20/30` | `10.6.101.0/30` | `10.6.111.0/30` | `10.6.112.0/30` |
| --------------- | --------------- | --------------- | --------------- | ---------------- | ---------------- | ---------------- | --------------- | --------------- | --------------- |
| `r1.at.final`   | `10.6.100.1`    | `10.6.100.5`    | -               | -                | `10.6.100.17`    | -                | -               | -               | `10.6.112.1`    |
| `r2.at.final`   | `10.6.100.2`    | -               | `10.6.100.9`    | -                | -                | `10.6.100.21`    | -               | -               | -               |
| `r3.at.final`   | -               | -               | `10.6.100.10`   | `10.6.100.14`    | `10.6.100.18`    | -                | `10.6.101.1`    | -               | -               |
| `r4.at.final`   | -               | `10.6.100.6`    | -               | `10.6.100.13`    | -                | `10.6.100.22`    | -               | -               | -               |
| `r5.at.final`   | -               | -               | -               | -                | -                | -                | `10.6.101.2`    | `10.6.111.1`    | -               |
| `ovs1.at.final` | -               | -               | -               | -                | -                | -                | -               | `10.6.111.2`    | -               |
| `ovs2.at.final` | -               | -               | -               | -                | -                | -                | -               | -               | `10.6.112.2`    |

Et également les aires __OSPF__.

| Réseaux          | `area 0` | `area 1` | `area 2` | Commentaire                  |
| ---------------- | -------- | -------- | -------- | ---------------------------- |
| `10.6.100.0/30`  | X        | -        | -        | Liaison entre `r1` et `r2`   |
| `10.6.100.4/30`  | X        | -        | -        | Liaison entre `r1` et `r4`   |
| `10.6.100.8/30`  | X        | -        | -        | Liaison entre `r2` et `r3`   |
| `10.6.100.12/30` | X        | -        | -        | Liaison entre `r3` et `r4`   |
| `10.6.100.16/24` | X        | -        | -        | Liaison entre `r1` et `r3`   |
| `10.6.100.20/24` | X        | -        | -        | Liaison entre `r2` et `r4`   |
| `10.6.101.0/30`  | -        | X        | -        | Liaison entre `r3` et `r5`   |
| `10.6.111.0/30`  | -        | X        | -        | Liaison entre `r5` et `OVS1` |
| `10.6.112.0/30`  | -        | -        | X        | Liaison entre `r1` et `OVS2` |

Ces informations ont été modifiées pour correspondrent au besoins de cette infrastructure.

### Topologie

![Topologie préparation LAB](Final/Topologie_base.png)

## 1. Mise en place du lab

### Checklist IP Routeurs

Pour r1.at.final, r2.at.final, r3.at.final, r4.at.final et r5.at.final la définition des IPs statiques et des noms de domaine se fait avec la même procédure que pour le TP 5 en utilisant les valeurs rappelées dans la section [Tableau des Informations IP et OSPF](Infra_finale.md#tableau-des-informations-ip-et-ospf).\
r4.at.final aura également une IP attribuée par le serveur `DHCP` de sa carte reliée au réseau `NAT`.

### Checklist OVS

Après avoir cloné deux fois la `VM Patron` qui sera utilisée plus tard pour OVS ,on utilise la procédure habituelle pour:

* Définir les IPs statiques
* Définir les noms de domaine
* Remplir le fichier `hosts`

avec les valeurs de la section [Tableau des Informations IP et OSPF](Infra_finale.md#tableau-des-informations-ip-et-ospf).

### Configuration de OSPF

Pour la configuration d'OSPF il suffit de reprendre la configuration du TP 6 indiquée [ici](TP_6.md#configuration-de-ospf) en rajoutant les routes appropriées.

Les IDs ne changent pas:

* router1 : 1.1.1.1
* router2 : 2.2.2.2
* router3 : 3.3.3.3
* router4 : 4.4.4.4
* router5 : 5.5.5.5

Après modification les routes sont les suivantes:

* `router1.tp6.b1`
  * network 10.6.100.0 0.0.0.3 area 0
  * network 10.6.100.4 0.0.0.3 area 0
  * network 10.6.100.16 0.0.0.3 area 0
  * network 10.6.112.0 0.0.0.3 area 2
* `router2.tp6.b1`
  * network 10.6.100.0 0.0.0.3 area 0
  * network 10.6.100.8 0.0.0.3 area 0
  * network 10.6.100.20 0.0.0.3 area 0
* `router3.tp6.b1`
  * network 10.6.100.8 0.0.0.3 area 0
  * network 10.6.100.12 0.0.0.3 area 0
  * network 10.6.100.16 0.0.0.3 area 0
  * network 10.6.101.0 0.0.0.3 area 1
* `router4.tp6.b1`
  * network 10.6.100.4 0.0.0.3 area 0
  * network 10.6.100.12 0.0.0.3 area 0
  * network 10.6.100.20 0.0.0.3 area 0
* `router5.tp6.b1`
  * network 10.6.101.0 0.0.0.3 area 1
  * network 10.6.111.0 0.0.0.3 area 1

La configuration de OSPF est maintenant fonctionnelle.

### NAT

Pour founir un accès Internet au réseau il faut configurer le NAT sur `r4.at.final` et la partager aux autres routeurs du réseau OSPF, la carte g0/0 reliant `r4` au NAT est déjà configurée en DHCP.\
La procédure est la même que dans le [TP précédent](TP_6.md#12-configuration-du-nat).\
On peut maintenant activer le lookup DNS et déclarer un serveur DNS sur chaque routeur.

### Précautions

Par pécaution il est nécessaire d'enregister les configurations des routeurs (copie du fichier `running-config` vers `stratup-config`) et de faire un snapshot avec gns3, sur du matériel physique cette manipulation aurait été faite en faisant une copie du fichier `startup-config` sur un serveur fttp.\
*Comme il s'agit d'un environnement il n'y a pas de risque de casse mais devoir recommencer toute cette partie prendrait un peu de temps si un malheur arrivait*

## En route pour VxLan *(avec beaucoup d'espoir, de nicotine et de caféine)*

<!-- Il est 3h42 et j'ai pas mal de recherche sur le VxLan fait de différentes façons, VTEP, UNICAST, MULTICAST, BGP, l'OVERLAY, les SDN, Open vSwitch, GRE, la tunnelisation, l'encapsulation, les cisco csr et nexus, l'isolation, l'IPv6 (on verra pour se lancer là dedans plus tard sachant qu'avant mon changement de topologie réseau et matériel qui se resume à tout casser avec un formatage de mes hyperviseurs une bonne partie de mon infra avec de l'IPv6 mais les études ont fait que je n'ai plus le temps de m'occuper de mes pauvres bébés qui risque de se faire reformater à la fin des projets qui m'empêches de me lâcher et de tout casser et la faire rennaitre de ses cendre bien plus solide) et encore d'autres recherches sur des termes techniques trop violent vu l'état de mon cerveux que je vais aller reposer après une clope.
Avec tout ceci j'ai eu une idée pour la future reconstruction de mon infra mais qui au final n'est pas viable :'( en même temps un cluster avec uniquement 2 machines dont une sur la connexion Internet d'un particulier et un openvpn pour le supporter pas besoin d'être un génie pour savoir que ça commence déjà très bancale. 
Donc après réflexion de mon cerveau à peine allumé (la fatigue c'est très frustrant) je me suis dit que j'allais juste refaire sur papier mon plan d'adressage catastrophique (à force de faire de la virtu dans tous les sens) et y rajouter de l'IPv6, redéfinir mes plages DHcP, une verification et un nettoyage de mes deux pfsense, dégager toutes les VMs qui sont chez chez moi (sauf le pfsense bien-sûr) et formater mon pc portable de gestion qui tourne h24 donc ce qui revient à dégager toute mon infra Windows Server (malheureusement je vais devoir la remonter parce que je risque d'en avoir besoin pour des TPs ou autre) et je pense que ça devrait être bon. 
Comme ça je me range sans me laisser aller à mes plusions de nettoyage de printemps à coup de formatage intensif.
Pour les pfsense il suffit que je dégage une carte réseau et sa config qui devait servir pour une tentative d'OpenStack disuadé pour les compétences, le temps et surtout les ressources, ensuite peut-être dégager un des deux OpenVPNs  et nettoyer mon exeptions firewall et refaire mes règles NAT.
Et en fait un peu en premier lui il fautt que je vois comment je fais mon dns, si je fais tout sur Windows Server par rapport à l'AD (c'est un problème qui revient dans beaucoup de mes réflexions)ou si je fais un Master sur un serveur, un Slave sur l'autre et que le DNS de l'AD l'utilise en redirection ou encore si les trois se répliques (ça par contre je me suis pas renseigné sur la possibilité de la chose).
Sinon une autre des possibilités et de faire du full Linux d'un coté avec pfSense pour la réseau et full Windows de l'autre mais la gestion du NAT sur Windows Server 2016 je me souviens que je me suis arraché les cheveux dessus donc à voir si sur 2019 et avec un peu de motive ça fonctionne.
Un autre des problèmes c'est que je suis un peu impulsif là dessus (les restes de mon ancien état psychologique) et que pour mon Labo Infra j'ai une présentation de mon Zabbix tout pété à faire qui justement tourne sur une VM Hyper-V donc sur la partie Windows, je suis assez raisonnable pour ne rien faire ce soir mais je sais très bien que demain je dois m'en tenir à de la réflexion...
Sachant qu'il y a une faible possibilité que tu lises ceci je te souhaite une bonne nuit (4h28) -->