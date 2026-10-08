# Génération et installation de l'image

L'image Debian de la carte est générée sur un PC à partir des commandes suivantes de build SolidRun, puis flashée sur une carte microSD.

## Prérequis

- Un PC sous Linux avec Docker installé
- Une carte microSD (32 Go utilisés ici)

!!! note "Docker sans sudo"
    L'utilisateur doit appartenir au groupe `docker` :
```bash
    sudo usermod -aG docker $USER
```
    Puis se déconnecter et se reconnecter.

## 1. Récupérer les scripts de build SolidRun

La branche `develop-lf-6.6.52-2.2.0-imx8mp` est la version maintenue (SDK NXP 6.6.52, U-Boot v2024.04, Linux 6.6).

```bash
git clone -b develop-lf-6.6.52-2.2.0-imx8mp https://github.com/SolidRun/imx8mp_build.git
cd imx8mp_build
```

## 2. Configurer le build

Les options se trouvent au début du fichier `runme.sh` :

| Option | Valeur par défaut | Valeur choisie |
|---|---|---|
| `DISTRO` | `buildroot` | `debian` |
| `DEBIAN_ROOTFS_SIZE` | `936M` | `8G` |
| `DEBIAN_PACKAGES` | (liste des paquets) | + `wpasupplicant,iw,wireless-regdb,htop` |

Lignes modifiées dans `runme.sh` :

```bash
: ${DISTRO:=debian}
: ${DEBIAN_ROOTFS_SIZE:=8G}
```

Ajout des paquets à la fin de la ligne `DEBIAN_PACKAGES` :

```bash
sed -i 's/xterm,xz-utils"}/xterm,xz-utils,wpasupplicant,iw,wireless-regdb,htop"}/' runme.sh
```

## 3. Générer l'image

Construire l'environnement de build (une seule fois) :

```bash
docker build -t imx8mp_build docker
```

Lancer la génération de l'image :

```bash
docker run --rm -i -t -v "$PWD":/work imx8mp_build -u $(id -u) -g $(id -g)
```

En fin de build, l'image est disponible dans `images/` :

```
images/debian-bootimg-<hash>.img
```

## 4. Flasher la carte microSD

Identifier la carte SD :

```bash
lsblk
```

Pour vérifier si la carte SD est bien listée dans les périphériques.

```bash
sudo umount /dev/sdX*
sudo dd if=images/debian-bootimg-<hash>.img of=/dev/sdX bs=4M status=progress conv=fsync
sync
sudo eject /dev/sdX
```
Remplacer `sdX` par le nom du périphérique correspondant à la carte microSD.

Après le flash, la SD contient deux partitions : le `boot` (~60 Mo) et le `rootfs` (~8 Go).

## 5. Démarrer sur la carte

1. Configurer le boot select S3 pour la microSD : switches **1 et 2 sur ON**.
2. Ouvrir la console série avec :
```bash
   sudo minicom -b 115200 -D /dev/ttyUSB0
```
3. Se connecter avec l'utilisateur `root` et le mot de passe par défaut : `root`.