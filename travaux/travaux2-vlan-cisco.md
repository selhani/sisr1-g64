# Travaux 2 — VLANs, Trunk et SVI (Cisco)

TP de segmentation d'un réseau d'entreprise en plusieurs VLANs sur deux switches Cisco reliés par un lien trunk **802.1q**.

## Plan d'adressage

| VLAN | Nom | Ports | Réseau |
|------|-----|-------|--------|
| 10 | etudiant | Fa0/1-8 | 192.168.10.0/24 |
| 20 | enseignant | Fa0/9-16 | 192.168.20.0/24 |
| 30 | administration | Fa0/17-23 | 192.168.30.0/24 |
| 100 | native | - | - |
| 101 | management | Fa0/24 | 192.168.101.0/24 |

**Adresses SVI (administration à distance) :**

- SW1 : `192.168.101.254`
- SW2 : `192.168.101.253`

Le lien trunk entre les deux switches passe par **Gig0/1** sur chaque switch.

---

## Configuration SW1

Ouvrir l'onglet **CLI** de SW1 dans Packet Tracer :

```
enable
configure terminal

hostname SW1

vlan 10
name etudiant
vlan 20
name enseignant
vlan 30
name administration
vlan 100
name native
vlan 101
name management

interface range f0/1-8
switchport mode access
switchport access vlan 10

interface range f0/9-16
switchport mode access
switchport access vlan 20

interface range f0/17-23
switchport mode access
switchport access vlan 30

interface f0/24
switchport mode access
switchport access vlan 101

interface vlan 101
ip address 192.168.101.254 255.255.255.0
no shutdown

int g0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30,100,101
switchport trunk native vlan 100

end
```

### Vérifications

```
show vlan brief
show interfaces trunk
```

Résultat attendu : chaque VLAN apparaît avec ses ports, Gig0/1 est en mode trunk 802.1q avec native vlan 100.

---

## Configuration SW2

La configuration est **identique à SW1**, seule l'adresse IP de la SVI change (`192.168.101.253`) :

```
enable
configure terminal

hostname SW2

vlan 10
name etudiant
vlan 20
name enseignant
vlan 30
name administration
vlan 100
name native
vlan 101
name management

interface range f0/1-8
switchport mode access
switchport access vlan 10

interface range f0/9-16
switchport mode access
switchport access vlan 20

interface range f0/17-23
switchport mode access
switchport access vlan 30

interface f0/24
switchport mode access
switchport access vlan 101

interface vlan 101
ip address 192.168.101.253 255.255.255.0
no shutdown

int g0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30,100,101
switchport trunk native vlan 100

end
```

---

## Configuration SSH sur SW1 et SW2

```
enable
configure terminal

ip domain-name gsb.org
crypto key generate rsa
```

> ⚠️ Choisir **2048 bits** pour la clé RSA.

```
line vty 0 15
login local
transport input ssh

username abdelillah password ******
ip ssh version 2
ip ssh authentication-retries 5
ip ssh time-out 60

enable secret drowssap

end
copy running-config startup-config
```

### Vérification

```
show ip ssh
```

Résultat attendu : `SSH Enabled - version 2.0`

### Test de connexion

Depuis PC6 (`192.168.101.1`) :

```
ssh -l admin 192.168.101.254
```

---

## Sauvegarde de la configuration via FTP

### Mise en place du serveur FTP

Brancher un serveur sur le port **Fa0/23** de SW1 (VLAN 101) :

```
configure terminal
interface fastEthernet 0/23
switchport access vlan 101
end
```

Sur le serveur, activer le service FTP (Services → FTP) avec un compte ayant tous les droits (RWDNL).

Configurer l'IP du serveur :

- IP : `192.168.101.200`
- Masque : `255.255.255.0`

### Sauvegarder SW1 sur le serveur FTP

```
enable
config terminal
ip ftp username abdelillah
ip ftp password ******
copy running-config ftp:
```

- **Address or name of remote host** → `192.168.101.200`
- **Destination filename** → Entrée (nom par défaut `SW1-confg`)

### Restaurer la config SW1 sur SW2

```
enable
ip ftp username admin
ip ftp password drowssap
copy ftp: running-config
```

- **Address or name of remote host** → `192.168.101.200`
- **Source filename** → `SW1-confg`

> ⚠️ Après la copie, le prompt affiche `SW1#` alors qu'on est sur SW2. Corriger manuellement :

```
configure terminal
hostname SW2
interface vlan 101
ip address 192.168.101.253 255.255.255.0
end
wr
```

---

## Vérifications finales

| Test | Résultat attendu |
|------|-----------------|
| PC0 (192.168.10.1) ↔ PC3 (192.168.10.2) — même VLAN | ✅ OK |
| PC0 (VLAN 10) ↔ PC1 (VLAN 20) — VLANs différents | ❌ KO (normal, nécessite un routeur) |
| PC6 (VLAN 101) → SSH vers SW1 et SW2 | ✅ OK |

> ⚠️ Si le VLAN natif n'est pas identique des deux côtés du trunk, CDP génère des erreurs. Vérifier que les deux switches ont bien `native vlan 100` sur Gig0/1.
