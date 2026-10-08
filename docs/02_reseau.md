# Connexion réseau (Wi-Fi)

La carte a besoin d'Internet pour installer Docker et ROS 2.
Sur le réseau de l'université, la prise Ethernet ne fournit pas d'adresse IP aux appareils non déclarés : la carte est donc connectée en Wi-Fi eduroam.

!!! note "Prérequis"
    Les paquets `wpasupplicant` et `iw` doivent être présents dans l'image
    (ajoutés dans `DEBIAN_PACKAGES`, voir page précédente).

## 1. Vérifier la puce Wi-Fi

```bash
rfkill unblock wifi
ip link set wlan0 up
iw dev wlan0 scan | grep SSID
```

Les réseaux wifi disponibles dont `eduroam` doivent apparaître dans la liste.

## 2. Configuration du réseau eduroam

Paramètres de l'UBS : 

- WPA2-Entreprise, **PEAP / MSCHAPv2**, 
- certificat **HARICA TLS RSA Root CA 2021**, 
- serveur `radius.univ-ubs.fr`.

```bash
nano /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

```
network={
    ssid="eduroam"
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="identifiant@univ-ubs.fr"
    password="mot_de_passe_UBS"
    phase2="auth=MSCHAPV2"
    ca_cert="/etc/ssl/certs/HARICA_TLS_RSA_Root_CA_2021.pem"
    domain_suffix_match="radius.univ-ubs.fr"
}
```

Remplacer `identifiant` et `mot_de_passe_UBS` par ses propres identifiants UBS, puis protéger le fichier (il contient le mot de passe) :

```bash
chmod 600 /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

## 3. Tester la connexion

```bash
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
iw dev wlan0 link        # affiche "Connected to ..."
dhclient -v wlan0        # affiche "bound to ..."
ping -c 3 deb.debian.org
```

!!! tip "Diagnostic"
    En cas d'échec, lancer `wpa_supplicant` sans l'option `-B` pour afficher
    les messages. `CTRL-EVENT-EAP-SUCCESS` puis `CTRL-EVENT-CONNECTED`
    indiquent une connexion réussie.

## 4. Connexion automatique au démarrage

```bash
nano /etc/network/interfaces.d/wlan0
```

```
allow-hotplug wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

Vérification après redémarrage :

```bash
reboot
```

```bash
ip -br addr show wlan0      # adresse IP attribuée
ping -c 3 deb.debian.org    # accès Internet
```

!!! tip "Ajouter un autre réseau"
    D'autres réseaux (box, point d'accès…) peuvent être ajoutés dans le même
    fichier, avec un bloc `network={ ssid="..." psk="..." }` par réseau.