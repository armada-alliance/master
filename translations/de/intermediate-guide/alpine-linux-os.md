# Alpine Linux OS 🗻

![](../../.gitbook/assets/image (1).png)

## Warum das Alpine Betriebsystem auf dem Raspberry Pi verwenden? Hier einige Gründe:

* **Very low memory consumption (\~50MB utilized during idle vs \~350MB for Ubuntu 20.04).**
* **Lower CPU overhead** **(27 tasks/ 31 threads active for Alpine vs 57 tasks / 111 threads for Ubuntu when cardano-node is running).**
* **Cooler Pi 😎 (Literally, CPU runs cooler because of the lower CPU overhead).**
* **Und zuletzt, wieso nicht? Wenn Sie statische Binärdateien verwenden wollen, können Sie auch die Vorteile von AlpineOS nutzen**

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

## Ersteinrichtung für AlpineOS auf Raspberry Pi 4B 8GB:

1\) Download the AlpineOS for RPi 4 aarch64 here: [https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz)

2\) Dekomprimieren Sie die .tar.gz Datei und kopieren Sie ihren Inhalt auf eine SSD/SD-Karte

3\) Schliessen Sie eine Tastatur und einen Monitor an.

4\) Melden Sie sich mit dem Benutzernamen 'root' an.

5\) Führen Sie den Befehl `setup-alpine` aus folgen Sie den Anweisungen.

{% hint style="info" %}
When you are in `setup-alpine` you will be prompted to choose the system disk. Wenn Sie an diesem Punkt sind, geben Sie `y` ein, um die Festplatte einzurichten und die Partition für `sys` zu erstellen.
{% endhint %}

6\) Führen Sie einen Neustart aus.

7\) Fügen Sie einen neuen Benutzer namens cardano über den Befehl `adduser cardano` hinzu und setzten Sie ein Passwort.

8\) Führen Sie die folgenden Befehle aus, um dem neuen Benutzer Root Rechte zu gewähren

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

9\) Beenden Sie entweder root über den Befehl `exit` oder starten Sie neu und melden Sie sich als cardano an

10\) Installieren Sie Bash, um die Kompatibilität von Bash Skripten zu gewährleisten

```
    sudo apk add bash
```

11\) Installieren Sie auch git und wget, denn wir werden sie später brauchen.

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

* Herunterladen der Binärdateien

```
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.30.1.zip https://ci.zw3rk.com/build/409517/download/1/aarch64-unknown-linux-musl-cardano-node-1.30.1.zip
```

* Entpacken und installieren Sie die Binärdateien mit den Befehlen

```
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.30.1.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Installieren Sie den Armada Alliance Alpine Linux Cardano Node Dienst

{% hint style="success" %}
### Wenn Sie sich entschieden haben, AlpineOS für Ihre Cardano-Stake-Pool-Operationen zu verwenden, könnten Sie diese Sammlung von Skripten und Diensten nützlich finden.
{% endhint %}

{% hint style="info" %}
### Um die Skripte und Dienste korrekt zu installieren, sollten Sie diese Schritte auf keinen Fall überspringen🏴‍☠️😎
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

2\) Führen Sie die folgenden Befehle aus, um anschliessend den Ordner **cnode**, die Skripte und die Dienste in den richtigen Ordnern zu installieren. The **cnode** folder contains everything a **Cardano node** needs to start as a functional relay node.

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

3\) Um die Synchronisierung zu beschleunigen, sollten Sie diesen optionalen Befehl zum Herunterladen des neuesten db Ordners, der von einem unserer Alliance-Mitglieder gehostet wird, in Betracht ziehen.

```
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4\) Folgen Sie der Anleitung in **README.txt** im **$HOME** Verzeichnis nach erfolgreicher Installation von **cnode**, Skripten und Dienste.

```
    more ~/README.txt
```

## Einrichten von Prometheus und Node Exporter

1\) Laden Sie Prometheus und node-exporter in das Home Verzeichnis herunter

```
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.29.2/prometheus-2.29.2.linux-arm64.tar.gz
```

```
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-arm64.tar.gz
```

2\) Extrahieren der tarballs

```
tar -xzvf prometheus.tar.gz
```

```
tar -xzvf node_exporter.tar.gz
```

3\) Benenne die Ordner mit den folgenden Befehlen um

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

## Allgemeine Fehlerbehebung

* Wenn Sie Probleme mit der Port-Weiterleitung über SSH haben, führen Sie folgenden Befehl aus

```
sudo nano /etc/ssh/sshd_config
```

* Bearbeiten Sie die Zeile `AllowTcpForwarding no` to `AllowTcpForwarding yes`

{% hint style="info" %}
Stellen Sie sicher, dass diese Zeile nicht mit einem`#` auskommentiert ist
{% endhint %}

{% hint style="success" %}
We would like to give a special shoutout to our [alliance member](https://armada-alliance.com), [Sayshar](https://armada-alliance.com/identities/sayshar-srn), operator of [\[SRN\] Pool](https://armada-alliance.com/stake-pools/cc1b1c03798884c636703443a23b8d9e827d6c0417921600394198a0), for providing this tutorial 🏴‍☠️ 🙏 😎
{% endhint %}
