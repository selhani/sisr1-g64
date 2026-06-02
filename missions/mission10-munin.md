# Mission 10 — Supervision avec Munin

La **supervision** informatique permet de surveiller l'état et la disponibilité du système d'information (réseau, serveurs, applications, services...). Elle permet de détecter rapidement les anomalies et garantir le bon fonctionnement des services.

**Munin** est une solution flexible pour créer des graphiques de supervision à travers le réseau, avec une installation et configuration simple.

## Conteneurs Munin

| Conteneur | Adresse IP |
|-----------|------------|
| munin1 | `10.31.64.49` |
| munin2 | `10.31.64.50` |

---

## Configuration du serveur Munin

### Installation

```bash
apt-get install munin munin-node munin-plugins-extra
```

### Fichier `/etc/munin/munin.conf`

```bash
nano /etc/munin/munin.conf
```

Décommenter ces lignes :

```
dbdir   /var/lib/munin
htmldir /var/cache/munin/www
logdir /var/log/munin
rundir  /var/run/munin
includedir /etc/munin/munin-conf.d
```

Ajouter les nœuds à surveiller à la fin du fichier :

```
[munin2.m2l.org]
    address 127.0.0.1
    use_node_name yes
[backup2.m2l.org]
    address 10.31.64.71
    use_node_name yes
[dhcp2.m2l.org]
    address 10.31.64.68
    use_node_name yes
[ftp2.m2l.org]
    address 10.31.64.21
    use_node_name yes
[nginx2.m2l.org]
    address 10.31.64.91
    use_node_name yes
[ns5.m2l.org]
    address 10.31.64.57
    use_node_name yes
[ns6.m2l.org]
    address 10.31.64.58
    use_node_name yes
[web2.m2l.org]
    address 10.31.64.81
    use_node_name yes
```

### Fichier `/etc/munin/munin-node.conf`

```bash
nano /etc/munin/munin-node.conf
```

```
host_name munin2.m2l.org
allow ^127\.0\.0\.1$
allow ^::1$
host *
```

### Démarrage et vérification

```bash
systemctl restart munin-node
netstat -nat | grep 4949    # vérifier que le port 4949 est en écoute
```

---

## Configuration des clients

Sur **tous les conteneurs** à surveiller (web, dns, ftp, backup, dhcp, nginx) :

```bash
apt-get install munin-node munin-plugins-extra
```

Configurer `/etc/munin/munin-node.conf` sur chaque client (adapter le nom du conteneur et l'adresse du serveur munin) :

```
host_name nginx2.m2l.org
allow ^10\.31\.64\.50$
allow ^::1$
```

```bash
systemctl restart munin-node
```

> **PS :** `munin-node` doit être installé sur **tous** les équipements à surveiller.

---

## Interface web — VirtualHost Nginx

Sur munin1 et munin2, créer un VirtualHost pour accéder à l'interface web :

```bash
nano /etc/nginx/sites-available/munin.m2l.org.conf
```

```nginx
server {
    listen 80;
    server_name munin.m2l.org;
    root /var/cache/munin/www;
    index index.html;
    access_log /var/log/nginx/munin-access.log;
    error_log /var/log/nginx/munin-error.log;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Activer le site :

```bash
ln -s /etc/nginx/sites-available/munin.m2l.org.conf /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

---

## Test manuel

```bash
su - munin --shell=/bin/bash
/usr/bin/munin-cron
```

Vérification des logs en cas de problème :

```bash
tail -f /var/log/munin/munin-node.log
```
