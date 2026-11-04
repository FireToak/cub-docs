# Recette - DHCP Actif Passif

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 04/11/2026
- **Domaine :** Administration Windows

---

## 1. Contexte du test

Ce plan de recette a pour but de valider le bon fonctionnement et la haute disponibilité de la grappe DHCP configurée en mode **Actif/Passif** (Hot Standby). Il s'agit de s'assurer que le serveur principal traite la majorité des requêtes, et qu'en cas d'interruption de son service, le serveur partenaire (en attente) prend immédiatement le relais en utilisant son pool d'adresses de réserve (10%) pour assurer la continuité de service.

> [!warning] Interruption de service simulée
> Le test de basculement implique l'arrêt volontaire du service DHCP sur le nœud primaire. Bien que le relais soit pris par le serveur passif, il est recommandé de réaliser ce test lors d'une fenêtre de maintenance pour s'assurer que le relais DHCP (IP Helper) transmet bien les requêtes au second nœud.

## 2. Procédures de validation

### 2.1. Vérification de l'état du partenariat Actif/Passif

**Objectif :** S'assurer que le partenariat DHCP est actif, synchronisé et correctement configuré en mode secours (Hot Standby) avec l'attribution des rôles adéquats.

**Commande utilisée :**

```powershell title="Sur le serveur DHCP Primaire"
Get-DhcpServerv4Failover -ComputerName localhost
```

* `Get-DhcpServerv4Failover` : Cmdlet PowerShell qui interroge et retourne les propriétés de la configuration de basculement DHCP IPv4.
* `-ComputerName` : Spécifie le serveur cible sur lequel exécuter la requête.

**Résultat attendu :**

```powershell hl_lines="3-5"
Name                 : Failover-CUB-Actif-Passif
PartnerServer        : 192.168.4.4
Mode                 : HotStandby
ReservePercent       : 10
ServerRole           : Active
State                : Normal
```

*(Sur le serveur secondaire, la valeur `ServerRole` doit afficher `Standby`)*

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.2. Validation de l'attribution IP initiale par le nœud actif

**Objectif :** Confirmer qu'un client du réseau obtient bien une configuration IP dynamique fournie par le serveur primaire.

**Commande utilisée :**

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all

```

* `ipconfig` : Utilitaire réseau Windows affichant la configuration TCP/IP.
* `/release` : Libère le bail IPv4 actuel.
* `/renew` : Renouvelle le bail IPv4 auprès du serveur DHCP.
* `/all` : Affiche les informations détaillées, dont l'adresse du serveur DHCP fournisseur.

**Résultat attendu :**

```cmd hl_lines="4"
Carte Ethernet Ethernet0 :
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.4.150
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Serveur DHCP . . . . . . . . . . . . . : 192.168.4.3

```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.3. Test de basculement (Failover) vers le nœud passif

**Objectif :** Simuler la chute du serveur DHCP principal et vérifier que le serveur partenaire (passif) attribue un bail IP de secours aux clients via ses 10% d'adresses réservées.

**Commande utilisée :**

```powershell title="Sur le serveur DHCP 1 (Primaire)"
Stop-Service -Name dhcpserver

```

```cmd title="Sur le poste client"
ipconfig /release
ipconfig /renew
ipconfig /all

```

* `Stop-Service` : Arrête un service Windows en cours d'exécution pour simuler une panne.
* `-Name` : Cible le nom court du service DHCP.
* `ipconfig /release` et `/renew` : Forcent le client à redemander une IP alors que le serveur 1 est éteint.

**Résultat attendu :**

```cmd hl_lines="4"
Carte Ethernet Ethernet0 :
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.4.240
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Serveur DHCP . . . . . . . . . . . . . : 192.168.4.4

```

*(Le client reçoit une adresse issue du pool de réserve depuis le serveur 192.168.4.4)*

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................
