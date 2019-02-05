# TP4

## I. Mise en place du lab

### 2. Création des VMs

La désactivation de "SELinux", l'installation des paquets réseau et la désactivation de la carte "NAT" sont déjà décrites dans le sujet.
Les cartes réseau réliants des VMS ne permettant pas de communication avec le PC client utilisé pour le TP la carte "NAT" du le VM routeur restera active pour permettre une connexion SSH aux trois machines.

Définition des IPs statique:

- Pour la carte réseau de la VM routeur le réseau 1

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
hostname router1.tp4
```

ou de façon permanente

```bash
echo 'router1.tp4' | tee /etc/hostname
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
> _édit_: Bon méthode un peu plus propre un fichier "/etc/systemd/system/tp.service" qui renvoit sur un fichier /root/tp.sh
>
> ```bash
> sysctl -w net.ipv4.conf.all.forwarding=1
> sysctl -w net.ipv4.conf.eth0.forwarding=0
> ```
>
> Puis
>
> ```bash
> systemctl enable tp
> ```
>
> _édit2_: bon mon idée avait pour moi une certaine logique mais ne fonctionne pas donc j'ai trouvé une autre solution bien plus propre (BAC +5 recherhce Google)
> Ajouter `net.ipv4.ip_forward = 1` au fichier /etc/sysctl.conf
> Cette fois ça fonctionne !
>
> _édit3_: En fait avant la méthode de l'édit2 j'étais parti sur un script service en utilisant systemctl mais j'ai pas réussi et je viens juste de comprendre pourquoi, en pensant à faire un script pour l'automatisation de l'installation de docker et kubernetes sur centos je me suis dit que j'avais dû oublier quelque chose et c'était le `#!/bin/bash` à mon avis. Je viens d'ouvrir vscode en deux secondes juste pour écrire ça mais je ferai le script kube plus tard si j'ai un peu de temps et on pourra en parler.

Ensuite on désactive le pare-feu

```bash
systemctl disable firewalld && systemctl stop firewalld
```

Vérification des routes

```bash
[root@router1 ~]# ip route show
default via 192.168.20.1 dev eth0 proto dhcp metric 100
10.1.0.0/24 dev eth1 proto kernel scope link src 10.1.0.254 metric 101
10.2.0.0/24 dev eth2 proto kernel scope link src 10.2.0.254 metric 102
```

Maintenant il faut définir les routes pour le client et le serveur:

- La vesrion temporaire a déjà été vu dans le précédent avec ip route add
- Pour une version permanentre de la commande il faut éditer le fichier `/etc/sysconfig/network-scripts/route-<LOCAL_INTERFACE_NAME>` et ajouter `<NETWORK_ADDRESS> via <IP_GATEWAY> dev <LOCAL_INTERFACE_NAME>`

Maintenant les VMs se ping.

Résultats des `traceroute`

- client1:

```bash
[root@client1 ~]# traceroute 10.2.0.10
traceroute to 10.2.0.10 (10.2.0.10), 30 hops max, 60 byte packets
 1  router1 (10.1.0.254)  0.179 ms  0.147 ms  0.125 ms
 2  server1 (10.2.0.10)  0.540 ms !X  0.511 ms !X  0.497 ms !X
```

- server1:

```bash
[root@server1 ~]# traceroute 10.1.0.10
traceroute to 10.1.0.10 (10.1.0.10), 30 hops max, 60 byte packets
 1  router1 (10.2.0.254)  0.213 ms  0.170 ms  0.105 ms
 2  client1 (10.1.0.10)  0.304 ms !X  0.260 ms !X  0.190 ms !X
```

## II. Spéléologie réseau

### 1. ARP

> Pour cette partie j'ai eu de gros problèmes surement dû à mes cartes réseau qui étaient Open vSwitch et non des cartes KVM standard, je pouvais pas trop me permettre de redémarrer le serveur mais bon j'ai changé le type de carte puis `apt update && apt dist-upgrade` et un petit `reboot` et visiblement problème résolu. Je ne sais pas exactement d'où vient le problème mais si j'ai le temps la piste des cartes OVS est a creusé un peu plus que ce que j'ai fait mais en effet dans certains cas en fonction du paramétrage elles peuvent faire automatiquement un filtrage de certains paquets.

#### A .Manip 1

##### Après un `ip neigh flush all`

Pour le client1:

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 REACHABLE
```

Pour le server1:

```bash
[root@server1 ~]# ip neigh show
10.2.0.254 dev eth1 lladdr 56:35:7f:db:96:6a DELAY
```

Ces lignes correspondent à l'adresse IP et MAC de la Gateway, comme il s'agit de la passerelle par défaut la table arp garde toujours cette adresse à jour.

##### Après un ping

> Léo concrètement j'ai un gros problème dont je cherche la source, j'ai pensé que c'était mes cartes réseaux comme je te l'ai dit mais visible ça ne vient pas de là parce que même après le changement de carte toujours le même problème, pour le `ping` pas de problème mais l'arp le passe pas même en désactivant le pare-feu de proxmox. J'ai donc tenté avec la commande `arping` et dans mon désespoire j'ai aussi tenté avec une machine en écoute et l'autre en demande mais toujours rien sur l'autre vm dans la table, `arping` et je précise aussi que `arping` passe aussi du client1 ou server1 vers la carte du routeur dans le réseau opposé. Le routeur lui a bien le client1 et le server1 dans sa table \
>
> _édit_: Bon c'est résolu mais j'ai pas trop compris comment mais l'arp passe après 3 jours a chercher une solution \
>
> _édit 2_: au moment où j'ai fait le premier édit j'étais heureux mais en fait ça fonctionnait... Donc après avoir fouillé un peu (Internet m'a pas vraiment aidé) j'ai modifier à la main des variables dans `sysctl` et donc après du `sysctl -w "net.ipv4.conf.all.proxy_arp"=1` et du `sysctl -w "net.ipv4.conf.all.arp_accept"=1`

Client1:

```bash
[root@server1 ~]# ping -c 2 client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.388 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=0.701 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.388/0.544/0.701/0.158 ms
```

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 DELAY
10.2.0.10 dev eth1 lladdr 6e:5d:c1:82:8a:05 STALE
```

Server1

```bash
[root@server1 ~]# ping -c 2 client1
PING client1 (10.1.0.10) 56(84) bytes of data.
64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.388 ms
64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=0.701 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.388/0.544/0.701/0.158 ms
```

```bash
[root@server1 ~]# ip neigh show
10.1.0.10 dev eth1 lladdr 56:35:7f:db:96:6a STALE
10.2.0.254 dev eth1 lladdr 56:35:7f:db:96:6a DELAY
```

Lors du ping la machine expéditrice a besoin de l'adresse mac de la machine destinataire, cette première machine envoie une requête en broadcast pour savoir quelle est l'adresse MAC du la machine ayant l'ip demandée.

_Info_: _Il ne faut pas prendr en compte la carte eth0 sur le routeur, il s'agit de la carte NAT qui est acctivée pour l'accès SSH_

#### B. Manip 2

```bash
ip neigh flush all
```

La table ARP du routeur juste après le flush.

```bash
[root@router1 ~]# ip neigh show all
192.168.20.1 dev eth0 lladdr c2:59:6b:bf:53:4e REACHABLE
```

Ensuite après le ping du client1 vers server1

```bash
[root@router1 ~]# ip neigh show all
192.168.20.1 dev eth0 lladdr c2:59:6b:bf:53:4e REACHABLE
10.1.0.10 dev eth1 lladdr 42:c1:ee:c4:c5:ab REACHABLE
10.2.0.10 dev eth2 lladdr fe:02:46:4f:31:bd REACHABLE
```

Le router1 fait un pont entre les deux réseaux donc le client1 a dû passer par le router1 pour envoyé son ping puis de même pour le retour. Le router1 a donc fait des demandes arp pour savoir quelles machines avaient les IPs concernées.

#### C. Manip 3

On flush les 3 machines.

La table ARP

```bash
root@sys:~# ip neigh show
91.121.118.254 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
91.121.118.250 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
192.168.20.1 dev vmbr1 lladdr c2:59:6b:bf:53:4e STALE
fe80::2ff:ffff:feff:fffe dev vmbr0 lladdr 00:ff:ff:ff:ff:fe router REACHABLE
fe80::418:1 dev vmbr0 lladdr 00:00:5e:00:02:01 router STALE
fe80::2ff:ffff:feff:fffd dev vmbr0 lladdr 00:ff:ff:ff:ff:fd router REACHABLE
```

Après avoir vidé la table :

```bash
91.121.118.254 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
192.168.20.12 dev vmbr1 lladdr f6:9b:f9:f3:e3:ff DELAY
91.121.118.250 dev vmbr0 lladdr 00:00:5e:00:01:01 REACHABLE
fe80::2ff:ffff:feff:fffe dev vmbr0 lladdr 00:ff:ff:ff:ff:fe router STALE
fe80::418:1 dev vmbr0 lladdr 00:00:5e:00:02:01 router STALE
fe80::2ff:ffff:feff:fffd dev vmbr0 lladdr 00:ff:ff:ff:ff:fd router STALE
```

Le résultat est sensiblement le même sachant que l'ip "192.168.20.1" est la Gateway
pour toutes les VMs Ayant accès au NAT. Et 192.168.20.12 est l'IP de la carte NAT du router1 que j'ai utilisé pour la connexion SSH au Promox.
91.121.118.254 est l'IP de Gateway fournie par So You Sart (OVH) et je ne sais pas à quoi correspond la 91.121.118.250, surement une redondance.
A cause mes problèmes ARP l'hyperviseur lui même n'a plus d'IP sur les cartes utilisées pour le TP et n'a donc plus accès qu'au réseau utilisé en local pour les autres VM et les cartes NAT, "192.168.20.0/24".

#### D. Manip 4

La table ARP:

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 DELAY
```

```bash
[root@client1 ~]# ifup eth0
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/2)
```

```bash
[root@client1 ~]# ip neigh show
10.1.0.254 dev eth1 lladdr 6e:5d:c1:82:8a:05 REACHABLE
192.168.20.1 dev eth0 lladdr c2:59:6b:bf:53:4e REACHABLE
```

Maintenant nous avons l'adresse IP et MAC de la passerelle utilisée pour l'accès au Internet (un petit PfSense).

### 2. Wireshark

#### A. Interception d'ARP et `ping`

Lien vers le fichier pccap pour le
[ping](./TP4/ping.pcap)

On voit bien qu'ici notre client1 ne communique avec le protocol ARP (niveau 2) qu'avec le router1 et pas directement avec le server1.
Ceci est dû au fonctionnement du protocol arp, sauf dans certains précis l'ARP est envoyé en boradcast or le client1 et le server1 ne sont pas sur le même réseau donc n'ont pas la même adresse de broadcast et au final pas d'arp entre les deux machines. Pour régler celà la communication passe par le router qui lui est sur les deux réseaux.

#### B. Interception d'une communication `netcat`

Firewall:

```bash
[root@server1 ~]# firewall-cmd --add-port=8888/tcp --permanent
success
```

Table ARP:

```bash
[root@router1 ~]# ip neigh flush all
```

> Bon Léo là pour nc j'ai un
>
> ```bash
> [root@client1 ~]# nc -p 8888 10.2.0.10
> Ncat: No route to host.
> ```
>
> Je ne comprends pas d'où ça vient alors que mes routes sont bonnes, ma tables arp ne contient que la Gateway et `arping` passe bien cette fois.
> On verra tout à l'heure si c'est quelque chose que je fais mal ou si ça restera un mystère

#### C. Interception d'un trafic HTTP (BONUS)

##### Installation du serveur WEB

> Tant que j'ai un peu de motive et que mon mal de crâne reste n'empire pas je vais comment l'installation du GUI et de nginx

Server1:

```bash
ifup eth0

yum install -y nginx

firewall-cmd --add-port=80/tcp
firewall-cmd --reload

systemctl enable nginx
systemctl start nginx

ifdown eth0
```

server1:

```bash
[root@server1 ~] systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since mar. 2019-02-05 11:07:28 CET; 2min 45s ago
  Process: 4528 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 4525 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 4524 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 4530 (nginx)
   CGroup: /system.slice/nginx.service
           ├─4530 nginx: master process /usr/sbin/nginx
           └─4531 nginx: worker process

févr. 05 11:07:28 server1.tp4 systemd[1]: Starting The nginx HTTP and reverse proxy server...
févr. 05 11:07:28 server1.tp4 nginx[4525]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
févr. 05 11:07:28 server1.tp4 nginx[4525]: nginx: configuration file /etc/nginx/nginx.conf test is successful
févr. 05 11:07:28 server1.tp4 systemd[1]: Failed to read PID from file /run/nginx.pid: Invalid argument
févr. 05 11:07:28 server1.tp4 systemd[1]: Started The nginx HTTP and reverse proxy server.
```

```bash
[root@server1 ~]# curl localhost:80
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }
            h1 {
                text-align: center;
                margin: 0;
                padding: 0.6em 2em 0.4em;
                background-color: #294172;
                color: #fff;
                font-weight: normal;
                font-size: 1.75em;
                border-bottom: 2px solid #000;
            }
            h1 strong {
                font-weight: bold;
                font-size: 1.5em;
            }
            h2 {
                text-align: center;
                background-color: #3C6EB4;
                font-size: 1.1em;
                font-weight: bold;
                color: #fff;
                margin: 0;
                padding: 0.5em;
                border-bottom: 2px solid #294172;
            }
            hr {
                display: none;
            }
            .content {
                padding: 1em 5em;
            }
            .alert {
                border: 2px solid #000;
            }

            img {
                border: 2px solid #fff;
                padding: 2px;
                margin: 2px;
            }
            a:hover img {
                border: 2px solid #294172;
            }
            .logos {
                margin: 1em;
                text-align: center;
            }
            /*]]>*/
        </style>
    </head>

    <body>
        <h1>Welcome to <strong>nginx</strong> on Fedora!</h1>

        <div class="content">
            <p>This page is used to test the proper operation of the
            <strong>nginx</strong> HTTP server after it has been
            installed. If you can read this page, it means that the
            web server installed at this site is working
            properly.</p>

            <div class="alert">
                <h2>Website Administrator</h2>
                <div class="content">
                    <p>This is the default <tt>index.html</tt> page that
                    is distributed with <strong>nginx</strong> on
                    Fedora.  It is located in
                    <tt>/usr/share/nginx/html</tt>.</p>

                    <p>You should now put your content in a location of
                    your choice and edit the <tt>root</tt> configuration
                    directive in the <strong>nginx</strong>
                    configuration file
                    <tt>/etc/nginx/nginx.conf</tt>.</p>

                </div>
            </div>

            <div class="logos">
                <a href="http://nginx.net/"><img
                    src="nginx-logo.png"
                    alt="[ Powered by nginx ]"
                    width="121" height="32" /></a>

                <a href="http://fedoraproject.org/"><img
                    src="poweredby.png"
                    alt="[ Powered by Fedora ]"
                    width="88" height="31" /></a>
            </div>
        </div>
    </body>
</html>
```

Pour router1 -> server1 toujours une erreur sur les routes, il y a visiblement que le `ping` et le `arping` qui passent correctement.

##### Installation du gui

```bash
ifup eth0

yum groupinstall "X Window system"

yum groupinstall xfce

systemctl isolate graphical.target
systemctl set-default graphical.target

reboot
```
