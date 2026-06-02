# Mission 8 — Netfilter & Iptables

**Netfilter** est un pare-feu implémenté au niveau du noyau Linux.  
**Iptables** est un jeu de commandes permettant de manipuler Netfilter.

Le **filtrage** consiste à accepter ou refuser des paquets en fonction de paramètres : IP source, IP destination, protocole, port source, port destination, état de la connexion, etc.

Le but de cette mission est de sécuriser le réseau en n'autorisant que les permissions strictement nécessaires.

---

## Schéma du réseau et protocoles nécessaires
![Schéma du réseau](images/Beaupeyrat.png)

---
## Règles de pare-feu — `/etc/rc.local`

```bash
#!/bin/bash

# Vider les règles existantes
iptables -F

# Bloquer TOUT
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

# Stateful - Autoriser les retours
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT

# ICMP (ping)
iptables -A INPUT -p icmp -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT

# SSH vers le routeur
iptables -A INPUT -p tcp -s 10.187.20.0/24 --dport 22 -j ACCEPT

# SSH vers le serveur et tous les conteneurs
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.0/20 --dport 22 -j ACCEPT

# Web (HTTP/HTTPS) vers web1 et web2
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.80 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.80 --dport 443 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.81 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.81 --dport 443 -j ACCEPT

# DNS vers dns1/dns2/dns5/dns6 (UDP + TCP)
iptables -A FORWARD -p udp -s 10.187.20.0/24 -d 10.31.64.53 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.53 --dport 53 -j ACCEPT
iptables -A FORWARD -p udp -s 10.187.20.0/24 -d 10.31.64.54 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.54 --dport 53 -j ACCEPT
iptables -A FORWARD -p udp -s 10.187.20.0/24 -d 10.31.64.57 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.57 --dport 53 -j ACCEPT
iptables -A FORWARD -p udp -s 10.187.20.0/24 -d 10.31.64.58 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.58 --dport 53 -j ACCEPT

# DNS vers Internet (résolution récursive)
iptables -A FORWARD -p udp -s 10.31.64.53 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.31.64.53 --dport 53 -j ACCEPT
iptables -A FORWARD -p udp -s 10.31.64.54 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.31.64.54 --dport 53 -j ACCEPT
iptables -A FORWARD -p udp -s 10.31.64.57 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.31.64.57 --dport 53 -j ACCEPT
iptables -A FORWARD -p udp -s 10.31.64.58 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.31.64.58 --dport 53 -j ACCEPT

# FTP vers ftp1 et ftp2
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.54.20 --dport 21 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.21 --dport 21 -j ACCEPT

# Backup (SFTP/rsync) vers backup1 et backup2
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.70 --dport 22 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.187.20.0/24 -d 10.31.64.71 --dport 22 -j ACCEPT

# Mises à jour : conteneurs → Internet
iptables -A FORWARD -p tcp -s 10.31.64.0/20 --dport 80  -j ACCEPT
iptables -A FORWARD -p tcp -s 10.31.64.0/20 --dport 443 -j ACCEPT

# DNS externes pour tout le sous-réseau
iptables -A FORWARD -p udp -s 10.31.64.0/20 --dport 53 -j ACCEPT
iptables -A FORWARD -p tcp -s 10.31.64.0/20 --dport 53 -j ACCEPT

# Pings
iptables -A FORWARD -p icmp --icmp-type echo-request -j ACCEPT
```

---

## Logique du script

1. **DROP général** : tout est bloqué par défaut (INPUT, OUTPUT, FORWARD)
2. **Stateful** : les connexions déjà établies sont autorisées en retour
3. **Règles spécifiques** : on ouvre uniquement les protocoles/ports strictement nécessaires pour chaque conteneur
