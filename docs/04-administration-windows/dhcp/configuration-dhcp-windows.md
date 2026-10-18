# Configuration du rôle DHCP via PowerShell

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 18/10/2026
- **Domaine :** Administration Windows

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Configuration](#3-configuration)
- [4. Vérification](#4-verification)

## 2. Contexte

Cette procédure détaille la configuration généralisée en ligne de commande (PowerShell) du rôle Serveur DHCP. Dans une démarche d'Infrastructure as Code (IaC) et d'automatisation, ces commandes permettent de couvrir l'intégralité du cycle de vie d'une étendue : création, gestion des baux, exclusions, réservations et options réseaux distribuées. Les adresses IP et noms utilisés sont des exemples génériques à adapter selon le plan d'adressage cible.

> [!warning] Prérequis de routage
> Pensez à configurer le relais DHCP (IP Helper) sur votre équipement de routage si les clients ne se trouvent pas dans le même domaine de diffusion (VLAN) que le serveur DHCP.

## 3. Configuration

3.1. **Vérification du service.** S'assurer que le service DHCP est bien installé et en cours d'exécution avant d'appliquer la configuration.

```powershell
Get-Service DHCPServer
```

- `DHCPServer` : Nom interne du service de distribution DHCP sous Windows.

**Résultat attendu :**
Le statut du service doit indiquer `Running`.

3.2. **Création d'une étendue DHCP.** Définir la plage d'adresses IP dynamiques principale qui sera allouée aux clients du réseau (ex: étendue "Clients").

```powershell
Add-DhcpServerv4Scope -Name "Clients" -StartRange 192.168.1.100 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0 -State InActive
```

- `-Name` : Label d'identification de l'étendue.
- `-StartRange` / `-EndRange` : Bornes de début et de fin de la plage IP distribuable.
- `-SubnetMask` : Masque de sous-réseau associé au segment réseau.
- `-State` : Définit l'état initial (il est recommandé de la laisser `InActive` le temps de finaliser la configuration des options).

3.3. **Modification du bail DHCP.** Ajuster la durée de validité du bail IP alloué aux clients (ex: paramétrage d'un bail court de 4 heures).

```powershell
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -LeaseDuration 04:00:00
```

- `-ScopeId` : Identifiant réseau (ID) de l'étendue à modifier.
- `-LeaseDuration` : Durée du bail formatée en `Jours.Heures:Minutes:Secondes` (ici 0 jour et 4 heures).

3.4. **Exclusion d'adresses IP.** Retirer une plage spécifique d'adresses IP de la distribution dynamique pour conserver des adresses disponibles pour des équipements fixes.

```powershell
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.180 -EndRange 192.168.1.189
```

- `-ScopeId` : Identifiant réseau de l'étendue cible.
- `-StartRange` / `-EndRange` : Délimitation de la plage d'adresses IP à bloquer au sein de l'étendue.

3.5. **Création d'une réservation.** Assigner de manière statique et exclusive une adresse IP à un équipement spécifique (ex: serveur de service, imprimante) via son adresse MAC.

```powershell
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.179 -ClientId "00-11-22-33-44-55" -Name "Poste-Service-Specifique"
```

- `-ScopeId` : Identifiant réseau de l'étendue cible.
- `-IPAddress` : Adresse IP à réserver pour l'équipement.
- `-ClientId` : Adresse MAC de la carte réseau cible (le format attend généralement des tirets).
- `-Name` : Description explicite ou nom d'hôte permettant d'identifier la réservation dans la console.

3.6. **Configuration des options DHCP.** Définir les paramètres réseau complémentaires indispensables (Passerelle, Serveurs DNS, Suffixe DNS) distribués avec le bail IP.

```powershell
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.254 -DnsServer 8.8.8.8, 8.8.4.4 -DnsDomain "domaine.local"
```

- `-ScopeId` : Identifiant réseau de l'étendue concernée.
- `-Router` : Adresse IP de la passerelle par défaut du sous-réseau (Option 003).
- `-DnsServer` : Liste des adresses IP des serveurs de résolution de noms (Option 006).
- `-DnsDomain` : Suffixe de nom de domaine réseau attribué aux clients (Option 015).

3.7. **Activation de l'étendue.** Basculer l'étendue en production une fois la configuration achevée.

```powershell
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State Active
```

- `-State` : Modifie le statut de l'étendue en `Active` pour autoriser la distribution des baux.

## 4. Vérification {#4-verification}

4.1. **Vérification globale de l'étendue.** S'assurer que l'étendue a correctement été provisionnée et activée sur le serveur.

```powershell
Get-DhcpServerv4Scope -ScopeId 192.168.1.0
```

4.2. **Vérification des options distribuées.** Contrôler la bonne application de la passerelle, du DNS et du domaine sur l'étendue.

```powershell
Get-DhcpServerv4OptionValue -ScopeId 192.168.1.0
```

4.3. **Vérification des exclusions.** Lister les plages configurées pour être exclues de la distribution.

```powershell
Get-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0
```

4.4. **Vérification des réservations et des baux.** Valider la présence de l'équipement réservé et des hôtes clients dans la liste des baux actifs du serveur DHCP.

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.1.0
```
