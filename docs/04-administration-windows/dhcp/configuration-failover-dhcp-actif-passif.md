# Configuration du failover DHCP en Actif/Passif

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
- [3. Préparation de l'environnement](#3-preparation-de-lenvironnement)
- [4. Configuration de la relation de Failover](#4-configuration-de-la-relation-de-failover)

## 2. Contexte

Cette procédure documente la mise en place d'une redondance haute disponibilité pour le service DHCP en mode **Actif/Passif** (également appelé *Hot Standby*). Dans cette topologie, un serveur principal (Actif) traite l'intégralité des requêtes d'attribution IP, tandis que le second serveur (Passif) reste en attente et prend le relais uniquement en cas de défaillance du nœud principal.

Une part restreinte des adresses (généralement 10%) est réservée au serveur passif afin qu'il puisse distribuer de nouveaux baux pendant la période de transition (MCLT) s'il détecte une panne du serveur primaire.

> [!warning] Relais DHCP
> En mode Actif/Passif, l'agent relais DHCP (IP Helper) de vos équipements de routage doit impérativement pointer vers les adresses IP des **deux** serveurs DHCP de la grappe. Cela garantit que le serveur passif recevra bien les requêtes de diffusion (broadcast) s'il doit assumer la charge suite à une interruption.

## 3. Préparation de l'environnement {#3-preparation-de-lenvironnement}

3.1. **Nettoyage du serveur partenaire (esclave).** Pour éviter les erreurs de synchronisation lors de la création du partenariat, le serveur DHCP secondaire ne doit héberger aucune configuration existante pour l'étendue cible.

```powershell
Remove-DhcpServerv4Scope -ScopeId 192.168.1.0 -Force
```

- `-ScopeId` : Identifiant réseau de l'étendue à purger sur le serveur passif.
- `-Force` : Force la suppression silencieuse sans inviter l'utilisateur à confirmer l'action.

## 4. Configuration de la relation de Failover

4.1. **Création du partenariat Actif/Passif.** Déployer la relation de basculement *Hot Standby* depuis le serveur principal vers le serveur partenaire.

```powershell
Add-DhcpServerv4Failover -ComputerName "ServeurDHCP0" -Name "Failover-CUB-Actif-Passif" -PartnerServer "192.168.1.4" -ScopeId 192.168.1.0 -Mode HotStandby -ReservePercent 10 -MaxClientLeadTime 01:00:00 -SharedSecret "cub" -Force
```

- `-ComputerName` : Nom d'hôte ou adresse IP du serveur DHCP local (nœud Actif).
- `-Name` : Label d'identification de la grappe de basculement dans le système.
- `-PartnerServer` : Adresse IP ou nom d'hôte du serveur DHCP partenaire (nœud Passif).
- `-ScopeId` : Identifiant du sous-réseau de l'étendue incluse dans la réplication.
- `-Mode` : Fixe le comportement de la grappe, ici `HotStandby` pour de l'Actif/Passif.
- `-ReservePercent` : Pourcentage des adresses IP (ici 10%) alloué au serveur de secours pour répondre aux nouveaux clients pendant la phase d'interruption.
- `-MaxClientLeadTime` : Délai de transition maximal (MCLT, ici 1 heure) accordé avant que le serveur passif n'assume le contrôle absolu du pool d'adresses.
- `-SharedSecret` : Clé de chiffrement partagée pour sécuriser et authentifier les échanges entre les nœuds.

4.2. **Vérification de l'état du Failover.** S'assurer de la bonne communication inter-serveurs et de l'état nominal du partenariat.

```powershell
Get-DhcpServerv4Failover -ComputerName "ServeurDHCP0"
```

- `-ComputerName` : Serveur cible interrogé pour afficher le statut de la relation.

**Résultat attendu :**

La console doit afficher une propriété `State` avec la valeur `Normal` et la propriété `Mode` avec la valeur `HotStandby`. L'étendue est désormais répliquée et sécurisée.
