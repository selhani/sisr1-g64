# Mission 5 — Serveurs FTP

## Partie 1 — Installation de base

### Installation de ProFTPd

```bash
apt-get install proftpd
systemctl status proftpd
```

### Création des utilisateurs

```bash
useradd abdelillah
useradd shirine
```

| Utilisateur | Adresse IP |
|-------------|------------|
| abdelillah | `10.31.64.20` |
| shirine | `10.31.64.21` |

### Activation du chroot

Dans `/etc/proftpd/proftpd.conf`, décommenter :

```
Defaultroot ~
```

```bash
systemctl restart proftpd
```

### Installation du client FTP

```bash
apt-get install ftp
```

**Test de connexion :**

```bash
ftp 10.31.64.1
# Login : abdelillah ou shirine
# Mot de passe : motdepasse
```

**Vérification du port d'écoute (port 21) :**

```bash
grep "Port" /etc/proftpd/proftpd.conf
ss -tlnp | grep proftpd
```

---

## Partie 2 — Accès anonyme

### Créer le répertoire anonyme

```bash
mkdir -p /home/ftpdocs
echo "Fichier de test anonyme" > /home/ftpdocs/test.txt
```

### Modification de proftpd.conf

Dans `/etc/proftpd/proftpd.conf`, décommenter le bloc :

```
<Anonymous /home/ftpdocs>
  User ftp
  Group nogroup
  UserAlias anonymous ftp
  RequireValidShell off
  MaxClients 10
  DisplayLogin welcome.msg
  DisplayChdir .message
  <Directory *>
    <Limit WRITE>
      DenyAll
    </Limit>
  </Directory>
</Anonymous>
```

```bash
systemctl restart proftpd
```

### Test de connexion anonyme

```bash
ftp 10.31.64.1
```

| Commande | Résultat attendu |
|----------|-----------------|
| `get test.txt` | ✅ Fonctionne |
| `put test.txt` | ❌ Doit être refusé |

---

## Partie 3 — VirtualHosts

### Ajouter les interfaces virtuelles dans `/etc/network/interfaces`

```
auto eth0:0
iface eth0:0 inet static
  address 10.31.64.22
  netmask 255.255.240.0

auto eth0:1
iface eth0:1 inet static
  address 10.31.64.23
  netmask 255.255.240.0
```

Vérification :

```bash
ip a | grep 10.31.64
```

### Créer les utilisateurs et répertoires

```bash
useradd -m intra && passwd intra
useradd -m extra && passwd extra
mkdir -p /srv/ftp/intranet
mkdir -p /srv/ftp/extranet
```

### Configuration dans `/etc/proftpd/virtuals.conf`

```
<VirtualHost 10.31.64.22>
  ServerName "FTP INTRANET"
  Port 2100
  <Limit LOGIN>
    Order Allow,Deny
    AllowGroup intra
    DenyAll
  </Limit>
  DefaultRoot /srv/ftp/intranet
  AllowOverwrite yes
</VirtualHost>

<VirtualHost 10.31.64.23>
  ServerName "FTP EXTRANET"
  Port 2200
  <Limit LOGIN>
    Order Allow,Deny
    AllowGroup extra
    DenyAll
  </Limit>
  DefaultRoot /srv/ftp/extranet
  AllowOverwrite no
</VirtualHost>
```

Dans `/etc/proftpd/proftpd.conf`, décommenter :

```
Include /etc/proftpd/virtuals.conf
```

```bash
systemctl restart proftpd
ss -tlnp | grep proftpd
```

---

## Partie 4 — Analyseur de trame

### Capture avec tcpdump

```bash
apt-get install tcpdump
```

**Terminal 1 — lancer la capture :**

```bash
tcpdump -i lo -A port 21
```

**Terminal 2 — se connecter en FTP :**

```bash
ftp 10.31.64.20
```

> ⚠️ **Les informations circulent en clair** (login, mot de passe...) et sont visibles dans la capture tcpdump.
