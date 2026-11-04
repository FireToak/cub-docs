# Recette - DHCP Actif Actif

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 04/11/2026
- **Domaine :** Administration Windows

---

## 1. Contexte du test

Ce plan de recette a pour but de valider le bon fonctionnement et la haute disponibilité de la grappe DHCP configurée en mode **Actif/Actif** (Load Balance). Il s'agit de s'assurer que les deux serveurs se répartissent la charge d'attribution des baux IP (50/50) et qu'en cas d'interruption du service sur l'un des nœuds, le serveur partenaire prend immédiatement le relais sans impact pour les postes clients du réseau.

> [!warning] Tolérance aux pannes
> Le test de basculement (Failover) implique l'arrêt volontaire du service DHCP sur le nœud primaire. Bien que transparent pour les utilisateurs finaux grâce au partenariat, ce test de continuité de service doit être planifié pour valider l'infrastructure.

## 2. Procédures de validation

### 2.1. Vérification de l'état du partenariat de basculement

**Objectif :** S'assurer que le partenariat DHCP est bien actif, synchronisé et configuré en répartition de charge entre les deux serveurs.

**Commande utilisée :**

```powershell
Get-DhcpServerv4Failover -ComputerName localhost
```

* `Get-DhcpServerv4Failover` : Cmdlet PowerShell qui interroge et retourne les propriétés de la configuration de basculement DHCP IPv4.
* `-ComputerName` : Spécifie le serveur cible sur lequel exécuter la requête (ici, le serveur local).

**Résultat attendu :**

```powershell
Name                 : Failover-CUB-Actif-Actif
PartnerServer        : 192.168.4.4
Mode                 : LoadBalance
LoadBalancePercent   : 50
ServerRole           : Normal
State                : Normal
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.2. Validation de l'attribution IP sur un poste client

**Objectif :** Confirmer qu'un client du réseau parvient à joindre la grappe DHCP et à obtenir une configuration IP dynamique valide.

**Commande utilisée :**

```cmd
ipconfig /release
ipconfig /renew
```

* `ipconfig` : Utilitaire en ligne de commande Windows affichant la configuration TCP/IP actuelle.
* `/release` : Libère le bail IPv4 actuel de la carte réseau.
* `/renew` : Renouvelle le bail IPv4 en diffusant une requête DHCP sur le réseau.

**Résultat attendu :**

```cmd
Carte Ethernet Ethernet0 :
   Suffixe DNS propre à la connexion. . . : cub.local
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.4.135
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . : 192.168.4.1
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.3. Test de basculement (Failover) en condition de panne

**Objectif :** Simuler la chute du serveur DHCP principal et vérifier que le serveur partenaire attribue correctement un bail IP aux clients.

**Commande utilisée :**

```powershell title="Sur le serveur DHCP 1 (Primaire)"
Stop-Service -Name dhcpserver
```

```cmd title="Sur le poste client"
ipconfig /release
ipconfig /renew
ipconfig /all
```

* `Stop-Service` : Cmdlet PowerShell permettant d'arrêter un service Windows en cours d'exécution.
* `-Name` : Cible spécifiquement le nom du service à interrompre (ici, le Serveur DHCP).
* `ipconfig /all` : Affiche l'intégralité de la configuration TCP/IP, y compris l'adresse IP du serveur DHCP ayant fourni le bail.

**Résultat attendu :**

```cmd hl_lines="2"
Bail obtenu. . . . . . . . . . . . . . : mercredi 4 novembre 2026
Serveur DHCP . . . . . . . . . . . . . : 192.168.4.4
```

*(Le client a bien reçu une adresse IP depuis le serveur partenaire 192.168.4.4 suite à la coupure du serveur 192.168.4.3)*

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................
