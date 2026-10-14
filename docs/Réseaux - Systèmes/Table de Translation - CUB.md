___

**Auteur :** BISERAY Louis
**Date :** 09/09/2026
**Sujet :** Création de la table de Translation de l'agence d'Edimbourg

___

## Table de Translation `edg-firewall`

| Inside Local     | Inside Global |
| ---------------- | ------------- |
| 192.168.5.0/24   | 192.36.253.50 |
| 192.36.5.1 (DMZ) | deny          |

## Table de Translation ``main-firewall`

| Inside Local     | Inside Global |
| ---------------- | ------------- |
| 192.36.253.50/24 | 172.16.31.250 |
| 192.36.5.1 (DMZ) | deny          |

## Exemple de trajet ``Production` => `Serveur Internet`

| Etape                   | Source        | Destination   |
| ----------------------- | ------------- | ------------- |
| Emission PC             | 192.168.5.1   | 172.16.31.251 |
| Aprés NAT edg-firewall  | 192.36.253.50 | 172.16.31.251 |
| Aprés NAT main-firewall | 172.16.31.250 | 172.16.31.251 |
