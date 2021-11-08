# Linux Alpine OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

### Pourquoi utiliser AlpineOS sur le Raspberry Pi ? Voici quelques raisons :

* **Très faible consommation de mémoire \(~50Mo utilisés pendant l'inactivité vs ~350Mo pour Ubuntu 20.04\\).**
* **Diminuer la surcharge CPU** **\(27 tâches/ 31 threads actifs pour Alpine vs 57 tâches / 111 threads pour Ubuntu lorsque cardano-node est en cours d'exécution\).**
* **Pi plus cool 😎 \(Litéralement, la réduction de la surcharge garde le Processeur plus frais\\).**
* **Et finalement, pourquoi pas? Si vous planifiez d'utiliser des exécutables statiques, vous pourriez aussi tirer parti de AlpineOS 😜**

## Configuration initiale pour AlpineOS sur Raspberry Pi 4B 8GB :

1\) Téléchargez AlpineOS pour RPi 4 aarch64 ici : [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)

2\) Décompresser le fichier .tar.gz et copier son contenu sur une carte SSD/SD

3\) Branchez un clavier et un moniteur.

4\) Connectez-vous avec le nom d'utilisateur 'root'.

5\) Exécutez la commande `setup-alpine` et suivez les instructions.

{% hint style="info" %}
Lorsque vous êtes dans `setup-alpine`  vous serez invité à choisir le disque système. Une fois que vous êtes à ce stade, saisissez, `y`, pour configurer le disque et créer la partition pour `sys`.
{% endhint %}



6\) Redémarrez.

7\) Ajouter un nouvel utilisateur appelé cardano via la commande `adduser cardano` et son mot de passe comme indiqué. \(Pour un nom d'utilisateur autre que **cardano**, reportez-vous au **Dépannage général**\)

8\) Exécutez les commandes suivantes pour accorder au nouvel utilisateur les privilèges "root"

```text
apk add sudo
echo '%wheel ALL=(ALL) ALL' > /etc/sudoers.d/wheel
addgroup cardano wheel
addgroup cardano sys
addgroup cardano adm
addgroup cardano root
addgroup cardano bin
addgroup cardano daemon
addgroup cardano disk
addgroup cardano floppy
addgroup cardano dialout
addgroup cardano tape
addgroup cardano video
```

9\) Quitter l'utilisateur "root" via la commande `exit` ou redémarrer et se reconnecter en tant que "cardano"

10\) Installez bash pour assurer la compatibilité avec les scripts bash

```text
    sudo apk add bash
```

11\) Installez également git et wget, nous en aurons besoin plus tard.

```text
    sudo apk add git wget
```

### Installer les exécutables statiques 'cardano-node' et 'cardano-cli' \\(AlpineOS utilise presque exclusivement des exécutables statiques, donc vous devriez éviter les compilations non statiques\\)

{% hint style="info" %}
**Vous pouvez obtenir les exécutables statiques pour la version 1.27. via ce lien** [****](https://ci.zw3rk.com/build/1758) **grâce à la courtoisie de Moritz Angermann, le SPO du pool ZW3RK 🙏**
{% endhint %}

**Exécutez les commandes suivantes pour installer les exécutables et les placer dans le bon répertoire.**

* Télécharger les exécutables

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
```

* Décompressez et installez les exécutables via les commandes

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Installer le service de l'Alliance Armada pour Alpine Linux Cardano Node

{% hint style="success" %}
#### Si vous avez décidé d'utiliser AlpineOS pour vos opérations de stake pool Cardano, vous trouverez peut-être cette collection de scripts et de services utiles.
{% endhint %}

{% hint style="info" %}
#### Pour installer correctement les scripts et les services, ne sautez pas les étapes 🏴‍☠️😎
{% endhint %}

1\) Clonez ce dépôt pour obtenir le dossier et les scripts nécessaires pour démarrer rapidement votre noeud Cardano. Utilisez la commande :

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

2\) Exécutez les commandes suivantes pour installer le dossier **cnode** , les scripts et les services dans les bons dossiers. Le dossier **cnode** contient tout ce que **cardano-node** a besoin pour démarrer un relais fonctionnel :

```text
    cp -r alpine-rpi-os/alpine_cnode_scripts_and_services/home/cardano/* ~/
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/init.d/* /etc/init.d/
```

```text
    chmod +x ~/start_stop_cnode_service.sh ~/cnode/autorestart_cnode.sh
```

```text
    sudo chmod +x /etc/init.d/cardano-node /etc/init.d/prometheus /etc/init.d/node-exporter
```

3\) Pour une synchronisation plus rapide, considérez cette commande optionnelle pour télécharger la dernière base de données hébergée par l'un des membres de l'Alliance.

```text
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4\) Suivez le guide **README.txt** contenu dans le répertoire **$HOME** après avoir installé **cnode**, scripts et services.

```text
    more ~/README.txt
```

## Configuration de Prometheus et de Node-Exporter

1\) Téléchargez Prometheus et node-exporter dans le répertoire personnel

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
```

2\) Extraire les archives tarballs

```text
tar -xzvf prometheus.tar.gz
```

```text
tar -xzvf node_exporter.tar.gz
```

3\) Renommer les dossiers avec les commandes suivantes

```text
    mv prometheus-2.27.1.linux-arm64 prometheus
```

```text
    mv node_exporter-1.1.2.linux-arm64 node_exporter
```

4\) Suivez le guide README.txt contenu dans le répertoire $HOME après avoir installé cnode, scripts et services.

```text
    more ~/README.txt
```

## Dépannage général

* Si vous utilisez un autre nom d'utilisateur que celui de cardano, utilisez les commandes suivantes et remplacez `nom d'utilisateur` par le nom d'utilisateur que vous avez choisi.

```text
    sed -i 's@/home/cardano@/home/<username>@g' ~/cnode_env
```

```text
    sudo sed -i 's@/home/cardano@/home/<username>@g' /etc/init.d/cardano-node
```

```text
    sudo sed -i 's@/home/cardano@/home/<username>@g' /etc/init.d/prometheus
```

```text
    sudo sed -i 's@/home/cardano@/home/<username>@g' /etc/init.d/node-export
```

* Si vous avez des problèmes avec la redirection de port via SSH, exécutez la commande suivante

```text
sudo nano /etc/ssh/sshd_config
```

* Modifier la ligne `AllowTcpForwarding no` pour `AllowTcpForwarding yes`

{% hint style="info" %}
  Assurez-vous que cette ligne n'est pas commentée avec un`#`
{% endhint %}

{% hint style="success" %}
Nous aimerions souligner spécialement notre [membre d'alliance](https://armada-alliance.com) Sayshar, opérateur de [\[SRN\] Pool](https://www.adasrn.com/), pour fournir ce tutoriel 🏴‍☠️ 🙏 😎
{% endhint %}



