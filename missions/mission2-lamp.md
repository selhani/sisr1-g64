# Mission 2 — Installation LAMP

## Apache2

### Installation

```bash
apt update
apt install apache2
```

### Vérification

```bash
systemctl status apache2
netstat -natp
```

### Port d'écoute

```bash
nano /etc/apache2/ports.conf
```

Apache écoute sur le port **80** (HTTP).

---

## SQLite3

### Installation

```bash
apt update
apt install sqlite3 php-sqlite3 libapache2-mod-php
```

### Créer le fichier info.php

```bash
nano /var/www/html/info.php
```

### Créer et utiliser une base SQLite3

```bash
sqlite3 myCDS.db
```

Pour quitter : `.quit`

### Création des tables

```sql
CREATE TABLE artist (art_id INTEGER PRIMARY KEY, art_name TEXT);

CREATE TABLE cd (
  cd_id INTEGER PRIMARY KEY,
  art_id INTEGER NOT NULL,
  cd_title TEXT NOT NULL,
  cd_date TEXT
);
```

### Insertion des données

```sql
INSERT INTO artist (art_id,art_name) VALUES (NULL,'Peter Gabriel');
INSERT INTO artist (art_id,art_name) VALUES (NULL,'Bruce Hornsby');
INSERT INTO artist (art_id,art_name) VALUES (NULL,'Lyle Lovett');
INSERT INTO artist (art_id,art_name) VALUES (NULL,'Beach Boys');

INSERT INTO cd (cd_id,art_id,cd_title,cd_date) VALUES (NULL,1,'Us','1992');
INSERT INTO cd (cd_id,art_id,cd_title,cd_date) VALUES (NULL,2,'The Way It Is','1986');
INSERT INTO cd (cd_id,art_id,cd_title,cd_date) VALUES (NULL,2,'Scenes from the Southside','1990');
INSERT INTO cd (cd_id,art_id,cd_title,cd_date) VALUES (NULL,1,'Security','1990');
INSERT INTO cd (cd_id,art_id,cd_title,cd_date) VALUES (NULL,3,'Joshua Judges Ruth','1992');
INSERT INTO cd (cd_id,art_id,cd_title,cd_date) VALUES (NULL,4,'Pet Sounds','1966');
```

### Créer le fichier mycds.php

```bash
nano /var/www/html/mycds.php
```

```php
<?php
try {
    $db = new PDO('sqlite:myCDS.db');
    print("<b>Connecté à la base</b><br /><br />");

    print("<b>Les artistes</b><br />");
    $sql = 'SELECT * FROM artist';
    $result = $db->query($sql);
    foreach ($result as $row) {
        print($row['art_id'] . ".\t" . $row['art_name'] . '<br />');
    }

    print("<br /><b>Les albums</b><br />");
    $sql = 'SELECT artist.art_name, cd.cd_title, cd.cd_date
            FROM cd
            JOIN artist ON cd.art_id = artist.art_id';
    $result = $db->query($sql);
    foreach ($result as $row) {
        print('- ' . $row['art_name'] . ' - ' . $row['cd_title'] . ' - ' . $row['cd_date'] . '<br />');
    }
    $db = null;
} catch (PDOException $e) {
    print('Exception : ' . $e->getMessage());
}
?>
```

Tester dans le navigateur : `http://10.31.64.80/mycds.php`

> **Page blanche ?** Activer l'affichage des erreurs PHP :
> ```bash
> nano /etc/php/8.2/apache2/php.ini
> # Modifier : display_errors = On
> systemctl restart apache2
> ```

---

## MariaDB

### Installation

```bash
apt update
apt install mariadb-server php-mysql
```

### Vérification

```bash
systemctl status mariadb
ss -tlnp | grep mariadb
```

MariaDB écoute sur le port **3306**.

### Sécurisation (`mysql_secure_installation`)

| Question | Réponse |
|----------|---------|
| Configurer le mot de passe root ? | OUI |
| Supprimer les utilisateurs anonymes ? | OUI |
| Interdire la connexion root à distance ? | OUI |
| Supprimer la base de test ? | OUI |
| Recharger les tables de privilèges ? | OUI |

### Connexion

```bash
mysql -u root -p
# mot de passe : drowssap
```

### Créer un compte `dba`

```sql
CREATE USER 'dba'@'localhost' IDENTIFIED BY 'drowssap';
GRANT ALL PRIVILEGES ON *.* TO 'dba'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
SHOW DATABASES;
```

---

## phpMyAdmin

### Installation des modules PHP

```bash
apt install php-json php-mbstring php-zip php-gd php-xml php-curl
systemctl restart apache2
```

### Installation de phpMyAdmin

```bash
apt update
apt install wget
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
tar xvf phpMyAdmin-latest-all-languages.tar.gz
mv phpMyAdmin-*-all-languages /var/www/html/phpMyAdmin
chown -R www-data:www-data /var/www/html/phpmyadmin
```

Connexion : `http://10.31.64.80/phpmyadmin`

| Champ | Valeur |
|-------|--------|
| Login | `dba` |
| Mot de passe | `drowssap` |

### Création de la base de données

1. Créer la base `phpcrud`
2. Onglet **Privilèges** → Ajouter un utilisateur
3. Nouvel utilisateur : `gulumser` — Hôte : `localhost`
4. Accorder tous les privilèges sur `phpcrud`

### Créer la table `users`

```sql
CREATE TABLE `users` (
 `id` int(11) NOT NULL auto_increment,
 `name` varchar(100) NOT NULL,
 `age` int(3) NOT NULL,
 `email` varchar(100) NOT NULL,
 PRIMARY KEY (`id`)
);
```

### Déployer l'application CRUD

```bash
scp crud.zip root@10.31.64.80:/var/www/html/
unzip crud.zip
mv crud phpcrud
```

Tester : `http://10.31.64.80/phpcrud/`

---

## Finalisation — Écoute sur toutes les interfaces

Par défaut MariaDB écoute sur `127.0.0.1`. Pour accepter les requêtes depuis tout le réseau :

```bash
cd /etc/mysql/mariadb.conf.d
nano 50-server.cnf
# Modifier : bind-address = 0.0.0.0
systemctl restart mariadb
netstat -natp
```

### Vérifier les utilisateurs MariaDB

```sql
SELECT user, host FROM mysql.user;
```
