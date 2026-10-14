___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 10/09/2026
**Sujet :** Déploiement d'un Serveur DNS sur Debian 13

___
## 1. Objectif

Cette documentation décrit l'installation et la configuration d'un **serveur DNS Resolver sur Debian 13 à l'aide d'**Unbound**.

Le serveur a pour rôle de :

- résoudre les requêtes DNS des clients du réseau local en interrogeant directement les serveurs racine et les serveurs faisant autorité ;
- mettre en cache les réponses pour accélérer les résolutions suivantes ;
- valider les réponses avec **DNSSEC** ;
- ne répondre qu'aux clients autorisés (pas de resolver ouvert sur Internet).

> **Note :** ce serveur est un _resolver_, pas un serveur _faisant autorité_. Il n'héberge pas de zone DNS publique.

---

## 2. Prérequis

| Élément               | Valeur                                                                                     |
| --------------------- | ------------------------------------------------------------------------------------------ |
| Système               | Debian 13                                                                                  |
| Accès                 | Root ou utilisateur avec `sudo`                                                            |
| Réseau                | Adresse IP **statique**                                                                    |
| Connectivité sortante | UDP/TCP 53 vers Internet                                                                   |
| Filtrage              | Géré par le pare-feu Stormshield (UDP/TCP 53 entrant depuis le LAN, sortant vers Internet) |
| Logiciel              | Unbound                                                                                    |

### Paramètres utilisés dans cette documentation

|Paramètre|Valeur d'exemple|
|---|---|
|Nom d'hôte|`dns01`|
|IP du serveur|`192.168.1.10`|
|Réseau client autorisé|`192.168.1.0/24`|
|Domaine local|`lan.example`|

---

## 3. Préparation du système

### 3.1 Mise à jour

```bash
sudo apt update && sudo apt full-upgrade -y
```

### 3.2 Vérifier la configuration réseau

```bash
ip -br a
hostnamectl
```

L'adresse IP du serveur doit être fixe. Exemple avec `/etc/network/interfaces` :

```
auto ens18
iface ens18 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
```

### 3.3 Vérifier que le port 53 est libre

```bash
sudo ss -tulpn | grep ':53 '
```

Aucune sortie ne doit apparaître. Si un service occupe déjà le port (par exemple `systemd-resolved`), le désactiver :

```bash
sudo systemctl disable --now systemd-resolved
```

---

## 4. Installation

```bash
sudo apt install -y unbound dnsutils
```

- `unbound` : le serveur resolver ;
- `dnsutils` : fournit `dig` pour les tests.

Vérification de la version installée et de l'état du service :

```bash
unbound -V
systemctl status unbound
```

---

## 5. Configuration

Debian charge automatiquement tous les fichiers `*.conf` du dossier `/etc/unbound/unbound.conf.d/`. On crée donc un fichier dédié plutôt que de modifier `unbound.conf`.

```bash
sudoedit /etc/unbound/unbound.conf.d/resolver.conf
```

### 5.1 Fichier de configuration

```yaml
server:
    # --- Écoute ---
    interface: 127.0.0.1
    interface: 192.168.1.10
    port: 53

    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    # --- Contrôle d'accès ---
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.1.0/24 allow
    access-control: 0.0.0.0/0 refuse

    # --- Confidentialité / durcissement ---
    hide-identity: yes
    hide-version: yes
    qname-minimisation: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    harden-below-nxdomain: yes
    aggressive-nsec: yes
    use-caps-for-id: no
    
    # --- Performances / cache ---
    num-threads: 2
    msg-cache-size: 64m
    rrset-cache-size: 128m
    cache-min-ttl: 300
    cache-max-ttl: 86400
    prefetch: yes
    prefetch-key: yes

    # --- Logs ---
    verbosity: 1
    use-syslog: yes
    log-queries: no
```

### 5.2 Explication des principaux paramètres

| Paramètre                        | Rôle                                                                                                 |
| -------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `interface`                      | Adresses sur lesquelles Unbound écoute.                                                              |
| `access-control`                 | Définit qui peut interroger le serveur. Tout ce qui n'est pas autorisé est refusé.                   |
| `hide-identity` / `hide-version` | Empêche la divulgation du nom et de la version du serveur.                                           |
| `qname-minimisation`             | N'envoie aux serveurs amont que la partie du nom strictement nécessaire (meilleure confidentialité). |
| `harden-*`                       | Rejette les réponses suspectes ou incohérentes.                                                      |
| `prefetch`                       | Renouvelle en avance les entrées de cache bientôt expirées.                                          |
| `cache-min-ttl`                  | Impose un TTL minimum de 5 minutes pour limiter les requêtes répétées.                               |

### 5.3 Contrôle à distance (unbound-control)

Pour pouvoir utiliser `unbound-control` (statistiques, vidage du cache), créer :

```bash
sudoedit /etc/unbound/unbound.conf.d/remote-control.conf
```

```yaml
remote-control:
    control-enable: yes
    control-interface: /run/unbound.sock
```

---

## 6. Validation de la configuration et démarrage

Toujours valider la syntaxe avant de redémarrer :

```bash
sudo unbound-checkconf
```

Résultat attendu :

```
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

Activation et démarrage du service :

```bash
sudo systemctl enable --now unbound
sudo systemctl restart unbound
sudo systemctl status unbound
```

Vérifier l'écoute sur le port 53 :

```bash
sudo ss -tulpn | grep unbound
```

---

## 7. Entrées DNS locales

Pour résoudre des noms internes sans serveur DNS dédié, ajouter dans `/etc/unbound/unbound.conf.d/local.conf` :

```yaml
server:
    local-zone: "lan.example." static

    local-data: "dns01.lan.example.     IN A 192.168.1.10"
    local-data: "nas.lan.example.       IN A 192.168.1.20"
    local-data-ptr: "192.168.1.10 dns01.lan.example."
    local-data-ptr: "192.168.1.20 nas.lan.example."
```

Puis :

```bash
sudo unbound-checkconf && sudo systemctl reload unbound
```

> Avec `static`, tout nom non déclaré dans la zone `lan.example.` renverra `NXDOMAIN`.

---

## 8. Tests

### 8.1 Résolution locale

```bash
dig @127.0.0.1 debian.org
```

Vérifier :

- `status: NOERROR` ;
- une section `ANSWER` non vide.

### 8.2 Résolution depuis un client du réseau

```bash
dig @192.168.1.10 debian.org
```

### 8.3 Test du cache

Exécuter deux fois la même requête et comparer le temps de réponse :

```bash
dig @192.168.1.10 debian.org | grep "Query time"
dig @192.168.1.10 debian.org | grep "Query time"
```

Le second résultat doit être nettement plus faible (généralement `0 msec`).

### 8.4 Test DNSSEC

Domaine correctement signé : le flag `ad` (Authenticated Data) doit être présent.

```bash
dig @192.168.1.10 +dnssec debian.org | grep flags
```

Domaine volontairement mal signé : la réponse doit être `SERVFAIL`.

```bash
dig @192.168.1.10 sigfail.verteiltesysteme.net
```

Domaine correctement signé de contrôle (doit répondre `NOERROR` avec le flag `ad`) :

```bash
dig @192.168.1.10 sigok.verteiltesysteme.net
```

### 8.5 Test du contrôle d'accès

Depuis une machine **hors** du réseau autorisé, la requête doit être refusée (`REFUSED`) ou ne pas aboutir (timeout si le Stormshield bloque le flux).

---

## 9. Configuration des clients

### 9.1 Serveur lui-même

Éditer `/etc/resolv.conf` :

```
nameserver 127.0.0.1
```

### 9.2 Clients du réseau

Renseigner `192.168.1.10` comme serveur DNS :

- soit manuellement sur chaque machine ;
- soit via le serveur DHCP (option 6 _domain-name-servers_), ce qui est la méthode recommandée.

---

## 10. Exploitation et maintenance

### 10.1 Commandes utiles

bash

```bash
# Statistiques
sudo unbound-control stats_noreset

# Vider le cache d'un domaine
sudo unbound-control flush debian.org

# Vider tout le cache
sudo unbound-control flush_zone .

# Recharger la configuration sans coupure
sudo unbound-control reload
```

### 10.2 Logs

bash

```bash
sudo journalctl -u unbound -f
```

La verbosité reste fixée à `1` (`verbosity: 1`) en fonctionnement normal. Pour un débogage ponctuel, il est possible d'activer temporairement `log-queries: yes`, puis de recharger le service. **Penser à le remettre à `no` ensuite**, car les logs de requêtes contiennent des données personnelles.

### 10.3 Mises à jour

bash

```bash
sudo apt update && sudo apt upgrade
```

---

## 11. Dépannage

|Symptôme|Cause probable|Solution|
|---|---|---|
|`unbound` ne démarre pas|Erreur de syntaxe|`sudo unbound-checkconf`, puis `journalctl -u unbound`|
|`address already in use`|Port 53 déjà utilisé|`ss -tulpn \| grep :53`, arrêter le service concurrent|
|`REFUSED` côté client|IP du client absente d'`access-control`|Ajouter le réseau et recharger|
|Timeout côté client|Flux bloqué par le Stormshield ou mauvaise `interface`|Vérifier la règle de filtrage (UDP/TCP 53) sur le Stormshield et les lignes `interface`|
|`SERVFAIL` sur tous les domaines|Pas de sortie Internet ou DNSSEC en échec|Tester `dig @127.0.0.1 debian.org +cd` ; vérifier l'heure système (NTP)|
|`SERVFAIL` DNSSEC généralisé|Horloge incorrecte|`timedatectl` et activer la synchronisation NTP|

---

## 12. Récapitulatif des fichiers

|Fichier|Rôle|
|---|---|
|`/etc/unbound/unbound.conf`|Configuration principale (inclut le dossier `unbound.conf.d`)|
|`/etc/unbound/unbound.conf.d/resolver.conf`|Configuration du resolver|
|`/etc/unbound/unbound.conf.d/local.conf`|Entrées DNS locales (optionnel)|
|`/etc/unbound/unbound.conf.d/remote-control.conf`|Activation d'`unbound-control`|

---

## 13. Conclusion

Le serveur Debian 13 fournit désormais un service de résolution DNS récursif, avec cache, validation DNSSEC et accès restreint au réseau local. Les tests de la section 8 permettent de valider le bon fonctionnement avant la mise en production.