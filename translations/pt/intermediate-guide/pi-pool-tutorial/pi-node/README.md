---
description: Quickly bootstrap a synced configured node in a hour!
---

# Pi-Node \(quick start\)

{% hint style="info" %}
It will take about 15 minutes to download the chain and another 45 to sync to the tip. You will not be able to do much until your node has synced with the tip of the block chain.

It can take anywhere from 5 to 50 minutes to sync after a reboot depending how the node was shut down or restarted. Check if process is running with htop. If it is, use gLiveView.sh or go for walk. It will sync and the socket will be created.

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

{% hint style="warning" %}
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

