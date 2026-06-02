# Travaux 1 — Connexion SSH avec clé

## Génération de la clé SSH

Sur la machine cliente, générer une paire de clés :

```bash
ssh-keygen
```

Cela crée deux fichiers dans le répertoire `.ssh` :

| Fichier | Type | Description |
|---------|------|-------------|
| `id_rsa` | 🔴 Clé privée | À ne **jamais** partager |
| `id_rsa.pub` | 🟢 Clé publique | À copier sur le serveur |

---

## Copier la clé publique sur le serveur

### Localisation de la clé publique (Windows)

```
C:\Users\<NomUtilisateur>\.ssh\id_rsa.pub
```

### Ajouter la clé sur le serveur

Se connecter en root sur le serveur :

```bash
su -
```

Aller dans le répertoire `.ssh` et modifier le fichier `authorized_keys` :

```bash
nano /root/.ssh/authorized_keys
```

Coller le contenu de `id_rsa.pub` dans ce fichier.

> On peut avoir plusieurs clés publiques dans `authorized_keys` (une par ligne), une par machine cliente.

---

## Connexion SSH sans mot de passe

| Hôte | Adresse IP |
|------|------------|
| Serveur | `10.31.64.1` |
| Routeur | `172.31.64.254` |

```bash
ssh root@10.31.64.1
ssh rtr-g64@172.31.64.254
```

La connexion s'établit directement sans demander de mot de passe grâce à la clé.
