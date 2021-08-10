---
description: Quickly bootstrap a synced configured node in a hour!
---

# Pi-Node \(pikaopas\)

{% hint style="info" %}
Ketjun lataaminen kestää noin 15 minuuttia ja lisäksi 45 minuuttia kun node synkronoidaan ketjun kärkeen. Et voi tehdä paljoakaan ennen kuin node on synkronoitu lohkoketjun kärkeen asti.

Uudelleenkäynnistyksen jälkeen voi kestää 5-50 minuuttia synkronoida ketju uudlleen riippuen siitä, miten node suljettiin tai käynnistettiin uudelleen. Tarkista htopilla, onko prosessi käynnissä. If it is, use gLiveView.sh or go for walk. It will sync and the socket will be created.

It is best to just leave it running. 🏃♀
{% endhint %}

## Quick Start

### **1. Download and flash the** [**Pi-Node.img.gz**](https://db.adamantium.online/Pi-Node.img.gz)**.**

### 2. ssh into the server.

```bash
ssh ada@<pi-node private IPv4>
```

Default credentials = **ada:lovelace**

### 3. Enter the pi-pool folder.

```bash
cd /home/ada/pi-pool
```

### 4. Download database snapshot.

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/
```

### 5. Enable & start the cardano-service.

{% hint style="Huomaa" %}
Wait for wget to finish downloading the chain before starting the cardano-service. While you are waiting update Ubuntu by entering the server from another terminal.

```bash
sudo apt update
sudo apt upgrade
```
{% endhint %}

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

### 8. gliveview.sh

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

### 9. Grafana.

Enter your Node's IPv4 address in your browser.

Default credentials = **admin:admin**

#### Dashboards can be found here.

{% embed url="https://github.com/armada-alliance/dashboards" caption="" %}

{% embed url="https://api.pooldata.live/" caption="" %}

{% hint style="info" %}
The following guide builds out the image, use it as a reference and please feel free to ask for clarification in our Telegram channel. [https://t.me/armada\_alli](https://t.me/armada_alli)
{% endhint %}

