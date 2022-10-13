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

```
[root@localhost ~]# firewall-cmd --get-active-zone
public
  interfaces: enp0s8
[root@localhost ~]# firewall-cmd --add-masquerade --zone=public --permanent
success
```
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

**[Capture ARP-PING Marcel](./tp3_capture_marcel.pcapng)**
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
DIG : 
```
; <<>> DiG 9.16.23-RH <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45231
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;gitlab.com.                    IN      A

;; ANSWER SECTION:
gitlab.com.             14      IN      A       172.65.251.78

;; Query time: 23 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Oct 13 14:22:17 CEST 2022
;; MSG SIZE  rcvd: 55
```

🌞**Analyse de trames**

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |
|-------|------------|--------------------|-------------------------|----------------|-----------------|
| 1     | ping       | `john` `10.3.1.12` | `john` `08:00:27:7a:2d:04` | `8.8.8.8`      | `routeur` `08:00:27:9b:8e:87` |
| 2     | pong       | `8.8.8.8` | `routeur` `08:00:27:9b:8e:87`  | `john` `10.3.1.12` |`john` `08:00:27:7a:2d:04` |

**[Capture PING](./tp3_routage_internet.pcapng)**

## III. DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").
Install dhclient on John with :

sudo dnf install dhcp-server -y

Next, open the DHCP server configuration file with the command:

sudo nano /etc/dhcp/dhcpd.conf

Add this in file

```default-lease-time 900;
max-lease-time 10800;

authoritative;

subnet 10.3.1.0 netmask 255.255.255.0 {
range 10.3.1.2 10.3.1.253;
option routers 10.3.1.254;
option subnet-mask 255.255.255.0;
option domain-name-servers 1.1.1.1;
}
```
Configur firewall

sudo firewall-cmd --add-port=67/udp --permanent

Now, reload the firewall to apply the new rule with:

sudo firewall-cmd --reload

Start DHCP server

sudo systemctl enable --now

🌞**Améliorer la configuration du DHCP**


```[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:78:be:3e brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 765sec preferred_lft 765sec
```
```
[root@localhost ~]# ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.818 ms
64 bytes from 10.3.1.254: icmp_seq=2 ttl=64 time=0.667 ms
64 bytes from 10.3.1.254: icmp_seq=3 ttl=64 time=0.514 ms
64 bytes from 10.3.1.254: icmp_seq=4 ttl=64 time=0.631 ms
```
```
[root@localhost ~]# ip r s
default via 10.3.1.254 dev enp0s8 proto dhcp src 10.3.1.2 metric 100
```

```
[root@localhost ~]# ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.12 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.889 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.833 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.09 ms
```

```
[root@localhost ~]# dig gitlab.com

; <<>> DiG 9.16.23-RH <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33526
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;gitlab.com.                    IN      A

;; ANSWER SECTION:
gitlab.com.             144     IN      A       172.65.251.78

;; Query time: 35 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Oct 13 15:47:40 CEST 2022
;; MSG SIZE  rcvd: 55
```
```
[root@localhost ~]# ping gitlab.com
PING gitlab.com (172.65.251.78) 56(84) bytes of data.
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=1 ttl=53 time=26.3 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=2 ttl=53 time=33.6 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=3 ttl=53 time=23.8 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=4 ttl=53 time=38.3 ms
```
### 2. Analyse de trames

🌞**Analyse de trames**

**[Capture DHCP](./tp3_capture_dhcp.pcapng)**
