---
description: Pi-Node to Pi-Relay
---

# Pi-Relay

To turn Pi-Node into a active relay we have to.

1. Configure hostname.
2. Configure static IP.
3. Configure port for cardano-service.
4. Configure port forwarding on router.
5. Update port in env file.
6. Enable cron job.
7. Configure both topology scripts.
8. Wait for service on boarding(4 hours).
9. Pull in new list of peers.
10. Prune list of best peers.
11. Update gLiveView's env file.
12. Edit the alias name for Prometheus.
13. Reboot.

## Hostname

To set a fully qualified domain name (FQDN) for our relay edit /etc/hostname & /etc/hosts.

```
sudo nano /etc/hostname
```

Replace ubuntu with your desired FQDN.

```
r1.example.com
```

Save and exit.

```
sudo nano /etc/hosts
```

Edit the file accordingly, take note that you may not be using the 192.168.1.xxx IP range.

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

Save and exit.

## Network

### Static IP

Open **50-cloud-init.yaml** and replace the contents of the file with below.

{% hint style="warning" %}
Be sure to use an address on your LAN subnet. In this example I am using **192.168.1.xxx**. Your network may very well be using a different private range.
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

Create a file named **99-disable-network-config.cfg** to disable cloud-init.

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

Add the following, save and exit.

```bash
network: {config: disabled}
```

Apply your changes.

```
sudo netplan apply
```

## Configure service port

Open the cardano service file and change the port it listens on.

```bash
nano /home/ada/.local/bin/cardano-service
```

Save and exit. **ctrl+x then y**.

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

Enable cardano-service at boot & restart the service to load changes.

```bash
cardano-service enable
cardano-service restart
```

## Forward port on router

{% hint style="danger" %}
Do not forward a port to your Core machine it only connects to your relay(s) on your LAN
{% endhint %}

Log into your router and forward port 3001 to your relay nodes LAN IPv4 address port 3001. Second relay forward port 3002 to LAN IPv4 address for relay 2 to port 3002.

## Topology Updater

```bash
cd $NODE_HOME/scripts
```

Configure the script to match your environment.

{% hint style="warning" %}
If you are using IPv4 leave CNODE_HOSTNAME the way it is. The service will pick up your public IP address on it's own. I repeat only change the port to 3001. For DNS change only the first instance. Do not edit "CHANGE ME" further down in the file.
{% endhint %}

```bash
cd /home/ada/pi-pool/scripts/
```

```bash
nano topologyUpdater.sh
```

Run the updater once to confirm it is working.

```bash
./topologyUpdater.sh
```

Should look similar to this.

> `{ "resultcode": "201", "datetime":"2021-05-20 10:13:40", "clientIp": "1.2.3.4", "iptype": 4, "msg": "nice to meet you" }`

Enable the cron job by removing the # character from crontab.

```bash
crontab -e
```

```bash
33 * * * * /home/ada/pi-pool/scripts/topologyUpdater.sh
```

Save and exit.

### Pull in your list of peers

Wait four hours or so and run the relay-topology_pull.sh to replace your mainnet-topology file with the list created in the log directory.

Open relay-topology_pull.sh and configure it for your environment.

```bash
nano /home/ada/pi-pool/scripts/relay-topology_pull.sh
```

```bash
#!/bin/bash
BLOCKPRODUCING_IP=<core nodes private IPv4 address>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o /home/ada/pi-pool/files/${NODE_CONFIG}-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&customPeers=${BLOCKPRODUCING_IP}:${BLOCKPRODUCING_PORT}:1|relays-new.cardano-mainnet.iohk.io:3001:2"
```

Save and exit.

After four hours of on boarding your relay(s) will start to be available to other peers on the network. **topologyUpdater.sh** will create a list in /home/ada/pi-pool/logs.

relay-topology_pull.sh will replace the contents of your relays mainnet-topology file.

```bash
cd /home/ada/pi-pool/scripts
./relay-topology_pull.sh
```

### Prune the list

Open your topolgy file and use **ctrl+k** to cut the entire line of any peer over 5,000 miles away.

{% hint style="warning" %}
Remember to remove the last entries comma in your list or cardano-node will fail to start.
{% endhint %}

```bash
nano /home/ada/pi-pool/files/mainnet-topology.json
```

### Enable blockfetch tracing

```bash
sed -i ${NODE_FILES}/mainnet-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g"
```

## Update gLiveView port

Open the env file in the scripts directory.

```bash
nano /home/ada/pi-pool/scripts/env
```

Update the port number to match the one set in the cardano-service file. 3001 in this guide.

Reboot your new relay and let it sync back to the tip of the chain.

Use gLiveView.sh to view peer info.

```bash
cd /home/ada/pi-pool/scripts
./gLiveView.sh
```

Many operators block icmp syn packets(ping) because of a security flaw that was patched a decade ago. So expect to see --- for RTT because we are not receiving a response from that server.

More incoming connections is generally a good thing, it increases the odds that you will get network data sooner. Though you may want to put a limit on how many connect. The only way to stop incoming connections would be to block the IPv4 address with ufw.

## Prometheus

Last thing we should do is change the name Prometheus is serving to Grafana.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

{% hint style="warning" %}
You can change.

```bash
alias: 'N1'
```

to

```bash
alias: 'R1'
```

In an upcoming guide I will show how to have Prometheus running on a separate Pi scraping data from the pool instead of having Prometheus using system resources on those machines.
{% endhint %}

Update, save and exit.

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

Reboot the server and give it a while to sync back up. That is just about it. Please feel free to join our Telegram channel for support. [https://t.me/armada_alli](https://t.me/armada_alli)
