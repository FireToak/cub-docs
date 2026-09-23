# Renommer le compte Administrateur local

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 16/09/2026
- **Domaine :** Windows

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Renommage via l'interface graphique (GUI)](#3-renommage-via-linterface-graphique-gui)
- [4. Renommage via PowerShell (CLI)](#4-renommage-via-powershell-cli)

## 2. Contexte

Le renommage du compte Administrateur local par défaut est une mesure de sécurité (obfuscation) limitant les attaques par force brute standardisées. Au sein de l'infrastructure CUB, cette modification permet d'éviter l'exploitation automatisée ciblant l'identifiant "Administrateur" ou "Administrator" bien connu.

> [!warning] Gestion des accès
> Assurez-vous de documenter de manière sécurisée (gestionnaire de mots de passe, solution LAPS) le nouveau nom d'utilisateur. La perte de cet identifiant complique grandement les opérations de récupération d'urgence.

## 3. Renommage via l'interface graphique (GUI)

3.1. **Ouvrir la console de gestion.** Exécutez `compmgmt.msc` ou faites un clic droit sur le menu Démarrer pour sélectionner **Gestion de l'ordinateur**.

3.2. **Accéder aux utilisateurs.** Dans le panneau latéral gauche, développez **Outils système**, puis **Utilisateurs et groupes locaux**, et cliquez sur le dossier **Utilisateurs**.

3.3. **Appliquer le nouveau nom.** Faites un clic droit sur le compte nommé **Administrateur**, choisissez **Renommer**, saisissez le nouveau nom souhaité et appuyez sur Entrée pour valider la modification.

## 4. Renommage via PowerShell (CLI)

4.1. **Exécuter le script de modification.** Ouvrez une invite PowerShell en tant qu'administrateur pour renommer directement le compte utilisateur cible.

```powershell
Rename-LocalUser -Name "Administrateur" -NewName "Admin_CUB"
```

- `Rename-LocalUser` : Commande principale utilisée pour modifier l'identifiant (le nom) d'un compte utilisateur présent localement sur la machine.
- `-Name "Administrateur"` : Cible l'utilisateur exact à modifier en fonction de son nom actuel.
- `-NewName "Admin_CUB"` : Spécifie la nouvelle valeur textuelle qui remplacera le nom du compte ciblé.
