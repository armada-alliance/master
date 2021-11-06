---
description: Configure the environment for Cardano Node
---

# Ympäristön Asetukset

## Choose testnet or mainnet.

{% hint style="danger" %}
There is a 500 ₳ Registration deposit and another 5 ₳ in registration costs. First time users are strongly reccomended to use testnet. You can get tada (test ada) from the testnet faucet or ask Alliance members in Telegram. Try not to lose it please.
{% endhint %}

Create an .adaenv file, choose which network you want to be on and source the file. This file will hold the variables for operating a Pi-Node.

```shell
echo -e NODE_CONFIG=testnet >> ${HOME}/.adaenv && source ${HOME}/.adaenv
```

Tee muutamia kansioita.

```bash
mkdir -p ${HOME}/.local/bin
mkdir -p ${HOME}/pi-pool/files
mkdir -p ${HOME}/pi-pool/scripts
mkdir -p ${HOME}/pi-pool/logs
mkdir ${HOME}/git
mkdir ${HOME}/tmp
```

### Create bash variables & add \~/.local/bin to our $PATH 🏃

{% hint style="info" %}
[Ympäristömuuttujat Linux/Unix](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells/247769#247769).
{% endhint %}

{% hint style="Huomaa" %}
You must reload environment files after updating them. Same goes for cardano-node, changes to the topology or config files require a cardano-service restart.
{% endhint %}

```bash
echo PATH="${HOME}/.local/bin:$PATH" >> ${HOME}/.bashrc
echo . ~/.adaenv >> ${HOME}/.bashrc
echo export NODE_HOME=${HOME}/pi-pool >> ${HOME}/.adaenv
echo export NODE_PORT=3003 >> ${HOME}/.adaenv
echo export NODE_FILES=${HOME}/pi-pool/files >> ${HOME}/.adaenv
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> ${HOME}/.adaenv
echo export CARDANO_NODE_SOCKET_PATH="${HOME}/pi-pool/db/socket" >> ${HOME}/.adaenv
source ${HOME}/.bashrc && source .adaenv
```

### Nouda palvelintiedostot

```bash
cd $NODE_FILES
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-alonzo-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
```

Run the following to modify ${NODE_CONFIG}-config.json and update TraceBlockFetchDecisions to "true" & listen on all interfaces with Prometheus Node Exporter.

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g" \
    -e "s/127.0.0.1/0.0.0.0/g"
```

{% hint style="info" %}
**Tip for relay nodes**: It's possible to reduce memory and cpu usage by setting "TraceMemPool" to "false" in **{NODE_CONFIG}-config.json.** This will turn off mempool data in Grafana and gLiveView.sh.
{% endhint %}

### Nouda aarch64-binäärit

{% hint style="info" %}
**Epäviralliset** käyttöön saamamme cardano-node & cardano-cli binäärit on rakentanut IOHK insinööri omalla **vapaa-ajallaan**. Ole hyvä ja tutustu '[Arming Cardano](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w)' Telegram ryhmään saadaksesi lisätietoja.
{% endhint %}

```bash
cd ${HOME}/tmp
wget -O cardano_node_$(date +"%m-%d-%y").zip wget https://ci.zw3rk.com/build/410011/download/1/aarch64-unknown-linux-musl-cardano-node-1.31.0.zip
unzip *.zip
mv cardano-node/* ${HOME}/.local/bin
rm -r cardano*
cd ${HOME}
```

{% hint style="Huomaa" %}
If binaries already exist (if updating) you will have to confirm overwriting the old ones.
{% endhint %}

Confirm binaries are in $USER's $PATH.

```bash
cardano-node version
cardano-cli version
```

### Systemd yksikön tiedostot

Create the systemd unit file and startup script so systemd can manage cardano-node.

```bash
nano ${HOME}/.local/bin/cardano-service
```

Liitä seuraavat, tallenna & sulje nano.

```bash
#!/bin/bash
. /home/ada/.adaenv
#DIRECTORY=/home/${USER}/pi-pool
TOPOLOGY=${NODE_FILES}/${NODE_CONFIG}-topology.json
DB_PATH=${NODE_HOME}/db
CONFIG=${NODE_FILES}/${NODE_CONFIG}-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node run +RTS -N4 -RTS \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${CARDANO_NODE_SOCKET_PATH} \
  --port ${NODE_PORT} \
  --config ${CONFIG}
```

Salli uuden käynnistyskomentosarjan suorittaminen.

```bash
chmod +x ${HOME}/.local/bin/cardano-service
```

Avaa /etc/systemd/system/cardano-node.service.

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

Paste the following, You will need to edit the username here if you chose to not use ada. save & exit.

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
EnvironmentFile=-/home/ada/.adaenv

[Install]
WantedBy= multi-user.target
```

Reload systemd so it picks up our new service file.

```bash
sudo systemctl daemon-reload
```

Let's add a function to the bottom of our .adaenv file to make life a little easier.

```bash
nano ${HOME}/.adaenv
```

```bash
cardano-service() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-node.service
}
```

Tallenna & poistu.

```bash
source ${HOME}/.adaenv
```

Lisäämämme funktio antaa meidän hallita cardano-nodea kirjoittamatta pitkiä komentoja kuten:

> > sudo systemctl enable cardano-node.service sudo systemctl start cardano-node.service sudo systemctl stop cardano-node.service sudo systemctl status cardano-node.service

Nyt meidän täytyy vain:

* cardano-service enable (aktivoi cardano-node.servicen automaattisen käynnistyksen uudelleenkäynnistettäessä)
* cardano-service start (käynnistä cardano-node.service)
* cardano-service stop (pysäytä cardano-node.service)
* cardano-service status (näyttää cardano-node.service tilan)

## ⛓ Ketjun synkronointi ⛓

Olet nyt valmis käynnistämään cardano-noden. Käynnistäminen aloittaa oman nodesi synkronoinnin Cardano lohkoketjun kanssa. This is going to take about 48 hours and the db folder is about 13GB in size right now. Aiemmin ensimmäinen node tuli synkronoida kokonaan, alusta loppuun jonka jälkeen tietokanta voitiin kopioida toiseen nodeen. However...

### Lataa tilannekuva

Olen alkanut ottaa tilannekuvia oman vara noden tietokanta kansiosta ja se on saatavilla web-hakemistosta. With this service it takes around 20 minutes to pull the latest snapshot and maybe another hour to sync up to the tip of the chain. Palvelu tarjotaan sellaisenaan. Valinta on sinun. Jos haluat synkronoida ketjun omin avuin, yksinkertaisesti:

```bash
cardano-service enable
cardano-service start
cardano-service status
```

Otherwise, be sure your node is **not** running & delete the db folder if it exists and download db/.

```bash
cardano-service stop
cd $NODE_HOME
rm -r db/
```

#### Download Database

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://$NODE_CONFIG.adamantium.online/db/
```

Kun wget valmistuu, ota käyttöön cardano-node & käynnistä se.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## gLiveView.sh

Guild operators scripts has a couple useful tools for operating a pool. We do not want the project as a whole, though there are a couple scripts we are going to use.

{% embed url="https://github.com/cardano-community/guild-operators/tree/master/scripts/cnode-helper-scripts" %}

```bash
cd $NODE_HOME/scripts
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
```

Meidän täytyy muokata env tiedostoa, jotta se toimii meidän ympäristössämme. Porttinumero on päivitettävä, jotta se vastaa oman cardano-nodemme porttia. **Pi-nodessamme** se on portti 3003. Rakentaessamme poolia valitsemme edelliset portit. For example Pi-Relay(2) will run on port 3002, Pi-Relay(1) on 3001 and Pi-Core on port 3000.

{% hint style="info" %}
You can change the port cardano-node runs on in the .adaenv file in your home directory.
{% endhint %}

```bash
sed -i env \
    -e "s/\#CNODE_HOME=\"\/opt\/cardano\/cnode\"/NODE_HOME=\"\/home\/${USER}\/pi-pool\"/g" \
    -e "s/"6000"/"3003"/g" \
    -e "s/\#CONFIG=\"\${CNODE_HOME}\/files\/config.json\"/CONFIG=\"\${NODE_FILES}\/${NODE_CONFIG}-config.json\"/g"
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

Download the topologyUpdater script.

```bash
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/topologyUpdater.sh

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
Pi-node-imagessassa tämä cron merkintä on oletuksena pois päältä. You can enable it by removing the #.
{% endhint %}

```bash
33 * * * * . $HOME/.adaenv; /home/$USER/pi-pool/scripts/topologyUpdater.sh
```

After 4 hours of on boarding you will be added to the service and can pull your new list of peers into the {NODE_CONFIG}-topology file.

Luo toinen tiedosto, relay-topology\_pull.sh ja liitä siihen seuraavat rivit.

```bash
nano relay-topology_pull.sh
```
{% hint style="Huomaa" %}
If your running just an active relay you can add a random ip and port. Just remove it from the topology file after you pull and restart node.
{% endhint %}

```bash
#!/bin/bash
NODE_CONFIG=$(grep NODE_CONFIG /home/${USER}/.adaenv | cut -d '=' -f2)
BLOCKPRODUCING_IP=<BLOCK PRODUCERS PRIVATE IP>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o /home/${USER}/pi-pool/files/{${NODE_CONFIG}-topology.json "https://api.clio.one/htopology/v1/fetch/?max=10&customPeers=${BLOCKPRODUCING_IP}:${BLOCKPRODUCING_PORT}:1|relays-new.cardano-${NODE_CONFIG}.iohk.io:3001:2"
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
You can use gLiveView.sh to view ping times in relation to the peers in your {NODE_CONFIG}-topology file. Käytä Ping:ää palvelimien nimien selvittämiseen IP-osoitteissa.
{% endhint %}

Muutokset tässä tiedostossa tulevat käyttöön vasta kun cardano-service käynnistetään uudelleen.

{% hint style="Huomaa" %}
Älä unohda poistaa viimeistä pilkkua topologiatiedostosta!
{% endhint %}

Tilan tulisi näyttää enabled & running.

Once your node syncs past epoch 208(shelley era) you can use gLiveView.sh to monitor.

{% hint style="danger" %}
Se voi kestää jopa tunnin, kun cardano-node synkronoituu takaisin lohkoketjun kärkeen. Käytä ./gliveView.sh, htop ja log tietoja tarkastellaksesi prosessia. Olipa kärsivällinen, kärki saavutetaan kyllä.
{% endhint %}

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

![](../../../../.gitbook/assets/pi-node-glive (7).png)

## Prometheus, Node Exporter & Grafana

Prometheus yhdistää cardano-noden backendiin ja lähettää metriikkaa http:n kautta. Grafana puolestaan voi käyttää näitä tietoja kaavioiden näyttämiseen ja hälytysten luomiseen. Meidän Grafana kojelautamme koostuu Ubuntu järjestelmän & cardano-noden datasta. Grafana can display data from other sources as well, like [adapools.org](https://adapools.org).

{% hint style="info" %}
Voit myös yhdistää Telegram botin Grafanaan, joka varoittaa sinua ongelmista palvelimen kanssa. Tämä on paljon helpompaa kuin yrittää määritellä sähköpostihälytyksiä.
{% endhint %}

{% embed url="https://github.com/prometheus" %}

![](../../../../.gitbook/assets/pi-pool-grafana (2) (2) (2) (2) (1) (7).png)

### Asenna Prometheus & Node Exporter.

{% hint style="info" %}
Prometheus pystyy onkimaan myös muiden node exporteria käyttävien palvelimien http päätetapahtumat. Tämä tarkoittaa, että Grafanaa ja Prometheusta ei tarvitse asentaa ydin tai relay nodeesi. Vain prometheus-node exporter paketti tarvitaan, jos haluat rakentaa Grafanaan keskitetyn kojelaudan poolillesi ja vapauttaa hieman resursseja.
{% endhint %}

```bash
sudo apt install prometheus prometheus-node-exporter -y
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
  - job_name: 'Prometheus' # To scrape data from Prometheus Node Exporter
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

### Asenna Grafana

{% embed url="https://github.com/grafana/grafana" %}

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

Open .adaenv.

```bash
cd ${HOME}
nano .adaenv
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
source .adaenv
```

Täällä yhdistimme kaikki kolme palvelua yhteen tehtävään. Ota Prometheus.service, prometheus-node-exporter.service & grafana-server.service käyttöön käynnistyksen yhteydessä ja käynnistä palvelut.

```bash
cardano-monitor enable
cardano-monitor start
```

{% hint style="Huomaa" %}
Tässä vaiheessa saatat haluta käynnistää cardano-servicen ja synkronoida nodesi lohkoketjun kanssa ennen kuin jatkamme Grafanan konfigurointia. Go to the syncing the chain section. Choose whether you want to wait 30 hours or download the latest chain snapshot. Palaa tähän kun gLiveView.sh näyttää, että olet ketjun kärjessä.
{% endhint %}

## Grafana, Nginx proxy\_pass & snakeoil

Let's put Grafana behind Nginx with self signed(snakeoil) certificate. Sertifikaatti luotiin, kun asensimme ssl-cert paketin.

Voit saada varoituksen selaimestasi. This is because ca-certificates cannot follow a trust chain to a trusted (centralized) source. Yhteys on kuitenkin salattu, ja se suojaa salasanojasi, jotka liitelevät bittiavaruudessa pelkkänä tekstinä.

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

You can now visit your pi-nodes ip address without any port specification, the connection will be upgraded to SSL/TLS and you will get a scary message(not really scary at all). Jatka kohti kojelautaasi.

![](../../../../.gitbook/assets/snakeoil.png)

### Määritä Grafana

On your local machine open your browser and enter your nodes private ip address.

Kirjaudu sisään ja aseta uusi salasana. Oletus käyttäjätunnus ja salasana on **admin:admin**.

#### Määritä tietolähde

In the left hand vertical menu go to **Configure** > **Datasources** and click to **Add data source**. Valitse Prometheus. Syötä [http://localhost:9090](http://localhost:9090) kaikki harmaa voidaan jättää oletusarvoiseksi. Alareunassa save & test. Sinun pitäisi saada vihreä "Data source is working", jos kardano-monitor on päällä. Jos jostain syystä nämä palvelut eivät käynnistyneet, käytä komentoa **cardano-service restart**.

#### Tuo kojelaudat

Tallenna kojelaudan json tiedostot paikalliseen koneeseen.

{% embed url="https://github.com/armada-alliance/dashboards" %}

In the left hand vertical menu go to **Dashboards** > **Manage** and click on **Import**. Valitse tiedosto, jonka juuri latasit tai loit ja tallenna. Head back to **Dashboards** > **Manage** and click on your new dashboard.

![](../../../../.gitbook/assets/pi-pool-grafana (2) (2) (2) (2) (1) (5).png)

### Määritä poolDataLive

Täällä voit käyttää poolData api -sovellusta tuodaksesi poolisi tiedot Grafanaan.

{% embed url="https://api.pooldata.live/dashboard" %}

Noudata ohjeita asentaaksesi Grafana plugin, määritä datasource ja tuo dashboard.

## Useful Commands

Seuraa lokin ulostuloa päiväkirjaan.

```bash
sudo journalctl --unit=cardano-node --follow
```

Seuraa lokin ulostuloa stdoutiin.

```bash
sudo tail -f /var/log/syslog
```

View network connections with netstat.

```bash
sudo netstat -puntw
```

Nyt sinulla on pi-node, jossa on työkaluja, joilla voit rakentaa stake poolin seuraavien sivujen ohjeiden ja tutoriaalien avulla. Best of luck and please join the [armada-alliance](https://armada-alliance.com), together we are stronger! :muscle:&#x20;
