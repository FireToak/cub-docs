---
icon: lucide/rocket
---

# Bienvenue 👋

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## 1. 🧭 Contexte

Le projet CUB concerne une entreprise spécialisée dans l'incubation de startups, disposant d'un siège social à Paris et de multiples agences internationales. Face aux évolutions de son infrastructure et aux menaces de cybersécurité comme le malware Emotet, la Direction des Systèmes d'Information déploie une refonte de son architecture réseau. Cette évolution s'articule autour d'une segmentation stricte des réseaux (VLAN, DMZ) et du remplacement des anciens pare-feux par des solutions de gestion unifiée des menaces (UTM) Stormshield, tout en appliquant les recommandations de l'ANSSI pour garantir la protection et la souveraineté des données.

---

## 2. 🗺️ Navigation

La présente documentation est séparée en 4 modules : [Administration et supervision des réseaux](./02-administration-supervision-reseaux/index.md), [Administration Windows](./04-administration-windows/index.md), [Cybersécurité](./05-cybersecurite/index.md), [Exploitation des services](./03-exploitation-services/index.md) et [Ressources](./01-ressources/index.md). Dans chaque module, vous retrouverez un dossier par technologie déployée qui contient toutes les procédures liées à cette technologie mise en place. La section Ressources centralise quant à elle la base documentaire commune à l'ensemble de ces modules.

---

## 3. 📚 Contenu

<!-- 
NE PAS SUPPRIMER CE COMMENTAIRE !!!

Message pour l'ia : Tu mets à jour avec les informations données en entrée dans le prompt et en utilisant la structure suivante

### [Nom de la compétence principale]

* **[Sous-compétence mobilisée]** : justification
-->

### Cybersécurité

* **Filtrage et contrôle des flux** : configuration des règles de pare-feu, du NAT, des ACL et de UFW afin de contrôler les communications entre les réseaux, les DMZ et Internet.
* **Sécurisation des accès d'administration** : centralisation des accès SSH et RDP avec le bastion Apache Guacamole, contrôle des habilitations par groupes et authentification TOTP.
* **Chiffrement des échanges** : mise en place de certificats SSL/TLS, d'un reverse proxy HTTPS et de connexions d'administration sécurisées.
* **Sauvegarde et traçabilité** : sauvegarde chiffrée des configurations Stormshield et versionnement des fichiers de configuration.

### Administration et supervision des réseaux

* **Conception d'une infrastructure réseau** : réalisation des schémas physiques et logiques, du plan d'adressage IPv4 et des tables de routage de l'architecture multisite.
* **Commutation et segmentation** : configuration des VLAN, du VTP, des commutateurs Cisco de niveau 3 et des sous-interfaces 802.1Q pour isoler les réseaux Production, Clients et Administration.
* **Routage et services réseau** : mise en œuvre du routage statique, du NAT/PAT et du relais DHCP sur les équipements Cisco et Stormshield.
* **Administration sécurisée des équipements** : activation de SSH, gestion des niveaux d'accès et validation des configurations à l'aide de fiches de recette.

### Administration Windows

* **Administration des systèmes Windows** : déploiement de Windows Server Core, configuration des postes clients et gestion des accès à distance par RDP.
* **Services d'annuaire et stratégies** : installation d'Active Directory et du DNS associé, puis déploiement de paramètres et de logiciels au moyen des GPO.
* **Gestion du parc** : déploiement de l'agent GLPI par GPO afin d'automatiser l'inventaire matériel et logiciel des postes.

### Exploitation des services

* **Services DNS (Debian)** : installation et administration de BIND9 en serveur maître et esclave, création de délégations DNS et mise en place d'un résolveur récursif Unbound.
* **Gestion des configurations** : administration des paquets et des noms d'hôte Debian, suivi des modifications avec Etckeeper et contrôle des journaux système.
* **Gestion des services et des demandes** : déploiement de GLPI, structuration des entités et catégories ITIL, création des comptes et suivi de l'inventaire.

---

## 4. 🧠 Compétences du référentiel de BTS SIO

### Gérer le patrimoine informatique

* **Recenser et identifier les ressources numériques** : réalisation de l'inventaire automatisé des équipements et logiciels dans GLPI, complété par les schémas, le plan d'adressage et les tables de routage.
* **Exploiter des référentiels, normes et standards** : application des conventions de nommage, des bonnes pratiques de l'ANSSI et des règles de sécurité liées aux VLAN, DMZ et comptes d'administration.
* **Gérer des sauvegardes** : sauvegarde chiffrée des configurations réseau et conservation de l'historique des fichiers de configuration avec Git et Etckeeper.

### Mettre à disposition des utilisateurs un service informatique

* **Déployer un service** : installation et configuration des services DNS BIND9/Unbound, d'Active Directory, de GLPI et du bastion Guacamole dans l'infrastructure CUB.
* **Réaliser les tests d'intégration et d'acceptation** : rédaction et exécution de fiches de recette pour vérifier la syntaxe, la résolution DNS, les transferts de zones, les accès SSH/RDP et le filtrage réseau.
* **Accompagner les utilisateurs dans la mise en place d'un service** : documentation des procédures d'accès, d'administration et de support afin de rendre les services exploitables par les équipes techniques.

### Répondre aux incidents et aux demandes d’assistance et d’évolution

* **Traiter des demandes concernant les services réseau et système** : diagnostic des erreurs de configuration DNS, des problèmes d'accès SSH/RDP et des incidents liés au pare-feu à partir des journaux et des tests de connectivité.
* **Traiter des demandes d'évolution** : adaptation du plan d'adressage, du routage, des VLAN et des règles de filtrage pour répondre aux besoins d'évolution de l'infrastructure.

### Concevoir une solution d'infrastructure réseau

* **Choisir les éléments nécessaires à la mise en place de la solution** : sélection et intégration des équipements Cisco, des appliances Stormshield et des serveurs Debian selon les contraintes de l'architecture multisite.
* **Installer et tester une solution d'infrastructure réseau** : configuration des commutateurs, du routage, du NAT, des relais DHCP et des services de sécurité, puis validation par des tests documentés.
* **Exploiter, dépanner et superviser une solution d'infrastructure réseau** : maintien en conditions opérationnelles des équipements et services, analyse des journaux et utilisation des fiches de recette pour contrôler leur disponibilité.

### Travailler en mode projet

* **Analyser les objectifs et les contraintes** : prise en compte des besoins de segmentation, de disponibilité et de sécurité de l'entreprise CUB dans la conception de l'infrastructure.
* **Produire et mettre à jour une documentation technique** : formalisation des procédures, schémas, configurations et recettes dans une base documentaire versionnée.

---

## 5. 🛠️ Comment utiliser la documentation ?

Ce site de documentation est généré automatiquement à partir de fichiers Markdown hébergés depuis le dépôt : [cub-docs](https://github.com/FireToak/cub-docs)

### 5.2 ✏️ Modifier la documentation

Pour contribuer ou mettre à jour la documentation, suivez cette procédure Git standard :

1. Cloner le dépôt en local :

```bash
git clone https://github.com/FireToak/cub-docs.git
cd cub-docs
```

2. Créer ou modifier les fichiers : éditez les fichiers .md situés dans l'arborescence correspondante.
3. Indexer les modifications :

```bash
git add .
```

4. Créer un commit descriptif :

```bash
git commit -m "docs: ajout de la procédure de réinitialisation du routeur"
```

5. Pousser les modifications :

```bash
git push origin main
```

*Une fois le `push` effectué, la chaîne CI/CD via GitHub Actions compilera automatiquement les fichiers et déploiera la nouvelle version du site MkDocs.*

---

## 👥 6. Auteur

Ce contexte est réalisé par un étudiant du BTS SIO du lycée Paul-Louis Courier (Tours).

* **Louis MEDO** : [LinkedIn](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr) | [GitHub](https://github.com/FireToak) | [Mail](mailto:louis.medo@loutik.fr)