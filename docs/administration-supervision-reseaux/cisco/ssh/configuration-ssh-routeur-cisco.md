# Implémentation du SSH sur un routeur Cisco

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

Le déploiement de SSH (Secure Shell) sur les routeurs Cisco vise à remplacer le protocole Telnet, qui fait transiter les flux d'administration en clair. Cette configuration garantit un accès distant chiffré de bout en bout pour l'administration de l'équipement de routage (routeur de bordure, passerelle inter-VLAN). Elle s'intègre dans la politique de sécurité globale de l'infrastructure CUB, en prévenant les interceptions de trames de gestion sur les réseaux LAN/WAN et en centralisant l'authentification sécurisée.

## 3. Configuration des prérequis globaux {#3-configuration-des-prerequis-globaux}

3.1. **Configuration du nommage et désactivation DNS**. L'équipement a besoin d'un nom d'hôte et d'un nom de domaine uniques pour générer la clé cryptographique asymétrique. La résolution DNS est désactivée pour éviter des requêtes inutiles et des temps de blocage lors d'erreurs de frappe dans l'invite de commande.

```text
Router> enable
Router# configure terminal
Router(config)# hostname RT-EDGE-01
RT-EDGE-01(config)# ip domain-name cub.local
RT-EDGE-01(config)# no ip domain-lookup

```

* `enable` : Passage en mode d'exécution privilégié.
* `configure terminal` : Passage en mode de configuration globale.
* `hostname` : Définit le nom de l'équipement de routage sur le réseau.
* `ip domain-name` : Spécifie le suffixe DNS du domaine réseau.
* `no ip domain-lookup` : Empêche le routeur d'interroger un serveur DNS pour tenter de traduire les commandes erronées en adresses IP.

## 4. Sécurisation cryptographique {#4-securisation-cryptographique}

4.1. **Génération de la paire de clés RSA**. Création du trousseau de clés asymétriques indispensable à la mise en place du chiffrement du tunnel SSH.

```text
RT-EDGE-01(config)# crypto key generate rsa modulus 2048

```

* `crypto key generate rsa` : Lance le moteur de création de clés pour l'algorithme de chiffrement RSA.
* `modulus 2048` : Définit la taille de la clé à 2048 bits, la norme minimale actuelle recommandée pour une sécurité robuste contre les attaques de force brute.

## 5. Authentification et paramètres SSH {#5-authentification-et-parametres-ssh}

5.1. **Création du compte local et renforcement du protocole**. Création d'un compte administrateur local hautement privilégié et application des paramètres de durcissement pour le service SSH.

!!! WARNING
    La version 1 de SSH comporte des failles de sécurité structurelles. Il est impératif de forcer l'usage exclusif de la version 2 en production.

```text
RT-EDGE-01(config)# username admin privilege 15 secret M0tDeP@sseF0rt!
RT-EDGE-01(config)# ip ssh version 2
RT-EDGE-01(config)# ip ssh time-out 60
RT-EDGE-01(config)# ip ssh authentication-retries 3

```

* `username [nom]` : Déclare l'identifiant de l'administrateur local.
* `privilege 15` : Accorde instantanément les droits d'administration complets (équivalent root) à cet utilisateur lors de la connexion.
* `secret [mot_de_passe]` : Stocke le mot de passe sous forme de hash cryptographique fort dans la configuration, protégeant contre la lecture en clair (contrairement à l'attribut `password`).
* `ip ssh version 2` : Désactive le protocole obsolète SSHv1 et impose l'utilisation de SSHv2.
* `ip ssh time-out 60` : Interrompt automatiquement la session d'administration après 60 secondes d'inactivité.
* `ip ssh authentication-retries 3` : Limite les tentatives de connexion à 3 échecs avant la rupture de la session TCP, mitigant les attaques par dictionnaire.

## 6. Sécurisation des accès virtuels (VTY) {#6-securisation-des-acces-virtuels-vty}

6.1. **Restriction des lignes d'administration VTY**. Activation de l'authentification locale pour l'accès distant et verrouillage des protocoles de management en n'autorisant exclusivement que SSH.

```text
RT-EDGE-01(config)# line vty 0 4
RT-EDGE-01(config-line)# login local
RT-EDGE-01(config-line)# transport input ssh
RT-EDGE-01(config-line)# end
RT-EDGE-01# write memory

```

* `line vty 0 4` : Accède au mode de configuration des 5 premières lignes de terminal virtuel (standard d'accès distant sur les routeurs Cisco).
* `login local` : Oblige le routeur à interroger la base de données locale (comptes configurés via `username`) pour authentifier les utilisateurs distants.
* `transport input ssh` : Restreint les flux de connexion en entrée au seul protocole SSH, rejetant explicitement et définitivement l'accès Telnet.
* `end` : Quitte le mode de configuration globale pour revenir au mode d'exécution privilégié.
* `write memory` : Sauvegarde la configuration active (running-config) dans la mémoire non volatile (NVRAM) pour la rendre persistante au prochain redémarrage.
