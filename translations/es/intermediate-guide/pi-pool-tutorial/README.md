---
description: >-
  A guide to building a 4 watt per Pi, Cardano Stake Pool. A reference guide for the Pi-Node.img.gz.
---

# Imagen del Pi-Node & Guías

The [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz) is a pre configured Cardano node that can be used for minting NFT's, submitting transactions and can be further configured into a relay or core node for stake pool creation. It is configured with everything you need to quickly bootstrap a synced node with Nginx proxypass of Grafana for TLS encryption with a self signed certificate, working topology updater and gLiveView.

![](../../../.gitbook/assets/photo\_2021-03-09-13.40.29.jpeg)

{% hint style="danger" %}
It is strongly recommended to work through the [Stake Pool School](https://cardano-foundation.gitbook.io/stake-pool-course/) course presented by the Cardano Foundation.
{% endhint %}

{% hint style="warning" %}
If you would like to create a .img file of your work that can be flashed for reuse on other Raspberry Pi's you should build on an 8GB sd card. It will take less time to make an image. See [image creation section](https://app.gitbook.com/@ada-pi/s/raspi-spo/intermediate-guide/pi-pool-tutorial/create-.img-file).
{% endhint %}

## Why this guide?

Consolidate and organize the various guides into a single document that can be followed or referenced _specifically_ for running a pool using two (or more) Raspberry Pi 4B (the 8GB version) and one offline Pi for cold key operations.

Provide documentation of every step taken while building the Pi-Node image available for bootstrapping pool creation. A reference & guide.

The most popular guides out there are aimed at x86 architecture and '_knowing what to throw away and knowing what to keep_' is not always clear. I aim to change that '_with a little help from my friends_'. 🎸

## Hardware

{% hint style="warning" %}
The cardano-node & cardano-cli binaries linked to in this guide require aarch64 architecture to run. You **must** use Pi4B 8GB for the Core & Relays, you can use the Pi3B+ or PI4B 4GB or 8GB version with a micro sd card for your cold offline machine.
{% endhint %}

{% hint style="info" %}
[Here is a list of working adapters.](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/)
{% endhint %}

### Shopping list

* 2 [Pi4B 8GB](https://thepihut.com/products/raspberry-pi-4-model-b?variant=31994565689406) version.
* 2 Drives : (NVMe **low power**, form & speed).
* M.2 key to USB3.1 adapter or whatever works with your drive.
* A 3'rd 64bit capable Pi as an offline machine(Cold).
* Class 10 micro sd card 8GB or larger.
* Extra USB flash drives for backing up keys and configurations.
* Consider a single 50+ watt power supply
* Consider a 5 volt gigabit switch
* Consider a case with a fan

## Credit & community

* [Alessandro konrad](https://github.com/alessandrokonrad) |[ Berry](https://adapools.org/pool/2a748e3885f6f73320ad16a8331247b81fe01b8d39f57eec9caa5091) (@berry_ales)
* Moritz Angermann | [zw3rk](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) (@zw3rk)
* [CoinCashew: guide-how-to-build-a-haskell-stakepool-node](https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node)
* [Chris-Graffagnino](https://github.com/Chris-Graffagnino)/[Setup Cardano Shelley staking node](https://github.com/Chris-Graffagnino/Jormungandr-for-Newbs/blob/master/docs/jormungandr_node_setup_guide.md)
* [Arming Cardano](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w) Telegram Group
* [Berry Pool](https://t.me/berry_pool) Telegram group
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

## Downloads

* [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz)
* Latest unofficial [static arm binaries](https://ci.zw3rk.com/build/1758)
  * [Moritz Angermann](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w)
* Raspberry Pi Imager ([rpi-imager](https://github.com/raspberrypi/rpi-imager))
  * update eeprom
  * flash .img files/install Ubuntu
* [PiShrink](https://github.com/Drewsif/PiShrink)
* [cardanocli-js](https://docs.pipool.online)
* Latest chain snapshot for quicker sync
  * wget -r -np -nH -R "index.html\*" -e robots=off [https://mainnet.adamantium.online/db/](https://mainnet.adamantium.online/db/)

## Links

* [https://cryptsus.com/blog/how-to-secure-your-ssh-server-with-public-key-elliptic-curve-ed25519-crypto.html](https://cryptsus.com/blog/how-to-secure-your-ssh-server-with-public-key-elliptic-curve-ed25519-crypto.html)
* [https://www.raspberrypi.org/forums/viewtopic.php?t=245931](https://www.raspberrypi.org/forums/viewtopic.php?t=245931)
