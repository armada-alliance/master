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

1) Kloonaa tämä repo saadaksesi tarvittavat kansiot ja skriptit cardano noden nopeaan käynnistämiseen. Käytä komentoa:

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

2) Suorita seuraavat komennot ja asenna sitten **cnode** -kansio, skriptit ja palvelut oikeisiin kansioihin. **cnode** kansio sisältää kaiken mitä **Cardano node** tarvitsee käynnistyäkseen toiminnallisena relay nodena:

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

3) Nopeampaa synkronointia varten, harkitse tätä valinnaista komentoa uusimman db-kansion lataamiseen yhden Alliance-jäsenemme ylläpitämältä serveriltä.

```text
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4) Asennettuasi **cnode**, skriptit ja palvelut, noudata **$HOME** hakemiston **README.txt** tiedoston sisältämää opasta.

```text
    more ~/README.txt
```

## Asenna prometheus ja node exporter

1) Lataa Prometheus ja node exporter kotihakemistoon

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
```

2) Pura tarballit

```text
tar -xzvf prometheus.tar.gz
```

```text
tar -xzvf node_exporter.tar.gz
```

3) Nimeä kansiot uudelleen seuraavilla komennoilla

```text
    mv prometheus-2.27.1.linux-arm64 prometheus
```

```text
    mv node_exporter-1.1.2.linux-arm64 node_exporter
```

4) Käynnistääksesi palvelut asianmukaisesti, seuraa $HOME hakemistossa olevan README.txt tiedoston ohjeita asennettuasi cnoden, skriptit ja palvelut.

```text
    more ~/README.txt
```

## Yleinen Vianmääritys

* Jos satut käyttämään muuta käyttäjänimeä kuin cardano, käytä seuraavia komentoja ja vaihda `käyttäjätunnus` valitsemaasi käyttäjätunnukseen.

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

* Jos sinulla on vaikeuksia siirtää porttia SSH:n kautta, suorita seuraava komento

```text
sudo nano /etc/ssh/sshd_config
```

* Muokkaa riviä `AllowTcpForwarding no` vastaamaan `AllowTcpForwarding yes`

{% hint style="info" %}
  Varmista, ettei tätä riviä ole kommentoitu pois `#` -merkillä
{% endhint %}

{% hint style="success" %}
Haluamme antaa erityisen kiitoksen [Alliancen jäsen](https://armada-alliance.com) Saysharille, [\[SRN\] Poolin](https://www.adasrn.com/) operaattorille, tämän oppaan tuottamisesta 🏴‍☠️ 🙏 😎
{% endhint %}



