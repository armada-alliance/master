# Alpine Linux OS

![](../../.gitbook/assets/image (1).png)

## Miksi käyttää AlpineOS Raspberry Pi:ssä? Tässä muutamia syitä:

* **Erittäin alhainen muistinkulutus (~50MB käytetään idle vs ~350MB Ubuntu 20.04).**
* **Alempi suorittimen kuormitus** **(27 tehtävää / 31 threadiä aktiivisena Alpinessa vs 57 tehtävää / 111 threadiä Ubuntussa, kun cardano-node on käynnissä).**
* **Viileämpi Pi 😎 (kirjaimellisesti, CPU toimii viileämpänä alemman suorittimen kuormituksen ansiosta).**
* **Ja lopuksi, miksi ei? Jos tulet käyttämään staattisia binäärejä, voit yhtä hyvin hyödyntää AlpineOS:ää 😜**

## Jos olet aiemmin käyttänyt tätä opasta ja aiot päivittää komentosarjoja, seuraa näitä ohjeita. Seuraa sitten muita tässä oppaassa hahmoteltuja vaiheita vastaavasti 🙂.

1) Päivitä paikallinen git repo.

```
cd ~/alpine-rpi-os
```

```
git fetch --recurse-submodules --tags --all
```

2) Tunnista viimeisin tagi.

```
git tag
```

3) Korvaa `<tag>` tässä vaiheessa uusimmalla tunnisteella kuten `v1.2.1`.

```
git checkout tags/<tag>
```

## Alpine v3.13:n päivittäminen Alpine v3.14:ään:

1) Päivitä nykyinen AlpineOS-versio.

```
sudo apk update
```

```
sudo apk upgrade
```

2) Muokkaa versiovarastoa vastaamaan Alpine v3.14 -versiota.

```
sudo sed -i 's@v3.13@v3.14@g' /etc/apk/repositories
```

3) Päivitä pakettiluettelo.

```
sudo apk update
```

4) Pakettien päivittäminen versioon 3.14

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

5) Nyt AlpineOS pitäisi olla päivitetty versioon 3.14 🍷.

```
cat /etc/alpine-release
```

6) Ongelmatilanteissa tutustu linkkiin: [https://wiki.alpinelinux.org/wiki/Upgrading_Alpine](https://wiki.alpinelinux.org/wiki/Upgrading_Alpine)

## AlpineOS: ensiasennus Raspberry Pi 4B 8GB koneeseen:

1) Lataa AlpineOS RPi 4 aarch64 täältä: [https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz)

2) Pura .tar.gz tiedosto ja kopioi sen sisältö SSD/SD kortille

3) Kytke näppäimistö ja monitori.

4) Kirjaudu sisään käyttäjätunnuksella 'root'.

5) Suorita komento `setup-alpine` ja noudata ohjeita.

{% hint style="info" %}
Kun olet `setup-alpinessa`, sinua kehotetaan valitsemaan järjestelmälevy. Kun olet tässä vaiheessa, syötä, `y`, määrittääksesi levyn ja luodaksesi osion `sys`:lle.
{% endhint %}

6) Käynnistä kone uudelleen.

7) Lisää uusi käyttäjä nimeltä cardano käyttämällä komentoa `adduser cardano` ja sille salasana ohjeiden mukaisesti.

8) Suorita seuraavat komennot myöntääksesi uudelle käyttäjälle root-oikeudet

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

9) Joko poistu root roolista `exit` komennon avulla tai käynnistä uudelleen ja kirjaudu sisään käyttäjänä cardano

10) Asenna bash varmistaaksesi bash skriptien yhteensopivuus

```
    sudo apk add bash
```

11) Asenna myös git ja wget, tarvitsemme niitä myöhemmin.

```
    sudo apk add git wget
```

12) Oletuksena AlpineOS käyttää virransäästön hallintaa, joka asettaa suorittimen taajuuden alhaisimmille mahdolliselle. Käyttääksesi ondemand säätöä, joka skaalaa suorittimen taajuutta järjestelmän kuormituksen mukaan, `cpufreq.start` sisältyy tähän repositoryyn, joka tulee lisätä kansioon /etc/local.d/. Voit käyttää seuraavia komentoja tehdäksesi tämän.

```
    cd ~
```

```
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```
    git tag
```

Korvaa `<tag>` uusimmalla tunnisteella seuraavassa komennossa.

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

12\) **\[CPU Governor - Optional]** By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. Käyttääksesi ondemand säätöä, joka skaalaa suorittimen taajuutta järjestelmän kuormituksen mukaan, `cpufreq.start` sisältyy tähän repositoryyn, joka tulee lisätä kansioon /etc/local.d/. Voit käyttää seuraavia komentoja tehdäksesi tämän.

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

* Download the binaries

```
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.30.1.zip https://ci.zw3rk.com/build/409517/download/1/aarch64-unknown-linux-musl-cardano-node-1.30.1.zip
```

* Unzip and install the binaries via the commands

```
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.30.1.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Install the Armada Alliance Alpine Linux Cardano node service

{% hint style="success" %}
### If you have decided to use AlpineOS for your Cardano stake pool operations, you may find this collection of scripts and services useful.
{% endhint %}

{% hint style="info" %}
### To install the scripts and services correctly don't skip steps 🏴‍☠️😎
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

Korvaa `<tag>` uusimmalla tunnisteella seuraavassa komennossa.

```
    git checkout tags/<tag>
```

2\) Run the following commands to then install the **cnode** folder, scripts, and services into the correct folders. The **cnode** folder contains everything a **Cardano node** needs to start as a functional relay node.

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

3\) For faster syncing, consider this optional command for downloading the latest db folder hosted by one of our Alliance members.

```
    wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/ -P ~/cnode
```

4\) Follow the guide written in **README.txt** contained in the **$HOME** directory after installing **cnode**, scripts, and services.

```
    more ~/README.txt
```

## Setup prometheus and node exporter

1\) Download Prometheus and node-exporter into the home directory

```
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.29.2/prometheus-2.29.2.linux-arm64.tar.gz
```

```
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-arm64.tar.gz
```

2\) Extract the tarballs

```
tar -xzvf prometheus.tar.gz
```

```
tar -xzvf node_exporter.tar.gz
```

3\) Rename the folders with the following commands

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

## General Troubleshooting

* If you have trouble with port forwarding via SSH, run the following command

```
sudo nano /etc/ssh/sshd_config
```

* Edit the line `AllowTcpForwarding no` to `AllowTcpForwarding yes`

{% hint style="info" %}
Make sure this line is not commented out with a`#`
{% endhint %}

{% hint style="success" %}
We would like to give a special shoutout to our [alliance member](https://armada-alliance.com), [Sayshar](https://armada-alliance.com/identities/sayshar-srn), operator of [\[SRN\] Pool](https://armada-alliance.com/stake-pools/cc1b1c03798884c636703443a23b8d9e827d6c0417921600394198a0), for providing this tutorial 🏴‍☠️ 🙏 😎
{% endhint %}
