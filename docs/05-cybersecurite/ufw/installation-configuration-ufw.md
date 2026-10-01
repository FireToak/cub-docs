# Installation et configuration de UFW

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 01/10/2026
- **Domaine :** Cybersécurité

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Procédure de déploiement et configuration](#3-procedure-de-deploiement-et-configuration)

## 2. Contexte

UFW (Uncomplicated Firewall) est une surcouche simplifiée pour la gestion du pare-feu Linux (iptables/nftables). Il est déployé pour sécuriser les flux réseau de l'infrastructure en filtrant le trafic entrant et sortant. Ce service est un point de contrôle critique qui isole les applications et respecte le principe de moindre privilège au sein de l'environnement CUB.

## 3. Procédure de déploiement et configuration

3.1.  **Installation de UFW.** Installation des paquets nécessaires depuis les dépôts officiels.

```bash
# Mise à jour
sudo apt update && sudo apt upgrade -y
# Installation du paquet UFW
sudo apt install ufw -y
```

- `sudo` : Exécute la commande avec les privilèges d'administrateur (root).
- `apt update` : Met à jour la liste des paquets disponibles sur les dépôts.
- `apt install` : Installe le paquet spécifié (ici, ufw).

3.2.  **Définition des politiques par défaut.** Mise en place du principe de moindre privilège.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

- `default deny incoming` : Configure la politique par défaut pour bloquer tout trafic entrant non explicitement autorisé.
- `default allow outgoing` : Configure la politique par défaut pour autoriser tout trafic sortant initié par le serveur.

3.3.  **Ouverture des ports nécessaires.** Autorisation des flux légitimes vers les services hébergés.

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 8443/tcp
sudo ufw allow 443/tcp
```

- `allow` : Règle ajoutant une autorisation de trafic.
- `22/tcp` : Port par défaut pour le protocole SSH (accès à distance).
- `8080/tcp`, `8443/tcp`, `443/tcp` : Ports d'exemples pour autoriser le trafic web HTTP/HTTPS.

3.4.  **Activation des règles de pare-feu.** Mise en application immédiate et au démarrage du service.

> [!warning] Vérification critique de l'accès distant
> Avant d'exécuter la commande suivante, assurez-vous de bien vérifier que le port 22 (SSH) est autorisé. Dans le cas contraire, la connexion au serveur sera définitivement coupée.

```bash
sudo ufw enable
```

- `enable` : Active le pare-feu en production et configure son lancement automatique au démarrage du système.

3.5.  **Vérification de l'état.** Contrôle visuel du bon fonctionnement des règles.

```bash
sudo ufw status verbose
```

- `status` : Affiche l'état actuel d'UFW (actif/inactif).
- `verbose` : Ajoute un niveau de détail supplémentaire (politiques par défaut, logs, routage).

```text title="Exemple de sortie attendue"
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
8080/tcp                   ALLOW IN    Anywhere
8443/tcp                   ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
8080/tcp (v6)              ALLOW IN    Anywhere (v6)
8443/tcp (v6)              ALLOW IN    Anywhere (v6)
443/tcp (v6)               ALLOW IN    Anywhere (v6)
```
