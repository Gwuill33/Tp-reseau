**Ce tp à étais résaliser en colaboration avec guillaume Branche**

# Sujet Réseau et Infra

On va utiliser GNS3 dans ce TP pour se rapprocher d'un cas réel. On va focus sur l'aspect routing/switching, avec du matériel Cisco. On va aussi mettre en place des VLANs.

![best memes from cisco doc](./pics/the-best-memes-come-from-cisco-documentation.jpg)

# I. Dumb switch

🌞 **Commençons simple**

définissez les IPs statiques sur les deux VPCS :
```
PC1> ip 10.5.1.1/24
```

```
PC2> ip 10.5.1.2/24
```

`ping` un VPCS depuis l'autre :
```
PC2> ping 10.5.1.1

84 bytes from 10.5.1.1 icmp_seq=1 ttl=64 time=0.253 ms
84 bytes from 10.5.1.1 icmp_seq=2 ttl=64 time=0.395 ms
84 bytes from 10.5.1.1 icmp_seq=3 ttl=64 time=0.325 ms
```


# II. VLAN

🌞 **Adressage**

définissez les IPs statiques sur tous les VPCS :
```
PC1> ip 10.5.10.1/24
```

```
PC2> ip 10.5.10.2/24
```

```
PC3> ip 10.5.10.3/24
```

vérifiez avec des `ping` que tout le monde se ping :
```
PC2> ping 10.5.10.3

84 bytes from 10.5.10.3 icmp_seq=1 ttl=64 time=0.178 ms
84 bytes from 10.5.10.3 icmp_seq=2 ttl=64 time=0.314 ms
84 bytes from 10.5.10.3 icmp_seq=3 ttl=64 time=0.381 ms

PC2> ping 10.5.10.1

84 bytes from 10.5.10.1 icmp_seq=1 ttl=64 time=0.104 ms
84 bytes from 10.5.10.1 icmp_seq=2 ttl=64 time=0.332 ms
84 bytes from 10.5.10.1 icmp_seq=3 ttl=64 time=0.364 ms
```

🌞 **Configuration des VLANs**

```
IOU1# conf t
IOU1(config)# vlan 10
IOU1(config-vlan)# name admins
IOU1(config-vlan)# exit
IOU1(config)#
IOU1(config)# vlan 20
IOU1(config-vlan)# name guests
IOU1(config-vlan)# exit
IOU1(config)# exit
IOU1# show vlan

10   admins                           active
20   guests                           active

IOU1(config)# interface Ethernet0/0
IOU1(config-if)# switchport mode access
IOU1(config-if)# switchport access vlan 10
IOU1(config-if)# exit
IOU1(config)# interface Ethernet0/1
IOU1(config-if)# switchport mode access
IOU1(config-if)# switchport access vlan 10
IOU1(config-if)# exit
IOU1(config)# interface Ethernet0/2
IOU1(config-if)# switchport mode access
IOU1(config-if)# switchport access vlan 20
IOU1(config-if)# exit
IOU1(config)# exit
IOU1# show vlan br

10   admins                           active    Et0/0 Et0/1
20   guests                           active    Et0/2

```

🌞 **Vérif**

- `pc1` et `pc2` doivent toujours pouvoir se ping
```
PC1> ping 10.5.10.2

84 bytes from 10.5.10.2 icmp_seq=1 ttl=64 time=0.109 ms
84 bytes from 10.5.10.2 icmp_seq=2 ttl=64 time=0.228 ms

```
- `pc3` ne ping plus personne

```
PC3> ping 10.5.10.1

host (10.5.10.1) not reachable
```

# III. Routing

🌞 **Adressage**

définissez les IPs statiques sur toutes les machines :

```
PC1> ip 10.5.10.1/24
Checking for duplicate address...
PC1 : 10.5.10.1 255.255.255.0
```
```
PC2> ip 10.5.10.2/24
Checking for duplicate address...
PC2 : 10.5.10.2 255.255.255.0

```
```
PC3> ip 10.5.20.1/24
Checking for duplicate address...
PC3 : 10.5.20.1 255.255.255.0
```
```
[dreams@web ~]$ ip a

inet 10.5.30.1/24
```
🌞 **Configuration des VLANs**


```
IOU1#show vlan br

10   client                           active    Et0/0, Et0/1
20   admin                            active    Et0/2
30   server                           active    Et0/3

iOU1#show interface trunk 

Port        Mode             Encapsulation  Status        Native vlan
Et3/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Et3/3       1-4094

Port        Vlans allowed and active in management domain
Et3/3       1,10,20,30

Port        Vlans in spanning tree forwarding state and not pruned
Et3/3       none
```

🌞 **Config du *routeur***

```
R1#show ip int br 

FastEthernet0/0.10         10.5.10.254     YES manual administratively down down          
FastEthernet0/0.20         10.5.20.254     YES manual administratively down down          
FastEthernet0/0.30         10.5.30.254     YES manual administratively down down 
```

🌞 **Vérif**

tout le monde doit pouvoir ping le routeur sur l'IP qui est dans son réseau :
```
PC1> ping 10.5.10.254

10.5.10.254 icmp_seq=1 timeout
84 bytes from 10.5.10.254 icmp_seq=2 ttl=255 time=0.980 ms
84 bytes from 10.5.10.254 icmp_seq=3 ttl=255 time=6.025 ms
84 bytes from 10.5.10.254 icmp_seq=4 ttl=255 time=10.588 ms
84 bytes from 10.5.10.254 icmp_seq=5 ttl=255 time=2.075 ms

```
en ajoutant une route vers les réseaux, ils peuvent se ping entre eux :
```
PC3> ip 10.5.20.1/24 10.5.20.254

PC3 : 10.5.20.1 255.255.255.0 gateway 10.5.20.254

PC3> ping 10.5.10.1

84 bytes from 10.5.10.1 icmp_seq=1 ttl=63 time=30.273 ms
84 bytes from 10.5.10.1 icmp_seq=2 ttl=63 time=20.427 ms
84 bytes from 10.5.10.1 icmp_seq=3 ttl=63 time=19.430 ms

```

# IV. NAT

🌞 **Ajoutez le noeud Cloud à la topo**


côté routeur, il faudra récupérer un IP en DHCP:
```
FastEthernet0/1            10.0.3.16       YES DHCP   up                    up  
```
vous devriez pouvoir `ping 1.1.1.1`:
```
PC1> ping 1.1.1.1

1.1.1.1 icmp_seq=1 timeout
84 bytes from 1.1.1.1 icmp_seq=2 ttl=62 time=15.080 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=62 time=19.056 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=62 time=17.643 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=62 time=20.577 ms

```

🌞 **Configurez le NAT**

```
R1#conf t
R1(config)#int fastE0/0
R1(config-if)#ip nat inside
R1(config-if)#exit
R1(config)#int fastE0/1
R1(config-if)#ip nat outside
R1(config)#access-list 1 permit any
R1(config)#ip nat inside source list 1 int fastE0/1 overload
R1(config)#exit
```

🌞 **Test**

- ajoutez une route par défaut (si c'est pas déjà fait)
  - sur les VPCS
  - sur la machine Linux
- configurez l'utilisation d'un DNS
  - sur les VPCS :
  ```
  ip dns 8.8.8.8
  ```
  - sur la machine Linux :
  ```
  sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

  DNS=8.8.8.8
  ```
vérifiez un `ping` vers un nom de domaine :
```
PC1> ping google.com
google.com resolved to 216.58.214.78

84 bytes from 216.58.214.78 icmp_seq=1 ttl=112 time=31.441 ms
84 bytes from 216.58.214.78 icmp_seq=2 ttl=112 time=23.632 ms
84 bytes from 216.58.214.78 icmp_seq=3 ttl=112 time=32.040 ms
84 bytes from 216.58.214.78 icmp_seq=4 ttl=112 time=30.090 ms
84 bytes from 216.58.214.78 icmp_seq=5 ttl=112 time=23.963 ms

```

# V. Add a building

On a acheté un nouveau bâtiment, faut tirer et configurer un nouveau switch jusque là-bas.

On va en profiter pour setup un serveur DHCP pour les clients qui s'y trouvent.

## 1. Topologie 5

![Topo 5](./pics/topo5.png)

## 2. Adressage topologie 5

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
|-----------|----------------|--------------|
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node                | `clients`        | `admins`         | `servers`        |
|---------------------|------------------|------------------|------------------|
| `pc1.clients.tp5`   | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`   | `10.5.10.2/24`   | x                | x                |
| `pc3.clients.tp5`   | DHCP             | x                | x                |
| `pc4.clients.tp5`   | DHCP             | x                | x                |
| `pc5.clients.tp5`   | DHCP             | x                | x                |
| `dhcp1.clients.tp5` | `10.5.10.253/24` | x                | x                |
| `adm1.admins.tp5`   | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5`  | x                | x                | `10.5.30.1/24`   |
| `r1`                | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 3. Setup topologie 5

Vous pouvez partir de la topologie 4. 

🌞  **Vous devez me rendre le `show running-config` de tous les équipements**

- de tous les équipements réseau
  - le routeur
  - les 3 switches

> N'oubliez pas les VLANs sur tous les switches.

🖥️ **VM `dhcp1.client1.tp5`**, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞  **Mettre en place un serveur DHCP dans le nouveau bâtiment**

- il doit distribuer des IPs aux clients dans le réseau `clients` qui sont branchés au même switch que lui
- sans aucune action manuelle, les clients doivent...
  - avoir une IP dans le réseau `clients`
  - avoir un accès au réseau `servers`
  - avoir un accès WAN
  - avoir de la résolution DNS

> Réutiliser les serveurs DHCP qu'on a monté dans les autres TPs.

🌞  **Vérification**

- un client récupère une IP en DHCP
- il peut ping le serveur Web
- il peut ping `8.8.8.8`
- il peut ping `google.com`

> Faites ça sur n'importe quel VPCS que vous venez d'ajouter : `pc3` ou `pc4` ou `pc5`.

![i know cisco](./pics/i_know.jpeg)