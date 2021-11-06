---
description: >-
  Quickly bootstrap a synced configured node in a few hours.
---

# Pi-Node (pikaopas)

{% hint style="info" %}
After booting the image it will take about 30 minutes to download the chain and another couple hours or so to sync to the tip. Et voi tehdä paljoakaan ennen kuin node on synkronoitu lohkoketjun kärkeen asti.

It can take anywhere from 2 to 60 minutes to sync after a reboot depending how the node was shut down or restarted. Tarkista htopilla, onko prosessi käynnissä. Jos se on, käytä gLiveView.sh -skriptiä monitorointiin tai mene kävelylle. Node synkronoituu ja socket luodaan.

On parasta vain jättää se käyntiin. 🏃♀
{% endhint %}


### **1. Download and flash the** [**Pi-Node.img.gz**](https://mainnet.adamantium.online/Pi-Node.img.gz)**.**

### 2. Ota ssh-yhteys palvelimeen.

```bash
ssh ada@<pi-node private IPv4>
```

Default credentials = **ada:lovelace**

{% hint style="Huomaa" %}
Check which version of cardano-node is on the image. Follow the static build upgrade instructions to upgrade. [static-build.md](../updating-a-cardano-node/static-build.md "mention")

```bash
cardano-node version
```
{% endhint %}

## Choose testnet or mainnet. Defaults to testnet.
Switch between testnet & mainnet, for mainnet issue..
```bash
sed -i .adaenv -e "s/NODE_CONFIG=mainnet/NODE_CONFIG=testnet/g"
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

Run the following to modify ${NODE_CONFIG}-config.json and update TraceBlockFetchDecisions to "true" & listen on all interfaces with Prometheus Node Exporter.

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g" \
    -e "s/127.0.0.1/0.0.0.0/g"
```

### 3. Enter the pi-pool folder.

```bash
cd /home/ada/pi-pool
```

### 4. Download database snapshot.

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://$NODE_CONFIG.adamantium.online/db/
```
```bash
touch /home/ada/pi-pool/db/clean
```

### 5. Enable & start the cardano-service.

```bash
cardano-service enable
cardano-service start
```

### 6. Enable & start the cardano-monitor.

```bash
cardano-monitor enable
cardano-monitor start
```

### 7. Confirm they are running.

```bash
cardano-service status
cardano-monitor status
```

Follow journal output or syslog

```
sudo journalctl --unit=cardano-node --follow
sudo tail -f /var/log/syslog
```

### 8. gliveview.sh
allow these files to update if they wish to.

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

### 9. Grafana.

Enter your Node's IPv4 address in your browser.

Default credentials = **admin:admin**

#### Kojelaudat löytyvät täältä.

{% embed url="https://github.com/armada-alliance/dashboards" %}

{% embed url="https://api.pooldata.live/" %}

{% hint style="info" %}
The following guide builds out the image, use it as a reference and please feel free to ask for clarification in our Telegram channel. [https://t.me/armada\_alli](https://t.me/armada\_alli)
{% endhint %}
