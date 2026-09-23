# Implémentation du NAT (PAT) sur un routeur Cisco

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
* [3. Identification du trafic à traduire (ACL)](#3-identification-du-trafic-a-traduire-acl)
* [4. Configuration des zones NAT sur les interfaces](#4-configuration-des-zones-nat-sur-les-interfaces)
* [5. Déclaration de la règle de translation dynamique (PAT)](#5-declaration-de-la-regle-de-translation-dynamique-pat)

## 2. Contexte

Le déploiement du NAT (Network Address Translation), et spécifiquement du PAT (Port Address Translation / Overload), permet de traduire un ensemble d'adresses IP privées vers une adresse IP publique unique. Au sein de l'infrastructure CUB, cela garantit l'accès à Internet aux réseaux LAN tout en masquant la topologie interne et en préservant le pool d'adresses IPv4.

## 3. Identification du trafic à traduire (ACL) {#3-identification-du-trafic-a-traduire-acl}

3.1. **Création de la liste de contrôle d'accès (ACL).** Définition d'une liste standard pour identifier précisément les sous-réseaux internes autorisés à subir une traduction d'adresse.

!!! INFO
    La bonne pratique impose de documenter l'ACL avec un `remark` pour garantir la lisibilité de la configuration lors d'audits ou d'intégrations IaC.

```text
Router> enable
Router# configure terminal
Router(config)# access-list 1 remark ALLOW_NAT_LAN
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255

```

* `enable` : Passage en mode d'exécution privilégié.
* `configure terminal` : Passage en mode de configuration globale.
* `access-list 1` : Initialise une liste de contrôle d'accès standard (identifiant de 1 à 99).
* `remark [Texte]` : Ajoute un libellé descriptif à l'ACL, n'affectant pas le trafic mais documentant la règle.
* `permit` : Autorise le trafic correspondant à être pris en charge par le processus de routage/traduction.
* `192.168.10.0` : Adresse du réseau source interne autorisé.
* `0.0.0.255` : Masque générique (Wildcard mask) correspondant à un masque /24. Il indique au routeur de vérifier strictement les 3 premiers octets.

## 4. Configuration des zones NAT sur les interfaces

4.1. **Assignation des rôles interne et externe.** Le routeur doit catégoriser ses interfaces pour déterminer le sens de la traduction (du réseau privé vers le réseau public).

```text
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit

```

* `interface [Nom]` : Entre dans le mode de configuration de l'interface ciblée.
* `ip nat inside` : Qualifie l'interface comme appartenant au domaine privé (LAN). Les paquets entrant par ce port sont candidats à la traduction.
* `ip nat outside` : Qualifie l'interface comme appartenant au domaine public (WAN / Internet).
* `exit` : Remonte au niveau hiérarchique parent (configuration globale).

## 5. Déclaration de la règle de translation dynamique (PAT) {#5-declaration-de-la-regle-de-translation-dynamique-pat}

5.1. **Application de la traduction avec surcharge (Overload).** Lier le trafic autorisé (ACL) à l'interface de sortie externe en activant le multiplexage des ports.

!!! WARNING
    L'oubli de l'argument `overload` configure un NAT dynamique un-pour-un (sans notion de ports). Cela bloquera l'accès à Internet dès le deuxième équipement LAN, l'IP publique étant verrouillée par le premier.

```text
Router(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload
Router(config)# end
Router# write memory

```

* `ip nat inside source` : Initie la commande de translation pour les sources provenant de la zone "inside".
* `list 1` : Appelle l'ACL numéro 1 définie à l'étape 3.1 pour filtrer les IP éligibles.
* `interface GigabitEthernet0/1` : Utilise dynamiquement l'adresse IP publique actuellement affectée à cette interface externe.
* `overload` : Active le PAT. Permet à de multiples IP privées de partager simultanément l'IP publique en attribuant un port source TCP/UDP unique à chaque flux de session.
* `end` : Quitte instantanément la configuration pour retourner au mode privilégié.
* `write memory` : Grave la configuration en cours (`running-config`) dans la mémoire persistante (`startup-config`).
