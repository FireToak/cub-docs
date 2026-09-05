# Implémentation du SSH sur un commutateur Cisco

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

* **Auteur :** Louis MEDO
* **Date :** 05/09/2026
* **Domaine :** Réseaux

---

## 1. Sommaire

* [1. Sommaire](#1-sommaire)
* [2. Contexte](#2-contexte)
* [3. Configuration des prérequis globaux](#3-configuration-des-prerequis-globaux)
* [4. Sécurisation cryptographique](#4-securisation-cryptographique)
* [5. Authentification et paramètres SSH](#5-authentification-et-parametres-ssh)
* [6. Sécurisation des accès virtuels (VTY)](#6-securisation-des-acces-virtuels-vty)

## 2. Contexte

Le déploiement de SSH (Secure Shell) sur les commutateurs Cisco vise à remplacer le protocole Telnet, qui transmet les données en clair. Cette configuration garantit un accès distant chiffré et sécurisé pour l'administration de l'équipement. Elle s'intègre dans la politique de sécurité globale de l'infrastructure CUB, en prévenant les interceptions de trames de gestion sur le réseau local et en centralisant l'authentification sécurisée.

## 3. Configuration des prérequis globaux {#3-configuration-des-prerequis-globaux}

3.1. **Configuration du nommage et désactivation DNS**. L'équipement a besoin d'un nom d'hôte et d'un nom de domaine uniques pour générer la clé cryptographique. La résolution DNS est désactivée pour éviter les temps d'attente lors d'erreurs de saisie.

```text
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-CORE-01
SW-CORE-01(config)# ip domain-name cub.local
SW-CORE-01(config)# no ip domain-lookup

```

* `enable` : Passage en mode d'exécution privilégié.
* `configure terminal` : Passage en mode de configuration globale.
* `hostname` : Définit le nom de l'équipement sur le réseau.
* `ip domain-name` : Spécifie le suffixe DNS du domaine réseau.
* `no ip domain-lookup` : Empêche le commutateur de chercher un serveur DNS pour traduire les commandes mal tapées en adresses IP.

## 4. Sécurisation cryptographique {#4-securisation-cryptographique}

4.1. **Génération de la paire de clés RSA**. Création du trousseau de clés asymétriques nécessaire au chiffrement du tunnel SSH.

```text
SW-CORE-01(config)# crypto key generate rsa modulus 2048

```

* `crypto key generate rsa` : Lance le moteur de création de clés pour l'algorithme RSA.
* `modulus 2048` : Définit la taille de la clé à 2048 bits, norme standard actuelle pour une sécurité robuste (minimum 1024 requis pour SSHv2).

## 5. Authentification et paramètres SSH {#5-authentification-et-parametres-ssh}

5.1. **Création du compte local et renforcement du protocole**. Création d'un administrateur local sécurisé et application des bonnes pratiques (Version 2).

!!! WARNING
    La version 1 de SSH comporte des failles de sécurité connues. Il est critique de forcer l'usage exclusif de la version 2.

```text
SW-CORE-01(config)# username admin privilege 15 secret M0tDeP@sseF0rt!
SW-CORE-01(config)# ip ssh version 2
SW-CORE-01(config)# ip ssh time-out 60
SW-CORE-01(config)# ip ssh authentication-retries 3

```

* `username [nom]` : Déclare l'identifiant local.
* `privilege 15` : Octroie immédiatement les droits d'administration totaux à cet utilisateur.
* `secret [mot_de_passe]` : Stocke le mot de passe dans la configuration de manière hachée (Type 5 ou Type 9 selon l'IOS), contrairement à `password` qui le laisse vulnérable.
* `ip ssh version 2` : Désactive SSHv1 et force SSHv2.
* `ip ssh time-out 60` : Ferme la session après 60 secondes d'inactivité.
* `ip ssh authentication-retries 3` : Limite à 3 le nombre d'échecs de connexion avant de rejeter la requête (protection brute-force).

## 6. Sécurisation des accès virtuels (VTY) {#6-securisation-des-acces-virtuels-vty}

6.1. **Restriction des lignes d'administration**. Activation de l'authentification pour l'accès distant et blocage définitif de Telnet.

```text
SW-CORE-01(config)# line vty 0 15
SW-CORE-01(config-line)# login local
SW-CORE-01(config-line)# transport input ssh
SW-CORE-01(config-line)# end
SW-CORE-01# write memory

```

* `line vty 0 15` : Entre dans la configuration des 16 interfaces de terminal virtuel (accès distant).
* `login local` : Indique au commutateur d'utiliser la base de données locale (créée à l'étape 5.1) pour vérifier les identifiants.
* `transport input ssh` : N'autorise que le protocole SSH en entrée. Rejette implicitement Telnet.
* `end` : Retour au mode privilégié.
* `write memory` : Sauvegarde la configuration active dans la NVRAM pour qu'elle persiste au redémarrage.
