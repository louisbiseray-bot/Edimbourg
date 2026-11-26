___
## Informations Générales

**Auteur :** Biseray Louis
**Date :** 19/11/2026
**Sujet :** Sécurisation d'un réseau à l'aide d'ACL et de règles de Filtrage
___

## 1. Objectif

L'objectif de cette documentation est de sécuriser un réseau en utilisant des listes de contrôle d'accès (ACL) et des règles de filtrage. Nous allons voir comment configurer ces mécanismes pour restreindre l'accès aux ressources réseau et protéger les systèmes contre les accès non autorisés.

## 2.Maquette réseau

Tout d'abord nous allons créer une maquette réseau avec Cisco Packet Tracer avec les éléments suivants : 
- 1 Serveur DHCP
- 1 routeur
- 3 Switchs
- 4 PC

[Inséerer une image de la maquette réseau ici]

## 3. Configuration du réseau

En suite nous allons configurer les éléments de notre nouveau réseau :

Serveur DHCP :
- IP : 192.168.1.1
- Masque : 255.255.255.0

Nous activons le serveur DHCP (Services > DHCP) sur le serveur : 

serverPool :
- Default Gateway : 192.168.1.254
- DNS Server : 192.168.1.1
- Start IP Address : 192.168.1.120
- Subet Mask : 255.255.255.0
- Maximum Number of Users : 10

serverPoolWifi :
- Default Gateway : 192.168.1.254
- DNS Server : 192.168.1.1
- Start IP Address : 192.168.2.120
- Subet Mask : 255.255.255.0
- Maximum Number of Users : 3

Nous allons rajouter un server FTP sur le switch du serveur DHCP.

Server FTP :
- IP : 192.168.1.2
- Masque : 255.255.255.0

Dans server FTP nous allons créer un utilisateur dans Services > FTP avec comme nom d'utilisateur `invite` avec le mot de passe `invite` et lui donner les droits `Lecture et Liste`.

