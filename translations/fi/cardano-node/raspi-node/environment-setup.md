---
description: Asenna tarvittavat paketit, joita tarvitaan cardano-noden ylläpitämiseen ja määritetään ympäristömme
---

# Ympäristön Asetukset

![](../../.gitbook/assets/download-10-%20%281%29%20%282%29.jpeg)

## Asenna paketit

Asennetaan tarvittavat paketit.

```bash
sudo apt install build-essential libssl-dev tcptraceroute python3-pip \
         jq make automake unzip net-tools nginx ssl-cert pkg-config \
         libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev \
         zlib1g-dev g++ libncursesw5 libtool autoconf -y
```

## Ympäristö

Tee muutamia kansioita.

```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/pi-pool/files
mkdir -p $HOME/pi-pool/scripts
mkdir -p $HOME/pi-pool/logs
mkdir $HOME/git
mkdir $HOME/tmp
```

### Luo bash muuttujat & lisää ~/.local/bin meidän $PATH🏃

{% hint style="info" %}
[Ympäristömuuttujat Linux/Unix](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells/247769#247769).
{% endhint %}

{% hint style="Huomaa" %}
Muutokset tähän tiedostoon vaativat .bashrc:n uudelleenlataamista tai uloskirjautumista ja sitten uuden sisäänkirjautumisen.
{% endhint %}

{% tabs %}
{% tab title="Testnet" %}
```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
echo export NODE_HOME=$HOME/pi-pool >> $HOME/.bashrc
echo export NODE_CONFIG=mainnet >> $HOME/.bashrc
echo export NODE_FILES=$HOME/pi-pool/files >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$HOME/pi-pool/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
echo export NODE_HOME=$HOME/pi-pool >> $HOME/.bashrc
echo export NODE_CONFIG=mainnet >> $HOME/.bashrc
echo export NODE_FILES=$HOME/pi-pool/files >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$HOME/pi-pool/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}
{% endtabs %}

### Nouda palvelintiedostot

```bash
cd $NODE_FILES
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-alonzo-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
```

Suorita seuraavat käskyt muokataksesi testnet-config.json tiedostoa ja päivittääksesi TraceBlockFetchDecisions arvoon "true"

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g"
```

{% hint style="info" %}
**Vinkki relay palvelimille**: On mahdollista vähentää muistin ja cpu:n kuormitusta asettamalla "TraceMemPool" arvoon "false" **mainnet-config.json** tiedostossa. Tämä poistaa mempool datan Grafana ja gLiveView.sh työkalujen käytöstä.
{% endhint %}

### Nouda aarch64-binäärit

{% hint style="info" %}
**Epäviralliset** käyttöön saamamme cardano-node & cardano-cli binäärit on rakentanut IOHK insinööri omalla **vapaa-ajallaan**. Ole hyvä ja tutustu '[Arming Cardano](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w)' Telegram ryhmään saadaksesi lisätietoja.
{% endhint %}

```bash
cd $HOME/tmp
wget -O cardano_node_$(date +"%m-%d-%y").zip https://ci.zw3rk.com/build/1771/download/1/aarch64-unknown-linux-musl-cardano-node-1.29.0.zip
unzip *.zip
mv cardano-node/* $HOME/.local/bin
rm -r cardano*
cd $HOME
```

{% hint style="Huomaa" %}
Jos binäärit ovat jo olemassa, sinun on vahvistettava vanhojen binäärien ylikirjoittaminen.
{% endhint %}

Vahvista binäärit kohteessa ada $PATH.

```bash
cardano-node version
cardano-cli version
```

### Systemd yksikön tiedostot

Luodaan nyt systemd yksikön tiedosto ja käynnistyskomentosarja, jotta systemd järjestelmä voi hallita cardano-nodea.

```bash
nano $HOME/.local/bin/cardano-service
```

Liitä seuraavat, tallenna & sulje nano.

{% tabs %}
{% tab title="Testnet" %}
```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3003
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node run +RTS -N4 -disable-delayed-os-memory-return -qg -qb -c -RTS run \
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
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3003
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node +RTS -N4 -disable-delayed-os-memory-return -qg -qb -c -RTS run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```
{% endtab %}
{% endtabs %}

Salli uuden käynnistyskomentosarjan suorittaminen.

```bash
sudo chmod +x $HOME/.local/bin/cardano-service
```

Avaa /etc/systemd/system/cardano-node.service.

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

Liitä seuraavat, tallenna & sulje nano.

```bash
# The Cardano Node Service (part of systemd)
# file: /etc/systemd/system/cardano-node.service

[Unit]
Description     = Cardano node service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = ada
Type            = simple
WorkingDirectory= /home/ada/pi-pool
ExecStart       = /bin/bash -c "PATH=/home/ada/.local/bin:$PATH exec /home/ada/.local/bin/cardano-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=3
LimitNOFILE=32768
Restart=always
RestartSec=5
EnvironmentFile=-/home/ada/.pienv

[Install]
WantedBy= multi-user.target
```

Määritä käyttöoikeudet ja uudelleenlataa systemd niin se poimii uuden palvelutiedostomme.

```bash
sudo systemctl daemon-reload
```

Lisätään funktio .bashrc tiedostomme loppuun, jotta elämä olisi hieman helpompaa.

```bash
nano $HOME/.bashrc
```

```bash
cardano-service() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-node.service
}
```

Tallenna & poistu.

```bash
source $HOME/.bashrc
```

Lisäämämme funktio antaa meidän hallita cardano-nodea kirjoittamatta pitkiä komentoja kuten:

> > sudo systemctl enable cardano-node.service sudo systemctl start cardano-node.service sudo systemctl stop cardano-node.service sudo systemctl status cardano-node.service

Nyt meidän täytyy vain:

* cardano-service enable  \(aktivoi cardano-node.servicen automaattisen käynnistyksen uudelleenkäynnistettäessä\)
* cardano-service start      \(käynnistä cardano-node.service\)
* cardano-service stop       \(pysäytä cardano-node.service\)
* cardano-service status    \(näyttää cardano-node.service tilan\)

## ⛓ Ketjun synkronointi ⛓

Olet nyt valmis käynnistämään cardano-noden. Käynnistäminen aloittaa oman nodesi synkronoinnin Cardano lohkoketjun kanssa. Tähän menee noin 30 tuntaja tietokanta on kooltaan on noin 8.5GB. Aiemmin ensimmäinen node tuli synkronoida kokonaan, alusta loppuun jonka jälkeen tietokanta voitiin kopioida toiseen nodeen.

### Lataa tilannekuva

{% hint style="danger" %}
Älä yritä tätä 8Gt:n SD-kortilla. Tilaa ei ole tarpeeksi [Luo imagetiedosto](https://app.gitbook.com/@wcatz/s/pi-pool-guide/create-.img-file) ja asenna se ssd-asemaasi.
{% endhint %}

Olen alkanut ottaa tilannekuvia oman vara noden tietokanta kansiosta ja se on saatavilla web-hakemistosta. Tämän palvelun avulla kestää noin 15 minuuttia ladata uusin tilannekuva ja ehkä vielä 30 minuuttia synkronoida tietokanta ketjun kärkeen saakka. Palvelu tarjotaan sellaisenaan. Valinta on sinun. Jos haluat synkronoida ketjun omin avuin, yksinkertaisesti:

```bash
cardano-service enable
cardano-service start
cardano-service status
```

Muussa tapauksessa varmista, että palvelimesi **ei ole** käynnissä, poista db-kansio jos se on olemassa ja lataa db.

```bash
cardano-service stop
cd $NODE_HOME
rm -r db/
```

Lataa tietokannan tilannekuva.

{% tabs %}
{% tab title="Testnet" %}
```bash
wget -r -np -nH -R "index.html*" -e robots=off https://test-db.adamantium.online/db/
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/
```
{% endtab %}
{% endtabs %}

Kun wget valmistuu, ota käyttöön cardano-node & käynnistä se.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## gLiveView.sh

Guild operaattoreiden skripteissä on pari hyödyllistä työkalua stake poolin hallintaan. Emme halua hanketta kokonaisuudessaan, mutta siellä on pari skriptiä, joita aiomme käyttää.

{% embed url="https://github.com/cardano-community/guild-operators/tree/master/scripts/cnode-helper-scripts" caption="" %}

```bash
cd $NODE_HOME/scripts
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
```

Meidän täytyy muokata env tiedostoa, jotta se toimii meidän ympäristössämme. Porttinumero on päivitettävä, jotta se vastaa oman cardano-nodemme porttia. **Pi-nodessamme** se on portti 3003. Rakentaessamme poolia valitsemme edelliset portit. Esimerkiksi Pi-Relay\(2\) ajetaan portilla 3002, Pi-Relay\(1\) 3001 ja Pi-Core portilla 3000.

{% hint style="info" %}
Voit vaihtaa portin, jossa cardano-node toimii muokkaamalla /home/ada/.local/bin/cardano-service.
{% endhint %}

```bash
sed -i env \
    -e "s/\#CNODE_HOME=\"\/opt\/cardano\/cnode\"/CNODE_HOME=\"\home\/ada\/pi-pool\"/g" \
    -e "s/"6000"/"3001"/g" \
    -e "s/\#CONFIG=\"\${CNODE_HOME}\/files\/config.json\"/CONFIG=\"\${NODE_FILES}\/${NODE_CONFIG}-config.json\"/g" \
    -e "s/\#SOCKET=\"\${CNODE_HOME}\/sockets\/node0.socket\"/SOCKET=\"\${NODE_HOME}\/db\/socket\"/g"
```

Salli gLiveView.sh:n suorittaminen.

```bash
chmod +x gLiveView.sh
```

## topologyUpdater.sh

Kunnes vertaisverkko on otettu käyttöön verkko-operaattorit tarvitsevat tavan saada listan releistä/vertaisverkoista, joihin muodostaa yhteyden. Topologian päivityspalvelu toimii taustalla cron kanssa. Joka tunti skripti toimii ja kertoo palvelulle, että olet relay ja haluat olla osa verkkoa. Se lisää relaysi sen hakemistoon neljän tunnin kuluttua ja alkaa luoda listaa relaystä json tiedostoon $NODE\_HOME/logs hakemistoon. Toisella skriptillä, relay-topology\_pull.sh:lla, voidaan sitten manuaalisesti luoda mainnet-topolgy tiedosto, jossa on relayt, jotka ovat tietoisia sinusta ja jotka itse tiedät.

{% hint style="info" %}
Luotu lista näyttää sinulle etäisyyden maileina sekä arvion siitä, missä relay sijaitsee.
{% endhint %}

Avaa tiedosto nimeltä topologyUpdater.sh

```bash
cd $NODE_HOME/scripts
nano topologyUpdater.sh
```

Liitä seuraavat, tallenna & sulje nano.

{% hint style="Huomaa" %}
Porttinumero on päivitettävä, jotta se vastaa oman cardano-nodemme porttia. Jos käytät dns-tietueita, voit lisätä FQDN:n, joka vastaa riviä 6\(vain rivi 6 \). Jätä se niin kuin on, jos et käytä dns:ää. Palvelu hakee julkisen IP-osoitteen ja käyttää sitä.
{% endhint %}

```bash
#!/bin/bash
# shellcheck disable=SC2086,SC2034

USERNAME=ada
CNODE_PORT=3003 # must match your relay node port as set in the startup command
CNODE_HOSTNAME="CHANGE ME"  # optional. must resolve to the IP you are requesting from
CNODE_BIN="/home/ada/.local/bin"
CNODE_HOME="/home/ada/pi-pool"
LOG_DIR="${CNODE_HOME}/logs"
GENESIS_JSON="${CNODE_HOME}/files/mainnet-shelley-genesis.json"
NETWORKID=$(jq -r .networkId $GENESIS_JSON)
CNODE_VALENCY=1   # optional for multi-IP hostnames
NWMAGIC=$(jq -r .networkMagic < $GENESIS_JSON)
[[ "${NETWORKID}" = "Mainnet" ]] && HASH_IDENTIFIER="--mainnet" || HASH_IDENTIFIER="--testnet-magic ${NWMAGIC}"
[[ "${NWMAGIC}" = "1097911063" ]] && NETWORK_IDENTIFIER="--mainnet" || NETWORK_IDENTIFIER="--testnet-magic ${NWMAGIC}"

export PATH="${CNODE_BIN}:${PATH}"
export CARDANO_NODE_SOCKET_PATH="${CNODE_HOME}/db/socket"

blockNo=$(/home/ada/.local/bin/cardano-cli query tip ${NETWORK_IDENTIFIER} | jq -r .block )

# Note:
# if you run your node in IPv4/IPv6 dual stack network configuration and want announced the
# IPv4 address only please add the -4 parameter to the curl command below  (curl -4 -s ...)
if [ "${CNODE_HOSTNAME}" != "CHANGE ME" ]; then
  T_HOSTNAME="&hostname=${CNODE_HOSTNAME}"
else
  T_HOSTNAME=''
fi

if [ ! -d ${LOG_DIR} ]; then
  mkdir -p ${LOG_DIR};
fi

curl -s -f -4 "https://api.clio.one/htopology/v1/?port=${CNODE_PORT}&blockNo=${blockNo}&valency=${CNODE_VALENCY}&magic=${NWMAGIC}${T_HOSTNAME}" | tee -a "${LOG_DIR}"/topologyUpdater_lastresult.json
```

Tallenna, sulje ja tee se suoritettavaksi.

```bash
chmod +x topologyUpdater.sh
```

{% hint style="Huomaa" %}
Et pysty suorittamaan ./topologyUpdater.sh onnistuneesti ennen kuin nodesi on täysin synkronoitu ketjun kärkeen.
{% endhint %}

{% hint style="info" %}
Valitse nano pyydettäessä editoria.
{% endhint %}

Luo cron työ, joka suorittaa skriptin tunnin välein.

```bash
crontab -e
```

Lisää seuraava tiedoston loppuun omalle riville, tallenna & sulje nano.

{% hint style="info" %}
Pi-node-imagessassa tämä cron merkintä on oletuksena pois päältä. Voit ottaa sen käyttöön poistamalla \#.
{% endhint %}

```bash
33 * * * * /home/ada/pi-pool/scripts/topologyUpdater.sh
```

Neljän tunnin skriptin ajon jälkeen, nodesi lisätään palveluun ja voit vetää palvelusta uudet vertaisnodet mainnet-topology tiedostoosi.

Luo toinen tiedosto, relay-topology\_pull.sh ja liitä siihen seuraavat rivit.

```bash
nano relay-topology_pull.sh
```

```bash
#!/bin/bash
BLOCKPRODUCING_IP=<BLOCK PRODUCERS PRIVATE IP>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o /home/ada/pi-pool/files/testnet-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&customPeers=${BLOCKPRODUCING_IP}:${BLOCKPRODUCING_PORT}:1"
```

Tallenna, sulje ja tee se suoritettavaksi.

```bash
chmod +x relay-topology_pull.sh
```

{% hint style="danger" %}
Uuteen listan vetäminen korvaa olemassa olevan topologiatiedoston. Pidä tämä mielessä.
{% endhint %}

Neljän tunnin jälkeen voit vetää uuden listan ja käynnistää cardano-palvelun uudelleen.

```bash
cd $NODE_HOME/scripts
./relay-topology_pull.sh
```

{% hint style="info" %}
relay-topology\_pull.sh lisää 15 vertaista mainnet-topology tiedostoon. Yleensä poistan kauimmat 5 relaytä ja käyttän lähimpiä 10:tä.
{% endhint %}

```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```

{% hint style="info" %}
Voit käyttää gLiveView.sh -sovellusta tarkastellaksesi ping-aikoja suhteessa mainnet-topologiatiedossasi oleviin vertaajiin. Käytä Ping:ää palvelimien nimien selvittämiseen IP-osoitteissa.
{% endhint %}

Muutokset tässä tiedostossa tulevat käyttöön vasta kun cardano-service käynnistetään uudelleen.

{% hint style="Huomaa" %}
Älä unohda poistaa viimeistä pilkkua topologiatiedostosta!
{% endhint %}

Tilan tulisi näyttää enabled & running.

Kun noden synkronointi on ohittanut epochin 208\(shelley era\), voit käyttää gLiveView.sh skriptiä monitorointiin.

{% hint style="danger" %}
Se voi kestää jopa tunnin, kun cardano-node synkronoituu takaisin lohkoketjun kärkeen. Käytä ./gliveView.sh, htop ja log tietoja tarkastellaksesi prosessia. Olipa kärsivällinen, kärki saavutetaan kyllä.
{% endhint %}

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

![](../../.gitbook/assets/pi-node-glive%20%282%29.png)

## Prometheus, Node Exporter & Grafana

Prometheus yhdistää cardano-noden backendiin ja lähettää metriikkaa http:n kautta. Grafana puolestaan voi käyttää näitä tietoja kaavioiden näyttämiseen ja hälytysten luomiseen. Meidän Grafana kojelautamme koostuu Ubuntu järjestelmän & cardano-noden datasta. Grafana voi näyttää tietoja myös muista lähteistä, kuten [adapools.org](https://adapools.org/).

{% hint style="info" %}
Voit myös yhdistää Telegram botin Grafanaan, joka varoittaa sinua ongelmista palvelimen kanssa. Tämä on paljon helpompaa kuin yrittää määritellä sähköpostihälytyksiä.
{% endhint %}

{% embed url="https://github.com/prometheus" caption="" %}

![](../../.gitbook/assets/pi-pool-grafana%20%282%29%20%282%29%20%282%29%20%282%29%20%281%29%20%282%29.png)

### Asenna Prometheus & Node Exporter.

{% hint style="info" %}
Prometheus pystyy onkimaan myös muiden node exporteria käyttävien palvelimien http päätetapahtumat. Tämä tarkoittaa, että Grafanaa ja Prometheusta ei tarvitse asentaa ydin tai relay nodeesi. Vain prometheus-node exporter paketti tarvitaan, jos haluat rakentaa Grafanaan keskitetyn kojelaudan poolillesi ja vapauttaa hieman resursseja.
{% endhint %}

```bash
sudo apt-get install -y prometheus prometheus-node-exporter
```

Poista ne systemd:n käytöstä toistaiseksi.

```bash
sudo systemctl disable prometheus.service
sudo systemctl disable prometheus-node-exporter.service
```

### Määritä Prometheus

Avaa prometheus.yml.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Korvaa tiedoston sisältö alla olevan kanssa.

{% hint style="Huomaa" %}
Sisennyksen on oltava oikea YAML muoto tai Prometheus ei käynnisty.
{% endhint %}

```yaml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label job=<job_name> to any timeseries scraped from this config.
  - job_name: 'Prometheus' # To scrape data from the cardano node
    scrape_interval: 5s
    static_configs:
#      - targets: ['<CORE PRIVATE IP>:12798']
#        labels:
#          alias: 'C1'
#          type:  'cardano-node'
#      - targets: ['<RELAY PRIVATE IP>:12798']
#        labels:
#          alias: 'R1'
#          type:  'cardano-node'
      - targets: ['localhost:12798']
        labels:
          alias: 'N1'
          type:  'cardano-node'

#      - targets: ['<CORE PRIVATE IP>:9100']
#        labels:
#          alias: 'C1'
#          type:  'node'
#      - targets: ['<RELAY PRIVATE IP>:9100']
#        labels:
#          alias: 'R1'
#          type:  'node'
      - targets: ['localhost:9100']
        labels:
          alias: 'N1'
          type:  'node'
```

Tallenna & poistu.

Muokkaa mainnet-config.json tiedostoa niin, että cardano-node lähettää jälkiä kaikilla rajapintoja.

```bash
cd $NODE_FILES
sed -i ${NODE_CONFIG}-config.json -e "s/127.0.0.1/0.0.0.0/g"
```

### Asenna Grafana

{% embed url="https://github.com/grafana/grafana" caption="" %}

Lisää Grafanan gpg avain Ubuntuun.

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Lisää uusin vakaa repo apt lähteisiin.

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Päivitä pakettilistat & asenna Grafana.

```bash
sudo apt update
sudo apt install grafana
```

Muuta portti jota Grafana kuuntelee, jotta se ei ole ristiriidassa cardano-noden kanssa.

```bash
sudo sed -i /etc/grafana/grafana.ini \
-e "s/;http_port/http_port/" \
-e "s/3000/5000/"
```

### cardano-monitor bash-toiminto

Open .bashrc.

```bash
cd $HOME
nano .bashrc
```

Lisää tiedoston loppuun:

```bash
cardano-monitor() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" prometheus.service
    sudo systemctl "$1" prometheus-node-exporter.service
    sudo systemctl "$1" grafana-server.service
}
```

Tallenna, poistu & source.

```bash
source .bashrc
```

Täällä yhdistimme kaikki kolme palvelua yhteen tehtävään. Ota Prometheus.service, prometheus-node-exporter.service & grafana-server.service käyttöön käynnistyksen yhteydessä ja käynnistä palvelut.

```bash
cardano-monitor enable
cardano-monitor start
```

{% hint style="Huomaa" %}
Tässä vaiheessa saatat haluta käynnistää cardano-servicen ja synkronoida nodesi lohkoketjun kanssa ennen kuin jatkamme Grafanan konfigurointia. Hyppää eteenpäin [synkronoidaan ketju jaksoon](https://app.gitbook.com/@wcatz/s/pi-pool-guide/~/drafts/-MYFtFDZp-rTlybgAO71/pi-node/environment-setup/@drafts#syncing-the-chain). Valitse haluatko odottaa 30 tuntia tai ladata viimeisimmän tilannekuvani tietokannasta. Palaa tähän kun gLiveView.sh näyttää, että olet ketjun kärjessä.
{% endhint %}

### Määritä Grafana

Avaa paikallisessa koneessasi selaimesi ja mene osoitteeseen http://&lt;Pi-Node's private ip&gt;:5000

{% hint style="danger" %}
Älä muuta oletussalasanaa vielä, johdolla ei ole salausta. Valitse ohita kun kysytään. Seuraavan kerran kun käymme Grafana se on itse allekirjoitettu TLS sertifikaatti, jonka hoitaa Nginx webservers proxy\_pass ja salasanat ovat turvassa kaikelta sisäinen verkon kuuntelulta.
{% endhint %}

Kirjaudu sisään ja aseta uusi salasana. Oletus käyttäjätunnus ja salasana on **admin:admin**.

#### Määritä tietolähde

Vasemman puolen pystysuorassa valikossa siirry **Configure** &gt; **Datasources** ja napsauta **Add data source**. Valitse Prometheus. Syötä [http://localhost:9090](http://localhost:9090) kaikki harmaa voidaan jättää oletusarvoiseksi. Alareunassa save & test. Sinun pitäisi saada vihreä "Data source is working", jos kardano-monitor on päällä. Jos jostain syystä nämä palvelut eivät käynnistyneet, käytä komentoa **cardano-service restart**.

#### Tuo kojelaudat

Tallenna kojelaudan json tiedostot paikalliseen koneeseen.

{% embed url="https://github.com/armada-alliance/dashboards" caption="" %}

Vasemmalla puolen valikossa mene **Dashboards** &gt; **Manage** ja klikkaa **Import**. Valitse tiedosto, jonka juuri latasit tai loit ja tallenna. Suuntaa takaisin **Dashboards** &gt; **Manage** ja klikkaa uutta kojelautaasi.

![](../../.gitbook/assets/pi-pool-grafana%20%282%29%20%282%29%20%282%29%20%282%29%20%281%29.png)

### Määritä poolDataLive

Täällä voit käyttää poolData api -sovellusta tuodaksesi poolisi tiedot Grafanaan.

{% embed url="https://api.pooldata.live/dashboard" caption="" %}

Noudata ohjeita asentaaksesi Grafana plugin, määritä datasource ja tuo dashboard.

Seuraa lokin ulostuloa päiväkirjaan.

```bash
sudo journalctl --unit=cardano-node --follow
```

Seuraa lokin ulostuloa stdoutiin.

```bash
sudo tail -f /var/log/syslog
```

## Grafana, Nginx proxy\_pass & snakeoil

Asetetaan Grafana Nginxin taakse itse allekirjoitetulla\(snakeoil\) sertifikaatilla. Sertifikaatti luotiin, kun asensimme ssl-cert paketin.

Voit saada varoituksen selaimestasi. Tämä johtuu siitä, että ca-sertifikaatit eivät voi seurata luottamusketjua luotettuun \(keskitetty\) lähteeseen. Yhteys on kuitenkin salattu, ja se suojaa salasanojasi, jotka liitelevät bittiavaruudessa pelkkänä tekstinä.

```bash
sudo nano /etc/nginx/sites-available/default
```

Korvaa tiedoston sisältö alla olevan kanssa.

```bash
# Default server configuration
#
server {
        listen 80 default_server;
        return 301 https://$host$request_uri;
}

server {
        # SSL configuration
        #
        listen 443 ssl default_server;
        #listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # Katso: https://bugs.debian.org/773332
        #
        # Lukea ssl_ciphers varmistaaksesi turvallisen konfiguraation.
        # Katso: https://bugs.debian.org/765782
        #
        # Itse allekirjoitetut sertit luotu ssl-cert paketti
        # Älä käytä niitä tuotantopalvelimella!
        #
        include snippets/snakeoil.conf;

        add_header X-Proxy-Cache $upstream_cache_status;
        location / {
          proxy_pass http://127.0.0.1:5000;
          proxy_redirect      off;
          include proxy_params;
        }
}
```

Tarkista, että Nginx on tyytyväinen muutoksiimme ja käynnistä se uudelleen.

```bash
sudo nginx -t
## if ok do
sudo service nginx restart
```

Voit nyt käydä pi-noden ip osoitteessa ilman portin määrittelyä, yhteys päivitetään SSL / TLS ja saat pelottavan viesti\(ei oikeasti pelottava ollenkaan\). Jatka kohti kojelautaasi.

![](../../.gitbook/assets/snakeoil.png)

Nyt sinulla on pi-node, jossa on työkaluja, joilla voit rakentaa stake poolin seuraavien sivujen ohjeiden ja tutoriaalien avulla. Onnea projektiisi ja liity [armada-allianssiin](https://armada-alliance.com), yhdessä olemme vahvempi!

