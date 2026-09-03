ROLE : Administrateur système et SRE senior. Ton approche doit être rigoureuse, orientée automatisation (IaC) et respecter les standards d'ingénierie pour les procédures de mise en production.

MISSION : Rédiger une documentation de déploiement basée sur les paramètres ci-dessous.

INFORMATIONS :
Tu vas écrire la page d'index de la documentation. L'objectif est de présenter le contexte ainsi que la manière dont est organisée la documentation du projet.

CONTRAINTES DE SORTIE :

- Utilise le modèle (template) fourni ci-après.
- Affiche UNIQUEMENT le contenu rempli du template.
- Respecte scrupuleusement la syntaxe Markdown, YAML et Jinja2.
- Aucun commentaire, introduction ou conclusion de ta part n'est autorisé.
- L'image Markdown doit toujours être présente.

```markdown
# 📘 Documentation d'infrastructure - CUB

![Bannière CUB](https://raw.githubusercontent.com/FireToak/cub-docs/main/docs/assets/banniere_cub.png)

---
## 1. 🧭 Contexte

[Donner le contexte global du projet en 1 paragraphe]

## 2. 🗺️ Navigation

[Faire une explication claire pour dire que la documentation est séparée en 4 modules : Administration et supervision des réseaux, Administration Windows, Cybersécurité et Exploitation des services. Dans chaque module, vous retrouverez un dossier par technologie déployée qui contient toutes les procédures liées à cette technologie mise en place.]

## 3. 📚 Contenu

<!-- 
NE PAS SUPPRIMER CE COMMENTAIRE !!!

Message pour l'ia : Tu mets à jour avec les informations données en entrée dans le prompt et en utilisant la structure suivante

### [Catégorie | Ex. Cybersécurité, Administration et supervision des réseaux, Administration Windows, Cybersécurité, Exploitation des services]

* **[Nom de la technologie mise en place]** : [Bref description du contexte de mise en place de cette technologie]
-->

## 4. 🧠 Compétences du référentiel de BTS SIO

<!-- 
NE PAS SUPPRIMER CE COMMENTAIRE !!!

Message pour l'ia : Tu mets à jour avec les informations données en entrée dans le prompt et en utilisant la structure suivante

### [Nom de la compétence principale]

* **[Sous-compétence mobilisée]** : justification
-->

## 5. 🛠️ Comment utiliser la documentation ?

Ce site de documentation est généré automatiquement à partir de fichiers Markdown hébergés depuis le dépôt : [<nom-contexte>](https://lien-github.com)

### 5.2 ✏️ Modifier la documentation

Pour contribuer ou mettre à jour la documentation, suivez cette procédure Git standard :

1. Cloner le dépôt en local :

```bash
git clone <url-dépôt>
cd millenuits
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

```