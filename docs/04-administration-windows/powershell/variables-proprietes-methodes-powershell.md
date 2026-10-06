# Manipulation des variables, propriétés et méthodes des objets

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 06/10/2026
- **Domaine :** Administration Windows

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Variables, Propriétés et Méthodes des Objets](#3-variables-proprietes-et-methodes-des-objets)

## 2. Contexte

Cette procédure documente les standards de manipulation des variables et de l'approche orientée objet au sein de PowerShell. Contrairement aux scripts de type Batch (texte pur), PowerShell manipule des objets en mémoire. La maîtrise des variables, de leurs propriétés (états) et de leurs méthodes (actions) est un prérequis indispensable pour l'écriture de scripts d'automatisation d'infrastructure (IaC) fiables et maintenables.

## 3. Variables, Propriétés et Méthodes des Objets {#3-variables-proprietes-et-methodes-des-objets}

3.1.  **Déclarer et assigner une variable.** PowerShell permet de stocker la sortie d'une commande valide dans une variable, agissant ainsi comme un pointeur vers l'objet créé.

> [!info] Nomenclature des variables
> Le nom d'une variable doit obligatoirement commencer par le symbole `$`. Il peut être suivi de tout caractère alphanumérique ou du trait de soulignement (`_`).

```powershell title="assignation_variable.ps1"
$loc = Get-Location
```

- `$loc` : Déclaration du nom de la variable qui va encapsuler l'objet en mémoire.
- `=` : Opérateur d'affectation assignant le résultat de la commande de droite à la variable de gauche.
- `Get-Location` : Cmdlet qui récupère le contexte du répertoire de travail actuel et retourne un objet complexe.

3.2.  **Inspecter le contenu et le type des variables.** Il est impératif de connaître la structure de l'objet manipulé pour en exploiter le potentiel.

```powershell title="inspection_objet.ps1"
$loc | Get-Member
Get-Location | Get-Member
```

- `$loc` / `Get-Location` : Appel de la variable ou exécution directe de la commande générant l'objet.
- `|` : Opérateur pipeline qui transmet l'objet généré en tant que flux d'entrée à la commande suivante.
- `Get-Member` : Cmdlet d'introspection qui analyse l'objet reçu et liste l'intégralité de ses membres (propriétés, méthodes, événements, type d'objet).

3.3.  **Accéder aux propriétés d'un objet.** Lecture de l'état ou des attributs de l'objet stocké en utilisant la notation pointée.

> [!tip] Auto-complétion
> Lors de la saisie d'une propriété ou d'une méthode après le point, l’usage de la touche `<tab>` permet de compléter automatiquement le nom du membre ciblé.

```powershell title="acces_proprietes.ps1" hl_lines="1"
$loc.Path
```

- `.` (Point) : Opérateur d'accès aux membres (Member Access Operator), permettant d'entrer dans l'arborescence de l'objet.
- `Path` : Nom spécifique de la propriété ciblée, qui contient ici la valeur textuelle du chemin d'accès absolu au répertoire.

3.4.  **Exécuter les méthodes d'un objet.** Invocation d'une action ou d'un comportement associé à l'objet.

> [!warning] Syntaxe des méthodes
> L'exécution d'une méthode requiert obligatoirement l'ajout de parenthèses `()`, même si aucun paramètre n'est transmis à celle-ci. Oublier les parenthèses affichera uniquement la définition de la méthode au lieu de l'exécuter.

```powershell title="execution_methodes.ps1"
$fichier.Delete()
```

- `$fichier` : Variable contenant un objet de type système de fichiers (ex: `System.IO.FileInfo`).
- `.Delete()` : Appel de la méthode ordonnant la suppression physique du fichier. Les parenthèses déclenchent l'exécution immédiate de l'action définie par cette méthode.
