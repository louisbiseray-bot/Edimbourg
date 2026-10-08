___
### Informations Générales
  
**Auteur :** BISERAY Louis
**Date :** 09/09/2026
**Sujet :** Premier Paramétrage d'un pare-feu Stormshield sur un site d'entreprise

___

## Mise en place

Pour notre Contexte nous allons mettre en place un firewall Stormshield qui s'acquitteras de plusieurs tâches comme le filtrage, la NAT et la translation.

Avant tout il faut vérifier que le NTP soit diriger vers la capitale du pays dans le quel on se situe, ici ce sera Paris et penser a changer le mot de passe administrateur pour un mot de passe qui respecte les recommandation de l'ANSSI.
Par la suite redémarrer le Firewall pour appliquer la nouvelle configuration.

Secondement sur ce pare feu ils nous faut déclarer les interfaces IN et OUT ainsi que la DMZ, lors du changement pour l'interface IN ne pas oublier de changer sa propre adresse IP pour continuer a communiquer avec le Pare-Feu sinon l'accès sera perdu.

Enfin quand tout les étapes du dessus ont été faite il faut mettre en place les deux règles de NAT et de translation pour le LAN et l'Inter-Co les règles devront être comme ceci :

| Etat | Source   | Destination | Port Dest. |     | Source       | Port Sourc. | Destination | Port Dest. |
| ---- | -------- | ----------- | ---------- | --- | ------------ | ----------- | ----------- | ---------- |
| on   | LAN      | Internet    | Any        | =>  | Firewall_out | ephemera    | Any         |            |
| on   | Inter-Co | Internet    | Any        | =>  | Firewall_out | ephemera    | Any         |            |

## Questions

#### 6. Pourquoi est-il primordial que les firewall soit synchroniser sur des serveurs DMZ ?

Il est impératif que les firewall soit synchroniser sur les serveurs NTP de Stormshield pour pouvoir récupérer les dernière mise à jour du systèmes et de sécurité, mais également pour garder les logs à jour et à la bonne heure ainsi on possède une supervision des incidents clair et précis sur notre pare-feu.

#### 7. Interface Physiques

| Interface | Port | Type                 | Etat | Adresse IPv4      | Commentaire                            |
| --------- | ---- | -------------------- | ---- | ----------------- | -------------------------------------- |
| out       | 1    | Ethernet, 1 Gbit / s |      | 192.36.252.50/24  | Liaison WAN CUB                        |
| in        | 2    | Ethernet, 1 Gbit / s |      | 192.168.55.254/29 | Réseau Interne de l'agence d'Edimbourg |
| dmz1      | 3    | Ethernet, 1 Gbit / s |      | 192.36.5.254/24   | Liaison vers la DMZ                    |

#### 8. Filtrage NAT / Translation

| Etat | Source   | Destination | Port Dest. |     | Source       | Port Sourc. | Destination | Port Dest. |
| ---- | -------- | ----------- | ---------- | --- | ------------ | ----------- | ----------- | ---------- |
| on   | LAN      | Internet    | Any        | =>  | Firewall_out | ephemera    | Any         |            |
| on   | Inter-Co | Internet    | Any        | =>  | Firewall_out | ephemera    | Any         |            |
