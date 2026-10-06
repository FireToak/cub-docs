# Gestion des utilisateurs locaux et active directory

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 06/10/2026
- **Domaine :** Administration Windows / Active Directory

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Gestion des Utilisateurs Locaux](#3-gestion-des-utilisateurs-locaux)
- [4. Gestion des Utilisateurs Active Directory (AD DS)](#4-gestion-des-utilisateurs-active-directory-ad-ds)

## 2. Contexte

Cette procédure documente les standards d'ingénierie pour le provisionnement et la gestion du cycle de vie des identités. Elle définit les méthodes d'automatisation (IaC) en PowerShell pour opérer sur la base SAM des serveurs isolés (utilisateurs locaux) ainsi que sur la base de données NTDS d'un environnement Active Directory centralisé. Ce composant est indispensable pour garantir la conformité des accès et le durcissement des déploiements en production.

## 3. Gestion des Utilisateurs Locaux

3.1.  **Créer un compte utilisateur local.** Instanciation d'un nouveau compte de service ou d'administration dans la base SAM de la machine hôte.

```powershell title="creation_local.ps1" hl_lines="2"
$Password = Read-Host -AsSecureString "Saisir le mot de passe"
New-LocalUser -Name "SRE_Admin" -Password $Password -FullName "Administrateur SRE" -Description "Compte de secours infrastructure"
```

- `Read-Host -AsSecureString` : Capture l'entrée utilisateur et chiffre immédiatement la chaîne en un objet `SecureString` en mémoire.
- `New-LocalUser` : Cmdlet créant physiquement l'utilisateur local sur le système.
- `-Name` : Définit l'identifiant de connexion (SAMAccountName local).
- `-Password` : Paramètre exigeant un objet `SecureString` pour assigner le mot de passe de manière sécurisée.

3.2.  **Gérer l'appartenance aux groupes locaux.** Élévation de privilèges via l'affectation du compte à un groupe de sécurité du système.

```powershell title="groupe_local.ps1"
Add-LocalGroupMember -Group "Administrateurs" -Member "SRE_Admin"
```

- `Add-LocalGroupMember` : Cmdlet liant un compte utilisateur existant à un groupe local.
- `-Group` : Identifie le groupe cible recevant le nouveau membre (ex: "Administrateurs").
- `-Member` : Spécifie le compte utilisateur à ajouter au groupe.

3.3.  **Supprimer un utilisateur local.** Révocation des accès et destruction du compte système pour maintenir la surface d'attaque minimale.

```powershell title="suppression_local.ps1"
Remove-LocalUser -Name "SRE_Admin"
```

- `Remove-LocalUser` : Cmdlet ordonnant la suppression définitive du compte local spécifié.

## 4. Gestion des Utilisateurs Active Directory (AD DS)

> [!warning] Dépendance du module
> L'exécution des cmdlets Active Directory requiert l'installation préalable de la fonctionnalité RSAT (Outils d'administration de serveur distant) et le chargement du module associé en mémoire via la commande `Import-Module ActiveDirectory`.

4.1.  **Provisionner un compte Active Directory.** Création standardisée d'une identité réseau et assignation dans la structure hiérarchique (Unité d'Organisation).

```powershell title="creation_ad.ps1" hl_lines="2"
$SecurePwd = ConvertTo-SecureString -String "P@ssw0rd2026!" -AsPlainText -Force
New-ADUser -Name "JDOE" -GivenName "John" -Surname "Doe" -UserPrincipalName "jdoe@local.dortmund.cub.sioplc.fr" -Path "OU=Utilisateurs,DC=local,DC=dortmund,DC=cub,DC=sioplc,DC=fr" -AccountPassword $SecurePwd -Enabled $true
```

- `ConvertTo-SecureString` : Force la conversion d'une chaîne de caractères en clair vers un objet sécurisé requis par l'AD.
- `New-ADUser` : Cmdlet déclenchant l'écriture de l'objet utilisateur dans l'annuaire LDAP.
- `-UserPrincipalName` : Définit le suffixe de routage UPN (format email) utilisé pour l'authentification Kerberos.
- `-Path` : Désigne le chemin LDAP (Distinguished Name) pointant vers l'Unité d'Organisation (OU) de destination.
- `-Enabled $true` : Paramètre booléen activant directement le compte post-création.

4.2.  **Requêter l'annuaire LDAP.** Extraction d'objets utilisateurs avec filtrage côté serveur pour optimiser le trafic réseau.

> [!tip] Performance des requêtes
> Ne retournez jamais l'ensemble de l'annuaire. Utilisez systématiquement le paramètre `-Filter` pour limiter la charge sur le contrôleur de domaine.

```powershell title="requete_ad.ps1"
Get-ADUser -Filter {Department -eq "Infrastructure"} -Properties Description, Department
```

- `Get-ADUser` : Cmdlet de recherche ciblant la classe d'objets utilisateurs.
- `-Filter` : Applique un filtre logique restrictif. Ici, seuls les comptes rattachés au département "Infrastructure" sont retournés.
- `-Properties` : Force le chargement des attributs étendus (Description, Department) qui ne sont pas inclus dans l'objet renvoyé par défaut.

4.3.  **Affecter des permissions via les groupes de sécurité.** Gestion des habilitations réseau en liant l'utilisateur à des groupes de domaine (RBAC).

```powershell title="groupe_ad.ps1"
Add-ADGroupMember -Identity "GG_Admins_OpsBricks" -Members "JDOE"
```

- `Add-ADGroupMember` : Cmdlet ajoutant une ou plusieurs identités dans le groupe ciblé.
- `-Identity` : Spécifie le groupe de sécurité cible (par son nom SAM, SID ou DN).
- `-Members` : Désigne l'identité (utilisateur, ordinateur ou autre groupe) à intégrer.

4.4.  **Désactiver ou révoquer une identité AD.** Procédure d'offboarding (départ collaborateur) garantissant la désactivation immédiate des sessions.

```powershell title="revocation_ad.ps1"
Disable-ADUser -Identity "JDOE"
Remove-ADUser -Identity "JDOE" -Confirm:$false
```

- `Disable-ADUser` : Suspend l'objet Active Directory sans le supprimer. Indispensable pour la rétention légale des boîtes mails ou dossiers partagés.
- `Remove-ADUser` : Purge définitivement l'objet de la base NTDS.
- `-Confirm:$false` : Inhibe l'invite de confirmation interactive, paramètre obligatoire pour un fonctionnement silencieux (mode non-interactif) dans les pipelines d'automatisation.
