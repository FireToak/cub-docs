# BTS SIO - CUB - DOCUMENTATION

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

## Contexte

Ce dépôt contient le code source de la documentation technique dédiée à l’infrastructure réseau multisite de l’entreprise fictive CUB (incubateur de startups). Ce projet est réalisé dans le cadre du BTS SIO au lycée Paul-Louis Courier de Tours.

L’objectif de cette base de connaissances est de documenter de bout en bout la conception, la configuration, la sécurisation et l’exploitation de l'infrastructure (siège à Paris et agences internationales). Elle s'adresse aux étudiants, administrateurs systèmes et réseaux, ainsi qu'aux techniciens d’exploitation.

L'infrastructure s'appuie sur des équipements Cisco et serveurs Debian, structurée autour de VLAN, de DMZ, et sécurisée par des solutions UTM Stormshield selon les recommandations de l’ANSSI.

- **Documentation en production** : [cub.bts.loutik.fr](https://cub.bts.loutik.fr/)
- **Documentation de référence (professeurs)** : [cubdocumentation.sioplc.fr](https://cubdocumentation.sioplc.fr/)

-----

## Structure du dépôt

L’organisation du dépôt suit la logique suivante :

```text
.
├── .github/          
├── docs/             
│   ├── administration-supervision-reseaux/ 
│   ├── administration-windows/             
│   ├── cybersecurite/                      
│   ├── exploitation-services/              
│   └── ressources/                         
├── templates/        
├── requirements.txt  
└── zensical.toml     

```

- **`docs/`** : Contient l'intégralité des articles techniques au format Markdown, répartis par modules métier métiers (réseau, système, cybersécurité).
- **`docs/ressources/schemas/`** : Regroupe les architectures physiques, logiques et plans de brassage (fichiers sources Draw.io et exports PDF).
- **`templates/`** : Regroupe les modèles de documents (procédures, pages d'index) pour garantir la standardisation des contributions.
- **`zensical.toml`** : Fichier de configuration principal pour le générateur de site statique Zensical.

-----

## Utilisation en local

### 1. Cloner le dépôt localement

```bash
git clone https://github.com/FireToak/cub-docs.git
cd cub-docs

```

### 2. Préparer l'environnement virtuel

Il est impératif d'isoler les dépendances Python (Zensical et extensions) pour éviter les conflits systèmes.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

```

### 3. Lancer la prévisualisation locale

Démarrez le serveur de développement pour compiler le Markdown et visualiser les modifications en temps réel.

```bash
zensical serve

```

-----

## Bonnes pratiques et sécurité

1. **Standardisation documentaire** : Utilisez toujours les modèles présents dans `templates/` (ex: `template_procedure.md`) lors de la création d'une nouvelle page afin de maintenir la cohérence structurelle.
2. **Gestion des secrets** : Ne commitez jamais de mots de passe, de clés privées, ou de configurations sensibles en clair dans le dépôt. Anonymisez systématiquement les adresses IP publiques réelles et les identifiants dans les extraits de code.
3. **Validation de l'infrastructure as Code (IaC)** : Vérifiez la syntaxe de vos fichiers avant de les soumettre.

```bash
# Vérification de l'état de l'arbre de travail avant commit
git status

```

-----

## 👨‍💻 Mainteneurs

- **Louis MEDO** | [LinkedIn](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr/) | [GitHub](https://github.com/FireToak) | [louis.medo@loutik.fr](mailto:louis.medo@loutik.fr)

-----

<div align="center">
<br>
<small><i>Dernière mise à jour : 5 septembre 2026</i></small>
</div>
