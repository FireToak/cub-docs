# Implémentation du VTP sur commutateur Cisco

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

* **Auteur :** Louis MEDO
* **Date :** 05/09/2026
* **Domaine :** Réseaux

---

## 1. Sommaire

* [1. Sommaire](#1-sommaire)
* [2. Contexte (Fonctionnement du VTP)](#2-contexte-fonctionnement-du-vtp)
* [3. Configuration des prérequis (Liaisons Trunk)](#3-configuration-des-prerequis-liaisons-trunk)
* [4. Configuration du domaine VTP (Mode Serveur)](#4-configuration-du-domaine-vtp-mode-serveur)
* [5. Configuration des commutateurs secondaires (Mode Client)](#5-configuration-des-commutateurs-secondaires-mode-client)
* [6. Configuration de l'isolation (Mode Transparent)](#6-configuration-de-lisolation-mode-transparent)

## 2. Contexte (Fonctionnement du VTP)

Le protocole VTP (VLAN Trunking Protocol) de Cisco permet de centraliser et d'automatiser la gestion des VLANs (Création, modification, suppression) sur l'ensemble d'une infrastructure de commutation. Il assure la cohérence de la base de données VLAN via des annonces de couche 2 diffusées sur les liens Trunk.

Le protocole s'articule autour de trois concepts fondamentaux :

* **Le domaine VTP :** Un regroupement logique de commutateurs partageant la même configuration VLAN.
* **Le numéro de révision :** Le cœur du mécanisme. C'est un compteur incrémenté à chaque modification de la base VLAN. Un commutateur écrasera toujours sa base locale s'il reçoit une annonce VTP dont le numéro de révision est supérieur au sien.
* **Les modes de fonctionnement :**
* **Serveur (Défaut) :** Autorise la gestion totale des VLANs. Génère et transfère les annonces VTP.
* **Client :** Ne peut ni créer, ni modifier, ni supprimer de VLANs. Il synchronise sa base de données sur celle du Serveur et relaie les annonces.
* **Transparent :** Ne synchronise pas sa base de données avec le domaine. Il gère ses propres VLANs locaux, mais relaie les annonces VTP aux autres équipements pour ne pas rompre la chaîne.

## 3. Configuration des prérequis (Liaisons Trunk) {#3-configuration-des-prerequis-liaisons-trunk}

3.1. **Établissement des liaisons Trunk.** Les trames VTP ne transitent exclusivement que sur les interfaces configurées en Trunk (marquage 802.1Q). Il est impératif d'initialiser l'interconnexion avant le protocole.

```text
Switch-Core> enable
Switch-Core# configure terminal
Switch-Core(config)# interface GigabitEthernet1/0/1
Switch-Core(config-if)# switchport trunk encapsulation dot1q
Switch-Core(config-if)# switchport mode trunk
Switch-Core(config-if)# exit

```

* `switchport trunk encapsulation dot1q` : Force l'utilisation de la norme ouverte IEEE 802.1Q pour le tag des trames (étape requise sur certains commutateurs de niveau 3 avant de pouvoir activer le trunk).
* `switchport mode trunk` : Configure l'interface en mode agrégation statique de VLANs.

## 4. Configuration du domaine VTP (Mode Serveur)

4.1. **Déclaration du VTP Serveur.** Initialisation du domaine de diffusion sur le commutateur de cœur (Core), avec une sécurisation obligatoire par mot de passe pour prévenir les injections malveillantes.

!!! CAUTION
    L'insertion d'un commutateur "de récupération" avec un ancien numéro de révision VTP élevé écrasera intégralement les VLANs de la production. Réinitialisez toujours le numéro de révision d'un commutateur à 0 (en le passant temporairement en `vtp mode transparent`) avant de le brancher au réseau.

```text
Switch-Core(config)# vtp version 2
Switch-Core(config)# vtp domain CUB-INFRA
Switch-Core(config)# vtp password S3cur3VTP!
Switch-Core(config)# vtp mode server

```

* `vtp version 2` : Active la version 2 du protocole (vérifications de cohérence accrues). Tous les commutateurs du domaine doivent utiliser la même version.
* `vtp domain [Nom]` : Définit le nom du domaine de gestion. Il est sensible à la casse.
* `vtp password [Mot_de_passe]` : Chiffre les mises à jour VTP via un hash MD5. Empêche un commutateur non autorisé d'altérer la base VLAN de la production.
* `vtp mode server` : Assigne le rôle de maître de la topologie à l'équipement (mode par défaut, mais explicité par convention IaC).

## 5. Configuration des commutateurs secondaires (Mode Client)

5.1. **Raccordement d'un commutateur d'accès (Client).** Déploiement des paramètres sur les commutateurs périphériques pour qu'ils héritent automatiquement de la configuration dictée par le Serveur.

```text
Switch-Access> enable
Switch-Access# configure terminal
Switch-Access(config)# vtp version 2
Switch-Access(config)# vtp domain CUB-INFRA
Switch-Access(config)# vtp password S3cur3VTP!
Switch-Access(config)# vtp mode client

```

* `vtp mode client` : Verrouille la configuration VLAN locale de l'équipement. Le commutateur attendra les mises à jour en provenance du domaine `CUB-INFRA` validées par le hachage de `S3cur3VTP!` pour peupler sa table VLAN.

## 6. Configuration de l'isolation (Mode Transparent)

6.1. **Isolement d'un équipement de la synchronisation dynamique.** Configuration d'un commutateur pour qu'il gère ses VLANs indépendamment tout en laissant passer les trames VTP pour le reste de la grappe.

!!! INFO
    En production, le mode `transparent` est souvent généralisé à l'ensemble du parc réseau pour éviter tout risque de propagation non désirée, la configuration des VLANs étant plutôt gérée par des outils d'automatisation externes (Ansible, Python).

```text
Switch-Isol(config)# vtp domain CUB-INFRA
Switch-Isol(config)# vtp mode transparent
Switch-Isol(config)# end
Switch-Isol# write memory

```

* `vtp mode transparent` : Désolidarise le commutateur de la base de données commune. Son numéro de révision reste à 0. Il transmettra les annonces VTP reçues sur ses ports Trunk sans jamais les appliquer à sa propre configuration.
* `end` : Retour direct au mode d'exécution privilégié.
* `write memory` : Sauvegarde l'état du protocole dans la configuration de démarrage persistante.
