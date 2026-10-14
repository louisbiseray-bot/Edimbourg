___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 16/09/2026
**Sujet :** Déploiement d'un Windows Admin Center 

___

## Partie 2 - Déploiement de Windows Admin Center

### 1. Introduction

Windows Admin Center est une solution d’administration gratuite développée par Microsoft, conçue pour moderniser et simplifier la gestion des infrastructures Windows grâce à une interface accessible depuis un navigateur web. Concrètement, Windows Admin Center permet d’administrer à distance des serveurs Windows Server et, si nécessaire, des postes de travail Windows, directement depuis un navigateur.

Windows Admin Center est régulièrement mis à jour par Microsoft et la solution intègre un système d’extensions permettant d’ajouter des fonctions supplémentaires. Il y a notamment des extensions développées par certains fabricants de serveurs (Fujitsu, Lenovo, etc.) afin de faciliter la remontée d’informations dans l’interface de WAC. La liste des fonctionnalités est déjà longue et continue de s'agrandir au fil des versions mais beaucoup d'informations peuvent déjà être remonter par WAC.

### 2. Installation de WAC

Pour Installer le WAC nous allons utiliser l'installateur 2026 disponible sur le site de Windows, une fois l'installateur télécharger lancer le et suivez ces étapes :

- Cliquez sur installation personnaliser 
- Choisissez ``à distance à partir d'autres machines``
- Cliquer sur ``Connexion au formulaire HTML``
- Mettez le port ``10443``
- Entrez le nom de domaine ``wac0.local.edimbourg.cub.sioplc.fr``
- Sélectionner un certificat Auto-Signé
- Et choisissez `N'importe quels ordinateurs` 

Avant de lancer le WAC nous allons créer un utilisateur pour se connecter à notre interfacer Windows Admin Center :

```PowerShell
$password = read-host -assecurestring
set-localuser -name "administrateurWAC1" -password $password
```

```Powershell
Add-localGroupMember -group "Administrateurs" -member "administrateurWAC1"
```

Une fois l'installation terminer ouvrer la page du WAC en entrant `https://wac0.local.edimbourg.cub.sioplc.fr:10443` et connecter vous avec le nouvel utilisateur `administrateurWAC1`

### 3. Ajouter des serveurs

Pour ajouter les serveurs cliquer sur ajouter en au de l'interface une menu latérale va s'ouvrir renseigner dans le champ vide soit le nom soit l'adresse IP de la machine et attendre que Windows Admin Center détecte cette machine.

Répéter l'opération autant de fois qu'il y à de machine à ajouter pour au final voir l'entièreté de votre parc sur votre interface WAC