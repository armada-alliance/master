# Linux Alpine OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

## Pourquoi utiliser AlpineOS sur le Raspberry Pi ? Voici quelques raisons :

* **Très faible consommation de mémoire \(~50Mo utilisés pendant l'inactivité vs ~350Mo pour Ubuntu 20.04\\).**
* **Diminuer la surcharge CPU** **\(27 tâches/ 31 threads actifs pour Alpine vs 57 tâches / 111 threads pour Ubuntu lorsque cardano-node est en cours d'exécution\).**
* **Pi plus cool 😎 \(Litéralement, la réduction de la surcharge garde le Processeur plus frais\\).**
* **Et finalement, pourquoi pas? Si vous planifiez d'utiliser des exécutables statiques, vous pourriez aussi tirer parti de AlpineOS 😜**

## If you have previously used this guide and intend to update the scripts. Follow these steps. Then follow the rest of the steps outlined in this guide accordingly 🙂.

1\) Update the git local repo.

```text
cd ~/alpine-rpi-os
```

```text
git fetch --recurse-submodules --tags --all
```

2\) Identify the latest tag.

```text
git tag
```

3\) Replace `<tag>` in this step with the latest tag such as `v1.2.1`.

```text
git checkout tags/<tag>
```

## Upgrading to Alpine v3.14 from Alpine v3.13:

1\) Update your current version of AlpineOS.

```text
sudo apk update
```

```text
sudo apk upgrade
```

2\) Edit the repository to reflect Alpine v3.14.

```text
sudo sed -i 's@v3.13@v3.14@g' /etc/apk/repositories
```

3\) Update the package list.

```text
sudo apk update
```

4\) Upgrading packages to v3.14

```text
sudo apk add --upgrade apk-tools
```

```text
sudo apk upgrade --available
```

```text
sudo sync
```

```text
sudo reboot now
```

5\) Now you should have AlpineOS upgraded to v3.14 🍷.

```text
cat /etc/alpine-release
```

6\) To troubleshoot the upgrade, refer to the link: [https://wiki.alpinelinux.org/wiki/Upgrading\_Alpine](https://wiki.alpinelinux.org/wiki/Upgrading_Alpine)

## Configuration initiale pour AlpineOS sur Raspberry Pi 4B 8GB :

1\) Download the AlpineOS for RPi 4 aarch64 here: [https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz)

2\) Décompresser le fichier .tar.gz et copier son contenu sur une carte SSD/SD

3\) Branchez un clavier et un moniteur.

4\) Connectez-vous avec le nom d'utilisateur 'root'.

5\) Exécutez la commande `setup-alpine` et suivez les instructions.

{% hint style="info" %}
When you are in `setup-alpine` you will be prompted to choose the system disk. Une fois que vous êtes à ce stade, saisissez, `y`, pour configurer le disque et créer la partition pour `sys`.
{% endhint %}

6\) Redémarrez.

7\) Ajouter un nouvel utilisateur appelé cardano via la commande `adduser cardano` et son mot de passe comme indiqué.

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

12\) By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. To use the ondemand governor which scales CPU frequency according to system load, `cpufreq.start` is included in this repo which should be added to /etc/local.d/. You may run the following commands to do this for you.

```text
    cd ~
```

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```text
    git tag
```

Replace `<tag>` with the latest tag in the next command.

```text
    git checkout tags/<tag>
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```text
    sudo chmod +x /etc/local.d/cpufreq.start
```

```text
    sudo rc-update add local default
```

12\) **\[CPU Governor - Optional\]** By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. To use the ondemand governor which scales CPU frequency according to system load, `cpufreq.start` is included in this repo which should be added to /etc/local.d/. You may run the following commands to do this for you.

```text
    cd ~
```

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```text
    cd alpine-rpi-os
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```text
    sudo chmod +x /etc/local.d/cpufreq.start
```

```text
    sudo rc-update add local default
```

13\) **\[ZRAM - Optional\]** To alleviate RAM limitation on RPi, ZRAM is recommended to enable RAM compression. Use the following steps to install zram-init and install the scripts. The scripts provided will enable a 50% boost in useable RAM capacity. This step assumes you have followed step 12.

```text
    sudo apk add zram-init
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/zram.* /etc/local.d/
```

```text
    sudo chmod +x /etc/local.d/zram.*
```

14\) Reboot the system. For the Raspberry Pi 4B 8GB, you should expect around 3.81GB of swap via ZRAM when checking with `htop` \(`sudo apk add htop` if htop is unavailable\).

## Installer les exécutables statiques 'cardano-node' et 'cardano-cli' \\(AlpineOS utilise presque exclusivement des exécutables statiques, donc vous devriez éviter les compilations non statiques\\)

{% hint style="info" %}
**You can obtain the static binaries for version 1.29.0 via this** [**link**](https://ci.zw3rk.com/build/1758) **courtesy of Moritz Angermann, the SPO of ZW3RK pool 🙏**
{% endhint %}

**Exécutez les commandes suivantes pour installer les exécutables et les placer dans le bon répertoire.**

* Télécharger les exécutables

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.29.0.zip https://ci.zw3rk.com/build/1771/download/1/aarch64-unknown-linux-musl-cardano-node-1.29.0.zip
```

* Décompressez et installez les exécutables via les commandes

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.29.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Installer le service de l'Alliance Armada pour Alpine Linux Cardano Node

{% hint style="success" %}
### Si vous avez décidé d'utiliser AlpineOS pour vos opérations de stake pool Cardano, vous trouverez peut-être cette collection de scripts et de services utiles.
{% endhint %}

{% hint style="info" %}
### Pour installer correctement les scripts et les services, ne sautez pas les étapes 🏴‍☠️😎
{% endhint %}

1\) Clone this repo to obtain the neccessary folder and scripts to quickly start your cardano node. You may skip this step if you have already clonned this repo from step 12 when setting up AlpineOS.

```text
    cd ~
```

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```text
    git tag
```

Replace `<tag>` with the latest tag in the next command.

```text
    git checkout tags/<tag>
```

2\) Exécutez les commandes suivantes pour installer le dossier **cnode** , les scripts et les services dans les bons dossiers. The **cnode** folder contains everything a **Cardano node** needs to start as a functional relay node.

```text
    cd ~
```

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
    wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/ -P ~/cnode
```

4\) Suivez le guide **README.txt** contenu dans le répertoire **$HOME** après avoir installé **cnode**, scripts et services.

```text
    more ~/README.txt
```

## Configuration de Prometheus et de Node-Exporter

1\) Téléchargez Prometheus et node-exporter dans le répertoire personnel

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.29.2/prometheus-2.29.2.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-arm64.tar.gz
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
    mv prometheus-2.29.2.linux-arm64 prometheus
```

```text
    mv node_exporter-1.2.2.linux-arm64 node_exporter
```

4\) Suivez le guide README.txt contenu dans le répertoire $HOME après avoir installé cnode, scripts et services.

```text
    more ~/README.txt
```

## Dépannage général

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

