___

**Auteur :** BISERAY Louis
**Date :** 02/09/2026
**Sujet :** Analyse des besoins d'une entreprise pour la mise en place du SOC

___
## Ressources / Synthèse

- Synthèse 1 :  Les attaquants et leurs Outils
- Synthèse 2 :  Les menaces et les attaques Communes

___ 

**2. Expliquer ce qui a poussé le service RSI à opter pour une solution UTM par rapport à un simple pare-feu stateful traditionnel**

- Un pare-feu traditionnel est un service permet de filtrer et de contrôler les trafics entrants et sortants et d'y appliquer des règles précise dans une machine ou un réseau, dans notre cas nous nous plaçons dans un réseau. 
   Un pare-feu **UTM** lui est plus qu'un unique service c'est une solution de sécurité qui contient   plusieurs outils ainsi qu'un pare-feu traditionnel permettant la surveillance et la sécurisation d'un réseau l'**UTM** agit également sur des couches supérieur sur le modèle **OSI** atteignant la couche **7** contre la couche **4** pour un pare-feu classique.

   La Raison de l'utilisation d'une solution **UTM** plutôt qu'une solution de pare-feu stateful est qu'un **pare-feu** traditionnel est limiter à la couche **4** du modèle **OSI**, une solution **UTM** elle regroupe beaucoup d'outils pour faire face aux attaques et pirates comme : Un **logiciel anti-virus**, une **protection spam** et elle inclut également un pare-feu permettant à CUB d'avoir accès a beaucoup d'outils puissant sans avoir besoin de mettre en place d'autres services augmentant la surface de vulnérabilité.

   Une solution **UTM** permet de couvrir un champ d'action bien plus grand et d'apporter à une entreprise une solution globale permettant de centraliser facilement les journaux d'attaques, cependant centraliser autant d'outils peut générer un **SPOF** (Single Point Of Failure), avec un UTM nous limitons la surface d'attaque mais nous ne corrigeons pas le problème nous ne faisons que le déplacer.


**3. Donner 2 arguments en faveur d'un boîtier UTM Stormshield par rapport  à ceux proposer par des entreprises concurrentes telles que Palo Alto ou CheckPoint**

- Tout d'abord un l'entreprise **Stormshield** est une entreprise Française contrairement à **Palo Alto** qui est une entreprise Américaine et **Checkpoint** qui est une entreprise Israélienne, les objectifs de l'entreprise CUB est de maximiser la **souveraineté** de sont système informatique et la **réglementation** de l'**ANSSI** recommande d'utiliser des services Français pour une simplicité législative les lois informatiques ne s'applique pas pareilles dans différents pays, notamment les **Etats** **Unis** et la **France** qui ont des **réglementation** très différentes.

  Il est donc plus avantageux pour CUB de choisir Stormshield qui est une entreprise Française et qui leurs permettras de vanter une sécurité souveraine.

| Entreprise                     | Stormshield | Palo Alto  | Checkpoint |
| ------------------------------ | ----------- | ---------- | ---------- |
| **Pays**                       | France      | Etats-Unis | Israel     |
| **Prix**                       | 1 200 €     | 2 500 €    | 3 000 €    |
| **Formations / Security Pack** | 3 000 €     | 1 200 €    | 2 500 €    |
| **Total**                      | 4 200 €     | 3 700 €    | 5 500 €    |

- Les formations dispenser par Stormshield sont solide, complète et accompagner par des professionnels de Stormshield ainsi que du milieu du travail ce qui permet aux personnes former de connaitre la technologie mais également de savoir quand la déployer dans un contexte professionnel.

  Pour les entreprises un suivis et une formation solide sont très importante pour que les employer puisse utiliser les technologies de Stormshield tout en respectant les règles de sécurité.

**4. Expliquer pourquoi la présence d'un réseau local unique pose des problèmes de sécurité**

- La présence d'un seul réseau local est dangereux, sans segmentation du réseau un pirate qui s'infiltre dans le systèmes aurait accès à tout les services sans limitations, une segmentation permet de limiter les accès et d'autoriser seulement les services qui ont besoin de communiquer entre eux et en cas de piratage de limiter la zone d'action du pirate.

  Pour segmenter le réseau nous allons utiliser le VLSM qui consiste à séparer une plage d'adresse en d'autres plus petites.

**5. Réaliser le schéma logique représentant la nouvelle proposition de segmentation**

- Dans une prévision d'évolution du réseau il faut prévoir le double du nombre d'hôtes prévu

| Services | Production       | Clients         | AdminSys      |
| -------- | ---------------- | --------------- | ------------- |
| Hôtes    | 60 x 2 = **120** | 16 x 2 = **32** | 3 x 2 = **6** |

| Services        | Production        | Clients         | AdminSys      |
| --------------- | ----------------- | --------------- | ------------- |
| Ajout Broadcast | 120 + 1 = **121** | 32 + 1 = **33** | 6 + 1 = **7** |

- Voici le schéma du réseau logique 

| ID              | 55                  | 10                    | 20                    |
| --------------- | ------------------- | --------------------- | --------------------- |
| Services        | Production          | Clients               | AdminSys              |
| Masque          | 255.255.255.128 /25 | 255.255.255.192 /26   | 255.255.255.240 /28   |
| Plage d'adresse | 192.168.5.**1-126** | 192.168.5.**128-190** | 192.168.5.**192-206** |
| Passerelles     | 192.168.5.**126**   | 192.168.5.**190**     | 192.168.5.**206**     |
| Broadcast       | 192.168.5.**127**   | 192.168.5.**191**     | 192.168.5.**207**     |

- Ainsi que la  justification des calculs du sous réseau de Production

| Valeurs        | 128 | 64  | 32  | 16  | 8   | 4   |
| -------------- | --- | --- | --- | --- | --- | --- |
| Puissance de 2 | 7   | 6   | 5   | 4   | 3   | 2   |

Nombre d'hôtes : 128 > 120 => 7
Masque : 32 - 7 = 25 => /25
Plage d'adresse : 192.168.5.1-127
Passerelle : 128 - 2 => .126
Broadcast : 128 - 1 => .127