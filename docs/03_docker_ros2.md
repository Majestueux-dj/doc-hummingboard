# Docker et ROS 2

La carte fonctionne sous Debian. ROS 2, lui, est installé dans un conteneur Docker qui contient un système Ubuntu 22.04, isolé du reste de la carte.

Ce conteneur n'a pas son propre noyau : il utilise celui de Debian. Il a donc accès au matériel de la carte (Wi-Fi, NPU, etc.) grâce aux drivers déjà présents dans Debian.

## 1. Installer Docker sur la carte

```bash
apt update
apt install -y docker.io
systemctl status docker      # affiche "active (running)"
```

Docker démarre automatiquement à chaque démarrage de la carte (voir `.bashrc`)

## 2. Télécharger l'image ROS 2

L'image officielle contient déjà Ubuntu 22.04 et ROS 2 Humble :

```bash
docker pull ros:humble-ros-base
```

## 3. Créer le conteneur du projet

Dossier de travail côté Debian :

```bash
mkdir -p /root/project
```

Création du conteneur (**une seule fois**) :

```bash
docker run -it --name project_ros --network host \
  -v /root/project:/project ros:humble-ros-base bash
```

| Option | Rôle |
|---|---|
| `--name project_ros` | Nom du conteneur, pour le retrouver |
| `--network host` | Le conteneur utilise directement le réseau de la carte |
| `-v /root/project:/project` | Dossier partagé : les fichiers sont stockés côté Debian |
| (pas de `--rm`) | Le conteneur est conservé à la sortie |

Vérification dans le conteneur :

```bash
cat /etc/os-release | head -2   # Ubuntu 22.04
ls /project
ros2 topic list                 # affiche /parameter_events et /rosout
```

## 4. Installer les outils de base dans le conteneur

L'image ROS 2 est minimale. Outils ajoutés :

```bash
apt update
apt install -y nano git python3-pip python3-colcon-common-extensions
```

Ces installations sont conservées, car le conteneur n'est pas supprimé.

## 5. Utilisation au quotidien

| Action | Commande |
|---|---|
| Entrer dans le conteneur | `docker start -ai project_ros` |
| En sortir | `exit` |
| Lister les conteneurs | `docker ps -a` |
| Fichiers côté Debian | `/root/project/` |
| Fichiers côté Ubuntu | `/project/` |

!!! danger "À éviter"
    - Ne **pas** relancer `docker run` pour ce conteneur : il existe déjà
      (utiliser `docker start -ai project_ros`).
    - Ne **pas** utiliser `docker container prune` : cette commande supprime
      tous les conteneurs arrêtés, y compris `project_ros`.