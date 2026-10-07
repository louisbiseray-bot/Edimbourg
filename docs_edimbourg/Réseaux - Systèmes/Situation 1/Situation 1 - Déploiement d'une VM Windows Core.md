___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 09/09/2026
**Sujet :** Déploiement d'un serveur Windows Core 2025 

___

## 1. Créer une VM Windows Core 2025

## 2. Vérifier la configuration IP

Tout d'abord il faut vérifier la configuration IP de notre machine avec la commande suivante

```powershell
ipconfig
```

Le résultat afficher est vide affichant seulement ceci : `Configuration IP de Windows`, cela signifie qu'il faut installer le ``Virt IO`` pour que la machine se voit attribuer une configuration IP.

## 3. Installation des Drivers Carte réseau

Il faut en premier lieu vérifier les lecteurs pour savoir le quel contient le `Virt IO`

```powershell
Get-Volume
```

Un lecteur nommer ``D     virtio-win-x-x-xxx`` doit apparaitre dans le résultat de la commande, le ``D`` devant `virtio-win-x-x-xxx` correspond au répertoire dans le quel ce situe le `virtio`

```powershell
cd D:\   #Se situer dans le répertoire D:\
ls       #Afficher tout les fichier et dossier du répertoire
```

Une grande liste va s'afficher avec tout les ficher et dossier dans le répertoire `D` , le fichier qui nous intéresse est `virtio-win-guest-tools.exe` c'est avec cette installateur que nous allons installer `VirtIO`

```powershell
Start-Process
FilePath : virtio-win-guest-tools.exe
```

Un installateur devrait ce lancer.
Une fois le l'installation terminer il faut mettre en place le service `QEMU-GA` 

```powershell
Set-Service -Name "QEMU-GA" -StartupType Automatic
Start-Service QEMU-GA
ipconfig
```

Maintenant une configuration réseau devrait s'afficher après `Configuration IP de Windows`

## 4. Paramétrer la Carte Réseau

Entrer les paramètres réseau qui suivent dans la carte réseau du Windows core

```
IP : 192.168.55.1 
Passerelle : 192.168.55.126
DNS 8.8.8.8
```

Pour modifier l'interface de la carte :

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1
```

Et pour ajouter les serveurs DNS :

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("8.8.8.8","8.8.4.4")
```

## 5. Changer le nom de la machine

```powershell
Rename-Computer -NewName "NOUVEAU-NOM" -Restart
```

Sans le paramètre `-Restart`, le changement sera pris en compte seulement au prochain redémarrage manuel 

## 6. Vérification des recommandation de l'ANSSI

Ajout de deux serveur NTP pour la redondance ainsi qu'une meilleur synchronisation

```powershell
w32tm /config /manualpeerlist:"serveur1.ntp.com,0x8 serveur2.ntp.com,0x8" /syncfromflags:manual /reliable:yes /update
```

Pour vérifier la date taper la commande 

```
date
```

Si tout est bien paramétrer l'heure actuel devrait s'afficher.

### 6.2 Vérifier les fonctionnalités de sécurité

Il nous faut maintenant vérifier ``l'UAC`` qui demande le mot de passe de l'administrateur aux utilisateurs qui demande l'exécution d'un programme ou qui essayent de modifier un paramètres qui demande des droits plus élever.

Voici la commande pour savoir  `UAC` est activer sur le Windows

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" | Select-Object EnableLUA
```

Le résultat obtenu devrait être celui ci :

```Powershell
EnableLUA
_________
        1
```

Dans le cas ou UAC est désactiver taper cette commande qui le réactiveras

```Powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA" -Value 1

Restart-Computer
```

Revérifier ``l'UAC`` pour confirmer que la commande a bien été appliquer

Enfin il faut vérifier que le pare-feu est bien activer sur les 3 service Domaine, Private et Public

```powershell
Get-NetFirewallProfile
```

Cela devrait afficher l'état (`Enabled`) pour les 3 profils :

- **Domain** (réseau de domaine)
- **Private** (réseau privé)
- **Public** (réseau public)

### 6.3 Mettre à jour

Server Core n'a généralement pas accès direct à Internet configuré pour NuGet, donc on installe d'abord le fournisseur NuGet puis le module :

```powershell
Install-PackageProvider -Name NuGet -Force
Install-Module -Name PSWindowsUpdate -Force
```

Importer le module Windows Update

```powershell
Import-Module PSWindowsUpdate
```

**Vérifier que le module est bien chargé :**

```powershell
Get-Command -Module PSWindowsUpdate
```

Vérifier les mise à jour disponible 

```powershell
Get-WindowsUpdate
```

Une version de la commande plus détailler

```powershell
Get-WindowsUpdate -Verbose
```

Installer tout les mise à jour proposer

```powershell
Install-WindowsUpdate -AcceptAll -IgnoreReboot
```

### 6.4 Sécuriser le compte Admin

Taper cette commande pour faire apparaitre tout les local User avec leur Nom et SSID

```powershell
Get-LocalUser | Select-Object Name, SID, Enabled
```

Enfin pour renommer le compte Admin Local

```powershell
Rename-user -Name "Administrateur" -NewName "ADM-SRV-00"
```

