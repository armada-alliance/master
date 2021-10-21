# Alpine Linux OS

![](../.gitbook/assets/image%20%281%29.png)

### Miksi käyttää AlpineOS Raspberry Pi:ssä? Tässä muutamia syitä:

* **Erittäin alhainen muistinkulutus (~50MB käytetään idle vs ~350MB Ubuntu 20.04\).**
* **Alempi suorittimen kuormitus** **(27 tehtävää / 31 threadiä aktiivisena Alpinessa vs 57 tehtävää / 111 threadiä Ubuntussa, kun cardano-node on käynnissä).**
* **Viileämpi Pi 😎 (kirjaimellisesti, CPU toimii viileämpänä alemman suorittimen kuormituksen ansiosta\).**
* **Ja lopuksi, miksi ei? Jos tulet käyttämään staattisia binäärejä, voit yhtä hyvin hyödyntää AlpineOS:ää 😜**

## AlpineOS: ensiasennus Raspberry Pi 4B 8GB koneeseen:

1) Lataa AlpineOS RPi 4 aarch64 täältä: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)

2) Pura .tar.gz tiedosto ja kopioi sen sisältö SSD/SD kortille

3) Kytke näppäimistö ja monitori.

4) Kirjaudu sisään käyttäjätunnuksella 'root'.

5) Suorita komento `setup-alpine` ja noudata ohjeita.

{% hint style="info" %}
Kun olet `setup-alpinessa`, sinua kehotetaan valitsemaan järjestelmälevy. Kun olet tässä vaiheessa, syötä, `y`, määrittääksesi levyn ja luodaksesi osion `sys`:lle.
{% endhint %}



6) Käynnistä kone uudelleen.

7) Lisää uusi käyttäjä nimeltä cardano käyttämällä komentoa `adduser cardano` ja sille salasana ohjeiden mukaisesti. (Asettaaksesi muun kuin **cardano** käyttäjänimen, katso **Yleinen vianetsintä**\)

8) Suorita seuraavat komennot myöntääksesi uudelle käyttäjälle root-oikeudet

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

9) Joko poistu root roolista `exit` komennon avulla tai käynnistä uudelleen ja kirjaudu sisään käyttäjänä cardano

10) Asenna bash varmistaaksesi bash skriptien yhteensopivuus

```text
    sudo apk add bash
```

11) Asenna myös git ja wget, tarvitsemme niitä myöhemmin.

```text
    sudo apk add git wget
```

### 'cardano-node' ja 'cardano-cli' staattisten binäärien asentaminen (AlpineOS käyttää lähes yksinomaan staattisia binäärejä, joten sinun pitäisi välttää ei-staattisia rakennelmia)

{% hint style="info" %}
**Saat staattiset binäärit versiolle 1.27.1 tästä ** [**linkistä**](https://ci.zw3rk.com/build/1758) **kiitokset Moritz Angermanille, ZW3RK poolin SPO 🙏**
{% endhint %}

**Suorita seuraavat komennot asentaaksesi binäärit ja laita ne oikeaan hakemistoon.**

* Lataa binäärit

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
```

* Pura ja asenna binäärit komentojen kautta

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Asenna Armada Alliancen Alpine Linux Cardano node -palvelu

{% hint style="success" %}
#### Jos olet päättänyt käyttää AlpineOS käyttöjärjestelmää Cardano stake poolissasi, saatat löytää tästä skripti ja palvelu kokoelmasta hyödyllisiä työkaluja.
{% endhint %}

{% hint style="info" %}
#### Asentaaksesi skriptit ja palvelut oikein, älä ohita vaiheita 🏴‍☠️😎
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



