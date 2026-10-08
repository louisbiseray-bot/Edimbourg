___
## Informations Générales

**Auteur :** BISERAY Louis
**Date :** 08/10/2026
**Sujet :** Mise en place de deux serveurs DNS maitre et esclave

___

## Introduction

En premier lieu il nous faut affecter les informations principales aux deux serveurs. Cette documentation ne couvrira que le serveur maitre.

Mettre le serveur Debian à jour :

```bash
sudo apt-get update && sudo apt-get upgrade
```

Adressage dans le réseau :

**Adresse Ip** : 192.36.5.10
**Passerelle** : 192.36.5.254
**DNS** : 172.16.20.11, 8.8.8.8

```bash
sudoedit /etc/network/interfaces
auto ens18
iface ens18 inet static
        address 192.36.5.10
        netmask 255.255.255.0
        gateway 192.36.5.254
```

```bash
sudoedit /etc/resolv.conf
nameserver 172.16.20.11
nameserver 8.8.8.8
```

Changer la configuration Ip sur la machine Debian via le fichier ``interfaces``

```bash
sudo systemctl restart networking
sudo systemctl status networking
```

Redémarrer le service `networking` pour appliquer les changements effectuer dans le fichier, le status doit afficher `enable` et `active`.

## Bind9 et dnsutils

Pour commencer nous allons installer plusieurs paquets, rsyslog et Bind9.
Pour le DNS nous devons installer les paquets bind9 et dnsutis qui serviras d'outils de diagnostic pour le DNS.

```bash
sudo apt install rsyslog
sudo apt install bind9 dnsutils
```

Changer le nom de la machine pour faire correspondre au fichier hosts que nous allons modifier aprés.
```bash
sudoedit /etc/hostname
dns0
```

(rajoute la def du fichier hosts)

```bash
sudoedit /etc/hosts
127.0.0.1 localhost
127.0.1.1 dns0.edimbourg.cub.sioplc.fr

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Redémarrer le serveur pour prendre en compte les changements de nom.

```bash
sudo shutdown -r now
```

## Unbound

Nous allons installer ``Unbound`` pour mettre en place le service DNS

```bash
sudo apt install unbound dnsutils tcpdump tmux curl
```

Puis nous allons configurer le service grace au fichier ``unbound.conf``
 (Explique chaque partie du fichier de conf avec des commentaire)
```bash
sudoedit /etc/unbound/unbound.conf
```

```bash

include: "/etc/unbound/unbound.conf.d/*.conf"

server:

interface: 192.36.5.10
interface: 127.0.0.1

access-control: 192.168.5.0/24 allow_snoop
access-control: 127.0.0.0/8 allow_snoop

root-hints: "/var/lib/unbound/root.hints"

hide-version: yes
hide-identity: yes
qname-minimisation: yes

do-ip4: yes

logfile: /var/log/unbound.log
verbosity: 1
log-queries: yes
```

Une fois le fichier sauvegarder vérifier que aucune erreur ne se trouve dans le fichier de conf avec la commande :

````bash
sudo unbound-checkconf
```

## TTL (Time To Live)

**TTL court** => L'avantage d'un TTL court est que le resolver va demander plus souvent la liste des enregistrement ce qui est pratique lors de changements réguilers.
**TTL long** => Tandis que l'avantage d'un TTL long est de réduire les échanges et la quantitées de paquets présent entre les deux serveurs ce qui réduit la charge sur le réseau.

Pour répartir la charge :
```
www IN A 192.36.5.20
www IN A 192.36.5.21
```

Le service DNS va se chager de faire automatiquement le ``loadbalancing``, ici le load serat en 50/50.
Cependant si un serveur tombe en panne le serveur DNS va continuer a envoyer des paquets a ce serveur cette méthode ne remplace pas un loadbalancer.