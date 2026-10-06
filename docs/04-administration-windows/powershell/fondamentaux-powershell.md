# Fondamentaux de Powershell

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
- [3. Les commandes PowerShell](#3-les-commandes-powershell)

## 2. Contexte

Cette documentation documente les fondamentaux de l'administration via PowerShell. Ce composant est essentiel pour l'automatisation des infrastructures (IaC) en environnement Windows. Il permet une manipulation des données orientée objet et garantit la standardisation des procédures de requêtes et de configuration système.

## 3. Les Commandes PowerShell

3.1.  **Utiliser la touche Tabulation.** La touche `<tab>` complète automatiquement les noms (commande, paramètre, chemin ou fichier).

> [!info] Navigation
> L’utilisation répétée de la touche `<tab>` permet de parcourir séquentiellement tous les choix disponibles d'auto-complétion.

3.2.  **Comprendre les applets de commandes.** Les cmdlets sont systématiquement formées d’une paire `Verbe-Nom` pour faciliter la mémorisation. Le langage n'est pas sensible à la casse, aux espaces ou aux tabulations.

```powershell title="exemple_cmdlet.ps1"
Start-Service -name eventlog
```

- `Start-Service` : Cmdlet (Verbe-Nom) ordonnant le démarrage d'un service Windows.
- `-name` : Paramètre précisant que l'argument qui suit définit le nom du service cible.
- `eventlog` : Argument indiquant précisément le service à manipuler (le journal d'événements).

3.3.  **Exploiter les méthodes et propriétés.** PowerShell produit des objets. L'état de cet objet est défini par ses propriétés, et son comportement par ses méthodes associées.

```powershell title="get_member.ps1"
Get-Date | Get-Member
```

- `Get-Date` : Cmdlet générant un objet contenant la date et l'heure système actuelles.
- `|` : Opérateur pipeline transférant l'objet résultant à la commande suivante.
- `Get-Member` : Cmdlet analysant et listant toutes les méthodes et propriétés de l'objet transmis.

3.4.  **Gérer les alias.** Les alias permettent de raccourcir l'appel des cmdlets fréquemment utilisées.

> [!warning] Durée de vie d'un alias
> La durée de vie d'un alias personnalisé est limitée à l’invite de commande PowerShell. Si le processus est fermé, l’alias est supprimé.

```powershell title="gestion_alias.ps1"
Get-Alias
Set-Alias list Get-ChildItem
```

- `Get-Alias` : Cmdlet retournant la liste de tous les alias pré-définis sur le système.
- `Set-Alias` : Cmdlet permettant de déclarer un nouvel alias ou d'en modifier un.
- `list` : Nom de l'alias personnalisé défini par l'administrateur.
- `Get-ChildItem` : Cmdlet réelle à exécuter lors de l'appel de l'alias (équivalent d'un ls).

3.5.  **Rechercher et obtenir de l'aide.** Plusieurs commandes natives sont indispensables pour opérer efficacement sur le système.

```powershell title="commandes_utiles.ps1"
Get-Command -verb "Get"
Get-Help Clear-Host -examples
```

- `Get-Command` : Affiche l'ensemble des commandes disponibles (filtrable via `-verb` ou `-noun`).
- `Get-Help` : Affiche la documentation d'une commande (l'argument `-examples` ou `-?` montre des cas d'usage).
- `Clear-Host` : Efface le contenu de la console.
- `Get-History` : Liste l'historique des commandes entrées durant la session.
- `Get-Process` : Liste l'ensemble des processus actuellement en cours d’exécution.

3.6.  **Maîtriser le pipeline.** L’opérateur `|` canalise le flux : la sortie d'une commande devient instantanément l'entrée de la suivante.

```powershell title="utilisation_pipeline.ps1"
Get-Date | Format-Table
ipconfig | findstr "Adresse"
```

- `Format-Table` : Cmdlet de formatage structurant les propriétés de l'objet reçu en tableau lisible.
- `ipconfig` : Exécutable natif Windows affichant la configuration réseau IP de l'hôte.
- `findstr` : Outil de traitement de texte filtrant le résultat pour ne garder que certaines chaînes.
- `"Adresse"` : Chaîne de caractères exacte recherchée et filtrée par l'outil `findstr`.
