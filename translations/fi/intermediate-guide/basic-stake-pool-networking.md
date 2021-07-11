---
description: >-
  This section will help with understanding the relay and block producer topology.
---

# Basic Stake Pool Networking 💫

## Assumptions

For the sake of this tutorial we are assuming that the Raspberry Pi nodes you are running are in your home and connected to either your ISP's internet router or a separate switch connected to your ISP's internet router. Your nodes should have a firewall configured with as few ports open as possible and with your firewall rules as specific as possible. Furthermore, your ISP's internet router should also have firewall settings configured. If you are not familiar with them, leaving the firewall defaults from your ISP are generally okay.

If you have a block producer running, at a minimum it's firewall rules should have it's node port available only to your configured relay IPs and then the port you use for ssh. If you want to monitor your block producer metrics using Grafana, you'll also need to provide access to the Grafana port. Same thing if you want to monitor your relays.

We are not network experts here. This is only provided as a point of general understanding of how the node topology and network interact. For more advanced network discussions, feel free to use the NASEC discord channel.

## Overview

Your **relay nodes** should be pointed to other remote relay nodes and your block producer. Your **block producer** should only be pointing to your relay nodes.

{% tabs %}
{% tab title="Relay Node" %}
{% code title="mainnet-topology.json" %}
```bash
{
  "Producers": [
    {
      "addr": "block producers private IPv4",
      "port": 6000,
      "valency": 1
    },
    {
      "addr": "138.197.71.216",
      "port": 3001,
      "valency": 1
    },
    {
      "addr": "107.23.17.23",
      "port": 3001,
      "valency": 1
    },
    {
      "addr": "3.140.154.176",
      "port": 6002,
      "valency": 1
    }
  ]
}
```
{% endcode %}
{% endtab %}

{% tab title="Block Producer" %}
{% code title="mainnet-topology.json" %}
```bash
{
  "Producers": [
    {
      "addr": "10.20.30.1",
      "port": 6001,
      "valency": 1
    },
    {
      "addr": "10.20.30.2",
      "port": 6002,
      "valency": 1
    }
  ]
}
```
{% endcode %}

The **addr** and **port** entries above should be the IP addresses of your relay nodes. That's it. Your block producer should have firewall entries restricting access to only these IP addresses on the port you are running your block producer on. Example ufw firewall status below running the block producer on port 6000.

{% code title="> sudo ufw status" %}
```text
To                         Action      From
--                         ------      ----
6000/tcp                   ALLOW       10.20.30.1
6000/tcp                   ALLOW       10.20.30.2
```
{% endcode %}
{% endtab %}
{% endtabs %}

The first example **addr** line above **10.20.30.3** is your block producer's IP address and **port 6000** is the port you are running your block producer on. This object should be the exact same on your other relays.

The other three objects are remote peers. You can set those manually or you can use the **topologyUpdater.sh** script from Guild operators. If you choose to use the topologyUpdater.sh be sure you set the **CUSTOM\_PEERS** line in the script correctly before you run it. This is a pipe-delimited set of addr:port:valency pairs of peers that you want the script to add to your final topology.json file. This line should include your block producer. Default valency is 1 \(one\) if not specified. Example showing the first two objects from the mainnet-topology.json file above:

CUSTOM\_PEERS="10.20.30.3**:**6000**\|**138.197.71.216**:**6000"

{% hint style="info" %}
Set **valency** to 0 \(zero\) to disable a remote peer if you do not wish to delete the peer entirely from the file.
{% endhint %}

## Pool Registration

When you create your stake pool's **pool.json** metadata file you will notice a section called **poolRelays**. This is where you would add **public** relays, visible to others. You can add them as static IPs or as a domain name, such as **north.acme.com**. If you are running more than one relay on your internal network you will need to have them assigned to different ports, such as 6001 and 6002.

{% code title="pool.json" %}
```bash
"poolRelays": [
    {
      "relayType": "dns",
      "relayEntry": "north.acme.com",
      "relayPort": "6001"
    },
    {
      "relayType": "dns",
      "relayEntry": "north.acme.com",
      "relayPort": "6002"
    }
  ],
```
{% endcode %}

A typical home network will only expose a single external IP address to the world, dynamically assigned by your ISP \(Internet Service Provider\). Dynamically assigned external IP leases can be relatively static for a good long period, but this is not guaranteed and you should consider registering a domain name so you can use dns entries in the pool.json instead. Otherwise, each time your external IP address changes you'll have to re-register your pool with a new IP for your relays.

## DNS Client

Unless you have a static IP address assigned by your ISP, at some point you're going to have to consider setting up a dynamic DNS client that runs on your internal network and broadcasts your external IP address assigned by your ISP to your dynamic dns domain provider, such as Google domains. Then whenever your ISP changes your external dynamic IP address, your DNS client will see that, push the new IP address to your domain provider and there should be next to no impact to your domain addresses.

### DNS Client Examples

* [ddclient](https://support.google.com/domains/answer/6147083?hl=en)
* no-ip
* namecheap.com openwrt ddns-scripts

Was this information helpful? Earn rewards with us! [Consider delegating some ADA](../cardano-developer-guides/delegate.md).

