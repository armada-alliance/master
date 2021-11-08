---
description: >-
  Tämän oppaan avulla voit rakentaa Cardano Stake Pool, joka kuluttaa vain 4 wattia per Pi. Pi-Node.img.gz:n referenssiopas (reference guide for the Pi-Node.img.gz).
---

# Pi-Node Image & Ohjeet

[Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz) on ennalta määritetty Cardano Node, jota voidaan käyttää NFT:n luomiseen, lähettämään tapahtumia tai edelleen konfiguroida relay tai ydin nodeksi stake pool käyttöön. Se on konfiguroitu kaikella mitä tarvitset asentaaksesi nopeasti synkronoidun noden sisältäen Nginx proxypass Grafanan TLS salaukselle itse allekirjoitetulla varmenteella, toimintavalmiin topology updaterin ja gLiveView-ohjelman.

![](../../../.gitbook/assets/photo\_2021-03-09-13.40.29.jpeg)

{% hint style="danger" %}
On erittäin suositeltavaa käydä läpi Cardano Foundationin [Stake Pool School](https://cardano-foundation.gitbook.io/stake-pool-course/) -kurssi.
{% endhint %}

{% hint style="Huomaa" %}
Jos haluat luoda .img tiedoston työstäsi, joka voidaan ottaa uudelleenkäyttöön muissa Raspberry Pi:ssäsi sinun kannattaa rakentaa se 8GB sd-kortille. Kuvan tekemiseen kuluu näin vähemmän aikaa. Katso [kuvan luontiosio](https://app.gitbook.com/@ada-pi/s/raspi-spo/intermediate-guide/pi-pool-tutorial/create-.img-file).
{% endhint %}

## Miksi tämä opas?

Yhdistämme ja järjestelemme eri oppaita yhteen asiakirjaan, jota on helppo seurata tai johon voidaan viitata _erityisesti_ stake poolin ylläpitoon kahdella (tai useammalla) Raspberry Pi 4B:llä (8GB versio) ja yhdellä offline Pi:llä, joka tarvitaan kylmä avain operaatioihin.

Toimitetaan dokumentaatio jokaisesta vaiheesta kun samalla rakennetaan Pi-Node imagea pool luomiseen. Viite & opas.

Suosituimmat oppaat on tarkoitettu x86-arkkitehtuuriin ja '_tiedostaa se, mitä heittää pois ja, mitä pitää_' ei ole aina selvää. Aion muuttaa tämä '_with a little help from my friends_'. 🎸

## Laitteisto

{% hint style="Huomaa" %}
Cardano-node ja cardano-cli, joihin tässä oppaassa viitataan, tarvitsevat toimiakseen aarch64 arkkitehtuurin taakseen. Sinun **täytyy** käyttää Pi4B 8GB Core & Relay laitteina, voit käyttää Pi3B+ tai PI4B 4GB tai 8GB versiota mikro sd-kortilla kylmänä offline koneenasi.
{% endhint %}

{% hint style="info" %}
[Tässä pon lista toimivista adaptereista.](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/)
{% endhint %}

### Ostoslista

* 2 [Pi4B 8GB](https://thepihut.com/products/raspberry-pi-4-model-b?variant=31994565689406) versio.
* 2 Asemaa: (NVMe **pieni virrankulutus**, muoto & nopeus).
* M.2 avain USB3.1 adapteriin tai mikä tahansa mikä toimii oman asemasi kanssa.
* Kolmas 64bit kykenevä Pi offline-kone (Cold).
* Luokan 10 micro Sd-kortti 8GB tai suurempi.
* Ylimääräisiä USB flash-asemia avainten ja konfiguraatioiden varmuuskopiointiin.
* Harkitse yhtä 50 watin virtalähdettä
* Harkitse 5 voltin gigabitin kytkintä
* Harkitse koteloa, jossa tuuletin

## Kiitokset ja Yhteisö

* [Alessandro konrad](https://github.com/alessandrokonrad) |[ Berry](https://adapools.org/pool/2a748e3885f6f73320ad16a8331247b81fe01b8d39f57eec9caa5091) (@berry_ales)
* Moritz Angermann | [zw3rk](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) (@zw3rk)
* [CoinCashew: guide-how-to-build-a-haskell-stakepool-node](https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node)
* [Chris-Graffagnino](https://github.com/Chris-Graffagnino)/[Setup Cardano Shelley staking node](https://github.com/Chris-Graffagnino/Jormungandr-for-Newbs/blob/master/docs/jormungandr_node_setup_guide.md)
* [Arming Cardano](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w) Telegram Group
* [Berry Pool](https://t.me/berry_pool) Telegram group
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

## Lataukset

* [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz)
* Viimeisimmät epäviralliset [staattiset arm binäärit](https://ci.zw3rk.com/build/1758)
  * [Moritz Angermann](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w)
* Raspberry Pi Imager ([rpi-imager](https://github.com/raspberrypi/rpi-imager))
  * päivitä eeprom
  * flash .img tiedostot/asenna Ubuntu
* [PiShrink](https://github.com/Drewsif/PiShrink)
* [cardanocli-js](https://docs.pipool.online)
* Viimeisimmän ketjun tilannekuva nopeampaa synkronointia varten
  * wget -r -np -nH -R "index.html\*" -e robots=off [https://mainnet.adamantium.online/db/](https://mainnet.adamantium.online/db/)

## Linkit

* [https://cryptsus.com/blog/how-to-secure-your-ssh-server-with-public-key-elliptic-curve-ed25519-crypto.html](https://cryptsus.com/blog/how-to-secure-your-ssh-server-with-public-key-elliptic-curve-ed25519-crypto.html)
* [https://www.raspberrypi.org/forums/viewtopic.php?t=245931](https://www.raspberrypi.org/forums/viewtopic.php?t=245931)
