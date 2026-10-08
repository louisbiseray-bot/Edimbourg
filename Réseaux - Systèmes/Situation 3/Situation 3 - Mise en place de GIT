### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 07/10/2026
**Sujet :** Mise en place d'un Mkdocs sur GIT

___

## 1. Introduction

Pour commencer il faut installer la dernierr version de Git sur son appareil pour cette documentation ce sera pour le systéme d'exploitation Windows 11.

Une fois Git installer il faut vérifier sa version, cela nous permet de savoir si Git est bien installer et qu'il est dans la bonne version.

```powershell
git --version
```

Réponse attendu : 

```powershell
PS C:\Users\xxxxx\Edimbourg> git --version
git version 2.52.0.windows.1
```

Git va servir à sauvegarder un état antérieur de notre projet pour y revenir quand nous le voulons, le versionning est surtout utiliser en cas d'erreur ou de perte de documentation.

## 2. Traçabilité

Git à besoin d'un nom d'utilisateur et d'une adresse mail pour autoriser les commits, ses information sont primordiales car on souhaite savoir qui a modifier, quand et qu'elle ressources sans ceci nous perdons toute tracabilité sur notre dépot/projet.
De plus certain utilisateurs n'on pas les mêmes branches et autorisation sur de dépot un utilisateurs peut seulement avoir le droit de cloner et de créer des suggestions tandis qu'un autre a l'autoriter sur tout le dépot.

Pour déclarer l'identité de la personne qui va faire le commit nous utilisons ces commandes :

```powershell
git config --global user.name "votre nom"
git config --global user.email "votre mail"
```

## 3. Création du dépot

Un dépot va nous servir a sauvegarder les versions des fichiers de notre projet sur internet sur la plateforme GitHub qui est disponible pour tout le monde (sauf si le dépot créer est préciser comme privé), l'utilité d'un Github est de pouvoir rassembler plusieurs personnes à travailler sur un même projet sans devoir se partager de fichiers et pouvoir suivre le travail de chacun et Git lui sert a créer des version des fichiers les deux peuvent fonctionner ensemble mais ils n'ont pas la même utiliter

Il nous faut d'abord créer un dossier `C:/users/xxxxx/CUB/scripts`, remplir ce dossier avec les commandes **powershell** suivantes :

```powershell
Write-Host "Diagnostic réseau CUB"
hostname
Get-Date
Get-NetIpConfiguration
```

Puis initialiser le depot :

```powershell
cd C:/users/xxxxx/CUB/scripts
git init
git status
```

``Git init`` permet de transformer le fichier en dépot suivi par le système de versionning de Git.
``Git status`` permet de connaitre l'état du dépot : Si des fichiers sont à commit, les commits précedent, sur quelle branche on se situe

## 4. Ajout / Commit

Nous allons par la suite ajouter le fichier au suivi de GIT aprés l'avoir initialiser avec `git init` :

```
git add diagnostics-reseau.ps1
git status
```

Aprés la commande git add nous pouvons voir que le fichier et maintenant signaler comme nouveau fichier il est a présent répertorier dans le dépot des version GIT.

```
git commit -m "Création du script de diagnostic réseau"
git log
git log --oneline
```

Un commit permet d'envoyer le fichier dans le suivi de version de git.
L'écriture d'un commentaire sur le commit permet de savoir qu'elles modifications ont été apporter sans vérifier tout le document et tracer dans qu'elle version du document nous nous trouvons.

## 5. Modification

Essayons maitenant de modifier le fichier `diagnostics-reseau.ps1` en rajoutant les lignes suivantes :

```powershell
Write-Host "Diagnostic réseau CUB"
hostname
Get-Date
Get-NetIpConfiguration
Write-Host "Test de la pile TCP/IP"
ping 127.0.0.1
```

Nous vérifions les modifications du dépot avec :

```
git status 
git diff
```

Avec le git status nous pouvons voir que Git a détecter une modification du ficher ``diagnostics-reseau`` et git diff permet de voir ce qui à changer dand le fichier (les lignes supprimer en rouge et les lignes rajouter en vert ).

Rajoutons ces modifications au dépot :

```
git add .\diagnostics-reseau.ps1
git commit -m "Ajout du test TCP IP local"
git log -oneline
```

Avec le git log nous pouvons voir que deux versions sont maitenant disponible dans le dépot : 

```git
c01f22a (HEAD -> master) Ajout du test TCP IP local
ba23699 Création du scipt de diagnostic réseau
```

## 6. Fonctionnement Git

Pour comprendre Git il faut avant tout comprendre les commandes de base :

Git status : Permet de savoir l'état actuel du dépot
Git diff : Montre les modifications effectuer entre la version actuelle et la version enregistrer
Git add : Ajout le fichier pour être commit 
Git commit : Permet d'envoyer le fichier sur le depot avec un commentaire pour préciser les modifications / la raison de la création du fichier
Git log : Affiche tout les commit effectuer sur le dépot Git

## 7. Annulation 

Une des utiliter du versionning est de revenir sur un état précédent pour corriger une erreur.
Ajoutons une lignes dans le fichier sans commit 

```powershell
Write-Host "Test Incorrect"
```

Pour vérifier que la modification a bien été effectuer nous allons utiliser git diff et pout revenir a la version d'avant nous allons utiliser une nouvelle commande : ``git restore``

```
git diff
git restore diagnostics-reseau.ps1
git status
```

La commande ``restore`` va nous permettre de revenir à la dernière version enregistrer, en effectuant cette commande nous allons supprimer définitivement la modifications car nous n'avous pas commit le changement pour garder cette modification en mémoire ils fallait effectuer un ``git commit``.

## 8. GITHUB

GitHub est une platforme qui permet d'héberger ces projets et qui facilite le travail d'équipe tandis que Git est un outils de versionning qui s'installe directement sur votre machine pour sauvegarder les version de vos fichiers.

Il existe deux types de dépot sur GitHub, Privé qui n'est visible seulement par le créateur et les utilisateurs autoriser à visualiser et à modifier le dépot et Public qui est visible par tout les utilisateurs de GitHub et qui peut etre cloner par tout les utilisateurs sans restriction.

Une faille critique lorsque l'on publie des scripts en tant qu'administrateur réseau est de laisser des secrets ou des mots de passes sur les scripts visible par tout les utilisateurs.

En mesure d'exemple voici 3 informations sensisbles à ne pas partager sur GitHub :
    - Les secrets et mots de passes
    - Les configurations de switch / routeur / firewall
    - Des informations personnelles (Eviter le social Engineering)

## 9. Liaison Git / GitHub

Nous allons maitenant lier nos sauvegarde Git avec notre dépot GitHub en utilisant la nouvelle commande ``git remote add origin Adresse_du_depot``, Git remote add origin ajoute la branche local `origin` du dépot **CUB-Scripts-Administration** sur notre machine 

```
git remote add origin https://github.com/louisbiseray-bot/CUB-Scripts-Administration
git remote -v
```

La commande remote -v nous permet de voir les dépots distant (remote) qui sont initialiser.

```
git branch
git branch -M main  #Si le nom main n'apparait pas
git branch #Vérifier si la modification a été apporter
```

Maitenant nous voulons pousser notre projet sur GitHub en le passant de notre branche local (origin) sur notre branche distante (main), pour cela nous allons utiliser la commande `git push` qui nous permet d'envoyer notre projet sur GitHub.

```
git push -u origin main
```

C'est Git push qui permet d'envoyer les commits locaux vers GitHub, git commit permet seulement d'enregistrer des versions de fichier nous pouvons par la suite pousser ces enregistrements sur github avec git push.

## 9. Modification et Synchronisation

Nous allons rajouter une autre ligne a notre scritp :

```
Write-Host "Test de la passerelle"
Test-NetConnection 192.168.x.126
```

Et nous allons enregistrer cette modification dans git :

```
git status 
git --no-pager diff #Le --no-pager permet de seulement afficher le résultat
git add diagnostics-reseau
git commit -m "Ajout du test de la passerelle"
```

PS : Cette modification n'est pas visible sur GitHub tant que le push n'est pas encore effectuer.

```
git push origin main
git log --oneline
```

Résultat attendu : 

```
86d2b9b (HEAD -> main, origin/main, origin/HEAD) Ajout du test de la passerelle
ea08518 Ajout du diagnostic DNS
c01f22a Ajout du test TCP IP local
ba23699 Création du scipt de diagnostic réseau
```

## 10. Récupération d'un dépot

Pour récupérer un dépot GitHub déja existant nous allons utiliser la commande ``git clone Adresse_du_depot``, la différence entre `clone` et `init` est que init permet de créer un depot tandis que clone le copie sur la machine dans le dossier dans le quel on se situe.

Dans les deux cas avant de travailler sur un depot il faut utiliser la commande `pull` qui permet de récupérer les dernière modification du dépot, dans le cas contraire git nous interdit de pousser notre travail pour eviter les conflits de noms et d'emplacements.

## 11. Branches

Nous allons créer une nouvelle branche dédier pour les diagnostics DNS nommer ``feature-diagnostic-dns`` le feature devant est une convention de nommage pour le branche GitHub.

```
git switch -c feature-diagnostic-dns
git branch
```

``Git branch`` nous permet de savoir dans qu'elle branche nous nous situons et ``git switch`` nous permet de changer de branche, dans ce cas d'utilisation de ``git switch`` nous rajoutons ``-c`` pour créer la branche **feature-diagnostic-dns**, pour voir qu'elle branche est utiliser lors d'un git branch une petite etoile est afficher devant la branch actuelle.

L'intéret d'une branche est de tester les features avant de les lancer sur la branche main qui est la branche de production et cela évite également de rentrer en conflit avec le travail d'un autre utilisateur.

Nous allons tester cette branche en ajoutant un test DNS dans notre scripts :

```powershell
Write-Host "Test de résolution DNS"
Resolve-Dnsname www.google.com
```

Et l'ajouter sur notre branche :

```
git add diagnostics-reseau.ps1
git commit -m "Ajout du test de résolution DNS"
git switch main
git merge feature-diagnostic-dns
git log --oneline --graph --all
```

Une nouvelle commande qui permet de mélanger deux branches est ``git merge`` qui permet de mélanger les fichier des deux branches de manière fluide sans avoir à tout réecrire.


## 14. Pull-Request

Créons une nouvelle branche nommer `feature-informations-systeme`.

```
git branch -c `feature-informations-systeme`
````

Ajoutons au script une nouvelle feature :

```
Write-Host "Informations Systeme"
Get-ComputerInfo
```

Nous allons commit les changement et les ajouter dans la branche que nous venons de créer :

```
git add diagnostic-reseau.ps1
git commit -m "Ajout des informations système"
git push -u origin feature-informations-systeme
```

Maintenant cette branche est visible sur GitHub avec toute les modifications apporter avec Git, contrairement a une branche locale celle ci est consulatable par tout les utilisateurs de GitHub.

Sur GitHub nous allons initier une Pull-Request, l'intéret d'une Pull-Request est de visualiser les changement puis de les valider avant qu'ils n'arrivent sur la branche main en duo les utilisateurs vont valider ou non la Pull-Request de l'autre, l'interet principal est surtout de vérifier que aucun conflit ou mauvaise pratique ne vas se mélanger à la branche main.

## 15. Conflit 

Lors d'un merge des conflits peuvent apparaitre dans ce cas l'administrateur du depot ou le propriétaire de la branche doit trancher entre les deux versions, une fois la résolutions faite à la main il faut commit pour enregistrer les changements. Git ne peut choisir de meilleur version car il lui est impossible de savoir laqu'elle modification est la plus pertinante pour le projet.

## 16. Synthèse

git init : Initialise un depot dans le dossier où la commande a été entrer
git status : Connaitre le status en cours du GIT
git diff : Voir les modifications apporter au fichier avant de commit
git add : Ajoute ce fichier au Git
git commit : Envoie la version actuel du fichier dans Git
git log --oneline : Montre tout les commit en une seule ligne chacun
git restore : Ramene le dossier a sa version antérieure
git clone : Clone un depot dans le dossier
git pull : Récupère la dernière version du dépot Git
git push : Envoi les commit vers le dépot
git branch : Savoir dans qu'elle branche on se situe
git switch : Permet de changerde branche
git merge : Fusionne une branche avec la branch main

Git : outil qui suit l’historique des modifications d’un projet.
GitHub : site qui héberge des projets Git et permet de collaborer.
Commit : enregistrement d’un ensemble de modifications dans l’historique.
Branche (branch) : copie de travail parallèle pour développer ou tester sans modifier directement la branche principale.
Fusion (merge) : opération qui réunit les modifications de deux branches.
Pull request : proposition de fusionner une branche dans une autre, souvent après relecture et discussion sur GitHub.

Un Git add suivi d'un git commit n'envoie pas la version enregistrer sur le depot GitHub et par conséquent les autres administrateurs n'ont pas la visibiliter car git add sert a jouter le fichier au versionning de Git et commit enregistre dans Git mais pour l'envoyer.

Pour développer une nouvelles fonctionnalité il est imperatif de créer une nouvelle branche pour pouvoir développer cette partie sans conflit et sans risque de casser le projet.