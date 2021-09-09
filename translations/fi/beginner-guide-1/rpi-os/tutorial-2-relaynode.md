---
description: >-
  Kun Raspberry Pin asennus on valmis, olemme valmiita lataamaan testiverkkoa varten tarvittavat tiedostot.
---

# Määritä Relay Noden asetukset

{% hint style="Huomaa" %}
**Tämän tutoriaalin tarkoitus on saada yksi node synkronoitua Cardano lohkoketjuun! Olemme ohittaneet tiettyjä vaiheita ja turvallisuuskäytäntöjä, jotta tämä tutoriaali olisi mahdollisimman helppo - ÄLÄ KÄYTÄ tätä tutoriaalia rakentaessasi mainnet stake poolia. Ole hyvä ja käytä**[ **keskitason oppaitamme**](../../intermediate-guide/pi-pool-tutorial/pi-node/) **mainnettia varten.**
{% endhint %}

{% hint style="Huomaa" %}
**Tämä tutoriaali on tarkoitettu vain Raspberry Pi 64bit OS versiolle ja ainoa tarkoitus on saada Cardano node synkronoitumaan lohkoketjun kanssa.**
{% endhint %}

## Yhteenveto

1. Ympäristön asetukset
2. Cardano relay noden rakentamiseen tarvittavien binääritiedostojen lataaminen
3. Konfiguraatiotiedostojen lataaminen IOHK/Cardano-nodelta
4. Config-asetuksien muokkaus
5. Tietokannan tilannekuvan lataaminen synkronointiprosessin nopeuttamiseksi
6. Passiivisen relay noden käynnistäminen ja yhdistäminen testiverkkoon
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

* Voimme nyt käynnistää Pi:n uudelleen ja antaa päivitysten tulla voimaan suorittamalla tämä komento terminaalissa.

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

* [ **Staattinen versio**](https://en.wikipedia.org/wiki/Static_build) on [](https://en.wikipedia.org/wiki/Compiler) ohjelman rakennelma, joka on staattisesti linkattu [kirjastoihin](https://en.wikipedia.org/wiki/Library_%28computing%29).

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

Now we need to move the cardano-node folder into our local binary directory.

```bash
mv cardano-node/* ~/.local/bin
```

Before we proceed let's make sure the cardano-node and cardano-cli is in our $PATH

```bash
cardano-node version
cardano-cli version
```

Now we can move into our files folder, and download the four Cardano node configuration files we need from the official [IOHK website](https://hydra.iohk.io/build/5822084/download/1/index.html) and or [documentation](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html). We will be using the "wget" command to download the files.

```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
```

* **Use the nano bash editor to change a few things in our "testnet-config.json" file**
* [ ] Change the **"TraceBlockFetchDecisions"** line from "**false**" to "**true**"
* [ ] Change the **"hasEKG"** to **12600**
* [ ] Change  the **"hasPrometheus"** address/port to 12700

```text
sudo nano testnet-config.json
```

### Create the systemd files

We will use the Linux systemd service manager to handle the starting, stopping, and restarting of our Cardano node relay.

{% hint style="info" %}
If you'd like to find out more about Linux and systemd go to the [Linux manual page](https://www.man7.org/linux/man-pages/man1/systemd.1.html).
{% endhint %}

```bash
sudo nano $HOME/.local/bin/cardano-service
```

**Now we need to make the cardano-node startup script**

{% hint style="info" %}
How to start the cardano-node can be found here on the [Cardano documentation](https://docs.cardano.org/getting-started/installing-the-cardano-node#gatsby-focus-wrapper).
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

cardano-node +RTS -N4 --disable-delayed-os-memory-return -qg -qb -c -RTS run \
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

cardano-node +RTS -N4 --disable-delayed-os-memory-return -qg -qb -c -RTS run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```
{% endtab %}
{% endtabs %}

**Now we must give access permission to our new systemd service script**

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

We now should reload our systemd service to make sure it picks up our cardano-service

```bash
sudo systemctl daemon-reload
```

**If we don't want to call "sudo systemctl" everytime we want to start, stop, or restart the cardano-node service we can create a "function" that will be added into our .bashrc shell script that will do this for us**

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

## Download a snapshot of the blockchain to speed the sync process

{% hint style="info" %}
Olemme saaneet tilannekuvan testnet tietokannasta Star Forge Pool \[OTG\] ansiosta. Jos et halua ladata tietokantaa, **voit ohittaa tämän vaiheen**. Beware, if you skip downloading our snapshot it may take up to 28 hours to get the node fully synced.
{% endhint %}

{% hint style="danger" %}
**Varmista, ettet ole aloittanut Cardano nodea ennen jatkamista.**🛑
{% endhint %}

Ensinnäkin, varmista, että Cardano-palvelu loimme aiemmin on pysäytetty, niin me ladata tietokannan testiverkon releyn tiedostoja. Voit suorittaa seuraavat komennot aloittaaksesi lataamisen.

```bash
# Make sure you do not have the cardano-node running in the background
cardano-service stop
cd $NODE_HOME
# Remove old db and its contents if present
rm -r db/ 
#Download testnet db snapshot
wget -r -np -nH -R "index.html*" -e robots=off https://test-db.adamantium.online/db/
```

{% hint style="info" %}
Tämä lataus kestää 25 minuutista 2 tuntiin riippuen internetin nopeudesta.
{% endhint %}

* After the database has finished downloading add a clean file to it before we start the relay. Copy/paste the following command into your terminal window.

```bash
touch db/clean
```

## Finish syncing to the blockchain

* Now we can start the "passive" relay node to begin syncing to the blockchain.

```bash
cd $HOME
cardano-service enable
cardano-service start
cardano-service status
```

## Setting up gLiveView to monitor the node during its syncing process

#### Now you can change to the $NODE\_FILES folder and then download the gLiveView monitor service

```bash
sudo apt-get install jq
cd $NODE_FILES
curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
```

* Need to change the "**CNODE\_PORT**" to the port you set on your cardano-node, in our case let's change it to **3001.**

```bash
sudo nano env
```

* Finally, we can exit the nano editor and just run the gLiveView script.

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

## References:

{% tabs %}
{% tab title="📚" %}
{% embed url="https://github.com/wcatz/pi-pool" caption="" %}

{% embed url="https://github.com/alessandrokonrad/Pi-Pool" caption="" %}

{% embed url="https://github.com/angerman" caption="" %}

{% embed url="https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html" caption="" %}

{% embed url="https://cardano-community.github.io/guild-operators/\#/Scripts/gliveview" caption="" %}
{% endtab %}
{% endtabs %}

