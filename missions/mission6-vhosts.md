# Mission 6 — Serveurs web virtuels (VirtualHosts Apache)

Le serveur web héberge plusieurs sites :

- `www.m2l.org`
- `intranet.m2l.org`
- `extranet.m2l.org`
- `wiki.m2l.org`

---

## Configuration DNS (Bind9)

Dans `/etc/bind/db.m2l.org`, ajouter 3 lignes dans la section machines :

```bash
intranet IN A 10.31.64.80
extranet IN A 10.31.64.80
wiki     IN A 10.31.64.80
```

Incrémenter le numéro de série :

```
2015122601  →  2015122602
```

Relancer Bind9 sur les deux serveurs DNS :

```bash
systemctl restart bind9
```

---

## Fichiers de configuration VirtualHost

Les fichiers sont dans `/etc/apache2/sites-available/`.

### www.m2l.org

```bash
nano /etc/apache2/sites-available/www.m2l.org.conf
```

```apache
<VirtualHost *:80>
    ServerName    m2l.org
    ServerAlias   www.m2l.org
    DocumentRoot  /home/htdocs/m2l.org/www

    ErrorLog   /var/log/apache2/www-error.log
    CustomLog  /var/log/apache2/www-access.log combined

    <Directory /home/htdocs/m2l.org/www>
        Require all granted
    </Directory>
</VirtualHost>
```

### intranet.m2l.org

```bash
nano /etc/apache2/sites-available/intranet.m2l.org.conf
```

```apache
<VirtualHost *:80>
    ServerName    intranet.m2l.org
    DocumentRoot  /home/htdocs/m2l.org/intranet

    ErrorLog   /var/log/apache2/intranet-error.log
    CustomLog  /var/log/apache2/intranet-access.log combined

    <Directory /home/htdocs/m2l.org/intranet>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

> **Important :** `AllowOverride All` est indispensable pour le fonctionnement des fichiers `.htaccess`.

### extranet.m2l.org

```bash
nano /etc/apache2/sites-available/extranet.m2l.org.conf
```

```apache
<VirtualHost *:80>
    ServerName    extranet.m2l.org
    DocumentRoot  /home/htdocs/m2l.org/extranet

    ErrorLog   /var/log/apache2/extranet-error.log
    CustomLog  /var/log/apache2/extranet-access.log combined

    <Directory /home/htdocs/m2l.org/extranet>
        Require all granted
    </Directory>
</VirtualHost>
```

### wiki.m2l.org

```bash
nano /etc/apache2/sites-available/wiki.m2l.org.conf
```

```apache
<VirtualHost *:80>
    ServerName    wiki.m2l.org
    DocumentRoot  /home/htdocs/m2l.org/wiki

    ErrorLog   /var/log/apache2/wiki-error.log
    CustomLog  /var/log/apache2/wiki-access.log combined

    <Directory /home/htdocs/m2l.org/wiki>
        Require all granted
    </Directory>
</VirtualHost>
```

---

## Création des répertoires et fichiers de test

```bash
mkdir -p /home/htdocs/m2l.org/{www,intranet,extranet,wiki}
```

### Exemple — index.html

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="utf-8">
    <title>Site Web www.m2l.org</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div class="content">
      Bienvenue sur le site web www.m2l.org
    </div>
  </body>
</html>
```

### Exemple — style.css

```css
body {
    background-color: #111;
}
.content {
    width: 100%;
    text-align: center;
    color: white;
    font-size: 25px;
}
```

---

## Activation des sites

```bash
a2ensite www.m2l.org.conf
a2ensite intranet.m2l.org.conf
a2ensite extranet.m2l.org.conf
a2ensite wiki.m2l.org.conf
systemctl reload apache2
```

---

## Sécuriser l'intranet avec .htaccess

### Créer le fichier .htaccess

```bash
nano /home/htdocs/m2l.org/intranet/.htaccess
```

```apache
AuthType     Basic
AuthName     "Accès réservé"
AuthUserFile /home/htdocs/m2l.org/intranet/.htpasswd
Require      valid-user
```

### Créer le fichier .htpasswd

```bash
htpasswd -c /home/htdocs/m2l.org/intranet/.htpasswd abdelillah
htpasswd    /home/htdocs/m2l.org/intranet/.htpasswd gulumser
htpasswd    /home/htdocs/m2l.org/intranet/.htpasswd shirine
```

> ⚠️ L'option `-c` crée le fichier — ne l'utiliser que pour le **premier** utilisateur.

Accéder à `http://intranet.m2l.org` → une fenêtre d'authentification doit apparaître.

### Pages d'erreur personnalisées

Dans le `.htaccess` de chaque site :

```apache
ErrorDocument 401 /401.html
ErrorDocument 403 /403.html
ErrorDocument 404 /404.html
ErrorDocument 500 /500.html
```

Créer ensuite les fichiers `401.html`, `403.html`, `404.html`, `500.html` dans chaque répertoire.

---

## Répertoires personnels (module userdir)

```bash
a2enmod userdir
systemctl reload apache2
```

Créer les pages de test dans `/home/(utilisateur)/public_html/`.

Pour activer PHP dans les répertoires personnels :

```bash
nano /etc/apache2/mods-enabled/php8.2.conf
# Remplacer Off par On dans la section userdir
systemctl reload apache2
```
