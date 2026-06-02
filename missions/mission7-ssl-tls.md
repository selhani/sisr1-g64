# Mission 7 — SSL / TLS

## HTTPS sur Apache

### Installation d'OpenSSL

```bash
apt-get install openssl
```

Sur les deux conteneurs web (`10.31.64.80` et `10.31.64.81`).

### Génération du certificat

```bash
mkdir /etc/ssl/localcerts
DIR=/etc/ssl/localcerts
openssl req -x509 -newkey rsa:4096 -nodes \
  -keyout $DIR/mydomainkey.key \
  -out $DIR/mydomaincert.pem \
  -days 365
```

### Activation du module SSL et du VirtualHost

```bash
a2enmod ssl
a2ensite default-ssl
```

### Configuration du VirtualHost SSL

```bash
nano /etc/apache2/sites-available/default-ssl.conf
```

Modifier les directives :

```bash
SSLCertificateFile      /etc/ssl/localcerts/m2l.org.pem
SSLCertificateKeyFile   /etc/ssl/localcerts/m2l.org.key
```

### Renommer les fichiers

```bash
cd /etc/ssl/localcerts/
mv mydomaincert.pem m2l.org.pem
mv mydomainkey.key m2l.org.key
```

### Redémarrage et vérification

```bash
systemctl restart apache2
netstat -tlnp | grep 443
```

Tester dans le navigateur : `https://10.31.64.80/` ou `https://10.31.64.81/`

> Accepter le risque pour afficher la page (certificat auto-signé).

---

## Configuration SSL/TLS pour tous les VirtualHosts

Aller dans le répertoire des sites :

```bash
cd /etc/apache2/sites-available/
```

Pour **chaque site** (www, intranet, extranet, wiki) :

```bash
nano www.m2l.org.conf
```

Copier le bloc `VirtualHost *:80`, le coller en dessous en changeant le port en **443**, et ajouter :

```apache
SSLEngine on
SSLCertificateFile    /etc/ssl/localcerts/m2l.org.pem
SSLCertificateKeyFile /etc/ssl/localcerts/m2l.org.key
```

---

## FTPS (FTP sécurisé)

### Générer le certificat pour ProFTPd

```bash
mkdir /etc/proftpd/ssl/
DIR=/etc/proftpd/ssl/
openssl req -x509 -newkey rsa:4096 -nodes \
  -keyout $DIR/mydomainkey.key \
  -out $DIR/mydomaincert.pem \
  -days 365
```

### Activer TLS dans proftpd.conf

```bash
nano /etc/proftpd/proftpd.conf
```

Décommenter la ligne pour inclure `tls.conf`.

### Configurer tls.conf

```bash
nano /etc/proftpd/tls.conf
```

Décommenter toutes les lignes et modifier les chemins vers les certificats.

### Renommer les certificats

```bash
cd /etc/proftpd/ssl/
mv mydomaincert.pem m2l.org.pem
mv mydomainkey.key m2l.org.key
systemctl restart proftpd
```
