---
description: >-
  Kun Raspberry Pin asennus on valmis, olemme valmiita lataamaan testiverkkoa varten tarvittavat tiedostot.
---

# Käyttäjän Asetukset

{% hint style="Huomaa" %}
**Tämän tutoriaalin tarkoitus on saada yksi node synkronoitua Cardano lohkoketjuun! Olemme ohittaneet tiettyjä vaiheita ja turvallisuuskäytäntöjä, jotta tämä tutoriaali olisi mahdollisimman helppo - ÄLÄ KÄYTÄ tätä tutoriaalia rakentaessasi mainnet stake poolia. Ole hyvä ja käytä**[ **keskitason oppaitamme**](../../intermediate-guide/pi-pool-tutorial/pi-node/) **mainnettia varten.**
{% endhint %}

{% hint style="Huomaa" %}
**This tutorial is for use with the official Raspberry Pi 64-bit OS and is solely for development educational purposes to get a Raspberry Pi Cardano node syncing to the blockchain.**
{% endhint %}

## Yhteenveto

1. Ympäristön asetukset
2. Cardano relay noden rakentamiseen tarvittavien binääritiedostojen lataaminen
3. Konfiguraatiotiedostojen lataaminen IOHK/Cardano-nodelta
4. Config-asetuksien muokkaus
5. Tietokannan tilannekuvan lataaminen synkronointiprosessin nopeuttamiseksi
6. Perus passiivi relay noden käynnistäminen ja yhdistäminen testiverkkoon
7. Relay noden monitorointi [**Guild Operators gLiveView** ](https://cardano-community.github.io/guild-operators/#/) -ohjelmalla

![](../../.gitbook/assets/download-10-%20%281%29.jpeg)

## Ympäristön luominen

* Meidän täytyy ensin päivittää käyttöjärjestelmämme ja asentaa tarvittavat päivitykset, jos saatavilla.

{% hint style="info" %}
On erittäin suositeltavaa päivittää käyttöjärjestelmä aina kun käynnistät ja kirjaudu sisään **Raspberry Pi:lle** estääksesi tietoturvahaavoittuvuuksia.
{% endhint %}

```text
# Käytämme sudo etuliitettä komentojen suorittamiseen ei-root-userina  

sudo apt update
sudo apt upgrade -y
```

* Voimme nyt käynnistää Pi uudelleen ja antaa päivitysten tulla voimaan suorittamalla tämän komennon terminaalissa.

```text
sudo reboot
```

### Tee hakemistot

{% tabs %}
{% tab title="testnet" %}
```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/testnet-relay/files
```
{% endtab %}

{% tab title="mainnet" %}
```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/mainnet-relay/files
```
{% endtab %}
{% endtabs %}

### Lisää ~/.local/bin meidän $PATH -polkuumme

{% hint style="info" %}
[Kuinka lisätä hakemisto $PATH -polkuun Linuxissa](https://www.howtogeek.com/658904/how-to-add-a-directory-to-your-path-in-linux/)
{% endhint %}

```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
```

### Luo bash muuttujat

{% tabs %}
{% tab title="testnet" %}
```bash
echo export NODE_HOME=$HOME/testnet-relay >> $HOME/.bashrc
echo export NODE_FILES=$HOME/testnet-relay/files >> $HOME/.bashrc
echo export NODE_CONFIG=testnet >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}

{% tab title="mainnet" %}
```bash
echo export NODE_HOME=$HOME/mainnet-relay >> $HOME/.bashrc
echo export NODE_FILES=$HOME/mainnet-relay/files >> $HOME/.bashrc
echo export NODE_CONFIG=mainnet >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}
{% endtabs %}

```bash
sudo reboot
```

### Lataa Cardano-noden staattinen rakennelma

| Toimittanut:                                                                                                                     | Linkki Cardano Static Buildiin                                         |
|:-------------------------------------------------------------------------------------------------------------------------------- |:---------------------------------------------------------------------- |
| [**ZW3RK**](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) **1PCT Haskell CI Support Pool** | [**https://ci.zw3rk.com/build/1758**](https://ci.zw3rk.com/build/1758) |

* A[ **staattinen versio**](https://en.wikipedia.org/wiki/Static_build) on [**kasattu**](https://en.wikipedia.org/wiki/Compiler) versio ohjelmasta, joka on staattisesti yhdistetty kirjastoihin.

Nyt meidän täytyy yksinkertaisesti ladata edellä mainittu zip-tiedosto Pi's kotihakemistoon ja sitten siirtää se oikeaan paikkaan, jotta voimme myöhemmin käyttää sitä ja käynnistää node.

```bash
# Ensin mene kotihakemistoon
cd $HOME

# Nyt voimme ladata cardano-noden 
wget https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
```

* Käytä [**unzip**](https://linux.die.net/man/1/unzip) komentoa ladattuun zip-tiedostoon ja pura sen sisältö.

  ```bash
  unzip aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
  ```

* Seuraavaksi meidän on varmistettava, että äskettäin ladattu "cardano-node" kansio ja sen sisältö ovat läsnä.

{% hint style="info" %}
Jos olet epävarma, onko tiedosto ladattu oikein tai haluat tietää kansion/tiedostojen nimet, voit käyttää Linuxin [**ls**](https://www.man7.org/linux/man-pages/man1/ls.1.html) komentoa.
{% endhint %}

Nyt meidän täytyy siirtää cardano-node kansio paikalliseen binääri hakemistoon.

```bash
mv cardano-node/* ~/.local/bin
```

Ennen kuin jatkamme, varmista, että cardano-node ja cardano-cli ovat meidän $PATH

```bash
cardano-node version
cardano-cli version
```

Nyt voimme siirtyä meidän files kansioon, ja ladata tarvittavat neljä Cardano noden virallista asetustiedostoa [IOHK verkkosivuilla](https://hydra.iohk.io/build/5822084/download/1/index.html) ja tai niihin liittyvän [dokumentaation](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html). Käytämme "wget" -komentoa tiedostojen lataamiseen.

```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
```

* **Käytä nano bash editoria muuttaaksesi muutamia asioita meidän "testnet-config.json" tiedostossa**
* [ ] Muuta **"TraceBlockFetchDecisions"** riviltä arvo "**false**" arvoon "**true**"
* [ ] Muuta **"hasEKG"** portiksi **12600**
* [ ] Muuta **"hasPrometheus"** osoite / portti 12700

{% tabs %}
{% tab title="testnet" %}
```text
sudo nano testnet-config.json
```
{% endtab %}

{% tab title="mainnet" %}
```
sudo nano mainnet-config.json
```
{% endtab %}
{% endtabs %}

### Luo systemd järjestelmätiedostot

Käytämme linuxin systemd järjestelmänvalvojaa Cardano noden käynnistämiseen, pysäyttämiseen ja uudelleenkäynnistämiseen.

{% hint style="info" %}
Jos haluat tietää lisää Linuxista ja systemd järjestelmästä, mene [Linux-käsikirjan sivulle](https://www.man7.org/linux/man-pages/man1/systemd.1.html).
{% endhint %}

```bash
sudo nano $HOME/.local/bin/cardano-service
```

**Nyt meidän täytyy tehdä cardano-noden käynnistys skripti**

{% hint style="info" %}
Cardano noden käynnistämisen ohje löytyy täältä Cardanon dokumentaatiosta.[https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html](https://docs.cardano.org/getting-started/installing-the-cardano-node#gatsby-focus-wrapper).
{% endhint %}

{% tabs %}
{% tab title="Testnet" %}
```bash
#!/bin/bash
DIRECTORY=/home/pi/testnet-relay
FILES=/home/pi/testnet-relay/files
PORT=3001
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/testnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/testnet-config.json

cardano-node run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
#!/bin/bash
DIRECTORY=/home/pi/mainnet-relay
FILES=/home/pi/mainnet-relay/files
PORT=3001
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json

cardano-node run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```
{% endtab %}
{% endtabs %}

**Nyt meidän on annettava pääsyoikeus uuden järjestelmäpalvelun skriptille**

```bash
sudo chmod +x $HOME/.local/bin/cardano-service
```

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

{% tabs %}
{% tab title="Testnet" %}
```bash
# The Cardano Node Service (part of systemd)
# file: /etc/systemd/system/cardano-node.service 

[Unit]
Description     = Cardano node service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = pi
Type            = simple
WorkingDirectory= /home/pi/testnet-relay
ExecStart       = /bin/bash -c "PATH=/home/pi/.local/bin:$PATH exec /home/pi/.local/bin/cardano-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=3
LimitNOFILE=32768
Restart=always
RestartSec=5

[Install]
WantedBy= multi-user.target
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
# The Cardano Node Service (part of systemd)
# file: /etc/systemd/system/cardano-node.service 

[Unit]
Description     = Cardano node service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = pi
Type            = simple
WorkingDirectory= /home/pi/mainnet-relay
ExecStart       = /bin/bash -c "PATH=/home/pi/.local/bin:$PATH exec /home/pi/.local/bin/cardano-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=3
LimitNOFILE=32768
Restart=always
RestartSec=5

[Install]
WantedBy= multi-user.target
```
{% endtab %}
{% endtabs %}

Meidän pitää nyt käynnistää järjestelmäpalvelu uudelleen varmistaaksemme, että se meidän cardano-palvelumme on mukana

```bash
sudo systemctl daemon-reload
```

**Jos emme halua käyttää "sudo systemctl" joka kerta kun haluamme aloittaa, lopettaa, tai käynnistä uudelleen cardano-node palvelun voimme luoda "funktion", joka lisätään meidän .bashrc shell skriptiin, ja tekee tämän puolestamme**

[https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/](https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/)

```bash
nano $HOME/.bashrc
```

```bash
cardano-service() {
    sudo systemctl "$1" cardano-node.service
}
```

```bash
source $HOME/.bashrc
```

## Lataa tilannekuva lohkoketjusta ja nopeuta synkronointiprosessia

{% hint style="info" %}
Olemme saaneet tilannekuvan testnet tietokannasta Star Forge Pool \[OTG\] ansiosta. Jos et halua ladata tietokantaa, **voit ohittaa tämän vaiheen**. Huomaa että, jos jätät tilannekuvan lataamisen väliin, saattaa kestää jopa 28 tuntia, ennen kuin node on täysin synkronoitu.
{% endhint %}

{% hint style="danger" %}
**Varmista, ettet ole aloittanut Cardano nodea ennen jatkamista.**🛑
{% endhint %}

Ensinnäkin, varmista, että Cardano-palvelu loimme aiemmin on pysäytetty, niin me ladata tietokannan testiverkon releyn tiedostoja. Voit suorittaa seuraavat komennot aloittaaksesi lataamisen.

{% tabs %}
{% tab title="testnet" %}
```bash
# Varmista, ettei kardano-node ole käynnissä taustalla
cardano-service stop
cd $NODE_HOME
# Poista vanha db ja sen sisältö, jos läsnä
rm -r db/ 
#Lataa testnet db tilannekuva
wget -r -np -nH -R "index. tml*" -e robots=off https://test-db.adamantium.online/db/
```
{% endtab %}

{% tab title="mainnet" %}
```bash
# Varmista, ettei kardano-node ole käynnissä taustalla
cardano-service stop
cd $NODE_HOME
# Poista vanha db ja sen sisältö, jos läsnä
rm -r db/ 
#Lataa testnet db tilannekuva
wget -r -np -nH -R "index. tml*" -e robots=off https://test-db.adamantium.online/db/Finish
```
{% endtab %}
{% endtabs %}

* Nyt voimme käynnistää "passiivisen" relay noden aloittaaksemme synkronoinnin lohkoketjuun.

```bash
cd $HOME
cardano-service enable
cardano-service start
cardano-service status
```

## gLiveView-ohjelman asennus, noden seuraamiseksi synkronointiprosessin aikana

#### Nyt voit siirtyä $NODE\_FILES kansioon ja sitten ladata gLiveView- monitoripalvelun

```bash
sudo apt-get install jq
cd $NODE_FILES
curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
```

* Päivitä "**CNODE\_PORT**" porttiin jonka asetit cardano nodelle, tässä tapauksessa vaihdetaan siihen **3001.**

```bash
sudo nano env
```

* Lopuksi, voimme poistua nanoeditorista ja vain käynnistää gLiveViewen skripti.

```bash
./gLiveView.sh
```

{% hint style="success" %}
Jos haluat seurata Raspberry Pi suorituskykyä voit käyttää seuraavia komentoja.
{% endhint %}

{% tabs %}
{% tab title="Get Cpu Temp" %}
```bash
vcgencmd measure_temp
```
{% endtab %}

{% tab title="Käytä htop komentoa CPU ja RAM käytön monitorointiin" %}
```bash
htop
```
{% endtab %}
{% endtabs %}

## Viitteet:

{% tabs %}
{% tab title="📚" %}
{% embed url="https://github.com/wcatz/pi-pool" caption="" %}

{% embed url="https://github.com/alessandrokonrad/Pi-Pool" caption="" %}

{% embed url="https://github.com/angerman" caption="" %}

{% embed url="https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html" caption="" %}

{% embed url="https://cardano-community.github.io/guild-operators/\#/Scripts/gliveview" caption="" %}
{% endtab %}
{% endtabs %}

