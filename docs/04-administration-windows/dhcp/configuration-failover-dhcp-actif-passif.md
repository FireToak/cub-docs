# Configuration failover DHCP Actif Passif

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 04/11/2026
- **Domaine :** Administration Windows

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Prérequis](#3-prerequis)
- [4. Configuration de l'environnement](#4-configuration-de-lenvironnement)
- [5. Mise en place du partenariat DHCP](#5-mise-en-place-du-partenariat-dhcp)

## 2. Contexte

Cette procédure documente la mise en place d'une redondance haute disponibilité pour le service DHCP en mode **Actif/Passif** (également appelé *Hot Standby*). Dans cette topologie, un serveur principal (Actif) traite l'intégralité des requêtes d'attribution IP, tandis que le second serveur (Passif) reste en attente et prend le relais uniquement en cas de défaillance du nœud principal.

Une part restreinte des adresses (ici configurée à 10%) est réservée au serveur passif afin qu'il puisse distribuer de nouveaux baux pendant la période de transition (MCLT) s'il détecte une panne du serveur primaire. Le délai de transition maximal est défini sur 1 heure.

> [!warning] Relais DHCP
> En mode Actif/Passif, l'agent relais DHCP (IP Helper) de vos équipements de routage doit impérativement pointer vers les adresses IP des **deux** serveurs DHCP de la grappe. Cela garantit que le serveur passif recevra bien les requêtes de diffusion (broadcast) s'il doit assumer la charge suite à une interruption.

## 3. Prérequis {#3-prerequis}

3.1. **Nettoyage du serveur partenaire (esclave).** Pour éviter les erreurs de synchronisation lors de la création du partenariat, le serveur DHCP secondaire ne doit héberger aucune configuration existante pour l'étendue cible.

```powershell
Remove-DhcpServerv4Scope -ScopeId 192.168.1.0 -Force
```

- `Remove-DhcpServerv4Scope` : Commande permettant de supprimer une étendue IPv4 existante du serveur DHCP.
- `-ScopeId` : Spécifie l'identifiant réseau de l'étendue à supprimer.
- `-Force` : Exécute l'action immédiatement en ignorant les demandes de confirmation.

**Vérification :**

```powershell
Get-DhcpServerv4Scope -ComputerName localhost | Where-Object ScopeId -eq 192.168.1.0
```

**Attendu :* La commande ne doit retourner aucun résultat, confirmant la suppression effective de l'étendue.

## 4. Configuration de l'environnement

4.1. **Création du compte d'administration dédié.** Il est nécessaire de créer un compte `ADM-DHCP` sur les deux serveurs pour la gestion centralisée.

```powershell
$Password = ConvertTo-SecureString "etudiant_007" -AsPlainText -Force
New-LocalUser -Name "ADM-DHCP" -Password $Password -FullName "Administrateur DHCP"
```

- `New-LocalUser` : Crée un nouveau compte utilisateur dans la base SAM locale.

4.2. **Élévation des privilèges.** Le compte créé doit disposer des droits d'administration sur le système.

```powershell
Add-LocalGroupMember -Group "Administrateurs" -Member "ADM-DHCP"
```

- `Add-LocalGroupMember` : Ajoute l'utilisateur spécifié au groupe local existant.

**Vérification :**

```powershell
Get-LocalGroupMember -Group "Administrateurs" | Where-Object Name -Match "ADM-DHCP"
```

**Attendu :** Le compte `ADM-DHCP` doit apparaître dans la sortie standard, confirmant son existence et son appartenance au groupe Administrateurs.

4.3. **Désactivation du filtre UAC à distance.** Cette étape permet aux comptes administrateurs locaux d'exécuter des requêtes WMI et PowerShell à distance sans blocage.

```powershell
New-ItemProperty -Name LocalAccountTokenFilterPolicy -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -PropertyType DWord -Value 1 -Force
```

- `New-ItemProperty` : Crée ou modifie la clé de registre pour désactiver le filtrage UAC réseau.

**Vérification :**

```powershell
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy"
```

**Attendu :** La console doit retourner la valeur `1`.

4.4. **Approbation WinRM.** Ajout du serveur esclave dans les hôtes de confiance pour autoriser la connexion distante.

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "ServeurDHCP1,192.168.4.4"
```

- `Set-Item` : Configure la liste des machines de confiance de l'agent WinRM.

**Vérification :**

```powershell
(Get-Item WSMan:\localhost\Client\TrustedHosts).Value
```

**Attendu :** La chaîne retournée doit inclure `ServeurDHCP1,192.168.4.4`.

## 5. Mise en place du partenariat DHCP

5.1. **Création du partenariat Actif/Passif.** Après s'être connecté avec l'utilisateur `ADM-DHCP`, on initie la synchronisation entre les deux serveurs en définissant le rôle primaire et le pourcentage d'adresses de secours.

```powershell
Add-DhcpServerv4Failover -ComputerName "192.168.4.3" -Name "Failover-CUB-Actif-Passif" -PartnerServer "192.168.4.4" -ScopeId 192.168.4.128 -ServerRole Active -ReservePercent 10 -MaxClientLeadTime 01:00:00 -SharedSecret "SecretFort!"
```

- `Add-DhcpServerv4Failover` : Crée le partenariat de basculement DHCP.
- `-ComputerName` : Serveur principal.
- `-PartnerServer` : Serveur secondaire (passif).
- `-ServerRole Active` : Définit le serveur local comme étant le nœud primaire (Actif) de la topologie.
- `-ReservePercent 10` : Assigne 10% des adresses de l'étendue au serveur secondaire en mode de secours (Hot Standby).
- `-MaxClientLeadTime 01:00:00` : Délai de transition maximal d'une heure (MCLT) accordé au client lors d'un basculement.

**Vérification :**

```powershell
Get-DhcpServerv4Failover -ComputerName "192.168.4.3" -Name "Failover-CUB-Actif-Passif"
```

**Attendu :**

- La propriété `State` doit indiquer `Normal`.
- La propriété `Mode` doit indiquer `HotStandby`.
- La propriété `ServerRole` doit indiquer `Active` sur le nœud principal.
- La propriété `ReservePercent` doit indiquer `10`.
