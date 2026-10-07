# Configuration de l'enregistrement des connexions

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 07/10/2026
- **Domaine :** Administration Windows

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Configuration de l'enregistrement](#3-configuration-de-lenregistrement)
- [4. Vérification](#4-verification)

## 2. Contexte

Cette procédure détaille la configuration de l'enregistrement systématique des sessions de connexion (incluant la capture vidéo et les événements clavier). Dans une démarche orientée SRE et sécurité, la traçabilité complète des interventions sur les serveurs garantit l'observabilité des actions administrateurs, facilite les audits de conformité et permet le rejeu des incidents.

## 3. Configuration de l'enregistrement

3.1.  **Sélection de la connexion.** Se rendre dans les Paramètres > Connexions, puis sélectionner la connexion cible à configurer.

![Sélection de la connexion](./assets/configuration-enregistrement-connexions/01-acces-configuration.png)

3.2.  **Paramétrage de la capture.** Dans la section *Paramètres de la connexion*, appliquer la configuration de l'enregistrement d'écran.

> [!info] Utilisation de variables
> L'utilisation des variables `${HISTORY_PATH}` et `${HISTORY_UUID}` permet d'automatiser et d'isoler le stockage des enregistrements par session.

Définir les paramètres suivants :

- Chemin de l'enregistrement : `${HISTORY_PATH}/${HISTORY_UUID}`
- Inclure les événements clavier : `Coché`
- Créer automatiquement le chemin d'enregistrement : `Coché`
- Autoriser l'écriture dans le fichier d'enregistrement existant : `Coché`

![Paramétrage de la capture](./assets/configuration-enregistrement-connexions/02-configuration-connexion.png)

## 4. Vérification {#4-verification}

4.1.  **Génération d'événements de test.** Démarrer une session sur un serveur depuis les connexions configurées et exécuter quelques commandes d'administration courantes pour générer de la donnée d'historique.

![Génération d'événements de test](./assets/configuration-enregistrement-connexions/03-connexions.png)

4.2.  **Consultation des logs d'enregistrement.** Naviguer vers Paramètres > Historique et cliquer sur *Voir* pour accéder aux enregistrements.

![Consultation des logs d'enregistrement](./assets/configuration-enregistrement-connexions/04-consultation-log-enregistrement.png)
