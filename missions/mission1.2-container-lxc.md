# Mission 1.2 — Container LXC

## Installation LXC

### 1. Installation et mise à jour

**Mise à jour du serveur** (dans le host) :

```bash
apt-get update
apt-get upgrade
```

**Installation de LXC :**

```bash
apt-get install lxc lxc-templates
```

### 2. Vérification de la configuration noyau

```bash
lxc-checkconfig
```

Vérifie le support des cgroups, namespaces, etc.

### 3. Mise en place du bridge réseau

**Installation de bridge-utils :**

```bash
apt-get install bridge-utils
```

**Création du pont br0 :**

```bash
brctl addbr br0
```

**Configuration permanente dans `/etc/rc.local` :**

```bash
#!/bin/sh -e
brctl addbr br0
ifconfig eno1 0.0.0.0
ifconfig br0 10.31.64.1/20
route add default gw 10.31.79.254
brctl addif br0 eno1
```

> Le pont **br0** devient l'interface principale du serveur.

**Vérifications :**

```bash
brctl show   # affiche les bridges réseau
ifconfig     # affiche la configuration réseau
```

### 4. Configuration par défaut des conteneurs

Éditer `/etc/lxc/default.conf` :

```
lxc.net.0.type = veth
lxc.net.0.link = br0
lxc.net.0.flags = up
lxc.net.0.name = eth0
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```

> Tous les conteneurs utiliseront automatiquement **br0**.

---

## Création du template LXC

### 1. Créer le conteneur template

```bash
lxc-create -n template -t debian -- -r bookworm
```

Crée un conteneur Debian Bookworm nommé `template`.

### 2. Commandes LXC essentielles

```bash
lxc-start template    # démarrer
lxc-info template     # afficher les infos
lxc-attach template   # accéder au conteneur
lxc-stop template     # arrêter
```

### 3. Configuration réseau temporaire (dans le conteneur)

```bash
ifconfig eth0 10.31.64.2/16 up
route add default gw 10.31.0.254
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

### 4. Installation des outils de base

```bash
apt update
apt upgrade
apt install sudo net-tools tcpdump nano iputils-ping dbus
```

### 5. Configuration de la timezone

```bash
ln -fs /usr/share/zoneinfo/Europe/Paris /etc/localtime
dpkg-reconfigure -f noninteractive tzdata
```

### 6. Création d'un utilisateur sudo (optionnel)

```bash
adduser std
usermod -aG sudo std
```

### 7. Ajout des clés SSH

Ajouter les clés dans `/root/.ssh/authorized_keys`.

> Tous les conteneurs clonés hériteront de ces clés.

### 8. Désactivation du template

```bash
lxc-stop template
```

---

## Clonage et configuration du conteneur web

### 1. Cloner le template

```bash
lxc-copy -n template -N web
```

### 2. Démarrage et accès

```bash
lxc-start web
lxc-attach web
```

### 3. Configuration réseau permanente

Éditer `/etc/network/interfaces` :

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
 address 10.31.64.80/20
 gateway 10.31.79.254
 dns-nameservers 8.8.8.8
```

**Redémarrer le conteneur :**

```bash
lxc-stop web
lxc-start web
lxc-info web    # vérification
```

### 4. Installation d'Apache

```bash
apt install apache2
nano /var/www/html/index.html
```

Contenu de test :

```html
<h1>Welcome to the web container !</h1>
```

### 5. Test

Depuis un navigateur : `http://10.31.64.80/`
