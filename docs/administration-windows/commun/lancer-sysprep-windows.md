# Lancer un sysprep sur Windows

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 16/09/2026
- **Domaine :** Windows

---

## 1. Contexte

L'utilitaire Sysprep (System Preparation) prépare une installation Windows pour la création d'images (imaging) et le déploiement automatisé. Il supprime les informations de sécurité spécifiques au système (comme le SID) pour garantir l'unicité des futurs nœuds lors du clonage de machines virtuelles (VM).

## 2. Exécution de Sysprep

3.1.  **Lancement de la préparation du système.** Exécuter la commande de généralisation dans une invite de commandes (cmd) avec les privilèges administrateur.

```cmd
%WINDIR%\system32\sysprep\sysprep.exe /generalize /oobe /shutdown
```

- `%WINDIR%\system32\sysprep\sysprep.exe` : Fichier exécutable principal de l'outil de préparation système de Windows.
- `/generalize` : Argument qui supprime les données spécifiques au système d'exploitation (identifiants uniques, journaux, points de restauration). Indispensable pour créer une image master.
- `/oobe` : Argument (Out-of-Box Experience) qui configure Windows pour présenter l'assistant de configuration initiale au prochain démarrage de l'image clonée.
- `/shutdown` : Argument qui force l'extinction de la machine de manière propre à la fin du processus Sysprep, permettant ainsi de capturer l'image disque à froid.
