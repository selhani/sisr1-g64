# Intermission — Migration Apache → Nginx

Nginx est un serveur web plus léger et très performant, utilisé par plus de 30% des serveurs web mondiaux. Il est apprécié pour sa faible consommation mémoire et sa capacité à gérer des milliers de connexions simultanées.

## Conteneurs Nginx

| Conteneur | Adresse IP |
|-----------|------------|
| nginx1 | `10.31.64.90` |
| nginx2 | `10.31.64.91` |

---

## Partie 1 — Installation

```bash
apt-get install -y nginx
systemctl status nginx
```

---

## Partie 2 — Structure et VirtualHosts

### Vérifier les dossiers

```bash
ls /etc/nginx/    # vérifier que sites-available et sites-enabled existent
```

### Créer les dossiers web et donner les droits

```bash
mkdir -p /home/htdocs/m2l.org/{www,intranet,extranet,wiki}
chown -R www-data:www-data /home/htdocs/
chmod -R 755 /home/htdocs/
```

### Vérifier nginx.conf

S'assurer que la ligne suivante est bien présente dans `nginx.conf` :

```
include /etc/nginx/sites-enabled/*;
```

### Configuration `/etc/nginx/sites-available/www.m2l.org.conf`

```nginx
server {
    listen 80;
    server_name www.m2l.org m2l.org;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name www.m2l.org m2l.org;
    root /home/htdocs/m2l.org/www;
    index index.html index.htm;
    ssl_certificate /etc/ssl/nginx/m2l.org.pem;
    ssl_certificate_key /etc/ssl/nginx/m2l.org.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/www.m2l.org-access.log;
    error_log /var/log/nginx/www.m2l.org-error.log;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

> Les configurations de `wiki.m2l.org` et `extranet.m2l.org` sont identiques, il suffit de remplacer les noms de domaine.

### Configuration `/etc/nginx/sites-available/intranet.m2l.org.conf`

```nginx
server {
    listen 80;
    server_name intranet.m2l.org;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name intranet.m2l.org;
    root /home/htdocs/m2l.org/intranet;
    index index.html index.htm;
    ssl_certificate /etc/ssl/nginx/m2l.org.pem;
    ssl_certificate_key /etc/ssl/nginx/m2l.org.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/intranet.m2l.org-access.log;
    error_log /var/log/nginx/intranet.m2l.org-error.log;
    location / {
        auth_basic "Accès réservé – Intranet m2l";
        auth_basic_user_file /etc/nginx/auth/.htpasswd;
        try_files $uri $uri/ =404;
    }
}
```

> Le bloc `listen 80` avec `return 301` redirige automatiquement HTTP → HTTPS.

---

## Partie 3 — SSL/TLS

```bash
apt install -y openssl
mkdir -p /etc/ssl/nginx/

openssl req -x509 -newkey rsa:4096 -nodes \
    -keyout /etc/ssl/nginx/m2l.org.key \
    -out /etc/ssl/nginx/m2l.org.pem \
    -days 365
```

Informations à renseigner :

| Champ | Valeur |
|-------|--------|
| Country Name | `FR` |
| State | `Nouvelle-Aquitaine` |
| Locality | `Limoges` |
| Organization | `m2l` |
| Organizational Unit | `SIO` |
| Common Name | `*.m2l.org` |
| Email | `abdelillah@m2l.org` |

### Sécuriser la clé privée

```bash
chmod 600 /etc/ssl/nginx/m2l.org.key
chown root:root /etc/ssl/nginx/m2l.org.key
```

### Vérification

```bash
ls /etc/ssl/nginx/    # doit afficher m2l.org.key et m2l.org.pem
```

---

## Partie 4 — Authentification HTTP pour l'intranet

```bash
apt install -y apache2-utils
mkdir -p /etc/nginx/auth/
htpasswd -c /etc/nginx/auth/.htpasswd abdelillah
```

---

## Partie 5 — Activation et test

### Supprimer le site par défaut

```bash
rm /etc/nginx/sites-enabled/default
```

### Activer les 4 sites

```bash
ln -s /etc/nginx/sites-available/www.m2l.org.conf /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/intranet.m2l.org.conf /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/extranet.m2l.org.conf /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/wiki.m2l.org.conf /etc/nginx/sites-enabled/
```

### Pages de test

```bash
echo '<h1>www.m2l.org</h1>'      > /home/htdocs/m2l.org/www/index.html
echo '<h1>intranet.m2l.org</h1>' > /home/htdocs/m2l.org/intranet/index.html
echo '<h1>extranet.m2l.org</h1>' > /home/htdocs/m2l.org/extranet/index.html
echo '<h1>wiki.m2l.org</h1>'     > /home/htdocs/m2l.org/wiki/index.html
```

### Vérifier et recharger

```bash
nginx -t
systemctl reload nginx
```

### Vérifier les ports

```bash
netstat -natp | grep nginx
ss -tlnp | grep nginx
```

---

## Fichier hosts Windows (pour les tests)

Ajouter dans `C:\Windows\System32\drivers\etc\hosts` :

```
10.31.64.90   www.m2l.org
10.31.64.90   intranet.m2l.org
10.31.64.90   extranet.m2l.org
10.31.64.90   wiki.m2l.org
```
