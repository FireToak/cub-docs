# Recette - Service DHCP

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 18/10/2026
- **Domaine :** Administration Windows

---

## 1. Contexte du test

Cette fiche de recette a pour objectif de valider la conformité du paramétrage du service DHCP déployé sous Windows Server 2025. Le scénario de test permet de vérifier l'attribution dynamique des paramètres réseau pour l'étendue "Clients", la bonne application des options distribuées (DNS, nom de domaine, passerelle), le respect d'un bail de 4 jours, ainsi que le fonctionnement rigoureux des exclusions et des réservations IP spécifiques au service.

## 2. Procédures de validation

### 2.1. Obtention d'un bail DHCP et vérification des options réseau

**Objectif :** S'assurer que le poste client reçoit bien une adresse IP dynamique, le masque de sous-réseau, la passerelle, ainsi que les options DHCP exigées (serveur DNS `8.8.8.8` et nom de domaine `local.anvers.cub.sioplc.fr`).

**Commande utilisée :**

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

- `ipconfig /release` : Libère la configuration IPv4 actuelle attribuée à la carte réseau du client.
- `ipconfig /renew` : Force la diffusion d'une trame DHCPDISCOVER pour solliciter l'attribution d'un nouveau bail IP.
- `ipconfig /all` : Affiche la configuration réseau détaillée, incluant les options de domaine et la durée de validité du bail.

**Résultat attendu :**

```text
Configuration IP de Windows

   Nom de l'hôte . . . . . . . . . . : CLIENT-TEST
   Suffixe DNS principal . . . . . . : local.anvers.cub.sioplc.fr

Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion : local.anvers.cub.sioplc.fr
   Adresse IPv4. . . . . . . . . . . : 192.168.1.150(préféré)
   Masque de sous-réseau . . . . . . : 255.255.255.0
   Bail obtenu . . . . . . . . . . . : dimanche 18 octobre 2026 10:00:00
   Bail expirant . . . . . . . . . . : jeudi 22 octobre 2026 10:00:00
   Passerelle par défaut . . . . . . : 192.168.1.254
   Serveurs DHCP . . . . . . . . . . : 192.168.1.3
   Serveurs DNS. . . . . . . . . . . : 8.8.8.8
```

> [!note] Validation de la durée du bail
> L'écart temporel entre "Bail obtenu" et "Bail expirant" doit être très exactement de 4 jours, conformément au cahier des charges de l'agence d'Anvers.

**Statut :**

- [ ] Ok
- [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.2. Validation de la réservation d'adresse IP

**Objectif :** Vérifier que le poste spécifique du service (identifié de manière unique par son adresse MAC) reçoit systématiquement l'adresse IP réservée `192.168.1.179`.

**Commande utilisée :**

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.1.0 | Where-Object {$_.IPAddress -eq "192.168.1.179"}
```

- `Get-DhcpServerv4Lease` : Récupère la liste intégrale des baux actifs actuellement gérés par le serveur DHCP.
- `-ScopeId` : Spécifie l'identifiant du réseau cible sur lequel opérer la requête.
- `Where-Object` : Filtre la sortie de la cmdlet pour isoler uniquement la ligne correspondant à l'adresse IP réservée.

**Résultat attendu :**

```powershell
IPAddress       ScopeId         ClientId             HostName             AddressState
---------       -------         --------             --------             ------------
192.168.1.179   192.168.1.0     00-15-5d-a1-b2-c3    POSTE-SERVICE        ActiveReservation
```

**Statut :**

- [ ] Ok
- [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.3. Contrôle de la plage d'exclusion

**Objectif :** Confirmer que la plage d'adresses IP s'étendant de `192.168.1.180` à `192.168.1.189` est rigoureusement exclue du pool de distribution dynamique.

**Commande utilisée :**

```powershell
Get-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0
```

- `Get-DhcpServerv4ExclusionRange` : Affiche les plages d'adresses IP configurées pour être exclues de la distribution DHCP sur une étendue.
- `-ScopeId` : Identifiant du réseau correspondant à l'étendue à auditer.

**Résultat attendu :**

```powershell
StartRange      EndRange
----------      --------
192.168.1.180   192.168.1.189
```

> [!success] Critère de succès complémentaire
> Pour garantir un test exhaustif de l'exclusion, le renouvellement de multiples baux sur plusieurs clients virtuels ou physiques ne doit en aucun cas déclencher l'attribution d'une adresse comprise dans cette plage.

**Statut :**

- [ ] Ok
- [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.4. Contrôle des baux actifs et synchronisation côté serveur

**Objectif :** Vérifier que le serveur DHCP enregistre correctement le bail dans sa base de données et que ce dernier est visible par l'administrateur.

**Commande utilisée :**

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.1.0
```

* `Get-DhcpServerv4Lease` : Extrait la liste de tous les baux IP actifs ou expirés pour une étendue donnée.
* `-ScopeId` : Cible l'identifiant réseau de l'étendue à inspecter.

**Résultat attendu :**

```text
IPAddress       ScopeId         ClientId             HostName             AddressState
---------       -------         --------             --------             ------------
192.168.1.150   192.168.1.0     00-15-5d-22-33-44    CLIENT-TEST          Active
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.5. Vérification de l'état du partenariat Failover

**Objectif :** S'assurer que la grappe de haute disponibilité (Actif/Actif ou Actif/Passif) est saine et que la communication entre le serveur principal et le serveur secondaire est établie.

**Commande utilisée :**

```powershell
Get-DhcpServerv4Failover | Format-List Name, Mode, State, PartnerServer
```

* `Get-DhcpServerv4Failover` : Récupère les informations d'état et la configuration de la relation de basculement locale.
* `Format-List` : Filtre l'affichage de la console pour ne conserver que les paramètres vitaux de la grappe.

**Résultat attendu :**

```text
Name           : dhcp-failover-cub
Mode           : LoadBalance
State          : Normal
PartnerServer  : 192.168.1.4
```

> [!success] Critère de validation
> La propriété `State` doit obligatoirement retourner `Normal`. Toute autre valeur (ex: `CommunicationInterrupted`) indique une rupture critique de la synchronisation des baux.

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.6. Test de basculement (Résilience du service)

**Objectif :** Simuler la chute du serveur principal pour garantir que le délai de transition maximal est respecté et que le serveur partenaire prend le relais de la distribution réseau.

**Commande utilisée :**

```powershell
Stop-Service DHCPServer
# ou
Stop-Computer
```

```cmd
ipconfig /renew
```

* `Stop-Service` : Interrompt volontairement le processus DHCP sur le nœud actif pour simuler un crash.
* `ipconfig /renew` : Lance une requête d'attribution IP depuis le client pendant la panne du nœud principal.

**Résultat attendu :**

```text
   Adresse IPv4. . . . . . . . . . . : 192.168.1.150(préféré)
   Serveurs DHCP . . . . . . . . . . : 192.168.1.4
```

> [!note] Preuve de bascule
> En exécutant `ipconfig /all` sur le client, le champ **Serveurs DHCP** doit désormais indiquer l'adresse IP du nœud partenaire. La continuité de service est ainsi validée.

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................