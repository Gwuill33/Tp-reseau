# TP3 : On va router des trucs

### 1. Echange ARP

🌞**Générer des requêtes ARP**

ping Marcel to John :

```
[root@localhost ~]# ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.866 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.621 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.671 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=64 time=0.611 ms
^C
--- 10.3.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3038ms
rtt min/avg/max/mdev = 0.611/0.692/0.866/0.102 ms
```
John :

```
[root@localhost ~]# ip neigh show dev enp0s8
10.3.1.12 lladdr 08:00:27:dd:38:9b REACHABLE <- Adresse MAC de Marcel
```

Marcel :

```
[root@localhost ~]# ip neigh show dev enp0s8
10.3.1.11 lladdr 08:00:27:7a:2d:04 STALE <- Adresse MAC de John
```
Verif Adresse MAC 
John : 
```
[root@localhost ~]# ip a
    link/ether 08:00:27:7a:2d:04 
```

Marcel : 
```
[root@localhost ~]# ip a
    link/ether 08:00:27:dd:38:9b
```
### 2. Analyse de trames

🌞**Analyse de trames**

**[Capture ARP](./tp3_arp.pcapng)**
## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

> Je les appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

> Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme du matériel Cisco.

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

john : 
```
[root@localhost ~]# ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.781 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.794 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.833 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=0.712 ms
```

marcel :
```
[root@localhost ~]# ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=0.560 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=0.537 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=0.508 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=63 time=1.08 ms
```

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

marcel :
```
[root@localhost ~]# ip n s
10.3.2.254 dev enp0s8 lladdr 08:00:27:c4:92:cf REACHABLE <- Echange ARP entre marcel et le routeur
```

john :
```
[root@localhost ~]# ip n s
10.3.1.254 dev enp0s8 lladdr 08:00:27:9b:8e:87 REACHABLE <- Echange ARP entre john et le routeur
```

routeur
```
[root@localhost ~]# ip n s
10.3.2.12 dev enp0s3 lladdr 08:00:27:dd:38:9b STALE
10.3.1.11 dev enp0s8 lladdr 08:00:27:7a:2d:04 STALE
```

- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | 10.3.1.11         |`john` `08:00:27:7a:2d:04`| 10.3.1.254  | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | 10.3.1.254        | `routeur` `08:00:27:9b:8e:87`  | 10.3.1.11             | `john` `08:00:27:7a:2d:04`  |
| 3     | Ping         | 10.3.1.11       | `john` `08:00:27:7a:2d:04`  |   10.3.2.12              |`routeur` `08:00:27:9b:8e:87` |
| 4     | ARP        | 10.3.2.254      | `routeur` `08:00:27:9b:8e:87`  |        x        |   Broadcast `FF:FF:FF:FF:FF`  |
| 5     | ARP         | 10.3.2.12      |`marcel` `08:00:27:dd:38d9b`| 10.3.2.254           |   `routeur` `08:00:27:c4:92:cf` |
| 6     | Ping        | 10.3.2.254      |`routeur` `08:00:27:c4:92:cf`| 10.3.2.12  | `marcel` `08:00:27:dd:38d9b` |
| 7     | Pong        | 10.3.2.12      |`marcel` `08:00:27:dd:38d9b`| 10.3.2.254           |   `routeur` `08:00:27:c4:92:cf` |
| 8     | Pong        | 10.3.2.12      |`marcel` `08:00:27:dd:38d9b`| 10.3.1.11       | `john` `08:00:27:7a:2d:04`  |

**[Capture ARP-PING Marcel](./tp3_capteure_marcel.pcapng)**
**[Capture ARP-PING Routeur](./tp3_capture_routeur.pcapng)**

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

Avec IP :
```
[root@localhost ~]# ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=23.8 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=24.6 ms
```
Avec Nom de domaine :
```
[root@localhost ~]# ping google.com
PING google.com (216.58.198.206) 56(84) bytes of data.
64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=1 ttl=113 time=23.6 ms
64 bytes from par10s27-in-f14.1e100.net (216.58.198.206): icmp_seq=2 ttl=113 time=22.1 ms
```


🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |
|-------|------------|--------------------|-------------------------|----------------|-----------------|
| 1     | ping       | `john` `10.3.1.12` | `john` `08:00:27:7a:2d:04` | `8.8.8.8`      | `routeur` `08:00:27:9b:8e:87` |
| 2     | pong       | `8.8.8.8` | `routeur` `08:00:27:9b:8e:87`  | `john` `10.3.1.12` |`john` `08:00:27:7a:2d:04` |

**[Capture PING](./tp3_routage_internet.pcapng)**

## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`              | `10.3.2.0/24` |
|----------|----------------------------|---------------|
| `router` | `10.3.1.254`               | `10.3.2.254`  |
| `john`   | `10.3.1.11`                | no            |
| `bob`    | oui mais pas d'IP statique | no            |
| `marcel` | no                         | `10.3.2.12`   |

```schema
   john               router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   john        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur

> Il est possible d'utilise la commande `dhclient` pour forcer à la main, depuis la ligne de commande, la demande d'une IP en DHCP, ou renouveler complètement l'échange DHCP (voir `dhclient -h` puis call me et/ou Google si besoin d'aide).

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser
- récupérez de nouveau une IP en DHCP sur `bob` pour tester :
  - `marcel` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP
    - vérifier qu'il peut `ping` sa passerelle
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
    - vérifier que la route fonctionne avec un `ping` vers une IP
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
    - vérifier un `ping` vers un nom de domaine

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`

🦈 **Capture réseau `tp2_dhcp.pcapng`**
