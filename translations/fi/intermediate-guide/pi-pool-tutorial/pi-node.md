---
description: >-
  Asenna synkattu ja konfiguroitu node, jopa tunnissa.
---

# Pi-Node (pikaopas)

{% hint style="info" %}
Kun image on käynnistetty, ketjun lataaminen kestää noin 30 minuuttia ja sen jälkeen menee n. pari tuntia kun database synkronoidaan ketjun kärkeen. Et voi tehdä paljoakaan ennen kuin node on synkronoitu lohkoketjun kärkeen asti.

Uudelleenkäynnistyksen jälkeen voi kestää kahdesta 60:een minuuttia synkronoida ketju uudlleen riippuen siitä, miten node suljettiin tai käynnistettiin uudelleen. Tarkista htopilla, onko prosessi käynnissä. Jos se on, käytä gLiveView.sh -skriptiä monitorointiin tai mene kävelylle. Node synkronoituu ja socket luodaan.

On parasta vain jättää se käyntiin. 🏃♀
{% endhint %}


### **1. Lataa ja asenna** [**Pi-Node.img.gz**](https://mainnet.adamantium.online/Pi-Node.img.gz)**.**

### 2. Ota ssh-yhteys palvelimeen.

```bash
ssh ada@<pi-node private IPv4>
```

Oletustiedot = **ada:lovelace**

{% hint style="Huomaa" %}
Tarkista, mikä cardano-noden versio imagessa on. Noudata staattisen rakennelman päivityksen ohjeita päivittääksesi. [static-build.md](../updating-a-cardano-node/static-build.md "mention")

```bash
cardano-node version
```
{% endhint %}

## Valitse testnet tai mainnet. Oletusarvona on testnet.
Vaihda testnetin & mainnetin välillä, mainnetiä varten anna issue. Config tiedoston polku /home/ada/.adaenv
```bash
sed -i .adaenv -e "s/NODE_CONFIG=testnet/NODE_CONFIG=mainnet/g"
```
```bash
source .adaenv
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

Suorita seuraavat muokataksesi ${NODE_CONFIG}-config.json ja päivittääksesi TraceBlockFetchDecisions arvoon "true" & kuuntele kaikki yhteyksiä Prometheus Node Exporteriin.

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g" \
    -e "s/127.0.0.1/0.0.0.0/g"
```

### 3. Mene pi-poolin kansioon.

```bash
cd /home/ada/pi-pool
```

### 4. Lataa tietokannan tilannekuva.

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://$NODE_CONFIG.adamantium.online/db/
```
```bash
touch /home/ada/pi-pool/db/clean
```

### 5. Ota käyttöön & aloita cardano-palvelu.

```bash
cardano-service enable
cardano-service start
```

### 6. Ota käyttöön & aloita cardano-monitor.

```bash
cardano-monitor enable
cardano-monitor start
```

### 7. Vahvista että palvelut ovat käynnissä.

```bash
cardano-service status
cardano-monitor status
```

Seuraa päiväkirjan tulostetta tai syslogia

```
sudo journalctl --unit=cardano-node --follow
sudo tail -f /var/log/syslog
```

### 8. gliveview.sh
anna näiden tiedostojen päivittää itsensä, jos ne haluavat.

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

### 9. Grafana.

Syötä Node:n IPv4 -osoite selaimesi osoitekenttään.

Oletus käyttäjätunnus ja salasana = **admin:admin**

#### Kojelaudat löytyvät täältä.

{% embed url="https://github.com/armada-alliance/dashboards" %}

{% embed url="https://api.pooldata.live/" %}

{% hint style="info" %}
Seuraava opas rakentaa imagen, käytä sitä viitteenä ja voit vapaasti pyytää selvennystä Telegram kanavassamme. [https://t.me/armada\_alli](https://t.me/armada\_alli)
{% endhint %}
