# Implémentation du Relais DHCP sur équipement Cisco

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
* [3. Implémentation de l'agent relais (IP Helper)](#3-implementation-de-lagent-relais-ip-helper)

## 2. Contexte

Le protocole DHCP repose nativement sur des requêtes de diffusion (Broadcast) systématiquement bloquées par les limites de domaines de diffusion (routeurs ou commutateurs de niveau 3). L'implémentation d'un agent relais DHCP via la fonction IP Helper permet d'écouter ces diffusions locales et de les encapsuler dans des trames monodiffusées (Unicast) à destination d'un serveur DHCP centralisé. Au sein de l'infrastructure CUB, cette approche d'ingénierie centralisée évite le déploiement de serveurs DHCP dédiés dans chaque VLAN d'accès.

## 3. Implémentation de l'agent relais (IP Helper) {#3-implementation-de-lagent-relais-ip-helper}

4.1. **Configuration du relais DHCP sur l'interface passerelle.** Définition de l'adresse du serveur DHCP cible sur l'interface servant de passerelle par défaut pour le sous-réseau client (Interface physique sur un routeur, ou Interface VLAN / SVI sur un commutateur).

!!! INFO
    La commande `ip helper-address` transmet par défaut 8 services UDP (dont TFTP, DNS, Time, et DHCP). Dans des environnements de très haute sécurité, il est recommandé de filtrer les ports relayés avec `no ip forward-protocol udp`.

```text
Switch(config)# interface Vlan 10
Switch(config-if)# ip address 192.168.10.254 255.255.255.0
Switch(config-if)# ip helper-address 10.0.99.5
Switch(config-if)# end
Switch# write memory

```

* `interface Vlan 10` : Sélectionne l'interface de routage associée au sous-réseau devant bénéficier du service DHCP (Interface Virtuelle Switchée).
* `ip address` : Assigne l'adresse IP de passerelle. Cette adresse est cruciale, car elle est insérée dans le champ "GIADDR" (Gateway IP Address) du paquet DHCP relayé, permettant au serveur DHCP d'identifier depuis quel sous-réseau provient la demande.
* `ip helper-address 10.0.99.5` : Active la fonction relais. Transforme la diffusion locale en requête Unicast ciblée vers l'adresse IP du serveur DHCP central (`10.0.99.5` dans cet exemple).
* `end` : Quitte la configuration de l'interface pour revenir au mode privilégié.
* `write memory` : Sauvegarde la configuration active de l'agent relais dans la NVRAM.
