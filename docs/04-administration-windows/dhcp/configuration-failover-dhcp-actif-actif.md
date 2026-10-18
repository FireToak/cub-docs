# Configuration du failover DHCP en Actif/Actif

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

Cette procédure documente la mise en place d'une redondance haute disponibilité pour le service DHCP au sein de l'infrastructure CUB. Le mode **Actif/Actif** (également appelé *Load Balance* ou équilibrage de charge) permet à deux serveurs DHCP de répondre simultanément aux requêtes des clients réseau et de se répartir la charge de distribution des baux de manière équitable.

Cette architecture garantit non seulement une tolérance aux pannes (si l'un des nœuds tombe, l'autre prend le relais à 100%), mais optimise également les performances en évitant de saturer un seul serveur.

> [!warning] Relais DHCP
> Dans une architecture Actif/Actif, les requêtes des clients doivent parvenir simultanément aux deux nœuds pour que l'équilibrage de charge s'opère correctement. Il est impératif de configurer l'agent relais DHCP (IP Helper) sur vos équipements de routage (cœurs de réseau, routeurs ou pare-feux) afin qu'il pointe vers les adresses IP des **deux** serveurs DHCP de la grappe (ex: l'IP de `ServeurDHCP0` et celle de `ServeurDHCP1`).

## 3. Préparation de l'environnement {#3-preparation-de-lenvironnement}

3.1. **Nettoyage du serveur partenaire (esclave).** Avant d'initialiser la réplication, il est impératif de s'assurer qu'aucune étendue conflictuelle n'existe sur le serveur DHCP secondaire.

```powershell
Remove-DhcpServerv4Scope -ScopeId 192.168.1.0 -Force
```

- `-ScopeId` : Identifiant réseau de l'étendue à supprimer sur le serveur partenaire.
- `-Force` : Force la suppression sans demander de confirmation interactive, idéal pour l'automatisation.

## 4. Configuration de la relation de Failover

4.1. **Création du partenariat Actif/Actif.** Instancier la relation d'équilibrage de charge entre le serveur principal et le serveur partenaire.

```powershell
Add-DhcpServerv4Failover -ComputerName "ServeurDHCP0" -Name "Failover-CUB-Actif-Actif" -PartnerServer "192.168.1.4" -ScopeId 192.168.1.0 -Mode LoadBalance -LoadBalancePercent 50 -MaxClientLeadTime 01:00:00 -SharedSecret "cub" -Force
```

- `-ComputerName` : Nom d'hôte ou adresse IP du serveur DHCP local (nœud principal).
- `-Name` : Nom identifiant la relation de basculement dans la console d'administration.
- `-PartnerServer` : Adresse IP ou nom d'hôte du serveur DHCP partenaire (nœud secondaire).
- `-ScopeId` : Identifiant du sous-réseau de l'étendue à inclure dans le Failover.
- `-Mode` : Définit le mode de fonctionnement, ici `LoadBalance` pour de l'Actif/Actif.
- `-LoadBalancePercent` : Pourcentage de la charge (pool d'adresses) alloué au serveur local (ici 50% pour une répartition équitable).
- `-MaxClientLeadTime` : Délai de transition maximal (MCLT) accordé à un client avant que le partenaire ne prenne le contrôle total de son bail (ici 1 heure).
- `-SharedSecret` : Mot de passe partagé pour authentifier et chiffrer la communication entre les deux nœuds.

> [!info] Synchronisation automatique
> Dès l'exécution de cette commande, le serveur principal (`ServeurDHCP0`) va automatiquement provisionner l'étendue et ses options sur le serveur partenaire (`ServeurDHCP1`).

4.2. **Vérification de l'état du Failover.** S'assurer que le partenariat est correctement monté et que les deux serveurs communiquent.

```powershell
Get-DhcpServerv4Failover -ComputerName "ServeurDHCP0"
```

- `-ComputerName` : Cible le serveur sur lequel lire l'état de la relation.

**Résultat attendu :**

La propriété `State` doit afficher `Normal` et le `Mode` doit être `LoadBalance`.
