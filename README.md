**🌞 Affichez les infos des cartes réseau de votre PC**

```
PS C:\Users\guill> ipconfig /all

Carte réseau sans fil Wi-Fi :

   Adresse physique . . . . . . . . . . . : 14-13-33-86-6A-1B
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.127(préféré)
```

```
Carte Ethernet Ethernet :

   Adresse physique . . . . . . . . . . . : 50-EB-F6-C4-51-A8
```

**🌞 Affichez votre gateway**
```
  PS C:\Users\guill> ipconfig /all  
   Passerelle par défaut. . . . . . . . . : 10.33.19.254

```
**🌞 Déterminer la MAC de la passerelle**
```
PS C:\Users\guill> arp /a
Interface : 10.33.16.127 --- 0x4
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

**🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

![Ip](./pics/Ip.jpg)

![MAC](./pics/MAC.jpg)


### A. Modification d'adresse IP (part 1)  

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :
J'ai changé mon adresse ip par le panneau de configuration

![ChangementIp](./pics/ChangementIp.jpg)

```
PS C:\Users\guill> ipconfig /all
Carte réseau sans fil Wi-Fi :
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.128(préféré)
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
   ```


🌞 **Il est possible que vous perdiez l'accès internet.** .

En changeant d'ip on peut tomber sur une ip déjà utilisé et par conséquent le routeur va envoyer le paquet vers la première IP.
# II. Exploration locale en duo

## 3. Modification d'adresse IP

🌞 **Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau**

![Cable](./pics/Cable.jpg)

🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**
```
PS C:\Users\guill> ipconfig /all

Carte Ethernet Ethernet :

   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.12(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . : 10.10.10.255
```

🌞 **Vérifier que les deux machines se joignent**

```
PS C:\Users\guill> ping 10.10.10.11

Envoi d’une requête 'Ping'  10.10.10.11 avec 32 octets de données :
Réponse de 10.10.10.11 : octets=32 temps<1ms TTL=128
Réponse de 10.10.10.11 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.11 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.11 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 10.10.10.11:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 1ms, Moyenne = 0ms
```


🌞 **Déterminer l'adresse MAC de votre correspondant**

```
PS C:\Users\guill> arp -a
Interface : 10.10.10.12 --- 0xa
  Adresse Internet      Adresse physique      Type
  10.10.10.11           00-e0-4c-68-00-33     dynamique
```

## 4. Utilisation d'un des deux comme gateway

🌞**Tester l'accès internet**

```
PS C:\Users\guill> ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=21 ms TTL=55
Réponse de 1.1.1.1 : octets=32 temps=21 ms TTL=55
Réponse de 1.1.1.1 : octets=32 temps=54 ms TTL=55
Réponse de 1.1.1.1 : octets=32 temps=24 ms TTL=55

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 21ms, Maximum = 54ms, Moyenne = 30ms
```
🌞 **Prouver que la connexion Internet passe bien par l'autre PC**

```
Command : tracert -4 1.1.1.1
```

```
Détermination de l’itinéraire vers one.one.one.one [1.1.1.1]
avec un maximum de 30 sauts :

  1     1 ms    <1 ms    <1 ms  Gwuill [192.168.137.1] 
  2     *        *        *     Délai d’attente de la demande dépassé.
  3     4 ms    43 ms     3 ms  10.33.19.254 
  4     9 ms     6 ms     5 ms  137.149.196.77.rev.sfr.net [77.196.149.137] 
  5    20 ms    15 ms    15 ms  108.97.30.212.rev.sfr.net [212.30.97.108] 
  6    46 ms    32 ms    34 ms  222.172.136.77.rev.sfr.net [77.136.172.222] 
  7    32 ms    41 ms    34 ms  221.172.136.77.rev.sfr.net [77.136.172.221] 
  8    28 ms    30 ms    32 ms  221.10.136.77.rev.sfr.net [77.136.10.221] 
  9    21 ms    21 ms    22 ms  221.10.136.77.rev.sfr.net [77.136.10.221] 
 10    30 ms    44 ms    25 ms  141.101.67.254 
 11    36 ms    31 ms    31 ms  172.71.128.2 
 12    22 ms    21 ms    21 ms  one.one.one.one [1.1.1.1] 

Itinéraire déterminé.
```

## 5. Petit chat privé

![Netcat](./pics/netcat.jpg)

On va créer un chat extrêmement simpliste à l'aide de `netcat` (abrégé `nc`). Il est souvent considéré comme un bon couteau-suisse quand il s'agit de faire des choses avec le réseau.

Sous GNU/Linux et MacOS vous l'avez sûrement déjà, sinon débrouillez-vous pour l'installer :). Les Windowsien, ça se passe [ici](https://eternallybored.org/misc/netcat/netcat-win32-1.11.zip) (from https://eternallybored.org/misc/netcat/).  

Une fois en possession de `netcat`, vous allez pouvoir l'utiliser en ligne de commande. Comme beaucoup de commandes sous GNU/Linux, Mac et Windows, on peut utiliser l'option `-h` (`h` pour `help`) pour avoir une aide sur comment utiliser la commande.  

Sur un Windows, ça donne un truc comme ça :

```schema
C:\Users\It4\Desktop\netcat-win32-1.11>nc.exe -h
[v1.11 NT www.vulnwatch.org/netcat/]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [options] [hostname] [port]
options:
        -d              detach from console, background mode

        -e prog         inbound program to exec [dangerous!!]
        -g gateway      source-routing hop point[s], up to 8
        -G num          source-routing pointer: 4, 8, 12, ...
        -h              this cruft
        -i secs         delay interval for lines sent, ports scanned
        -l              listen mode, for inbound connects
        -L              listen harder, re-listen on socket close
        -n              numeric-only IP addresses, no DNS
        -o file         hex dump of traffic
        -p port         local port number
        -r              randomize local and remote ports
        -s addr         local source address
        -t              answer TELNET negotiation
        -u              UDP mode
        -v              verbose [use twice to be more verbose]
        -w secs         timeout for connects and final net reads
        -z              zero-I/O mode [used for scanning]
port numbers can be individual or ranges: m-n [inclusive]
```

L'idée ici est la suivante :

- l'un de vous jouera le rôle d'un *serveur*
- l'autre sera le *client* qui se connecte au *serveur*

Précisément, on va dire à `netcat` d'*écouter sur un port*. Des ports, y'en a un nombre fixe (65536, on verra ça plus tard), et c'est juste le numéro de la porte à laquelle taper si on veut communiquer avec le serveur.

Si le serveur écoute à la porte 20000, alors le client doit demander une connexion en tapant à la porte numéro 20000, simple non ?  

Here we go :

🌞 **sur le PC *serveur*** 
```
C:\Users\guill\Desktop\netcat-1.11>nc.exe -l -p 8888
ça marche
oui
Yes !

```
🌞 **sur le PC *client*** 

```
C:\Users\natha\Desktop\netcat-win32-1.12>nc.exe 192.168.137.1 8888
ça marche
Yes !
oui
```
🌞 **Visualiser la connexion en cours**

Server : 
```
 [nc.exe]
  TCP    192.168.137.1:51396    0.0.0.0:0              LISTENING
```

Client : 
```
[nc.exe]
  TCP    192.168.137.11:63663   13.69.68.64:443        ESTABLISHED
```

- si vous faites un `netstat` sur le serveur AVANT que le client `netcat` se connecte, vous devriez observer que votre serveur `netcat` écoute sur toutes vos interfaces
  - c'est à dire qu'on peut s'y connecter depuis la wifi par exemple :D
- il est possible d'indiquer à `netcat` une interface précise sur laquelle écouter
  - par exemple, on écoute sur l'interface Ethernet, mais pas sur la WiFI

```bash
# Sur Windows/MacOS
$ nc.exe -l -p PORT_NUMBER -s IP_ADDRESS
# Par exemple
$ nc.exe -l -p 9999 -s 192.168.1.37
```

## 6. Firewall

Toujours par 2.

Le but est de configurer votre firewall plutôt que de le désactiver

🌞 **Activez et configurez votre firewall**

- autoriser les `ping`
  - configurer le firewall de votre OS pour accepter le `ping`
  - aidez vous d'internet
  - on rentrera dans l'explication dans un prochain cours mais sachez que `ping` envoie un message *ICMP de type 8* (demande d'ECHO) et reçoit un message *ICMP de type 0* (réponse d'écho) en retour
- autoriser le traffic sur le port qu'utilise `nc`
  - on parle bien d'ouverture de **port** TCP et/ou UDP
  - on ne parle **PAS** d'autoriser le programme `nc`
  - choisissez arbitrairement un port entre 1024 et 20000
  - vous utiliserez ce port pour communiquer avec `netcat` par groupe de 2 toujours
  - le firewall du *PC serveur* devra avoir un firewall activé et un `netcat` qui fonctionne
  
# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

Bon ok vous savez définir des IPs à la main. Mais pour être dans le réseau YNOV, vous l'avez jamais fait.  

C'est le **serveur DHCP** d'YNOV qui vous a donné une IP.

Une fois que le serveur DHCP vous a donné une IP, vous enregistrer un fichier appelé *bail DHCP* qui contient, entre autres :

- l'IP qu'on vous a donné
- le réseau dans lequel cette IP est valable

🌞**Exploration du DHCP, depuis votre PC**

- afficher l'adresse IP du serveur DHCP du réseau WiFi YNOV
- cette adresse a une durée de vie limitée. C'est le principe du ***bail DHCP*** (ou *DHCP lease*). Trouver la date d'expiration de votre bail DHCP
- vous pouvez vous renseigner un peu sur le fonctionnement de DHCP dans les grandes lignes. On aura un cours là dessus :)

> Chez vous, c'est votre box qui fait serveur DHCP et qui vous donne une IP quand vous le demandez.

## 2. DNS

Le protocole DNS permet la résolution de noms de domaine vers des adresses IP. Ce protocole permet d'aller sur `google.com` plutôt que de devoir connaître et utiliser l'adresse IP du serveur de Google.  

Un **serveur DNS** est un serveur à qui l'on peut poser des questions (= effectuer des requêtes) sur un nom de domaine comme `google.com`, afin d'obtenir les adresses IP liées au nom de domaine.  

Si votre navigateur fonctionne "normalement" (il vous permet d'aller sur `google.com` par exemple) alors votre ordinateur connaît forcément l'adresse d'un serveur DNS. Et quand vous naviguez sur internet, il effectue toutes les requêtes DNS à votre place, de façon automatique.

🌞** Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

🌞 Utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main

- faites un *lookup* (*lookup* = "dis moi à quelle IP se trouve tel nom de domaine")
  - pour `google.com`
  - pour `ynov.com`
  - interpréter les résultats de ces commandes
- déterminer l'adresse IP du serveur à qui vous venez d'effectuer ces requêtes
- faites un *reverse lookup* (= "dis moi si tu connais un nom de domaine pour telle IP")
  - pour l'adresse `78.73.21.21`
  - pour l'adresse `22.146.54.58`
  - interpréter les résultats
  - *si vous vous demandez, j'ai pris des adresses random :)*

# IV. Wireshark

**Wireshark est un outil qui permet de visualiser toutes les trames qui sortent et entrent d'une carte réseau.**

On appelle ça un **sniffer**, ou **analyseur de trames.**

![Wireshark](./pics/wireshark.jpg)

Il peut :

- enregistrer le trafic réseau, pour l'analyser plus tard
- afficher le trafic réseau en temps réel

**On peut TOUT voir.**

Un peu austère aux premiers abords, une manipulation très basique permet d'avoir une très bonne compréhension de ce qu'il se passe réellement.

➜ **[Téléchargez l'outil Wireshark](https://www.wireshark.org/).**

🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :

- un `ping` entre vous et votre passerelle
- un `netcat` entre vous et votre mate, branché en RJ45
- une requête DNS. Identifiez dans la capture le serveur DNS à qui vous posez la question.
- prenez moi des screens des trames en question
- on va prendre l'habitude d'utiliser Wireshark souvent dans les cours, pour visualiser ce qu'il se passe

# Bilan

**Vu pendant le TP :**

- visualisation de vos interfaces réseau (en GUI et en CLI)
- extraction des informations IP
  - adresse IP et masque
  - calcul autour de IP : adresse de réseau, etc.
- connaissances autour de/aperçu de :
  - un outil de diagnostic simple : `ping`
  - un outil de scan réseau : `nmap`
  - un outil qui permet d'établir des connexions "simples" (on y reviendra) : `netcat`
  - un outil pour faire des requêtes DNS : `nslookup` ou `dig`
  - un outil d'analyse de trafic : `wireshark`
- manipulation simple de vos firewalls

**Conclusion :**

- Pour permettre à un ordinateur d'être connecté en réseau, il lui faut **une liaison physique** (par câble ou par *WiFi*).  
- Pour réceptionner ce lien physique, l'ordinateur a besoin d'**une carte réseau**. La carte réseau porte une adresse MAC  
- **Pour être membre d'un réseau particulier, une carte réseau peut porter une adresse IP.**
Si deux ordinateurs reliés physiquement possèdent une adresse IP dans le même réseau, alors ils peuvent communiquer.  
- **Un ordintateur qui possède plusieurs cartes réseau** peut réceptionner du trafic sur l'une d'entre elles, et le balancer sur l'autre, servant ainsi de "pivot". Cet ordinateur **est appelé routeur**.
- Il existe dans la plupart des réseaux, certains équipements ayant un rôle particulier :
  - un équipement appelé *passerelle*. C'est un routeur, et il nous permet de sortir du réseau actuel, pour en joindre un autre, comme Internet par exemple
  - un équipement qui agit comme **serveur DNS** : il nous permet de connaître les IP derrière des noms de domaine
  - un équipement qui agit comme **serveur DHCP** : il donne automatiquement des IP aux clients qui rejoigne le réseau
  - **chez vous, c'est votre Box qui fait les trois :)**