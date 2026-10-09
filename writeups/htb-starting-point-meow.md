
# DÉBUT DU WRITE-UP

# HTB Starting Point : MEOW

| | |
|---|---|
| **Plateforme** | Hack The Box, Starting Point, Tier 0 |
| **Date** | 8/10/2026 |
| **Système** | Linux |
| **Difficulté** | Very Easy |
| **Durée** |15 minutes|
| **Service visé** |Telnet|
| **Port** | 23/tcp |


## 1. Contexte

Meow est la première machine de Starting Point. Elle montre ce qui arrive quand un service d'administration à distance ancien (Telnet) est exposé et que le compte administrateur n'est pas protégé par un mot de passe. L'objectif est de se connecter à la machine et de récupérer le fichier flag.txt.

## 2. Reconnaissance

Je me connecte au réseau du lab (Pwnbox ou OpenVPN), je lance la machine et je note son adresse IP. Je vérifie d'abord que la cible répond :
```bash
ping IP_DE_LA_MACHINE
```

Puis je scanne les ports et les services :

```bash
nmap -sV -sC IP_DE_LA_MACHINE
```
Résultat utile :

PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd

## 3. Exploitation

1. Je me connecte au service : telnet IP_DE_LA_MACHINE. La machine affiche une bannière puis demande un nom d'utilisateur (Meow login:).
2. Je teste les noms de comptes administrateur les plus courants, un par un, en laissant le mot de passe vide : admin, administrator, puis root.
3. admin et administrator sont refusés. Le compte root est accepté sans mot de passe et j'obtiens un shell.
4. Je liste le dossier avec ls, je trouve le fichier flag.txt et je le lis avec cat flag.txt.

Où se trouve le flag : dans flag.txt, dans le dossier où l'on arrive après la connexion en root.

## 4. Ce qui n'a pas marché

Je n’ai pas rencontré de blocage majeur. Les seuls essais ratés ont été les comptes admin et administrator, avant d’arriver à root.



## 5. Ce que j'ai appris

- Le scan de ports est la première étape. Avant de chercher une faille, il faut savoir quels services la machine expose. Ici, nmap -sV a montré un seul port ouvert (23, Telnet), et c’était la seule piste.

- Telnet est dangereux. Il n’est pas chiffré (les mots de passe circulent en clair) et il donne un accès à distance à la machine. Il a été remplacé par SSH, qui chiffre les échanges.

- On teste d’abord les comptes par défaut. Les comptes comme admin, administrator et root sont souvent laissés avec un mot de passe vide ou faible, et c’est une erreur de configuration fréquente.

- La défense compte autant que l’attaque. Le correctif tient en trois mesures : désactiver Telnet, fixer un mot de passe sur chaque compte et filtrer le port 23.

## Outils utilisés

`nmap`, `ping`, `telnet`

