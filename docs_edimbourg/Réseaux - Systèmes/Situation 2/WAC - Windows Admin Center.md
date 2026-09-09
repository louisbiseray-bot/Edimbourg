___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 09/09/2026
**Sujet :** Déploiement d'un Windows Admin Center 

___

## Partie 1 - Installation du serveur Windows 2025 avec Bureau

### 1. VM WAC Windows 2025

Il faut tout d'abord lui affecter une IP ainsi qu'une passerelle 

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1
```
