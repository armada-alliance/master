# Linux Alpine OS 🗻

![](../../.gitbook/assets/image (1).png)

## Pourquoi utiliser AlpineOS sur le Raspberry Pi ? Voici quelques raisons :

* **Very low memory consumption (\~50MB utilized during idle vs \~350MB for Ubuntu 20.04).**
* **Lower CPU overhead** **(27 tasks/ 31 threads active for Alpine vs 57 tasks / 111 threads for Ubuntu when cardano-node is running).**
* **Cooler Pi 😎 (Literally, CPU runs cooler because of the lower CPU overhead).**
* **Et finalement, pourquoi pas? Si vous planifiez d'utiliser des exécutables statiques, vous pourriez aussi tirer parti de AlpineOS 😜**

## If you have previously used this guide and intend to update the scripts. Follow these steps. Then follow the rest of the steps outlined in this guide accordingly 🙂.

1\) Update the git local repo.

```
cd ~/alpine-rpi-os
```

```
git fetch --recurse-submodules --tags --all
```

2\) Identify the latest tag.

```
git tag
```

3\) Replace `<tag>` in this step with the latest tag such as `v1.2.1`.

```
git checkout tags/<tag>
```

## Upgrading to Alpine v3.14 from Alpine v3.13:

1\) Update your current version of AlpineOS.

```
sudo apk update
```

```
sudo apk upgrade
```

2\) Edit the repository to reflect Alpine v3.14.

```
sudo sed -i 's@v3.13@v3.14@g' /etc/apk/repositories
```

3\) Update the package list.

```
sudo apk update
```

4\) Upgrading packages to v3.14

```
sudo apk add --upgrade apk-tools
```

```
sudo apk upgrade --available
```

```
sudo sync
```

```
sudo reboot now
```

5\) Now you should have AlpineOS upgraded to v3.14 🍷.

```
cat /etc/alpine-release
```

6\) To troubleshoot the upgrade, refer to the link: [https://wiki.alpinelinux.org/wiki/Upgrading_Alpine](https://wiki.alpinelinux.org/wiki/Upgrading_Alpine)

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

```
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

```
    sudo apk add bash
```

11\) Installez également git et wget, nous en aurons besoin plus tard.

```
    sudo apk add git wget
```

12\) By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. To use the ondemand governor which scales CPU frequency according to system load, `cpufreq.start` is included in this repo which should be added to /etc/local.d/. You may run the following commands to do this for you.

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    git tag
```

Replace `<tag>` with the latest tag in the next command.

```
    git checkout tags/<tag>
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```
    sudo chmod +x /etc/local.d/cpufreq.start
```

```
    sudo rc-update add local default
```

12\) **\[CPU Governor - Optional]** By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. To use the ondemand governor which scales CPU frequency according to system load, `cpufreq.start` is included in this repo which should be added to /etc/local.d/. You may run the following commands to do this for you.

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    cd alpine-rpi-os
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```
    sudo chmod +x /etc/local.d/cpufreq.start
```

```
    sudo rc-update add local default
```

13\) **\[ZRAM - Optional]** To alleviate RAM limitation on RPi, ZRAM is recommended to enable RAM compression. Use the following steps to install zram-init and install the scripts. The scripts provided will enable a 50% boost in useable RAM capacity. This step assumes you have followed step 12.

```
    sudo apk add zram-init
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/zram.* /etc/local.d/
```

```
    sudo chmod +x /etc/local.d/zram.*
```

14\) Reboot the system. For the Raspberry Pi 4B 8GB, you should expect around 3.81GB of swap via ZRAM when checking with `htop` (`sudo apk add htop` if htop is unavailable).

## Installing/Upgrading the 'cardano-node' and 'cardano-cli' static binaries (AlpineOS uses static binaries almost exclusively so avoid non-static builds)

{% hint style="info" %}
**You may obtain the static binaries for version 1.30.1 via this** [**link**](https://ci.zw3rk.com/build/409517) **courtesy of Moritz Angermann, the SPO of ZW3RK pool 🙏**
{% endhint %}

**Run the following commands to download and install the binaries into the correct directory.**

* Télécharger les exécutables

```
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.30.1.zip https://ci.zw3rk.com/build/409517/download/1/aarch64-unknown-linux-musl-cardano-node-1.30.1.zip
```

* Décompressez et installez les exécutables via les commandes

```
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.30.1.zip

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

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    git tag
```

Replace `<tag>` with the latest tag in the next command.

```
    git checkout tags/<tag>
```

2\) Exécutez les commandes suivantes pour installer le dossier **cnode** , les scripts et les services dans les bons dossiers. The **cnode** folder contains everything a **Cardano node** needs to start as a functional relay node.

```
    cd ~
```

```
    cp -r alpine-rpi-os/alpine_cnode_scripts_and_services/home/cardano/* ~/
```

```
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/init.d/* /etc/init.d/
```

```
    chmod +x ~/start_stop_cnode_service.sh ~/cnode/autorestart_cnode.sh
```

```
    sudo chmod +x /etc/init.d/cardano-node /etc/init.d/prometheus /etc/init.d/node-exporter
```

3\) Pour une synchronisation plus rapide, considérez cette commande optionnelle pour télécharger la dernière base de données hébergée par l'un des membres de l'Alliance.

```
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4\) Suivez le guide **README.txt** contenu dans le répertoire **$HOME** après avoir installé **cnode**, scripts et services.

```
    more ~/README.txt
```

## Configuration de Prometheus et de Node-Exporter

1\) Téléchargez Prometheus et node-exporter dans le répertoire personnel

```
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.29.2/prometheus-2.29.2.linux-arm64.tar.gz
```

```
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-arm64.tar.gz
```

2\) Extraire les archives tarballs

```
tar -xzvf prometheus.tar.gz
```

```
tar -xzvf node_exporter.tar.gz
```

3\) Renommer les dossiers avec les commandes suivantes

```
    mv prometheus-2.29.2.linux-arm64 prometheus
```

```
    mv node_exporter-1.2.2.linux-arm64 node_exporter
```

4\) Follow the guide written in **README.txt** contained in the $HOME directory after installing cnode, scripts and services to start the services accordingly.

```
    more ~/README.txt
```

## Dépannage général

* Si vous avez des problèmes avec la redirection de port via SSH, exécutez la commande suivante

```
sudo nano /etc/ssh/sshd_config
```

* Modifier la ligne `AllowTcpForwarding no` pour `AllowTcpForwarding yes`

{% hint style="info" %}
Assurez-vous que cette ligne n'est pas commentée avec un`#`
{% endhint %}

{% hint style="success" %}
We would like to give a special shoutout to our [alliance member](https://armada-alliance.com), [Sayshar](https://armada-alliance.com/identities/sayshar-srn), operator of [\[SRN\] Pool](https://armada-alliance.com/stake-pools/cc1b1c03798884c636703443a23b8d9e827d6c0417921600394198a0), for providing this tutorial 🏴‍☠️ 🙏 😎
{% endhint %}
