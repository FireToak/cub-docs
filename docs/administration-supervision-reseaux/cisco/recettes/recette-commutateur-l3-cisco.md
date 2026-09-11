# Fiche recette - Commutateur L3

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 11/09/2026
- **Domaine :** Réseau

---

## 1. Contexte du test

Validation de l'infrastructure réseau (couches 2 et 3 du modèle OSI) du commutateur L3 `dmd-sw-c1` de l'agence de Dortmund. Ce document vérifie la bonne configuration des interfaces de routage (SVI), la connectivité interne vers la passerelle par défaut (`192.168.44.254`), ainsi que le routage externe (accès Internet) nécessitant l'intervention du pare-feu et de ses règles NAT.

## 2. Procédures de validation

### 2.1. Vérification des interfaces virtuelles (SVI)

**Objectif :** S'assurer que les interfaces VLAN sont actives et correctement adressées pour assurer le routage inter-VLAN.

**Commande utilisée :**

```bash
show ip interface brief | exclude unassigned
```

* `show ip interface brief` : Affiche un résumé de l'état matériel/logiciel et des adresses IP des interfaces.
* `| exclude unassigned` : Filtre la sortie de la commande pour masquer les interfaces n'ayant pas d'adresse IP.

**Résultat attendu :**

```bash
Interface              IP-Address      OK? Method Status                Protocol
Vlan2                  192.168.44.253  YES manual up                    up      
Vlan10                 192.168.4.190   YES manual up                    up      
Vlan20                 192.168.4.206   YES manual up                    up      
Vlan54                 192.168.4.126   YES manual up                    up      
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.2. Vérification de la table de routage globale

**Objectif :** Contrôler la présence de la route par défaut vers le pare-feu, indispensable pour atteindre les réseaux inconnus.

**Commande utilisée :**

```bash
show ip route static
```

* `show ip route` : Affiche la table de routage active (Couche 3 OSI).
* `static` : Argument filtrant l'affichage pour ne montrer que les routes configurées statiquement.

**Résultat attendu :**

```bash
S*    0.0.0.0/0 [1/0] via 192.168.44.254
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.3. Communication avec la passerelle par défaut

**Objectif :** Valider la connectivité réseau interne entre le commutateur et le pare-feu `dmd-fw-c01` sur le sous-réseau d'interconnexion.

**Commande utilisée :**

```bash
ping 192.168.44.254
```

* `ping` : Utilise le protocole ICMP (Echo Request) pour tester la joignabilité réseau d'un équipement.
* `192.168.44.254` : Adresse IP cible (le pare-feu agissant comme passerelle sur le VLAN 2).

**Résultat attendu :**

```bash
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.44.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.4. Validation de l'accès à Internet

> [!warning] Dépendance matérielle et sécurité
> Le pare-feu `dmd-fw-c01` doit obligatoirement être opérationnel. Sa règle NAT doit être active pour translater les adresses internes (ex: `192.168.4.0/24`) vers l'adresse externe `192.36.253.40`, sinon les paquets ICMP seront rejetés.

**Objectif :** Confirmer le routage de bout en bout vers Internet et valider le bon fonctionnement du NAT sur le pare-feu.

**Commande utilisée :**

```bash
ping 8.8.8.8 source vlan 10
```

* `ping` : Teste la connectivité vers un hôte distant.
* `8.8.8.8` : Adresse IP publique de test (serveur DNS).
* `source vlan 10` : Force le ping à utiliser l'adresse IP du VLAN 10 (`192.168.4.190`) comme IP source, permettant ainsi de vérifier que le flux des utilisateurs est bien autorisé et translaté par le pare-feu.

**Résultat attendu :**

```bash
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/15/20 ms
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................
