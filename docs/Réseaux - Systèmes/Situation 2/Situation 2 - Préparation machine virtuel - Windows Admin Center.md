___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 09/09/2026
**Sujet :** Déploiement d'un Windows Admin Center 

___

## Partie 1 - Installation du serveur Windows 2025 avec Bureau

### 1. VM WAC Windows 2025

Il faut tout d'abord lui affecter une IP ainsi qu'une passerelle  nous pouvons utiliser les menu intégrer de Windows 11 ou utiliser le PowerShell.

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.8.5 -PrefixLength 25 -DefaultGateway 192.168.8.126
```

Vérifier que l'adresse IP est bien affecter avec la commande `ipconfig /all`.

### 2. Effectuer un Sysprep

Pour réinitialiser les SID il faut effectuer un ``sysprep``,

Pour lancer un Sysprep :

- Appuyer sur ``Windows + r``
- Taper ``sysprep``
	Cela vous ouvre un dossier avec un exécutable nommer ``sysprep``
- Exécuter le ``sysprep``
- Rester en ``OOBE (Out Of the Box Experience)``
- Cocher la case ``généraliser``
- Et choisir ``redémarrer`` 

### 3. Modification de Base

#### 3.1 Mise à jour

Nous allons mettre a jour le machine virtuel en allant chercher les dernière mise à jour de Windows 11 :

- Aller dans l'onglet ``Paramètres``
- Par la suite dans ``Windows Update``
- Cliquer sur 
- Attendre que les mise à jour chargent
- Cliquer sur ``Installer tout``

#### 3.2 Modifier le nom du serveur

Modifions le nom de la machine pour coller avec l'utilisation que nous allons en faire

```powershell
Rename-Computer -NewName "NOUVEAU-NOM" -Restart
```

Sans le paramètre `-Restart`, le changement sera pris en compte seulement au prochain redémarrage manuel 

#### 3.3 Modifier le nom d'administrateur

Pour renommer le compte Admin Local

```powershell
Rename-LocalUser -Name "Administrateur" -NewName "ADM-SRV-00"
```

Et pour changer son mot de passe 

```powershell
$Password = Read-host -AsSecureString #Permet de taper un mot de passe qui sera stocker dans la variable $Password
Get-localuser -name "ADM-SRV-WAC" | Set-localuser -password $Password # Affecte le mot de passe à l'administrateur
```

#### 3.4 Serveurs NTP

Nous allons ajouter de deux serveurs NTP pour la redondance ainsi qu'une meilleur synchronisation.

```powershell
w32tm /config /manualpeerlist: "serveur1.ntp.com,0x8 serveur2.ntp.com,0x8" /syncfromflags:manual /reliable:yes /update
```

Pour relancer le service NTP et synchroniser sans attendre.

```powershell
w32tm /resync /nowait
```
