# Gestion des dossiers et fichiers en PowerShell

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
- [3. Création, déplacement et suppression](#3-creation-deplacement-et-suppression)
- [4. Gestion des fichiers (Lecture et Écriture)](#4-gestion-des-fichiers-lecture-et-ecriture)

## 2. Contexte

Cette procédure définit les standards de manipulation du système de fichiers (fichiers, répertoires, flux de données) via PowerShell. Ces opérations sont cruciales pour l'automatisation de l'infrastructure (IaC), le traitement des fichiers de journalisation et la gestion des exports structurels pour les scripts d'administration.

## 3. Création, déplacement et suppression {#3-creation-deplacement-et-suppression}

3.1. **Explorer l'arborescence.** Utilisation des cmdlets pour localiser le contexte d'exécution et lister les éléments.

```powershell title="exploration.ps1"
Get-Location
Set-Location "c:\temp"
Get-ChildItem "c:\temp"
```

- `Get-Location` : Retourne le chemin absolu du répertoire de travail courant (équivalent `pwd`).
- `Set-Location` : Modifie le répertoire de travail courant vers le chemin spécifié (équivalent `cd`).
- `Get-ChildItem` : Liste les fichiers et dossiers enfants d'un répertoire donné (équivalent `ls`).

3.2.  **Créer des éléments.** Génération de nouveaux fichiers ou répertoires dans le système de fichiers.

```powershell title="creation.ps1" hl_lines="1"
New-Item -Name "fichier.txt" -ItemType file -Value "Test"
New-Item -Name "dossier" -ItemType directory
```

- `New-Item` : Cmdlet instanciant un nouvel objet (fichier, dossier, registre) dans l'arborescence.
- `-Name` : Définit le nom de la ressource à créer.
- `-ItemType` : Spécifie la nature de l'objet (`file` pour un fichier textuel, `directory` pour un dossier).
- `-Value` : Injecte une chaîne de caractères initiale lors de la création d'un objet de type fichier.

3.3.  **Modifier et transférer.** Opérations de déplacement, de renommage et de copie de données.

> [!warning] Copie récursive
> L'utilisation du paramètre `-Recurse` avec `Copy-Item` est obligatoire pour copier intégralement un dossier contenant lui-même des sous-dossiers et des fichiers.

```powershell title="transfert.ps1"
Move-Item -Path *.txt -Destination "c:\temp\"
Rename-Item -Path fichier.txt -NewName fichier2.txt
Copy-Item -Path dossier -Destination c:\temp -Recurse
```

- `Move-Item` : Déplace un ou plusieurs éléments d'un emplacement à un autre (accepte les wildcards `*`).
- `-Path` : Cible l'emplacement et le nom de l'élément source.
- `-Destination` : Définit le répertoire d'arrivée cible.
- `Rename-Item` : Modifie le nom d'un élément existant sans altérer son emplacement sur le disque.
- `Copy-Item` : Duplique un élément vers la destination spécifiée.
- `-Recurse` : Force l'application de la commande à toute l'arborescence enfant (fichiers et sous-dossiers).

3.4.  **Supprimer des éléments.** Nettoyage de l'arborescence via la destruction de fichiers.

```powershell title="suppression.ps1"
Remove-Item "c:\temp\*.txt"
```

- `Remove-Item` : Détruit définitivement les éléments spécifiés par le chemin. L'utilisation du wildcard `*.txt` supprime tous les fichiers texte du répertoire.

## 4. Gestion des Fichiers (Lecture et Écriture) {#4-gestion-des-fichiers-lecture-et-ecriture}

4.1.  **Lire et analyser un fichier.** Extraction et traitement algorithmique des données textuelles sous forme de collection d'objets.

> [!info] Typage des données
> La cmdlet `Get-Content` traite les données du fichier comme un tableau (`Array`). Chaque ligne lue correspond à un index du tableau, en commençant par l'index `0`.

```powershell title="lecture_analyse.ps1" hl_lines="3"
$ligne = Get-Content -Path c:\temp\test.txt
$ligne[1]
ForEach($l in Get-Content c:\temp\test.txt) { ($l.split("/"))[0] }
```

- `Get-Content` : Lit le contenu d'un fichier texte et le stocke en mémoire sous forme de tableau de lignes.
- `$ligne[1]` : Appel d'index. Cible l'élément situé à l'index 1 du tableau `$ligne` (soit la deuxième ligne réelle du fichier).
- `ForEach` : Instruction itérative parcourant séquentiellement chaque élément (ligne) retourné par `Get-Content`.
- `.split("/")` : Méthode applicable aux objets de type `String` (chaîne de caractères). Découpe la ligne en un sous-tableau en utilisant le caractère `/` comme séparateur.
- `[0]` : Extrait immédiatement la première valeur du sous-tableau généré par la méthode `split()`.

4.2.  **Écrire et exporter des données.** Manipulation des flux de sortie pour modifier des fichiers ou générer des exports de configuration système.

```powershell title="ecriture_export.ps1"
Add-Content -Path C:\temp\test1.txt -Value "bonjour à tous"
Clear-Content -Path C:\temp\test1.txt
ipconfig | findstr "Adresse" | Out-File c:\temp\test2.txt
Get-Process | Export-Clixml c:\temp\process.xml
```

- `Add-Content` : Ajoute la chaîne spécifiée à la fin d'un fichier existant sans écraser son contenu précédent.
- `Clear-Content` : Purge intégralement le contenu d'un fichier tout en conservant le conteneur physique sur le disque.
- `Out-File` : Cmdlet redirigeant le flux du pipeline vers un fichier texte (encodage Unicode par défaut). Équivalent structurel aux redirecteurs CMD `>` (écraser) ou `>>` (ajouter).
- `Export-Clixml` : Convertit les objets PowerShell complexes (ici la liste des processus) en une représentation hiérarchique XML. Idéal pour sérialiser l'état d'un objet.
- `Export-Csv` : *Mentionné dans la théorie*. Exporte les propriétés des objets dans un format structuré séparé par des virgules pour une exploitation tierce.
