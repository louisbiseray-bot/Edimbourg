___

**Auteur :** BISERAY Louis
**Date :** 03/09/2026
**Sujet :** Création d'une VM Debian 13 avec les modules requis pour la mise en place d'un modèle

___

## Introduction

Pour la mise en place de plusieurs machines nous allons avoir besoin d'un modèles permettant ainsi de cloner une machine Debian 13 déjà préparer sans avoir à reconfigurer tout manuellement à chaque installations de services

## Installations
### (Nom de catégorie approprier pour les outils)

Nous allons tout d'abord installer les services suivant :

- HTOP
- TCPDUMP
- TMUX
(Explique leurs utilités dans la machine)

### Enregistrement Logs

Nous aurons besoin d'un enregistreur de Logs pour pouvoir débugger la machine en cas d'erreur nous utiliserons le daemon RSYSLOG qui récupère les Log sous forme de fichier texte plus lisible que des fichier binaire mais il est plus difficile de trouver la source de l'erreur il nous faut jouer avec des GREP dans linux

Nous voulons que les log soit envoyer dans le dossier **/var/log** nous allons donc préciser a la fin dossier de configuration de **RSYSLOG** 

Ouvir le dossier de configuration de **RSYSLOG**
```bash
sudoedit /etc/rsyslog.conf
```

Entrer cette ligne a la fin du fichier de configuration
```bash
*.* /var/log/all.log
```

### Versioning

Dans une machine Linux il existe toujours le dossier **/etc** qui contient tout les dossiers et fichiers de configuration des services de la machine mais également le fichier **shadow** qui contient tout les mot de passes hacher des utilisateurs, il est donc impératif pour le bon fonctionnement du système que ce fichier reste intact et qu'en cas de mauvaise manipulation nous puissions revenir a un état stable.

Nous allons donc mettre en place un outils de versioning nommer etckeeper

```bash
sudo apt install etckeeper
```

Une fois l'installation terminer il faut initialiser le depot git et déclarer notre premier Commit

```bash
sudo etckeeper init
sudo etckeeper "First Commit"
```

#### Gestionnaire de Paquets

Il est possible de joindre a **etckeeper** un gestionnaire de paquets ainsi lorsque de nouveaux paquets sont installer sur **/etc** un commit est automatiquement effectuer pour éviter la casse lors de l'installation de nouveaux modules sur le système ce qui permettras de revenir à une version antérieur ou le modules n'était pas présent.

Nous utiliserons **ngnix**

```bash
sudo apt install nginx
```

Grace a **nginx** je peut maintenant revenir a une version antérieur a toute installations de paquets, il est possible de voir les logs Git maintenant que nous avons installer **nginx**.

```bash
cd /etc
sudo git log
```

Un commit effectuer lors de l'installation de **nginx** devrais apparaitre.

```bash
commit acd94001bbdb8ed8c0e3196e8c760566d8b546e9 (HEAD -> master)
Author: etudiant <etudiant@template.sio.lan>
Date:   Thu Sep 3 16:58:42 2026 +0200

    committing changes in /etc made by "apt install nginx"

    Packages with configuration changes:
    +nginx-common 1.26.3-3+dev13u7 all

    Package changes:
    +nginx 1.26.3-3+dev13u7 amd64
    +nginx-common 1.26.3-3+dev13u7 all
```

Pour restaurer a une version précedente 

```bash
cs /etc
sudo git log --oneline
```

```bash
e2d9c24 Ajout de la config SSL nginx
a7d0f91 Commit après update des paquets
1f3c6b2 Commit initial
```

L'enchainement de nombre et de lettre est l'ID du commit c'est avec lui que nous allons restaurer a une ancienne version.

```bash
sudo git chekout <ID_du_commit>
```

>[!warning] Retourner a une ancienne version remplace tout les fichier de /etc penser a faire un commit avant de revenir a une version antérieure.


### SSH / TOTP

Il faut pouvoir accéder a la machine avec SSH nous allons donc créer un utilisateur `adminbastion` qui aura les droit d'utiliser toutes les commandes systémes.

Installons d'abord SSH et créons l'utilisateur en lui rajoutant les droits sudo

```bash 
sudo apt install openssh-server
sudo adduser adminbastion
sudo moduser -aG sudo adminbastion
```

Une fois l'utilisateur et SSH installer nous allons tenter de nous connecter depuis une autre machine avec cet utilisateur pour verifier le fonctionnement.

```bash
ssh adminbastion@172.16.55.1
```

Pour se connecter il faut accepter l'empreinte de la machine ainsi que saisir le mot de passe utilisateur définis cependant l'accès aux machine n'est pas sécuriser avec un simple mot de passe.

Pour sécuriser la connexion nous allons ajouter une double authentification et pour cette double authentification nous allons utiliser l'Algorithme TOTP.
![[TOTP | 800x550]]

Pour pouvoir utiliser TOTP il faut installer FreeOTP Authentificator sur son téléphone et installer le paquet qrencode sur la machine Debian 13.

```bash
sudo apt install qrencode
```

Dans un premier il faut installer les paquets capable de mettre en place le mécanisme OTP appeler OATH.

```bash
sudo apt install libpam-oath oathtool
```

Nous allons par la suite définir un secret en hexadécimal qui devra être absolument garder secret sinon toute la sécurité de l'authentification TOTP vole en éclat.

```bash
sudo -i 
KEY=$(openssl rand -hex 20)
echo "HOTP/T30/6 etudiant - ${KEY}" >> /etc/security.users.oath
chown root /etc/security/user.oath
chown 600 /etc/security/users.oath
```

Il faut également configurer le PAM (Pluggable Authentification Modules), c'est ce service qui contrôle les authentifications sur le serveur Debian.

```bash
sudoedit /etc/pam.d/sshd
```

```bash
# PAM Configuration for the Secure Shell Service

# Standard Un*x authentification
#@ include common-auth

auth required pam_unix.so nullok_secure
auth [success=1 default=ignore] pam_succeed_if.so user = adminbastion
auth required pam_oath.so usersfile=/etc/security/users.oath window=30 digits=6
```

Il faut commenter la ligne `#@ include common-auth` pour autoriser l'authentification OTP

(Expliquer les lignes écrite dans le shell au dessus)

Il nous reste maintenant à éditer le fichier de configuration du service SSH afin de définir l’usage de l’authentification 2FA.

```bash
nano /etc/ssh/sshd_config
```

```bash
ChallengeResponseAuthentication yes
#KbdInteractiveAuthentication no
UsePAM yes
```

Nous nous assurons de commenter la ligne _KdbInteractiveAuthentication_ et d’avoir les deux autres lignes activées avec la valeur yes. Nous pouvons ensuite redémarrer le service SSH.

```
systemctl restart ssh
```

Enfin, il nous reste à récupérer le secret en base 32 qui nous permettra ensuite de générer sur le poste client un QR code pour notre application Android.

```bash
cat /etc/security/users.oath
```

```bash
HOTP/T30/6 etudiant – 65f43c705ce51c9c058ec8bb4b7f64b656681866
root@serveur:~# oathtool -v -d 6 65f43c705ce51c9c058ec8bb4b7f64b656681866
Hex secret: 65f43c705ce51c9c058ec8bb4b7f64b656681866
Base32 secret: MX2DY4C44UOJYBMOZC5UW73EWZLGQGDG
Digits: 6
Window size: 0
Start counter: 0x0 (0)
```

Il s’agit maintenant de paramétrer correctement la machine cliente et l’application Android FreeOTP afin de rendre opérationnel l’authentification SSH 2FA.

Nous allons d’abord générer sur le poste client (Ubuntu ou Kali) un fichier png contenant un QR code que nous soumettrons à l’application FreeOTP.

```bash
qrencode -o etudiant.png 'otpauth://totp/etudiant@192.168.1.90?secret=MX2DY4C44UOJYBMOZC5UW73EWZLGQGDG'
```

Nous pouvons ouvrir ce fichier PNG sur le poste client puis ouvrir l’application FreeOTP+ sur le smartphone.

Scanner le QR code et cliquer sur le bouton qui apparaitra pour dévoiler le one time password

>[!note] Ne pas oublier que ce mot de passe expire et que l'horloge des machines doivent être synchroniser

Sur le client, nous pouvons lancer une connexion SSH vers le serveur avec le compte etudiant. Après avoir entré votre mot de passe, un OTP vous est demandé. Dans l’application FreeOTP+, sélectionnez la nouvelle configuration. Celle-ci vous fournit un code de 6 chiffres valable 30 secondes.

```bash
ssh etudiant@192.168.1.90               
(etudiant@192.168.1.90) Password: 
(etudiant@192.168.1.90) One-time password (OATH) for `etudiant': 
Linux serveur 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jan  3 22:17:13 2024 from 192.168.1.85
```