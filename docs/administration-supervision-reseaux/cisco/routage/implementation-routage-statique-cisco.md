# Implémentation du routage statique (Cisco)

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

* **Auteur :** Louis MEDO
* **Date :** 05/09/2026
* **Domaine :** Réseaux

---

## 1. Sommaire

* [1. Sommaire](#1-sommaire)
* [2. Contexte](#2-contexte)
* [3. Activation du routage (Commutateurs de niveau 3)](#3-activation-du-routage-commutateurs-de-niveau-3)
* [4. Configuration d'une route statique standard](#4-configuration-dune-route-statique-standard)
* [5. Configuration d'une route par défaut (Gateway of last resort)](#5-configuration-dune-route-par-defaut-gateway-of-last-resort)

## 2. Contexte

Le routage statique permet de définir manuellement les chemins d'acheminement des paquets IP vers des réseaux cibles. Au sein de l'infrastructure CUB, cette méthode est utilisée pour les topologies de bordure (Edge) afin de garantir la prédictibilité des flux réseau sans la surcharge logicielle d'un protocole de routage dynamique.

## 3. Activation du routage (Commutateurs de niveau 3)

3.1. **Activation du moteur de routage.** Les routeurs acheminent le trafic IP nativement, mais les commutateurs multicouches nécessitent une activation explicite de leur table de routage globale.

!!! WARNING
    Sans cette commande préliminaire, un commutateur de niveau 3 fonctionnera strictement au niveau 2 (Switching pur) et ignorera vos directives de routage.

```text
Switch> enable
Switch# configure terminal
Switch(config)# ip routing

```

* `enable` : Passage en mode d'exécution privilégié.
* `configure terminal` : Passage en mode de configuration globale.
* `ip routing` : Active les capacités de routage de niveau 3 sur l'équipement.

## 4. Configuration d'une route statique standard

4.1. **Création d'une entrée statique ciblée.** Déclaration explicite d'un chemin fixe pour joindre un sous-réseau distant précis.

```text
Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2

```

* `ip route` : Instruit l'ajout d'une ligne fixe dans la table de routage.
* `192.168.20.0` : Adresse réseau de la destination à atteindre.
* `255.255.255.0` : Masque définissant la taille du réseau cible.
* `10.0.0.2` : Adresse IP du prochain saut (Next-Hop), qui doit être l'équipement voisin directement connecté capable d'acheminer le paquet.

## 5. Configuration d'une route par défaut (Gateway of last resort) {#5-configuration-dune-route-par-defaut-gateway-of-last-resort}

5.1. **Mise en place de la passerelle de dernier recours.** Transfert systématique des paquets destinés à des réseaux inconnus de la table de routage (ex: Internet) vers une porte de sortie unique.

```text
Router(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.254
Router(config)# end
Router# write memory

```

* `0.0.0.0 0.0.0.0` : Route "Quad-Zero" agissant comme un filet de sécurité (Catch-all) pour toute adresse IP non couverte par une route plus spécifique.
* `10.0.0.254` : Adresse IP de la passerelle de sortie du site.
* `end` : Retour direct au mode d'exécution privilégié.
* `write memory` : Sauvegarde la table de routage configurée dans la NVRAM pour résister aux redémarrages.
