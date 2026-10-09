# HTB Starting Point : Fawn

| | |
|---|---|
| **Plateforme** | Hack The Box, Starting Point, Tier 0 |
| **Système** | Linux (Unix) |
| **Difficulté** | Very Easy |
| **Service visé** | FTP (vsftpd 3.0.3) |
| **Port** | 21/tcp |
| **Faille** | Accès FTP anonyme activé |
| **Outils** | `ping`, `nmap`, `ftp` |

## 1. Contexte

Fawn est la deuxième machine de Starting Point. Elle expose un serveur FTP (File Transfer Protocol), un protocole ancien de transfert de fichiers. Le serveur est mal configuré : il accepte les connexions anonymes. N'importe qui sur le réseau peut donc se connecter, lister les fichiers et les télécharger. L'objectif est de récupérer le fichier `flag.txt` stocké sur le serveur.

Cette machine montre un cas réel : un service de transfert de fichiers laissé ouvert « pour faciliter le travail » peut exposer des fichiers sensibles, comme des journaux, des sauvegardes ou des configurations.

## 2. Reconnaissance

Je vérifie d'abord que la cible répond sur le réseau du lab :

```bash
ping IP_DE_LA_MACHINE
```

Les réponses reçues (0 % de perte) confirment que la connexion est stable. Je scanne ensuite les ports :

```bash
nmap IP_DE_LA_MACHINE
```

```
PORT   STATE SERVICE
21/tcp open  ftp
```

Un seul port est ouvert : le 21, avec le service FTP. Pour connaître la version du service, je relance le scan avec la détection de version :

```bash
nmap -sV IP_DE_LA_MACHINE
```

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix
```

Analyse : le serveur FTP est `vsftpd 3.0.3`, sur un système Unix. Connaître la version permet de savoir si le logiciel est ancien ou vulnérable. Ici, on ne cherche pas à exploiter une vulnérabilité du logiciel : on va tester sa configuration.

## 3. Exploitation

1. **Vérifier que le client FTP est installé** sur ma machine d'attaque : `sudo apt install ftp -y`.
2. **Se connecter au serveur** : `ftp IP_DE_LA_MACHINE`. Le serveur affiche sa bannière (`vsFTPd 3.0.3`) puis demande un nom d'utilisateur.
3. **Tester le compte anonyme.** Sur un serveur FTP mal configuré, le compte `anonymous` est accepté et le mot de passe est ignoré. J'ai saisi `anonymous` comme nom d'utilisateur, puis un mot de passe quelconque.
4. **Connexion réussie** : le serveur répond `230 Login successful` et l'invite `ftp>` apparaît.
5. **Lister les fichiers** avec `ls`. Le dossier contient un seul fichier : `flag.txt`.
6. **Télécharger le fichier** : `get flag.txt`. Le serveur confirme `226 Transfer complete`.
7. **Quitter** avec `bye`, puis lire le fichier en local avec `cat flag.txt`.

Où se trouve le flag : dans le fichier `flag.txt`, à la racine du dossier accessible par le compte anonyme.

## 4. Points d'attention

- **Les codes de réponse FTP** donnent l'état de chaque commande : `220` (serveur prêt), `331` (mot de passe demandé), `230` (connexion réussie), `150` (le serveur prépare le transfert) et `226` (transfert terminé).
- **Le scan nmap simple n'affiche pas la version.** Il faut ajouter `-sV` pour identifier `vsftpd 3.0.3`.
- **Le message « Consider using PASV »** apparaît pendant les transferts. C'est un simple avertissement sur le mode de connexion, pas une erreur.
- **Le mot de passe saisi n'a pas d'importance** pour le compte `anonymous`. C'est justement ce qui rend la configuration dangereuse.
- **Le fichier est téléchargé dans le dossier courant** de ma machine, c'est-à-dire celui où j'ai lancé la commande `ftp`.

## 5. Comment corriger la faille

- **Cause** : le serveur FTP autorise l'accès anonyme. Toute personne qui atteint le port 21 peut lire et télécharger des fichiers sans compte. De plus, FTP n'est pas chiffré : les identifiants et le contenu des fichiers circulent en clair et peuvent être interceptés lors d'une attaque de type « homme du milieu » (MitM).
- **Correctif** :
  - désactiver l'accès anonyme dans la configuration du serveur (avec vsftpd : `anonymous_enable=NO`)
  - exiger une authentification avec des comptes nominatifs et des mots de passe forts
  - remplacer FTP par SFTP (transfert de fichiers sur SSH) ou par FTPS (FTP chiffré avec TLS)
  - limiter chaque utilisateur à son propre dossier
  - filtrer le port 21 avec le pare-feu, pour que seules les machines autorisées y accèdent
- **Détection** :
  - activer et surveiller les journaux du serveur FTP, pour repérer les connexions du compte `anonymous`
  - créer une alerte (par exemple avec Wazuh) sur toute connexion FTP anonyme
  - surveiller les téléchargements inhabituels de fichiers

## 6. Ce que j'ai appris

- **Un service exposé peut être dangereux même sans vulnérabilité logicielle.** Ici, le logiciel n'est pas en cause : c'est sa configuration qui est mauvaise.
- **Il faut toujours tester l'accès anonyme** sur un service FTP pendant l'énumération, avec `anonymous` comme nom d'utilisateur.
- **La détection de version (`-sV`) apporte de l'information utile** pour savoir si un service est obsolète et mérite une recherche de vulnérabilités.
- **FTP n'est pas sûr par défaut** : sans chiffrement, ses échanges sont lisibles par un attaquant sur le réseau. SFTP et FTPS sont les alternatives à privilégier.
- **Côté défense**, désactiver l'accès anonyme, chiffrer les transferts et surveiller les journaux suffisent à bloquer ce scénario.

---
*Aucun flag ni mot de passe réel n'est publié dans ce document.*
