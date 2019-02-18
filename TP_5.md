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
  -Routes

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
