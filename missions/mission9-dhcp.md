# Mission 9 — Service DHCP

Le protocole **DHCP** (Dynamic Host Configuration Protocol) permet d'attribuer dynamiquement des adresses IP à chaque hôte du réseau.

## Installation

Dans les conteneurs **dhcp1** et **dhcp2** :

```bash
apt install isc-dhcp-server
```

## Configuration de l'interface d'écoute

```bash
nano /etc/default/isc-dhcp-server
```

Ajouter `eth0` dans `INTERFACESv4` :

```
INTERFACESv4="eth0"
```

## Configuration du serveur DHCP

```bash
nano /etc/dhcp/dhcpd.conf
```

Voici la configuration du conteneur **dhcp2** :

```
ddns-update-style none;

subnet 10.31.64.0 netmask 255.255.240.0 {
        range 10.31.65.101 10.31.65.200;
        option routers 10.31.79.254;
        option broadcast-address 10.31.79.255;
        option domain-name-servers 10.31.64.57;
        option domain-name "m2l.org";
        default-lease-time 86400;
        max-lease-time 604400;

        group {
                use-host-decl-names true;
                host dhcp2 {
                        hardware ethernet 10:66:6a:82:e7:9e;
                        fixed-address 10.31.64.68;
                }
                host web2 {
                        hardware ethernet 10:66:6a:4e:0a:1c;
                        fixed-address 10.31.64.81;
                }
                host ns5 {
                        hardware ethernet 10:66:6a:bf:e8:6d;
                        fixed-address 10.31.64.57;
                }
                host ns6 {
                        hardware ethernet 10:66:6a:fa:39:fd;
                        fixed-address 10.31.64.58;
                }
                host ftp2 {
                        hardware ethernet 10:66:6a:0c:0d:b4;
                        fixed-address 10.31.64.21;
                }
                host backup2 {
                        hardware ethernet 10:66:6a:75:8c:c4;
                        fixed-address 10.31.64.71;
                }
                host nginx2 {
                        hardware ethernet 10:66:6a:4c:04:03;
                        fixed-address 10.31.64.91;
                }
        }
}
```

## Redémarrage du service

```bash
systemctl restart isc-dhcp-server
```
