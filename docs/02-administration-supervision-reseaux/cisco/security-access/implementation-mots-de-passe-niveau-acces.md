# Implémentation des mots de passe par niveau d'accès (Cisco)

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 06/10/2026
- **Domaine :** Réseaux

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Chiffrement global de la configuration](#3-chiffrement-global-de-la-configuration)
- [4. Variante 1 : Gestion par mots de passe partagés](#4-variante-1-gestion-par-mots-de-passe-partages)
- [5. Variante 2 : Gestion par base de données locale (Nominatif)](#5-variante-2-gestion-par-base-de-donnees-locale-nominatif)
- [6. Variante 3 : Séparation des privilèges (Enable vs Conf t)](#6-variante-3-separation-des-privileges-enable-vs-conf-t)

## 2. Contexte

Le durcissement des accès sur un équipement réseau Cisco (commutateur ou routeur) impose de verrouiller chaque niveau d'interaction. L'accès à la modification (`conf t`) est nativement protégé par le privilège d'administration maximum (niveau 15). Cette procédure détaille trois approches : les mots de passe partagés, la base de données locale (recommandée SRE), et la ségrégation des accès par niveaux de privilèges personnalisés pour dissocier la consultation (`enable`) de la modification (`conf t`).

## 3. Chiffrement global de la configuration

3.1.  **Activer le chiffrement des mots de passe.** Application d'un algorithme de brouillage à tous les mots de passe configurés en clair (Type 0) sur l'équipement.

> [!warning] Chiffrement faible (Type 7)
> L'algorithme utilisé par `service password-encryption` (Type 7) est réversible. Il ne sert qu'à masquer les identifiants lors d'une lecture par-dessus l'épaule (`show run`). La déclaration des mots de passe via l'argument `secret` (Type 5 ou 9) doit toujours être priorisée.

```text title="global_encryption.txt" hl_lines="3"
Switch> enable
Switch# configure terminal
Switch(config)# service password-encryption
```

- `enable` : Élévation vers le mode d'exécution privilégié.
- `configure terminal` : Accès au mode de configuration globale.
- `service password-encryption` : Directive globale convertissant instantanément tous les mots de passe stockés en texte clair dans le fichier de configuration.

## 4. Variante 1 : Gestion par mots de passe partagés

4.1.  **Sécuriser l'accès de base (Console).** Définition d'un mot de passe unique exigé lors du branchement physique d'un câble sur le port console.

```text title="console_shared.txt" hl_lines="2-3"
Switch(config)# line con 0
Switch(config-line)# password AccessSwitch!
Switch(config-line)# login
```

- `line con 0` : Entre dans le sous-mode de configuration de l'unique port console matériel.
- `password [Secret]` : Assigne le mot de passe partagé exigé à la connexion.
- `login` : Force le commutateur à vérifier le mot de passe avant d'accorder l'accès au mode utilisateur (Privilège 1).

4.2.  **Sécuriser l'élévation de privilèges globale.**

```text title="enable_shared.txt" hl_lines="1"
Switch(config)# enable secret P@ssw0rdEnable!
```

- `enable secret [Secret]` : Génère un hachage fort (Type 5/9) protégeant l'accès direct au privilège 15 (incluant nativement `conf t`).

## 5. Variante 2 : Gestion par base de données locale (Nominatif)

5.1.  **Créer des comptes utilisateurs dédiés.** Provisionnement d'identités locales avec gestion fine des droits d'administration (RBAC).

```text title="local_db_users.txt" hl_lines="1-2"
Switch(config)# username admin_sre privilege 15 secret SreSecret2026!
Switch(config)# username stagiaire privilege 1 secret ReadOnly!
```

- `username [Nom]` : Déclare un nouvel identifiant dans la base SAM locale de l'IOS.
- `privilege [Niveau]` : Assigne le niveau de droits au compte (1 = Utilisateur, 15 = Administrateur ayant un accès direct à `conf t`).
- `secret [Secret]` : Associe un mot de passe haché cryptographiquement à l'utilisateur.

5.2.  **Appliquer l'authentification locale.**

> [!success] Conformité et Auditabilité
> L'utilisation de comptes locaux garantit l'imputabilité des actions réseau via les logs systèmes (Syslog).

```text title="local_auth_apply.txt" hl_lines="2 6"
Switch(config)# line con 0
Switch(config-line)# login local
Switch(config-line)# exit
Switch(config)# line vty 0 4
Switch(config-line)# transport input ssh
Switch(config-line)# login local
```

- `login local` : Exige la saisie d'un identifiant et d'un mot de passe de la base locale.
- `line vty 0 4` : Configure les lignes virtuelles pour l'accès distant.
- `transport input ssh` : Désactive Telnet au profit exclusif du protocole chiffré SSH.

## 6. Variante 3 : Séparation des privilèges (Enable vs Conf t)

> [!info] Fonctionnement de la ségrégation
> Cisco IOS ne permet pas de mettre un mot de passe directement sur la commande `conf t`. La commande est liée au niveau de privilège 15. Pour séparer l'accès "Enable" de l'accès "Config", on attribue un mot de passe à un niveau intermédiaire (ex: niveau 7 pour l'observation) et un autre pour le niveau 15.

6.1.  **Définir des mots de passe distincts par niveau.**

```text title="privilege_levels.txt" hl_lines="1-2"
Switch(config)# enable secret level 7 MdpEnable!
Switch(config)# enable secret level 15 MdpConfT!
```

- `enable secret level 7` : Crée un mot de passe spécifique pour basculer au niveau de privilège intermédiaire 7 (accessible via la commande `enable 7`).
- `enable secret level 15` : Crée un mot de passe distinct pour le niveau d'administration suprême (accessible via `enable 15`), seul niveau autorisé par défaut à exécuter `configure terminal`.

6.2.  **Autoriser des commandes pour le niveau intermédiaire.** Par défaut, le niveau 7 n'a pas plus de droits que le niveau 1. Il faut explicitement lui autoriser les commandes d'observation souhaitées.

```text title="privilege_commands.txt" hl_lines="1"
Switch(config)# privilege exec level 7 show running-config
```

- `privilege exec level 7` : Transfère la commande spécifiée (ici `show running-config`, normalement réservée au niveau 15) vers le niveau 7. L'utilisateur tapant `enable 7` pourra ainsi lire la configuration sans pouvoir la modifier.
