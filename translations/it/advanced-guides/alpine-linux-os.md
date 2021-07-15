# Alpine Linux OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

### Why use AlpineOS on the Raspberry Pi? Here are some reasons:

* **Very low memory consumption \(~50MB utilized during idle vs ~350MB for Ubuntu 20.04\).**
* **Lower CPU overhead** **\(27 tasks/ 31 threads active for Alpine vs 57 tasks / 111 threads for Ubuntu when cardano-node is running\).**
* **Cooler Pi 😎 \(Literally, CPU runs cooler because of the lower CPU overhead\).**
* **And finally, why not? If you're gonna use static binaries, might as well take advantage of AlpineOS 😜**

## Initial Setup for AlpineOS on Raspberry Pi 4B 8GB:

1\) Download the AlpineOS for RPi 4 aarch64 here: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)

2\) Decompress the .tar.gz file and copy it's contents into an SSD/SD card

3\) Plug in a keyboard and monitor.

4\) Login with username 'root'.

5\) Run the command `setup-alpine` and follow the instructions.

{% hint style="info" %}
When you are in `setup-alpine`  you will be prompted to choose the system disk. Once you are at this point, enter, `y`, to setup disk and create the partition for `sys`.
{% endhint %}



6\) Reboot.

7\) Add a new user called cardano via the command `adduser cardano` and its password as instructed. \(For username other than **cardano**, refer to **General Troubleshooting**\)

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



