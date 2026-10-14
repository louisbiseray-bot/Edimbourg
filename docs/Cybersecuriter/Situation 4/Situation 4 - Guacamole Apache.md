# Gestion des accès, Traçabilité, Enregistrement des sessions et Authentification Forte
___
## Informations Générales

**Auteur :** Louis BISERAY
**Date :** 01/10/2026
**Sujet :** Mise en place de Gestion des accès, Traçabilité, Enregistrement des sessions et Authentification Forte (MFA)**

---

## 1. Présentation & Préalables

Apache Guacamole est une passerelle de bureau à distance sans client (HTML5). Elle permet d'accéder de manière sécurisée à des serveurs distants via des protocoles standardisés tels que **SSH** et **RDP**.

### URL d'accès au bastion

```text
https://<Adresse_IP_du_Bastion>:8443/guacamole/
```

### Identifiants par défaut lors de la première connexion

* **Nom d'utilisateur :** `guacadmin`
* **Mot de passe :** `guacadmin`

---

## 2. Gestion des Utilisateurs, Groupes et Sécurité des Accès

### A. Sécurisation du compte administrateur

Pour des raisons de sécurité, le compte d'administration par défaut doit être remplacé immédiatement :

1. Allez dans **Paramètres** (en cliquant sur l'identifiant en haut à droite) $\rightarrow$ Onglet **Utilisateurs**.
2. Créez un nouveau compte d'administration (exemple : `adminbastion`) et attribuez-lui l'ensemble des privilèges d'administration (sauf le droit d'auditeur si dédié).
3. Déconnectez-vous de la session courante.
4. Reconnectez-vous avec le nouveau compte (`adminbastion`), puis **supprimez définitivement** le compte par défaut `guacadmin`.

### B. Création et gestion des groupes

L'attribution des accès s'effectue préférentiellement par des groupes d'utilisateurs :

1. Accédez aux **Paramètres** $\rightarrow$ Onglet **Groupes**.
2. Cliquez sur **Nouveau groupe** pour définir un nom (ex. `ServeursLinux`, `ServeursWindows`) et ajuster les droits associés.

### C. Création d'un compte utilisateur

1. Rendez-vous dans **Paramètres** $\rightarrow$ Onglet **Utilisateurs**.
2. Cliquez sur **Nouvel utilisateur**.
3. Renseignez le nom d'utilisateur et définissez obligatoirement un mot de passe.
4. *Remarque :* Par défaut, un nouvel utilisateur ne possède aucun privilège ni accès à aucune connexion.

---

## 3. Configuration des Connexions Distantes

### A. Configuration d'une connexion SSH (Serveur Linux)

1. Rendez-vous dans **Paramètres** $\rightarrow$ Onglet **Connexions**.
2. Cliquez sur **Nouvelle connexion** :
* **Nom :** `CubDHCP` (ou tout autre nom explicite)
* **Lieu :** `ROOT`
* **Protocole :** `SSH`


3. **Paramètres réseau & authentification :**
* **Nom d'hôte (Host) :** Adresse IP du serveur SSH distant
* **Port :** `22`
* **Nom d'utilisateur / Mot de passe :** Identifiants du compte local du serveur cible.



### B. Configuration d'une connexion RDP (Serveur Windows)

1. **Préalable :** Assurez-vous que l'option *Bureau à distance* est activée sur la machine Windows cible.
2. Rendez-vous dans **Paramètres** $\rightarrow$ Onglet **Connexions**.
3. Cliquez sur **Nouvelle connexion** :
* **Nom :** `CubAD`
* **Lieu :** `ROOT`
* **Protocole :** `RDP`


4. **Paramètres réseau & authentification :**
* **Nom d'hôte (Host) :** Adresse IP du serveur Windows
* **Port :** `3389`
* **Authentification :** Identifiant, mot de passe et domaine (si applicable)
* **Sécurité :** Cochez *Ignorer le certificat du serveur* (si certificat auto-signé).



### C. Association des accès via les groupes

* **Groupe `ServeursLinux` :** Associez-y la connexion `CubDHCP`.
* **Groupe `ServeursWindows` :** Associez-y la connexion `CubAD`.

---

## 4. Traçabilité, Audit et Enregistrement Vidéo des Sessions

### A. Consultation de l'historique des connexions

L'historique permet de suivre l'ensemble des activités d'accès au bastion :

* Accédez aux **Paramètres** $\rightarrow$ Onglet **Historique**.
* Vous y trouverez l'ensemble des métadonnées : *Nom d'utilisateur*, *Date/Heure de début*, *Durée de session*, *Nom de la connexion*, *Adresse IP hôte*.

### B. Mise en place du module d'enregistrement des sessions

Pour conserver un enregistrement visuel des sessions interactives :

1. **Télécharger et déployer l'extension d'enregistrement :**
```bash
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-history-recording-storage-1.6.0.tar.gz
tar -xzf guacamole-history-recording-storage-1.6.0.tar.gz
mv guacamole-history-recording-storage-1.6.0/guacamole-history-recording-storage-1.6.0.jar /etc/guacamole/extensions/

```


2. **Créer le dossier de stockage et appliquer les permissions :**

```bash
mkdir -p /var/lib/guacamole/recordings
chown guacd:tomcat /var/lib/guacamole/recordings
chmod 2750 /var/lib/guacamole/recordings
ls -ld /var/lib/guacamole/recordings
```


3. **Redémarrer le service Tomcat :**

```bash
systemctl restart tomcat9
```


4. **Activation au niveau d'une connexion dans l'interface Web :**
* Éditez une connexion (ex. `CubDHCP` ou `CubAD`).
* Dans la section **Enregistrement écran** :
* **Chemin d'enregistrement :** `${HISTORY_PATH}/${HISTORY_UUID}`
* **Option :** Cochez *Créer automatiquement le chemin d'enregistrement*.




5. **Visualisation des vidéos :**
* Les vidéos sont rejouables directement dans le navigateur depuis **Paramètres** $\rightarrow$ **Historique** (colonne *Catégorie : Journal*).
* *Règles de sécurité (ANSSI) :* L'utilisateur visionnant les sessions doit posséder le droit **Audit system** (de préférence sur un compte dédié à l'audit).



---

## 5. Authentification Double Facteur (MFA / TOTP)

### A. Installation de l'extension TOTP

1. **Télécharger et extraire l'extension :**
```bash
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-auth-totp-1.6.0.tar.gz
tar -xzf guacamole-auth-totp-1.6.0.tar.gz
sudo mv guacamole-auth-totp-1.6.0/guacamole-auth-totp-1.6.0.jar /etc/guacamole/extensions/
```


2. **Éditer le fichier de configuration principal :**
```bash
sudo nano /etc/guacamole/guacamole.properties
```


3. **Ajouter le bloc de paramètres TOTP :**
```properties
# Configuration TOTP (MFA)
totp-issuer: Guacamole CUB
totp-digits: 6
totp-period: 30
totp-mode: sha256
```


* **`totp-issuer` :** Nom de l'organisation affiché dans l'application TOTP.
* **`totp-digits` :** Longueur du code à jeton temporaire (6, 7 ou 8 chiffres).
* **`totp-period` :** Durée de validité du jeton en secondes.
* **`totp-mode` :** Algorithme de hachage de sécurité (`sha1`, `sha256`, `sha512`).


4. **Redémarrer le serveur d'applications :**
```bash
systemctl restart tomcat9
```



### B. Enrôlement et utilisation par l'utilisateur

* À la connexion suivante, l'utilisateur est invité à scanner un **QR Code** depuis une application TOTP (Google Authenticator, FreeOTP, etc.).
* La saisie du code à 6 chiffres valide définitivement la liaison de la clé.
* En cas de perte d'appareil, un administrateur peut réinitialiser la clé TOTP directement depuis la fiche de l'utilisateur dans l'onglet **Utilisateurs**.

### C. Configuration avancée de la politique TOTP (Exemptions / Obligations par IP)

Il est possible de conditionner l'exigence du MFA en fonction de l'adresse IP source dans `guacamole.properties` :

* **Exempter un sous-réseau interne (Bypass) :**
```properties
totp-bypass-hosts: 192.168.10.0/24
```

* **Rendre le TOTP strictement obligatoire partout (Enforce) :**
```properties
totp-enforce-hosts: 0.0.0.0/0
```