# Configuration d'une Connexion SSH sur Guacamole

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 01/10/2026
- **Domaine :** Cybersécurité

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Déploiement d'une connexion SSH](#3-deploiement-dune-connexion-ssh)

## 2. Contexte

L'objectif de cette procédure est de documenter l'intégration sécurisée d'un équipement Linux via le protocole SSH au sein du bastion. L'opération inclut le renseignement des paramètres réseaux, l'affectation logique à un groupe de connexions pour le maintien de l'architecture RBAC, et la validation de l'étanchéité des droits d'accès entre les différents profils d'administration déléguée.

## 3. Déploiement d'une connexion SSH {#3-deploiement-dune-connexion-ssh}

3.1. **Initialisation des paramètres protocolaires.** Depuis l'interface web d'administration de Guacamole, naviguer dans le menu `Paramètres` > `Connexions`, puis cliquer sur le bouton `Nouvelle connexion`. Renseigner les caractéristiques de la machine cible :

- `Nom` : Définir un identifiant explicite respectant la convention de nommage (exemple : `ServeurDNS0`).
- `Protocole` : Sélectionner le protocole `SSH`.
- `Réseau` : Saisir l'adresse IP du serveur cible ainsi que le port de communication (par défaut `22`).

![Configuration connexion ssh](./assets/creer-connexion-ssh-linux/01-configuration-connexion-ssh.png)

3.2. **Affectation du groupe.** Aller dans `Utilisateurs` > `Selectionner un utilisateur` et ajouter le bon groupe. Afin de garantir l'application des stratégies de contrôle d'accès définies au niveau des rôles utilisateurs, descendre dans les paramètres de la connexion jusqu'à l'arborescence des groupes. Cochez l'appartenance de la nouvelle connexion au groupe correspondant à son environnement (exemple : groupe `Linux`).

> [!info] Structuration des accès
> L'intégration systématique des nouvelles ressources au sein de groupes logiques est indispensable. Cela garantit que les permissions associées aux groupes de sécurité (comme `ServeursLinux`) soient appliquées dynamiquement à chaque nouvel équipement sans action supplémentaire.

![Affection du groupe](./assets/creer-connexion-ssh-linux/02-affection-groupe.png)

3.3. **Validation.** Procéder à une vérification croisée des accès afin de s'assurer de la bonne ségrégation des flux et du respect du principe de moindre privilège.

- Se déconnecter de la session d'administration globale.
- S'authentifier avec le compte `adminlinux` : la connexion SSH nouvellement créée doit être visible et l'accès interactif au terminal doit être pleinement fonctionnel.
- S'authentifier avec le compte `adminwindows` : la connexion SSH doit être totalement invisible depuis l'interface de l'utilisateur.

> [!success] Validation de conformité
> Si la ressource n'est listée que pour les utilisateurs disposant explicitement des droits sur l'environnement Linux, la politique d'isolation RBAC est validée et opérationnelle.

![Validation de configuration](./assets/creer-connexion-ssh-linux/03-validation-configuration.png)
