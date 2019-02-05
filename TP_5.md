# TP5

## I. Préparation du lab

### 1. Préparation VMs

On fait correspondre la machines à la configuration demandée en paramétrant une nouvelle carte "host-only" permettant la connexion SSH.
Ensuite on clone le patron pour avoir 3 VMs et on les joute dans gns3 en précisant qu'il y a 2 interfaces réseau.

### 2. Préparation Routeurs Cisco

On importe l'image qui va servire pour mettre en place les 2 routeurs.
Le réseau a utilisé pour les routeurs peut être 10.5.3.0/30 pour utilisé les ip 10.5.12.1 et 10.5.12.2 pour les deux routeurs.

Récapitulation des IPs:

| Machines       |    net1    |    net2    |   net3   |
|----------------|:----------:|:----------:|:--------:|
| client1.tp5.b1 |      X     |  10.5.2.10 |     X    |
| client2.tp5.b1 |      X     |  10.5.2.11 |     X    |
| router1.tp5.b1 | 10.5.1.254 |      X     | 10.5.12.1 |
| router2.tp5.b1 |      X     | 10.5.2.254 | 10.5.12.2 |
| server1.tp5.b1 |  10.5.1.10 |      X     |     X    |

