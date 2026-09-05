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

La présente documentation est séparée en 4 modules : [Administration et supervision des réseaux](./administration-supervision-reseaux/index.md), [Administration Windows](./administration-windows/index.md), [Cybersécurité](./cybersecurite/index.md), [Exploitation des services](./exploitation-services/index.md) et [Ressources](./ressources/index.md). Dans chaque module, vous retrouverez un dossier par technologie déployée qui contient toutes les procédures liées à cette technologie mise en place. La section Ressources centralise quant à elle la base documentaire commune à l'ensemble de ces modules.

---

## 3. 📚 Contenu

<!-- 
NE PAS SUPPRIMER CE COMMENTAIRE !!!

Message pour l'ia : Tu mets à jour avec les informations données en entrée dans le prompt et en utilisant la structure suivante

### [Nom de la compétence principale]

* **[Sous-compétence mobilisée]** : justification
-->

### Cybersécurité

* **Stormshield UTM** : implémentation d'une solution de sécurité unifiée pour assurer le filtrage applicatif et contrer les menaces avancées.

### Administration et supervision des réseaux

* **Segmentation réseau (VLAN)** : Cloisonnement du réseau local des agences (Production, Clients, Administration) afin de réduire la surface d'exposition aux attaques, selon les recommandations de l'ANSSI.
* **Routage et adressage public** : Implémentation du plan d'adressage IPv4 LIR de CUB (192.36.253.0/24) et configuration des liaisons inter-agences.

### Administration Windows

* **Postes clients Windows** : Gestion et intégration des postes de travail utilisateurs (Windows Client) répartis dans les réseaux locaux des différentes agences de l'incubateur.

### Exploitation des services

* **Serveurs DNS (Debian)** : Déploiement et administration des serveurs DNS maîtres et esclaves situés en DMZ pour assurer la résolution de noms de domaine de chaque agence.
* **Serveurs Web (Debian)** : Hébergement et maintien en conditions opérationnelles des services web vitrines de l'entreprise sur l'ensemble des sites.

---

## 4. 🧠 Compétences du référentiel de BTS SIO

### Gérer le patrimoine informatique

* **Recenser et identifier les ressources numériques** : Cartographie des services présents sur le réseau local et analyse de l'adressage IP des différentes agences CUB.

### Mettre à disposition des utilisateurs un service informatique

* **Déployer un service** : Intégration logique et physique des nouvelles appliances UTM et des serveurs Debian (Web/DNS) au sein des DMZ de l'architecture.

### Répondre aux incidents et aux demandes d’assistance et d’évolution

* **Traiter des demandes concernant les services réseau et système, applicatifs** : Conception de maquettes Packet Tracer et schémas logiques pour répondre aux besoins d'évolution sécuritaire exigés par le service RSSI.

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