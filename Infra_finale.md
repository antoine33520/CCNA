# Infrastructure finale

Ce fichier correspond à la mise en place de l'infrastructure finale pour le cours de B1.\
Les bases de l'infrastructure seront celles du [TP 6](TP_6.md) et la configuration ne sera pas entièrement détaillée à nouveau.

## GNS3

Après une bonne configuration personnalisée de gns3 on commence l'ajout et le placement du matériel.\
Ici les routeurs cisco __C3640__ sont remplacés par des __C7200__, derrière il y a des VMs __CentOS 7__ avec __OpenVSwitch__, des Switchs Cisco __vIOS__ et deux containers Docker __gns3-endhost__ pour effectuer les tests qui ne seront connectés qu'en cas de besoin.

## Topologie IP

