# Configuration failover DHCP Actif Actif

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

Cette procédure documente la mise en place d'une redondance haute disponibilité pour le service DHCP. Initialement pensée en mode **Actif/Passif** (également appelé *Hot Standby* avec 10% des adresses réservées), la configuration documentée ci-dessous établit finalement une topologie **Actif/Actif** (Load Balance). Les deux serveurs se répartissent ainsi la charge des requêtes à parts égales (50/50) tout en assurant une redondance mutuelle.

> [!warning] Relais DHCP
> En mode Actif/Actif ou Actif/Passif, l'agent relais DHCP (IP Helper) de vos équipements de routage doit impérativement pointer vers les adresses IP des **deux** serveurs DHCP de la grappe. Cela garantit que le partenaire recevra bien les requêtes de diffusion (broadcast) s'il doit assumer la charge suite à une interruption.

## 3. Prérequis {3-prerequis}

3.1. **Nettoyage du serveur partenaire (esclave).** Pour éviter les erreurs de synchronisation lors de la création du partenariat, le serveur DHCP secondaire ne doit héberger aucune configuration existante pour l'étendue cible.

```powershell
Remove-DhcpServerv4Scope -ScopeId 192.168.1.0 -Force
```

- `Remove-DhcpServerv4Scope` : Commande permettant de supprimer une étendue IPv4 existante du serveur DHCP.
- `-ScopeId` : Spécifie l'identifiant réseau (l'adresse IP) de l'étendue à supprimer.
- `-Force` : Exécute l'action immédiatement en ignorant les demandes de confirmation.

**Vérification :**

```powershell
Get-DhcpServerv4Scope -ComputerName localhost | Where-Object ScopeId -eq 192.168.1.0
```

**Attendu :** La commande ne doit retourner aucun résultat, confirmant la suppression effective de l'étendue.

## 4. Configuration de l'environnement

Vous devez faire cette partie sur les deux serveurs.

4.1.  **Création du compte d'administration dédié.** Il est nécessaire de créer un compte `ADM-DHCP` sur les deux serveurs pour la gestion centralisée.

```powershell
$Password = ConvertTo-SecureString "etudiant_007" -AsPlainText -Force
New-LocalUser -Name "ADM-DHCP" -Password $Password -FullName "Administrateur DHCP"
```

- `ConvertTo-SecureString` : Transforme une chaîne de caractères standard en chaîne sécurisée (chiffrée en mémoire).
- `-AsPlainText` : Indique que la donnée fournie est en texte clair.
- `New-LocalUser` : Crée un nouveau compte utilisateur dans la base SAM locale.
- `-Name` : Définit le login du compte local.
- `-Password` : Attribue le mot de passe sécurisé à l'utilisateur.
- `-FullName` : Définit le nom d'affichage de l'utilisateur.

4.2.  **Élévation des privilèges.** Le compte créé doit disposer des droits d'administration sur le système.

```powershell
Add-LocalGroupMember -Group "Administrateurs" -Member "ADM-DHCP"
```

- `Add-LocalGroupMember` : Ajoute un utilisateur spécifié à un groupe local existant.
- `-Group` : Désigne le groupe cible (ici, les Administrateurs locaux).
- `-Member` : Spécifie le compte à intégrer dans le groupe.

**Vérification :**

```powershell
Get-LocalGroupMember -Group "Administrateurs" | Where-Object Name -Match "ADM-DHCP"
```

**Attendu :** Le compte `ADM-DHCP` doit apparaître dans la sortie standard, confirmant son existence et son appartenance au groupe Administrateurs.

4.3. **Désactivation du filtre UAC à distance.** Cette étape permet aux comptes administrateurs locaux (autres que le compte "Administrateur" intégré) d'exécuter des requêtes WMI et PowerShell à distance sans blocage.

```powershell
New-ItemProperty -Name LocalAccountTokenFilterPolicy -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -PropertyType DWord -Value 1 -Force
```

- `New-ItemProperty` : Crée une nouvelle clé ou valeur dans la base de registre.
- `-Name` : Nom de la valeur à créer (LocalAccountTokenFilterPolicy).
- `-Path` : Chemin de destination dans l'arborescence du registre (HKLM).
- `-PropertyType DWord` : Spécifie que la valeur est de type registre entier 32 bits.
- `-Value 1` : Active la stratégie (désactive le filtrage UAC pour les accès réseau).

**Vérification :**

```powershell
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy"
```

**Attendu :** La console doit retourner la valeur `1`.

4.4. **Approbation WinRM.** Ajout du serveur esclave dans les hôtes de confiance pour autoriser la connexion distante.

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "ServeurDHCP1,192.168.4.4"
```

- `Set-Item` : Modifie la valeur d'un élément de configuration.
- `WSMan:\...\TrustedHosts` : Chemin d'accès à la configuration des hôtes de confiance de l'agent WinRM (gestion à distance).
- `-Value` : Définit la liste des machines (noms ou IPs) avec lesquelles la machine locale peut s'authentifier hors domaine.

**Vérification :**

```powershell
(Get-Item WSMan:\localhost\Client\TrustedHosts).Value
```

**Attendu :** La chaîne retournée doit inclure `ServeurDHCP1,192.168.4.4`.

## 5. Mise en place du partenariat DHCP

5.1.  **Création du partenariat Actif/Actif.** Après s'être connecté avec l'utilisateur `ADM-DHCP`, on initie la synchronisation entre les deux serveurs.

```powershell
Add-DhcpServerv4Failover -ComputerName "192.168.4.3" -Name "Failover-CUB-Actif-Actif" -PartnerServer "192.168.4.4" -ScopeId 192.168.4.128 -LoadBalancePercent 50 -MaxClientLeadTime 01:00:00 -SharedSecret "SecretFort!"
```

- `Add-DhcpServerv4Failover` : Crée un nouveau partenariat de basculement DHCP en IPv4.
- `-ComputerName` : Spécifie l'adresse IP (ou le nom) du serveur DHCP local/principal.
- `-Name` : Définit le nom d'identification de la grappe de basculement.
- `-PartnerServer` : Spécifie l'adresse IP (ou le nom) du second serveur DHCP.
- `-ScopeId` : Indique l'étendue réseau concernée par cette haute disponibilité.
- `-LoadBalancePercent 50` : Configure le mode en Actif/Actif avec une répartition des baux à 50% / 50% entre les nœuds.
- `-MaxClientLeadTime` : Temps additionnel de grâce (ici 1 heure) accordé à un client avant que son bail ne soit considéré comme expiré, critique pour éviter les conflits d'IP en cas de coupure de communication entre les serveurs.
- `-SharedSecret` : Mot de passe utilisé pour chiffrer et authentifier le trafic de synchronisation entre les deux nœuds DHCP.

**Vérification :**

```powershell
Get-DhcpServerv4Failover -ComputerName "192.168.4.3" -Name "Failover-CUB-Actif-Actif"
```

**Attendu :**

- La propriété `State` doit indiquer `Normal`.
- La propriété `Mode` doit indiquer `LoadBalance`.
- La propriété `LoadBalancePercent` doit indiquer `50`.
