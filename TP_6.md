# TP6

## Tableau des Informations IP et OSPF

### Réseaux IP et aires OSPF

Réseaux | `area 0` | `area 1` | `area 2` | Commentaire
--- | --- | --- | --- | ---
`10.6.100.0/30` | X | - | - | Liaison entre `r1` et `r2`
`10.6.100.4/30` | X | - | - | Liaison entre `r1` et `r4`
`10.6.100.8/30` | X | - | - | Liaison entre `r2` et `r3`
`10.6.100.12/30` | X | - | - | Liaison entre `r3` et `r4`
`10.6.101.0/30` | - | X | - | Liaison entre `r3` et `r5`
`10.6.201.0/24` | - | X | - | Réseau des clients
`10.6.202.0/24` | - | - | X | Réseau des serveurs

### Adressage IP de chacune des machines

Machines | `10.6.100.0/30` | `10.6.100.4/30` | `10.6.100.8/30` | `10.6.100.12/30` | `10.6.101.0/30` | `10.6.201.0/24` | `10.6.202.0/24`
--- | --- | --- | --- | --- | --- | --- | ---
`r1.tp6.b1` | `10.6.100.1` | `10.6.100.5` | - | - | - | - | `10.6.202.254`
`r2.tp6.b1` | `10.6.100.2` | - |  `10.6.100.9` | - | - | - | -
`r3.tp6.b1` | - | - | `10.6.100.10` | `10.6.100.14` | `10.6.101.1` | - | -
`r4.tp6.b1` | - |  `10.6.100.6` | - | `10.6.100.13` | - | - | -
`r5.tp6.b1` | - | - | - | - |  `10.6.101.2` |  `10.6.201.254` | -
`client1.tp6.b1` | - | - | - | - | - |  `10.6.201.10` | -
`client2.tp6.b1` | - | - | - | - | - |  `10.6.201.11` | -
`server1.tp6.b1` | - | - | - | - | - | - | `10.6.202.10`

<!-- ## __Informations__

- Ce TP a entièrement été réalisé sur [EVE-NG Community](https://www.eve-ng.net/) et la connexion `telnet` pour l'accès console aux routeurs et aux machines sont faites avec `"EVE-NG Intergration (Linux client side)"` utilisant le protocol `telnet` mais permetant aussi les protocols `vnc` et `rdp`.
- La virtualisation des trois VMs Centos a été faite avec `QEMU` intégré à `EVE-NG`, (par défaut la version 2.4.0).
- EVE-NG a été installé via son image iso sur une machine virtuelle hébergée `Proxmox`.
- Le routeur servant pour le réseau NAT et un pfsense hébergé également sur le `Proxmox`.
- Le serveur DNS est sur un réseau différent que celui attribué par le serveur DHCP mais le routage est bien éffectué entre les deux réseau via un tunnel IPsec.
- Topology du réseau sur EVE-NG [ici](./TP6/topology_eve.png). -->