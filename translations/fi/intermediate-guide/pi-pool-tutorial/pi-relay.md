---
description: Pi-Node Pi-Relayksi
---

# Pi-Relay

Jotta Pi-Node muuttuisi aktiiviseksi relayksi meidän on tehtävä seurrava.

1. Määritä hostname.
2. Määritä staattinen IP-osoite.
3. Määritä portti cardano-palveluun.
4. Määritä portin siirto reitittimessä.
5. Päivitä portti env-tiedostossa.
6. Ota cron työ käyttöön.
7. Määritä molemmat topologian komentosarjat.
8. Odota palvelua aktivointia\(4 tuntia).
9. Vedä uusi käyttäjäluettelo.
10. Jalosta luettelo parhaista käyttäjiä.
11. Päivitä gLiveViewin env-tiedosto.
12. Muokkaa Prometheuksen alias nimeä.
13. Käynnistä uudelleen

## Hostname

Määrittääksesi täysin hyväksytyn verkkotunnuksen (FQDN) relaylle muokkaa /etc/hostname & /etc/hosts.

```
sudo nano /etc/hostname
```

Korvaa ubuntu halutulla FQDN:llä.

```
r1.example.com
```

Tallenna ja sulje.

```
sudo nano /etc/hosts
```

Muokkaa tiedostoa vastaavasti, ota huomioon, että et ehkä käytä 192.168.1.xxx IP-aluetta.

```bash
127.0.0.1 localhost
127.0.1.1 r1.example.com r1

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

```

Tallenna ja sulje.

## Verkko

### Staattinen IP

Avaa **50-pilvi-init.yaml** ja korvaa tiedoston sisältö alla olevalla.

{% hint style="Huomaa" %}
Muista käyttää LAN aliverkon osoitetta. Tässä esimerkissä käytän **192.168.1.xxx**. Verkostosi voi hyvinkin käyttää erilaista yksityistä aluetta.
{% endhint %}

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.151/24
      gateway4: 192.168.1.1
      nameservers:
# Home router IP & QUAD9 https://quad9.net/
          addresses: [192.168.1.1, 9.9.9.9, 149.112.112.112]
```

Luo tiedosto nimeltä **99-disable-network-config.cfg** poistaaksesi cloud-init käytöstä.

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

Liitä seuraavat, tallenna & sulje nano.

```bash
network: {config: disabled}
```

Ota muutoksesi käyttöön.

```
sudo netplan apply
```

## Määritä palvelinportti

Avaa cardano-service tiedosto ja muuta portti jota se kuuntelee.

```bash
nano /home/ada/.local/bin/cardano-service
```

Tallenna ja sulje. **ctrl+x ja y**.

```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3001
TOPOLOGY=${FILES}/${NODE_CONFIG}-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/${NODE_CONFIG}-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --port ${PORT} \
  --config ${CONFIG}
```

Ota cardano-service käyttöön käynnistyksen yhteydessä & käynnistä palvelu uudelleen ladataksesi muutokset.

```bash
cardano-service enable
cardano-service restart
```

## Välitä portti reitittimessä

{% hint style="danger" %}
Älä tee portin siirtoa Core koneeseesi, sillä se muodostaa yhteyden vain LAN-verkossasi sijaitsevaan relayhin
{% endhint %}

Kirjaudu reitittimeen ja siltaa portti 3001 relay noden LAN IPv4 osoiteporttiin 3001. Toiselle relaylle sillataan portti 3002 relay LAN IPv4 osoitteesseen porttiin 3002.

## Topology Updater

```bash
cd $NODE_HOME/scripts
```

Määritä skripti vastaamaan ympäristöäsi.

{% hint style="Huomaa" %}
Jos käytät IPv4:ää, jätä CNODE_HOSTNAME niin kuin se on. Palvelu hakee julkisen IP-osoitteen ja käyttää sitä. Toistan, muuta vain portti arvoon 3001 DNS:llä muuta vain ensimmäinen ilmentymä. Älä muokkaa "CHANGE ME" tiedoston alemmilla riveillä.
{% endhint %}

```bash
cd /home/ada/pi-pool/scripts/topologyUpdater.sh
```

```bash
nano topologyUpdater.sh
```

Suorita päivitys kerran vahvistaa, että se toimii.

```bash
./topologyUpdater.sh
```

Pitäisi näyttää samankaltaiselta kuin tämä.

> `{ "resultcode": "201", "datetime":"2021-05-20 10:13:40", "clientIp": "1.2.3.4", "iptype": 4, "msg": "nice to meet you" }`

Ota cron työ käyttöön poistamalla # merkki crontabista.

```bash
crontab -e
```

```bash
33 * * * * /home/ada/pi-pool/scripts/topologyUpdater.sh
```

Tallenna ja sulje.

### Vedä uusi käyttäjäluettelo

Odota neljä tuntia ja suorita relay-topology_pull.sh korvataksesi mainnet-topologian tiedoston lokihakemistoon luodulla listalla.

Avaa relay-topology_pull.sh ja määritä se ympäristöllesi.

```bash
nano /home/ada/pi-pool/scripts/relay-topology_pull.sh
```

```bash
#!/bin/bash
BLOCKPRODUCING_IP=<core nodes private IPv4 address>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o /home/ada/pi-pool/files/${NODE_CONFIG}-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&customPeers=${BLOCKPRODUCING_IP}:${BLOCKPRODUCING_PORT}:1|relays-new.cardano-mainnet.iohk.io:3001:2"
```

Tallenna ja sulje.

Neljän tunnin kuluttua relaysi on saatavilla muiden käyttäjien verkossa. **topologyUpdater.sh** luo listan /home/ada/pi-pool/logs.

relay-topology_pull.sh korvaa relaysi mainnet-topology tiedoston sisällön.

```bash
nano /home/ada/pi-pool/scripts/relay-topology_pull.sh
```

### Jalosta lista

Avaa topolgy tiedosto ja käytä **ctrl+k** leikataksesi koko rivi, jossa käyttäjän etäisyys on yli 5000 mailia.

{% hint style="warning" %}
Muista poistaa listasta viimeisen syötteen pilkku tai cardano-node ei käynnisty.
{% endhint %}

```bash
nano /home/ada/pi-pool/files/mainnet-topology.json
```

### Ota blockfetch seuranta käyttöön

```bash
sed -i ${NODE_FILES}-mainnet-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g"
```

## Päivitä gLiveView- portti

Avaa env tiedosto skripts hakemistossa.

```bash
nano /home/ada/pi-pool/scripts/env
```

Päivitä portin numero, jotta se vastaa kardano-palvelun tiedostossa asetettua numeroa. 3001 tässä oppaassa.

Käynnistä uusi relay uudelleen ja anna sen synkronoida takaisin ketjun kärkeen.

Käytä gLiveView.sh nähdäksesi vertaistiedot.

```bash
cd /home/ada/pi-pool/scripts
./gLiveView.sh
```

Monet operaattorit estävät icmp syn packets(ping) vuosikymmen sitten korjatun turvavirheen vuoksi. Joten odota näkeväsi --- RTT, koska emme saa vastausta kyseiseltä palvelimelta.

Enemmän saapuvia yhteyksiä on yleensä hyvä asia, sillä se lisää mahdollisuutta, että saat verkon dataa nopeammin. Vaikkakin saatat haluta asettaa rajan sille, kuinka monta yhteyttä voidaan muodostaa. Ainoa tapa pysäyttää saapuvat yhteydet olisi estää IPv4-osoite ufw.

## Prometheus

Viimeinen asia, mitä meidän pitäisi tehdä, on muuttaa nimi jonka Prometheus vie Grafanaan.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

{% hint style="warning" %}
Voit muuttaa.

```bash
alias: 'N1'
```

arvoon

```bash
alias: 'R1'
```

Tulevassa oppaassa kerron kuinka Prometheus asennetaan erilliselle Pi:lle ja hakee dataa poolista sen sijaan, että Prometheus käyttäisi node koneiden järjestelmän resursseja.
{% endhint %}

Päivitä, tallenna ja poistu.

```bash
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
          alias: 'R1'
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
          alias: 'R1'
          type:  'node'
```

Käynnistä palvelin uudelleen ja anna sille aikaa synkronoida ketjun kärkeen. Siinäpä se. Ole hyvä ja liity vapaasti Telegram-kanavamme tukeen. [https://t.me/armada_alli](https://t.me/armada_alli)
