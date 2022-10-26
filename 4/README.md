# TP4 : TCP, UDP et services réseau

# I. First steps
🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

Discord :

  ip dst : 162.159.135.232  
  port dst : 443   
  port source : 60040

**[Capture TCP DISCORD](./tcp_discord.pcapng)**

```bash
PS C:\Users\guill> netstat
 TCP    10.33.19.152:60040     162.159.135.232:443    ESTABLISHED

```

Valorant :

  ip dst :  192.249.72.1  
  port dst : 7287   
  port source : 49713 

**[Capture UDP VALORANT](./tcp_valorant.pcapng)**

```bash
PS C:\Users\guill> netstat -n -b -p UDP -a
  UDP    0.0.0.0:57504          *:*    
 [System]
```

Youtube :

  ip dst : 216.58.198.163 
  port dst : 61632 
  port source : 443

**[Capture TCP YOUTUBE](./tcp_Youtube.pcapng)**

```bash
PS C:\Users\guill> netstat -n -b

  TCP    10.33.19.152:61632     216.58.198.193:443     ESTABLISHED
  [opera.exe]
```

Steam :

  ip dst : 91.68.245.140 
  port dst : 61809 
  port source : 443

**[Capture TCP STEAM](./tcp_steam.pcapng)**

```bash
PS C:\Users\guill> netstat -n -b
  TCP    10.33.19.152:61809     91.68.245.140:443      ESTABLISHED
 [Steam.exe]
```

Twitch :

  ip dst : 52.12.78.24 
  port dst : 61932 
  port source : 443  

**[Capture TCP TWITCH](./tcp_twitch.pcapng)**

```bash
PS C:\Users\guill> netstat -n -b
  TCP    10.33.19.152:61932     52.12.78.24:443        ESTABLISHED
 [opera.exe]
```

# II. Mise en place

## 1. SSH

🌞 **Examinez le trafic dans Wireshark**

**[Capture TCP SSH](./tcp_ssh.pcapng)**

🌞 **Demandez aux OS**
Linux :

```bash
tcp             ESTAB            0                0                                                            10.4.1.11:ssh                                          10.4.1.1:55688
```

Windows :

```bash
  Proto  Adresse locale         Adresse distante       État
  TCP    10.4.1.1:55688       10.4.1.11:22           ESTABLISHED
 [ssh.exe]
```
# III. DNS

## 1. Présentation

🌞 **Dans le rendu, je veux**
```bash
[gwuill@localhost ~]$ sudo cat /etc/named.conf
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "tp4.b1" IN {
        type master;
        file "tp4.b1.db";
        allow-update { none; };
        allow-query { any; };
};

zone "1.4.10.in-addr.arpa" IN {
        type master;
        file "tp4.b1.rev";
        allow-update { none; };
        allow-query { any; };
};

[gwuill@localhost ~]$ sudo cat /var/named/tp4.b1.db

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns-server IN A 10.4.1.201
node1      IN A 10.4.1.11

[gwuill@localhost ~]$ sudo cat /var/named/tp4.b1.rev

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

;Reverse lookup for Name Server
201 IN PTR dns-server.tp4.b1.
11 IN PTR node1.tp4.b1.
```

- un `systemctl status named` qui prouve que le service tourne bien
```bash
[gwuill@localhost ~]$ sudo systemctl restart named
[gwuill@localhost ~]$ sudo systemctl status named.service
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2022-10-25 16:31:13 CEST; 3s ago
    Process: 2585 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-check>
    Process: 2587 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 2588 (named)
      Tasks: 4 (limit: 5908)
     Memory: 13.8M
        CPU: 34ms
     CGroup: /system.slice/named.service
             └─2588 /usr/sbin/named -u named -c /etc/named.conf

Oct 25 16:31:13 localhost.localdomain named[2588]: configuring command channel from '/etc/rndc.key'
Oct 25 16:31:13 localhost.localdomain named[2588]: command channel listening on 127.0.0.1#953
Oct 25 16:31:13 localhost.localdomain named[2588]: configuring command channel from '/etc/rndc.key'
Oct 25 16:31:13 localhost.localdomain named[2588]: command channel listening on ::1#953
Oct 25 16:31:13 localhost.localdomain named[2588]: managed-keys-zone: loaded serial 0
Oct 25 16:31:13 localhost.localdomain named[2588]: zone 1.4.10.in-addr.arpa/IN: loaded serial 2019061800
Oct 25 16:31:13 localhost.localdomain named[2588]: zone tp4.b1/IN: loaded serial 2019061800
Oct 25 16:31:13 localhost.localdomain named[2588]: all zones loaded
Oct 25 16:31:13 localhost.localdomain systemd[1]: Started Berkeley Internet Name Domain (DNS).
Oct 25 16:31:13 localhost.localdomain named[2588]: running
```
- une commande `ss` qui prouve que le service écoute bien sur un port

```bash
[gwuill@localhost ~]$ ss -t -l -n
LISTEN    0    10    10.4.1.201:53     0.0.0.0:*
```

🌞 **Ouvrez le bon port dans le firewall**
```
[gwuill@localhost ~]$ sudo firewall-cmd --add-port=53/tcp --permanent
success
[gwuill@localhost ~]$ sudo firewall-cmd --reload
success
[gwuill@localhost ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 53/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
## 3. Test

🌞 **Sur la machine `node1.tp4.b1`**

```bash
[gwuill@localhost ~]$ dig node1.tp4.b1 @10.4.1.201

; <<>> DiG 9.16.23-RH <<>> node1.tp4.b1 @10.4.1.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13912
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: d2e5463b02b8cab20100000063599b5be682f5bcdc9be3b3 (good)
;; QUESTION SECTION:
;node1.tp4.b1.                  IN      A

;; ANSWER SECTION:
node1.tp4.b1.           86400   IN      A       10.4.1.11

;; Query time: 8 msec
;; SERVER: 10.4.1.201#53(10.4.1.201)
;; WHEN: Wed Oct 26 22:40:59 CEST 2022
;; MSG SIZE  rcvd: 85

[gwuill@localhost ~]$ dig dns-server.tp4.b1 @10.4.1.201

; <<>> DiG 9.16.23-RH <<>> dns-server.tp4.b1 @10.4.1.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24117
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 76093a20eca138740100000063599b73d9fe31b545b81f97 (good)
;; QUESTION SECTION:
;dns-server.tp4.b1.             IN      A

;; ANSWER SECTION:
dns-server.tp4.b1.      86400   IN      A       10.4.1.201

;; Query time: 6 msec
;; SERVER: 10.4.1.201#53(10.4.1.201)
;; WHEN: Wed Oct 26 22:41:24 CEST 2022
;; MSG SIZE  rcvd: 90

[gwuill@localhost ~]$ dig www.google.com @10.4.1.201

; <<>> DiG 9.16.23-RH <<>> www.google.com @10.4.1.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 32490
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 45b497cd9c8df15d0100000063599b96d71ca89bbcd196ef (good)
;; QUESTION SECTION:
;www.google.com.                        IN      A

;; Query time: 22 msec
;; SERVER: 10.4.1.201#53(10.4.1.201)
;; WHEN: Wed Oct 26 22:41:59 CEST 2022
;; MSG SIZE  rcvd: 71
```
🌞 **Sur votre PC**

- utilisez une commande pour résoudre le nom `node1.tp4.b1` en utilisant `10.4.1.201` comme serveur DNS

> Le fait que votre serveur DNS puisse résoudre un nom comme `www.google.com`, ça s'appelle la récursivité et c'est activé avec la ligne `recursion yes;` dans le fichier de conf.

🦈 **Capture d'une requête DNS vers le nom `node1.tp4.b1` ainsi que la réponse**
