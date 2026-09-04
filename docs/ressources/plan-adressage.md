# Plan d'adressage

![Bannière CUB](https://raw.githubusercontent.com/FireToak/cub-docs/main/docs/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 04/09/2026
- **Domaine :** Réseaux

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Plan d'adressage](#3-plan-dadressage)
- [4. Calcul du plan d'adressage](#4-calcul-du-plan-dadressage)

## 2. Contexte

Ce document définit le plan d'adressage IP (IPv4) de l'infrastructure réseau. La méthode VLSM (Variable Length Subnet Masking) est utilisée pour optimiser l'allocation des adresses IP en adaptant la taille de chaque sous-réseau aux stricts besoins d'hôtes requis par composant. L'objectif est de segmenter logiquement le réseau via des VLANs (DMZ, Production, Clients, Administration, Inter-connexion) afin d'assurer l'isolation des flux, la sécurité et un routage optimisé.

## 3. Plan d'adressage

| ID VLAN | Nom | Réseau | CIDR | Masque | Première @ | Dernière @ | Broadcast |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 84 | DMZ (externe) | 192.36.4.0/24 | /24 | 255.255.255.0 | 192.36.4.1 | 192.36.4.254 | 192.36.4.255 |
| 54 | Production | 192.168.4.0 | /25 | 255.255.255.128 | 192.168.4.1 | 192.168.4.126 | 192.168.4.127 |
| 10 | Clients | 192.168.4.128 | /26 | 255.255.255.192 | 192.168.4.129 | 192.168.4.190 | 192.168.4.191 |
| 20 | Administration | 192.168.4.192 | /28 | 255.255.255.240 | 192.168.4.193 | 192.168.4.206 | 192.168.4.207 |
| 2 | Inter-co SW L3 et FW | 192.168.44.248 | /29 | 255.255.255.248 | 192.168.44.249 | 192.168.44.254 | 192.168.44.255 |

## 4. Calcul du plan d'adressage

### 4.1. Figure 1 - Tableau des valeurs des puissances de 2

| Valeurs ² | Puissances ² |
| --- | --- |
| 128 | 7 |
| 64 | 6 |
| 32 | 5 |
| 16 | 4 |
| 8 | 3 |
| 4 | 2 |
| 2 | 1 |

### 4.2. Calcul du réseau `Production`

**1. Trouver la puissance de 2 (bits d'hôtes)**
Pour héberger un besoin d'au moins 121 hôtes, il faut trouver la puissance de 2 permettant d'obtenir ce nombre de machines (en retirant l'adresse réseau et l'adresse de broadcast) :

- Équation : `2^x - 2 >= 121`
- Calcul : `2^7 - 2 = 126` (qui est >= 121)
- Il faut donc **7 bits** réservés pour les hôtes.

**2. Déterminer le préfixe CIDR**
Le préfixe définit la partie réseau. Une adresse IPv4 comportant 32 bits au total :

- Calcul : `32 bits - 7 bits = 25 bits`
- Le CIDR est donc **/25**.

**3. Calculer le masque de sous-réseau en décimal**
Le CIDR /25 signifie que les 25 premiers bits du masque sont à 1 :

- Notation binaire : `11111111 . 11111111 . 11111111 . 10000000`
- Conversion d'un octet plein (`11111111`) : Addition des poids `128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255`.
- Conversion du dernier octet (`10000000`) : Seul le premier bit est actif, son poids est de `128`.
- Masque décimal : **255.255.255.128**

**4. Déterminer la plage d'adresses**
L'adresse réseau de départ est `192.168.4.0`. Le pas du sous-réseau (nombre total d'adresses dans le bloc) est de `2^7 = 128`.

- **Adresse de broadcast :** C'est la dernière adresse du bloc, soit l'adresse réseau à laquelle on ajoute toutes les valeurs d'hôtes possibles (`128 - 1 = 127`).
  - `192.168.4.0 + 127 = 192.168.4.127`
- **Première IP utilisable :** C'est l'adresse réseau incrémentée de 1.
  - `192.168.4.0 + 1 = 192.168.4.1`
- **Dernière IP utilisable :** C'est l'adresse de broadcast décrémentée de 1.
  - `192.168.4.127 - 1 = 192.168.4.126`
