# Mission 3 — Configuration DNS

## Serveur DNS Master (ns1)

### Création du conteneur

```bash
lxc-copy -n template -N ns1
```

Configuration réseau dans `/etc/network/interfaces` :

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
 address 10.31.64.53/20
 gateway 10.31.79.254
 dns-nameservers 8.8.8.8
```

### Installation de BIND9

```bash
apt install bind9 bind9utils dnsutils
```

### Configuration BIND9

**`/etc/bind/named.conf.local`** :

```
zone "m2l.org" IN {
 type master;
 file "/etc/bind/db.m2l.org";
 allow-transfer { localhost; 10.31.64.54; };
 notify yes;
};
```

> `10.31.64.54` = adresse de ns2 (serveur esclave)

**`/etc/bind/named.conf.options`** :

```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    forwarders { 8.8.8.8; 8.8.4.4; };
    forward only;
    dnssec-validation no;
    listen-on-v6 { any; };
};
```

**`/etc/bind/db.m2l.org`** :

```
$TTL 604800
@ IN SOA m2l.org. root.m2l.org. (
        2015122601 ; serial
        604800     ; Refresh
        86400      ; Retry
        2419200    ; Expire
        604800 )   ; Negative Cache TTL

; DNS Servers
@ IN A  10.31.64.53
@ IN NS ns1.m2l.org.
@ IN NS ns2.m2l.org.
ns1 IN A 10.31.64.53
ns2 IN A 10.31.64.54

; Machines
www IN A 10.31.64.80

; Aliases
console IN CNAME www
ftp     IN CNAME www
```

### Redémarrage et vérification

```bash
systemctl restart bind9
systemctl status bind9
```

### Configuration du conteneur web

Dans le conteneur **web**, modifier `/etc/resolv.conf` :

```
# Avant :
nameserver 8.8.8.8

# Après :
nameserver 10.31.64.53
```

Tester la résolution :

```bash
apt install dnsutils
dig a www.m2l.org
```

---

## Serveur DNS Slave (ns2)

### Création du conteneur

```bash
lxc-copy -n ns1 -N ns2
```

### Configuration BIND9

**`/etc/bind/named.conf.local`** :

```
zone "m2l.org" IN {
 type slave;
 file "/var/lib/bind/db.m2l.org";
 masters { 10.31.64.53; };
};
```

**`/etc/bind/named.conf.options`** :

```
options {
   directory "/var/cache/bind";
   recursion yes;
   forwarders { 8.8.8.8; 8.8.4.4; };
   forward only;
   dnssec-validation no;
   allow-query { any; };
};
```

> ⚠️ Il n'y a **pas** de fichier de zone `/etc/bind/db.m2l.org` sur ns2. Le supprimer s'il existe.

### Configuration de tous les conteneurs

Dans `/etc/resolv.conf` des **4 conteneurs** (web1, web2, ns1, ns2) :

```
nameserver 10.31.64.53
nameserver 10.31.64.54
```
