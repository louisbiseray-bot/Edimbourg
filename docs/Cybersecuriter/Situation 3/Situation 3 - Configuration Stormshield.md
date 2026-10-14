___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 09/09/2026
**Sujet :** Configuration du pare-feu Stormshield 

___

## 1. Objectif et périmètre

Le Stormshield doit assurer trois fonctions :

1. **Segmenter** le réseau en trois zones : **IN** (LAN interne), **OUT** (vers Internet / `main-firewall`) et **DMZ**.
2. **Router** le trafic entre ces zones et vers les VLAN du LAN (via `edg-sw-core`).
3. **Translater** (NAT / PAT) le trafic sortant du LAN, et **ne pas translater** la DMZ, conformément à la logique de la maquette.

---

## 2. Plan d'adressage de référence

### 2.1 Interfaces du Stormshield

|Zone|Nom d'interface SNS|Adresse IP / masque|Voisin direct|Rôle|
|---|---|---|---|---|
|**IN**|`in`|`192.168.55.254/29`|`edg-sw-core` (Vlan2 – `192.168.55.253`)|Interco vers le LAN|
|**OUT**|`out`|`192.36.253.50/24`|`main-firewall` Gi0/0 (`192.36.253.254`)|Sortie vers Internet|
|**DMZ**|`dmz1`|`192.36.5.254/24`|Serveur DMZ (`192.36.5.1`)|Zone démilitarisée|

### 2.2 Réseaux à connaître

|Réseau|CIDR|Passerelle|
|---|---|---|
|VLAN 55 – Production|`192.168.5.0/25`|`192.168.5.126` (SVI edg-sw-core)|
|VLAN 10 – Client|`192.168.5.128/26`|`192.168.5.190` (SVI edg-sw-core)|
|VLAN 20 – AdminSys|`192.168.5.192/28`|`192.168.5.206` (SVI edg-sw-core)|
|VLAN 2 – Interco cœur/pare-feu|`192.168.55.248/29`|`192.168.55.254` (Stormshield)|
|DMZ|`192.36.5.0/24`|`192.36.5.254` (Stormshield)|
|Interco pare-feux|`192.36.253.0/24`|point à point|
|Internet simulé|`172.16.28.0/22`|`172.16.31.250` (main-firewall)|

---

## 3. Prérequis et accès initial

### 3.1 Câblage

|Port Stormshield|Zone|Connecté à|
|---|---|---|
|Port « IN »|IN|`edg-sw-core` Gi0/1 (port en `access vlan 2`)|
|Port « OUT »|OUT|`main-firewall` Gi0/0|
|Port « DMZ1 »|DMZ|Serveur DMZ (câble direct ou commutateur dédié)|

### 3.2 Première connexion

1. Brancher un PC directement sur l'interface **IN** du Stormshield et lui donner une IP dans le même réseau que l'adresse d'usine. _(Adresse d'usine usuelle sur l'interface IN : `10.0.0.254/8` – à vérifier dans la fiche du modèle.)_
2. Ouvrir `https://10.0.0.254/admin`.
3. S'authentifier (compte `admin`, mot de passe d'usine à **changer**).

---

## 4. Configuration des interfaces (IN / OUT / DMZ)

Menu : **Configuration → Réseau → Interfaces**
### 4.1 Interface IN – LAN interne

Double-cliquer sur l'interface `in` :

| Champ               | Valeur                               |
| ------------------- | ------------------------------------ |
| Nom                 | `in`                                 |
| Commentaire         | `Interco vers edg-sw-core (VLAN 2)`  |
| Interface           | **Interne**                          |
| Adresse IPv4        | **Adresse fixe**                     |
| Adresse IP / masque | `192.168.55.254` / `255.255.255.248` |

Cliquer sur **Appliquer**.

### 4.2 Interface OUT – Sortie vers Internet

Double-cliquer sur l'interface `out` :

| Champ               | Valeur                            |
| ------------------- | --------------------------------- |
| Nom                 | `out`                             |
| Commentaire         | `Lien vers main-firewall`         |
| Interface           | **Externe (publique)**            |
| Adresse IPv4        | **Adresse fixe** (et non DHCP)    |
| Adresse IP / masque | `192.36.253.50` / `255.255.255.0` |

Cliquer sur **Appliquer**.

### 4.3 Interface DMZ

Double-cliquer sur l'interface `dmz1` :

| Champ               | Valeur                           |
| ------------------- | -------------------------------- |
| Nom                 | `dmz1`                           |
| Commentaire         | `Zone DMZ`                       |
| Interface           | **Interne**                      |
| Adresse IPv4        | **Adresse fixe**                 |
| Adresse IP / masque | `192.36.5.254` / `255.255.255.0` |

Cliquer sur **Appliquer**.
### 4.4 Vérification

Dans **Configuration → Réseau → Interfaces**, les trois interfaces doivent apparaître avec leur adresse et un état **actif** (lien up). Contrôle rapide depuis **Tableau de bord → Réseau**.

---
## 5. Configuration des objets réseau

Menu : **Configuration → Objets → Objets réseau**

Les objets rendent les règles de filtrage et de NAT lisibles et maintenables. 
### 5.1 Objets prédéfinis

|Objet|Signification|
|---|---|
|`Firewall_in`|IP du firewall sur l'interface IN (`192.168.55.254`)|
|`Firewall_out`|IP du firewall sur l'interface OUT (`192.36.253.50`)|
|`Firewall_dmz1`|IP du firewall sur l'interface DMZ (`192.36.5.254`)|
|`Network_in`|Réseau directement connecté à IN (`192.168.55.248/29`)|
|`Network_dmz1`|Réseau directement connecté à DMZ (`192.36.5.0/24`)|
|`Internet`|Tout ce qui n'est pas un réseau interne/protégé|
|`Any`|Tout|

### 5.2 Objets à créer – Machines (`Hôte`)

Cliquer sur **Ajouter → Machine**.

|Nom|Adresse IP|Description|
|---|---|---|
|`host_sw_core_vlan2`|`192.168.55.253`|SVI Vlan2 de `edg-sw-core` (prochain saut vers le LAN)|
|`host_main_firewall`|`192.36.253.254`|Passerelle par défaut (Gi0/0 de `main-firewall`)|
|`host_srv_dmz`|`192.36.5.1`|Serveur hébergé en DMZ|
|`host_srv_internet`|`172.16.31.251`|Serveur simulant Internet|
|`host_pc_adminsys`|`192.168.5.193`|Poste d'administration (VLAN 20)|

### 5.3 Objets à créer – Réseaux (`Réseau`)

Cliquer sur **Ajouter → Réseau**.

|Nom|Adresse réseau|Masque|Description|
|---|---|---|---|
|`net_vlan10_client`|`192.168.5.128`|`/26`|VLAN 10 – Client|
|`net_vlan20_adminsys`|`192.168.5.192`|`/28`|VLAN 20 – AdminSys|
|`net_vlan55_prod`|`192.168.5.0`|`/25`|VLAN 55 – Production|
|`net_interco_core`|`192.168.55.248`|`/29`|VLAN 2 – Interco cœur/pare-feu|
|`net_dmz`|`192.36.5.0`|`/24`|DMZ|
|`net_interco_fw`|`192.36.253.0`|`/24`|Interco entre pare-feux|
|`net_internet_sim`|`172.16.28.0`|`/22`|Internet simulé|

### 5.4 Objets à créer – Groupes (`Groupe`)

Cliquer sur **Ajouter → Groupe**.

|Nom du groupe|Membres|Usage|
|---|---|---|
|`grp_lan_users`|`net_vlan10_client`, `net_vlan20_adminsys`, `net_vlan55_prod`|Tous les VLAN utilisateurs (sources des règles NAT et filtrage)|

> L'usage du groupe évite de dupliquer trois règles. Si un VLAN est ajouté plus tard, il suffit de l'ajouter au groupe.

---

## 6. Configuration du routage

Menu : **Configuration → Réseau → Routage**

Le Stormshield connaît seulement ses réseaux directement connectés (IN, OUT, DMZ). Il faut lui indiquer :

- où envoyer le trafic Internet → **passerelle par défaut** vers `main-firewall` ;
- où se trouvent les VLAN du LAN → **routes statiques** vers `edg-sw-core`.

### 6.1 Route par défaut

Onglet **Routes statiques → Passerelle par défaut** :

| Champ            | Valeur                                  |
| ---------------- | --------------------------------------- |
| Route par défaut | `host_main_firewall` (`192.36.253.254`) |
### 6.2 Routes statiques vers les VLAN du LAN

Onglet **Routes statiques IPv4 → Ajouter** (une route par VLAN) :

|Réseau de destination|Passerelle|Interface|Commentaire|
|---|---|---|---|
|`net_vlan55_prod`|`host_sw_core_vlan2`|`in`|Retour vers VLAN 55|
|`net_vlan10_client`|`host_sw_core_vlan2`|`in`|Retour vers VLAN 10|
|`net_vlan20_adminsys`|`host_sw_core_vlan2`|`in`|Retour vers VLAN 20|

Cliquer sur **Appliquer** après chaque ajout.

### 6.3 Côté `edg-sw-core` 

La route par défaut du commutateur cœur doit pointer vers l'IP IN du Stormshield :

```
ip route 0.0.0.0 0.0.0.0 192.168.55.254
```

---

## 7. Configuration du NAT et du PAT

Menu : **Configuration → Politique de sécurité → Filtrage et NAT → onglet NAT**
### 7.1 Principe

| Flux                    | Traitement                                   | Justification                                                                                        |
| ----------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| LAN => DMZ              | **Pas de NAT**                               | Routage direct, les IP source restent visibles                                                       |
| DMZ => tous             | **Pas de NAT**                               | La DMZ est exclue du NAT sortant (cohérent avec `access-list 1 deny host 192.36.5.1` de la maquette) |
| LAN => Internet (`out`) | **PAT** sur `Firewall_out` = `192.36.253.50` | Toutes les IP du LAN sortent derrière une seule IP, comme sur `edg-firewall`                         |

Ce qui est ensuite fait par `main-firewall` : un second PAT sur `172.16.31.250` (double NAT en cascade).
### 7.2 Règle 1 – Pas de NAT entre le LAN et la DMZ

Cliquer sur **Nouvelle règle → Règle standard**.

| Colonne                                 | Valeur                     |
| --------------------------------------- | -------------------------- |
| État                                    | **On**                     |
| Trafic d'origine => Source              | `grp_lan_users`            |
| Trafic d'origine => Destination         | `net_dmz`                  |
| Trafic d'origine => Interface d'entrée  | `in`                       |
| Port destination                        | `Any`                      |
| Trafic après translation => Source      | `Any` (aucune translation) |
| Trafic après translation => Destination | `Any` (aucune translation) |
| Commentaire                             | `Pas de NAT LAN vers DMZ`  |

### 7.3 Règle 2 – Pas de NAT pour la DMZ (exclusion du NAT sortant)

|Colonne|Valeur|
|---|---|
|État|**On**|
|Source d'origine|`net_dmz`|
|Destination d'origine|`Any`|
|Interface d'entrée|`dmz1`|
|Interface de sortie|`any`|
|Source après translation|`Any` (aucune translation)|
|Commentaire|`DMZ exclue du NAT sortant`|

### 7.4 Règle 3 – PAT du LAN vers Internet

| Colonne                                                                  | Valeur                                                    |
| ------------------------------------------------------------------------ | --------------------------------------------------------- |
| État                                                                     | **On**                                                    |
| Trafic d'origine => Source                                               | `grp_lan_users`                                           |
| Trafic d'origine => Destination                                          | `Any`                                                     |
| Trafic d'origine => Interface d'entrée                                   | `in`                                                      |
| Trafic d'origine => Interface de sortie (_« Trafic après translation »_) | `out`                                                     |
| Source translatée                                                        | `Firewall_out`                                            |
| Port source translaté                                                    | `ephemeral_fw` (ports éphémères) – **mode « aléatoire »** |
| Destination translatée                                                   | `Any` (inchangée)                                         |
| Commentaire                                                              | `PAT LAN vers Internet (sur 192.36.253.50)`               |

> Sélectionner **Firewall_out** comme source translatée équivaut à `ip nat inside source list 1 interface Gi0/1 overload` sur le routeur Cisco de la maquette.

### 7.5 Tableau récapitulatif de la politique NAT

| N°  | Source          | Destination | Entrée => Sortie | Source translatée    | Rôle                  |
| --- | --------------- | ----------- | ---------------- | -------------------- | --------------------- |
| 1   | `grp_lan_users` | `net_dmz`   | `in` => `dmz1`   | —                    | Pas de NAT LAN => DMZ |
| 2   | `net_dmz`       | `Any`       | `dmz1` => `any`  | —                    | Pas de NAT DMZ        |
| 3   | `grp_lan_users` | `Any`       | `in` => `out`    | `Firewall_out` (PAT) | PAT LAN =>Internet    |

---

## 8. Politique de filtrage (A voir avec les professeurs)

Menu : **Configuration → Politique de sécurité → Filtrage et NAT → onglet Filtrage**

|N°|Action|Source|Destination|Service|Commentaire|
|---|---|---|---|---|---|
|1|**Passer**|`host_pc_adminsys`|`Firewall_in`|`https`, `ssh`|Administration du firewall depuis AdminSys uniquement|
|2|**Bloquer**|`Any`|`Firewall_*` (tous)|`https`, `ssh`|Interdit l'administration depuis d'autres zones|
|3|**Passer**|`grp_lan_users`|`net_dmz`|`http`, `https`, `icmp` _(ajuster)_|Le LAN accède aux services de la DMZ|
|4|**Bloquer**|`net_dmz`|`grp_lan_users`|`Any`|La DMZ ne doit **jamais** initier de connexion vers le LAN|
|5|**Passer**|`grp_lan_users`|`Internet`|`http`, `https`, `dns`, `icmp`|Navigation et résolution DNS|
|6|**Bloquer**|`Any`|`Any`|`Any`|Règle finale « tout interdire » (avec journalisation)|

Cliquer sur **Appliquer** pour activer la politique.

---

## 9. Tests de validation

Réaliser les tests dans cet ordre (du plus simple au plus complet) :

| #   | Test                | Depuis                         | Commande                          | Résultat attendu            |
| --- | ------------------- | ------------------------------ | --------------------------------- | --------------------------- |
| 1   | Lien IN             | `edg-sw-core`                  | `ping 192.168.55.254`             | Réponse                     |
| 2   | Lien OUT            | Stormshield (_Outils => Ping_) | ping `192.36.253.254`             | Réponse                     |
| 3   | Lien DMZ            | Stormshield                    | ping `192.36.5.1`                 | Réponse                     |
| 4   | Route de retour LAN | Stormshield                    | ping `192.168.5.190` (SVI Vlan10) | Réponse                     |
| 5   | LAN => DMZ          | PC Production                  | `ping 192.36.5.1`                 | Réponse (sans NAT)          |
| 6   | LAN => Internet     | PC Client                      | `ping 172.16.31.251`              | Réponse (double PAT)        |
| 7   | DMZ => LAN          | Serveur DMZ                    | ping `192.168.5.1`                | **Échec attendu** (règle 4) |
| 8   | Isolation admin     | PC Client                      | `https://192.168.55.254/admin`    | **Échec attendu** (règle 2) |

### 9.1 Vérifier le NAT / PAT

- **Supervision => Supervision des connexions** : repérer la connexion du test 6. L'adresse source translatée doit être `192.36.253.50`.
- Sur `main-firewall` (Cisco) :

```
show ip nat translations
```

La source affichée côté extérieur doit être `172.16.31.250`.

### 9.2 Consulter les logs

**Journaux => Trafic réseau** : filtrer sur l'adresse source du test pour voir la règle appliquée (n° de règle de filtrage et NAT) et l'action (Passer / Bloquer).

---

## 10. Dépannage 

| Symptôme                                              | Cause probable                                                          | Action                                                                                               |
| ----------------------------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Le LAN ne sort pas du tout                            | Route par défaut du Stormshield absente/erronée                         | Vérifier [[#6.2 Routes statiques vers les VLAN du LAN]]                                              |
| Ping OK vers le firewall mais pas au-delà             | Règle de filtrage manquante                                             | Consulter les journaux, ajouter la règle [[#8. Politique de filtrage (A voir avec les professeurs)]] |
| La requête part mais aucune réponse                   | Pas de route de retour vers les VLAN                                    | Vérifier [[#6.2 Routes statiques vers les VLAN du LAN]]                                              |
| Sortie Internet OK mais IP source non translatée      | Règle PAT absente ou placée **après** une règle « sans NAT » plus large | Réordonner les règles NAT [[#7.5 Tableau récapitulatif de la politique NAT]]                         |
| `main-firewall` refuse le trafic                      | Son ACL 1 n'autorise que `192.36.253.50`                                | Vérifier que le PAT utilise bien `Firewall_out`                                                      |
| DMZ non joignable depuis le LAN                       | NAT appliqué à tort sur LAN→DMZ                                         | Vérifier la règle 1 [[#7.2 Règle 1 – Pas de NAT entre le LAN et la DMZ]]                             |
| Perte d'accès à l'interface web après changement d'IP | Normal (nouvelle IP IN)                                                 | Se reconnecter via `https://192.168.55.254/admin` depuis le VLAN 20                                  |
| Interface « down »                                    | Câble / mauvais port / mauvais type d'interface                         | Contrôler le câblage [[#3.1 Câblage]]                                                                |
