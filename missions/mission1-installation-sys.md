# Mission 1 — Installation des systèmes

## Configuration du routeur

### Attribution d'une adresse IP (temporaire)

```bash
ip addr add 172.31.64.254/20 dev enp2s0
```

### Configuration permanente — `/etc/network/interfaces`

```
auto enp4s0
iface enp4s0 inet static
        address 10.31.79.254/20

auto enp2s0
iface enp2s0 inet static
        address 172.31.64.254/16
        gateway 172.31.0.1
post-up iptables -t nat -A POSTROUTING -s 10.31.64.0/16 -o enp2s0 -j MASQUERADE
```

### Activer le routage IP

1. Modifier `/etc/sysctl.conf`
2. Ajouter : `net.ipv4.ip_forward=1`
3. Appliquer : `sysctl -p`

### Configuration DNS

Dans `/etc/resolv.conf`, ajouter :

```
nameserver 8.8.8.8
```

### Vérification du routage

```bash
ip route                  # affiche les routes
ping 172.31.64.1          # test connectivité WAN
ping 10.31.64.1           # test connectivité LAN
```

---

## Configuration du serveur

### Attribution d'une adresse IP (temporaire)

```bash
ip addr add 10.31.64.1/20 dev eno1
```

### Configuration permanente — `/etc/rc.local`

```bash
#!/bin/sh -e
brctl addbr br0
ifconfig eno1 0.0.0.0
ifconfig br0 10.31.64.1/20
route add default gw 10.31.79.254
brctl addif br0 eno1
```

### Configuration DNS

Dans `/etc/resolv.conf` :

```
nameserver 8.8.8.8
```

### Tests réseau

```bash
ping 10.31.79.254
ping 172.31.64.254
ping 8.8.8.8
```

---

## Connexion à distance

### Ajouter les routes sur son PC

**Windows :**

```cmd
route add -p 10.31.0.0 mask 255.255.0.0 10.187.20.10
route add -p 172.31.0.0 mask 255.255.240.0 10.187.20.10
```

**Linux :**

```bash
route add 10.31.0.0/16 gw 10.187.20.10
route add 172.31.0.0/20 gw 10.187.20.10
```

### Connexion SSH

```bash
ssh rtr-g64@172.31.64.254   # routeur
ssh gulumser@10.31.64.1     # serveur
```
