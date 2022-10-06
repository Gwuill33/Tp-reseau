# TP2 : Ethernet, IP, et ARP

# I. Setup IP

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

>192.168.137.1    
11000000 10101000 10001001 00000001   
255.255.252.0   
11111111 11111111 11111100 00000000   
Adresse réseau : 11000000 10101000 10001000 00000000   
192.168.136.0   
Adresse broadcast : 11000000 10101000 10001011 11111111   
192.168.139.255   
Nombres adresse dispo : 2^8 - 2 = 1024 -2 = 1022

    PS C:\Users\guill> New-NetIPAddress –InterfaceIndex 10 –IPAddress 192.168.137.2 –PrefixLength 22

    IPAddress         : 192.168.137.2
    InterfaceIndex    : 10
    InterfaceAlias    : Ethernet
    AddressFamily     : IPv4
    Type              : Unicast
    PrefixLength      : 22
    PrefixOrigin      : Manual
    SuffixOrigin      : Manual
    AddressState      : Tentative
    ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
    PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
    SkipAsSource      : False
    PolicyStore       : ActiveStore

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

```
PS C:\Users\guill> ping 192.168.137.2

Envoi d’une requête 'Ping'  192.168.137.2 avec 32 octets de données :
Réponse de 192.168.137.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.137.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
```

🌞 **Wireshark it**

**[Capture ping](./ping.pcapng)**

  - Type : 0 - echo reply
  - Type : 8 - echo request

# II. ARP my bro

🌞 **Check the ARP table**

```
PS C:\Users\guill> arp -a

  192.168.137.2         e4-e7-49-42-6b-42     dynamique
  192.168.139.255       ff-ff-ff-ff-ff-ff     statique
```
🌞 **Manipuler la table ARP**

```
PS C:\Users\guill> netsh interface IP delete arpcache
Ok.

PS C:\Users\guill> arp -d
```
Avant ping : 
```
PS C:\Users\guill> arp -a

Interface : 192.168.137.1 --- 0xa
  Adresse Internet      Adresse physique      Type
  224.0.0.7             01-00-5e-00-00-07     statique
  224.0.0.22            01-00-5e-00-00-16     statique
```

Après ping :
```
PS C:\Users\guill> arp -a

Interface : 192.168.137.1 --- 0xa
  Adresse Internet      Adresse physique      Type
  192.168.137.2         e4-e7-49-42-6b-42     dynamique
  224.0.0.7             01-00-5e-00-00-07     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.1.60            01-00-5e-00-01-3c     statique
  224.0.1.187           01-00-5e-00-01-bb     statique
```
🌞 **Wireshark it**

**[Capture arp](./arp.pcapng)**

Première trame (ARP broadcast) :  
  Sender IP address: 192.168.137.2  Poste client 
  Target Ip address : 192.168.137.1  Poste server

Deuxième trame (ARP reply) :  
  Sender IP address: 192.168.137.1  Poste server
  Target IP address: 192.168.137.2  Poste client

# III. DHCP you too my brooo

🌞 **Wireshark it**

**[Capture DHCP](./DHCP.pcapng)**

>D : Src: 0.0.0.0, Dst: 255.255.255.255  
O : Src: 10.33.19.254, Dst: 255.255.255.255   
R : Src: 0.0.0.0, Dst: 255.255.255.255   
A : Src: 10.33.19.254, Dst: 255.255.255.255
 
 
1) 
> Your (client) IP address: 10.33.16.231

2)
>Source Address: 10.33.19.254

3)
>Option: (6) Domain Name Server   
  Length: 12    
  Domain Name Server: 8.8.8.8   
  Domain Name Server: 8.8.4.4   
  Domain Name Server: 1.1.1.1

