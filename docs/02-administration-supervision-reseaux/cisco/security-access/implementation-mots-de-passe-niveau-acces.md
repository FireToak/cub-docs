# Implémentation des mots de passe par niveau d'accès (Cisco)

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
* [3. Chiffrement global de la configuration](#3-chiffrement-global-de-la-configuration)
* [4. Sécurisation du mode d'exécution privilégié (Enable)](#4-securisation-du-mode-dexecution-privilegie-enable)
* [5. Sécurisation de l'accès physique (Console)](#5-securisation-de-lacces-physique-console)
* [6. Sécurisation du port de secours (AUX)](#6-securisation-du-port-de-secours-aux)

## 2. Contexte

Le durcissement des accès locaux sur un équipement Cisco (commutateur ou routeur) nécessite de verrouiller chaque point d'entrée matériel et logique. Une configuration propre proscrit l'utilisation de mots de passe en clair (`enable password`). Cette procédure standardise la sécurisation de l'élévation de privilèges et des ports d'administration physiques pour l'infrastructure CUB, en garantissant le chiffrement systématique des secrets dans le fichier de configuration (`running-config`).

## 3. Chiffrement global de la configuration

3.1. **Activation du chiffrement des mots de passe.** Cette commande applique un algorithme de chiffrement (Type 7) à tous les mots de passe actuels et futurs configurés en clair sur l'équipement.

!!! WARNING
    Le chiffrement de Type 7 est réversible et considéré comme faible. Il ne sert qu'à masquer les mots de passe lors d'une lecture par-dessus l'épaule (`show run`), c'est pourquoi il faut privilégier la commande `secret` (Type 5 ou 9) dès que possible.

```text
Router> enable
Router# configure terminal
Router(config)# service password-encryption

```

* `enable` : Passage en mode d'exécution privilégié.
* `configure terminal` : Passage en mode de configuration globale.
* `service password-encryption` : Chiffre instantanément tous les mots de passe en clair (console, aux, vty) dans le fichier de configuration.

## 4. Sécurisation du mode d'exécution privilégié (Enable) {#4-securisation-du-mode-dexecution-privilegie-enable}

4.1. **Verrouillage de l'élévation de privilèges.** Définition d'un mot de passe robuste pour protéger l'accès au mode administrateur.

```text
Router(config)# enable secret M0tDeP@sseF0rt!

```

* `enable secret` : Crée un mot de passe haché avec MD5 (Type 5) ou Scrypt/SHA (Type 9 selon l'IOS). Remplace et annule la commande non sécurisée `enable password`.
* `[M0tDeP@sseF0rt!]` : Chaîne de caractères constituant le secret d'administration.

## 5. Sécurisation de l'accès physique (Console) {#5-securisation-de-lacces-physique-console}

5.1. **Configuration du port console (CON 0).** Obligation de s'authentifier lors du branchement physique d'un câble console (RJ45/Série) sur l'équipement.

!!! TIP
    Bien que la méthode du mot de passe partagé soit présentée ici, la norme SRE recommande l'utilisation de `login local` pour lier l'accès physique à des comptes nominatifs.

```text
Router(config)# line con 0
Router(config-line)# password ConsoleSecurisee!
Router(config-line)# login
Router(config-line)# logging synchronous
Router(config-line)# exec-timeout 10 0

```

* `line con 0` : Entre dans le mode de configuration de l'unique port de console série (ligne 0).
* `password [mot_de_passe]` : Définit le mot de passe partagé de la ligne (qui sera chiffré grâce à l'étape 3.1).
* `login` : Active la vérification du mot de passe à la connexion.
* `logging synchronous` : Empêche les messages logs du système (syslog) d'interrompre la ligne de commande en cours de frappe.
* `exec-timeout 10 0` : Déconnecte la session console après 10 minutes et 0 seconde d'inactivité.

## 6. Sécurisation du port de secours (AUX) {#6-securisation-du-port-de-secours-aux}

6.1. **Configuration du port auxiliaire (AUX 0).** Sécurisation du port modem souvent présent sur les routeurs Cisco, qui représente une faille critique s'il est laissé sans mot de passe.

```text
Router(config-line)# exit
Router(config)# line aux 0
Router(config-line)# password AuxSecurise!
Router(config-line)# login
Router(config-line)# exec-timeout 5 0
Router(config-line)# end
Router# write memory

```

* `line aux 0` : Entre dans le mode de configuration du port auxiliaire.
* `password [mot_de_passe]` : Assigne un mot de passe d'accès pour ce port.
* `login` : Rend le mot de passe obligatoire pour établir la connexion.
* `exec-timeout 5 0` : Expulse l'utilisateur après 5 minutes d'inactivité, réduisant la fenêtre d'attaque physique.
* `end` : Retour immédiat au mode d'exécution privilégié.
* `write memory` : Sauvegarde les paramètres de sécurité dans la NVRAM.
