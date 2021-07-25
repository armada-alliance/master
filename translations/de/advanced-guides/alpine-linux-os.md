# Alpine Linux OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

### Warum das Alpine Betriebsystem auf dem Raspberry Pi verwenden? Hier einige Gründe:

* **Sehr geringer Speicherverbrauch \(~50MB im Leerlauf gegenüber ~350MB bei Ubuntu 20.04\).**
* **Niedrigerer CPU overhead** **\(27 tasks/ 31 threads aktiv bei Alpine gegenüber 57 tasks / 111 threads bei Ubuntu wenn der cardano-node läuft\).**
* **Kühler Pi 😎 \(Kein Schärz, die CPU läuft kühler, weil der CPU Overhead geringer ist\).**
* **Und zuletzt, wieso nicht? Wenn Sie statische Binärdateien verwenden wollen, können Sie auch die Vorteile von AlpineOS nutzen**

## Ersteinrichtung für AlpineOS auf Raspberry Pi 4B 8GB:

1\) Laden Sie das AlpineOS für RPi 4 aarch64 hier herunter: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)

2\) Dekomprimieren Sie die .tar.gz Datei und kopieren Sie ihren Inhalt auf eine SSD/SD-Karte

3\) Schliessen Sie eine Tastatur und einen Monitor an.

4\) Melden Sie sich mit dem Benutzernamen 'root' an.

5\) Führen Sie den Befehl `setup-alpine` aus folgen Sie den Anweisungen.

{% hint style="info" %}
Wenn Sie in `setup-alpine`  sind, werden Sie aufgefordert, das Systemlaufwerk auszuwählen. Wenn Sie an diesem Punkt sind, geben Sie `y` ein, um die Festplatte einzurichten und die Partition für `sys` zu erstellen.
{% endhint %}



6\) Führen Sie einen Neustart aus.

7\) Fügen Sie einen neuen Benutzer namens cardano über den Befehl `adduser cardano` hinzu und setzten Sie ein Passwort. \(Für andere Benutzernamen als **cardano**, siehe **Allgemeine Fehlerbehebung**\)

8\) Run the following commands to grant the new user root privileges

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

9\) Either exit root via the command `exit` or reboot and login to cardano

10\) Install bash to ensure bash script compatibility

```text
    sudo apk add bash
```

11\) Also install git and wget, we will need it later.

```text
    sudo apk add git wget
```

### Installing the 'cardano-node' and 'cardano-cli' static binaries \(AlpineOS uses static binaries almost exclusively so you should avoid non-static builds\)

{% hint style="info" %}
**You can obtain the static binaries for version 1.27.0 via this** [**link**](https://ci.zw3rk.com/build/1758) **courtesy of Moritz Angermann, the SPO of ZW3RK pool 🙏**
{% endhint %}

**Run the following commands to install the binaries and place them into the correct directory.**

* Download the binaries

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
```

* Unzip and install the binaries via the commands

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Install the Armada Alliance Alpine Linux Cardano node service

{% hint style="success" %}
#### If you have decided to use AlpineOS for your Cardano stake pool operations, you may find this collection of scripts and services useful.
{% endhint %}

{% hint style="info" %}
#### To install the scripts and services correctly don't skip steps 🏴‍☠️😎
{% endhint %}

1\) Clone this repo to obtain the necessary folder and scripts to quickly start your Cardano node. Use the command:

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

2\) Run the following commands to then install the **cnode** folder, scripts, and services into the correct folders. The **cnode** folder contains everything a **Cardano node** needs to start as a functional relay node:

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

3\) For faster syncing, consider this optional command for downloading the latest db folder hosted by one of our Alliance members.

```text
    wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/ -P ~/cnode
```

4\) Follow the guide written in **README.txt** contained in the **$HOME** directory after installing **cnode**, scripts, and services.

```text
    more ~/README.txt
```

## Setup prometheus and node exporter

1\) Download Prometheus and node-exporter into the home directory

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
```

2\) Extract the tarballs

```text
tar -xzvf prometheus.tar.gz
```

```text
tar -xzvf node_exporter.tar.gz
```

3\) Rename the folders with the following commands

```text
    mv prometheus-2.27.1.linux-arm64 prometheus
```

```text
    mv node_exporter-1.1.2.linux-arm64 node_exporter
```

4\) Follow the guide written in README.txt contained in the $HOME directory after installing cnode, scripts and services to start the services accordingly.

```text
    more ~/README.txt
```

## General Troubleshooting

* If you happen to use another than username other than cardano, do use the following commands and replace `username` with your chosen username.

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

* If you have trouble with port forwarding via SSH, run the following command

```text
sudo nano /etc/ssh/sshd_config
```

* Edit the line `AllowTcpForwarding no` to `AllowTcpForwarding yes`

{% hint style="info" %}
  Make sure this line is not commented out with a`#`
{% endhint %}

{% hint style="success" %}
We would like to give a special shoutout to our [alliance member](https://armada-alliance.com) Sayshar, operator of [\[SRN\] Pool](https://www.adasrn.com/), for providing this tutorial 🏴‍☠️ 🙏 😎
{% endhint %}



