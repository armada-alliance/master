# Dynaaminen Versio

_**Me Armada Allianssissa rakennamme aktiivisesti tarvittavia ohjelmistopaketteja, joita tarvitaan Cardano stake poolin ylläpitoon ARM-pohjaisilla tietokoneilla, kuten Raspberry Pi:llä tai Applen MacMini M1:llä.**_

{% hint style="warning" %}
### Käyttääksesi dynaamista arm64 cardano-node versiota sinulla täytyy olla [libsodium](https://github.com/input-output-hk/libsodium) asennettuna.
{% endhint %}

{% hint style="success" %}
### Nykyinen Virallinen Cardano Node Versio: 1.30.1
{% endhint %}

### Yleiskatsaus :notepad_spiral:

* [ ] Tarkista, onko libsodium asennettu paikalliseen koneeseen
  * Rakenna libsodium, jos ei ole jo asennettu
* [ ] Lataa Cardano Noden Dynaaminen versio & konfiguraatiotiedosto
* [ ] Pura tiedoston sisältö
* [ ] Tarkista, jos sinulla on jo Cardano Node -palvelu käynnissä
  * Sammuta Cardano node turvallisesti, jos se on käynnissä
* [ ] Korvaa vanhat binaarit uudella cardano-nodella ja cardano-cli:llä
* [ ] Tarkista, että cardano-node ja -cli versio on päivitetty nykyiseen versioon
* [ ] Korvaa vanhat asetustiedostot uusilla (jos tarpeen)
* [ ] Käynnistä Cardano node uudelleen
* [ ] Tarkista, että palvelin on käynnistynyt oikein

## Libsodiumin Rakentaminen

Tarkista ensin, onko libsodium jo asennettu.

```bash
which libsodium
```

Jos tämä ei palauta mitään tulostetta, sinun täytyy asentaa libsodium.

### Ohjeet libsodiumin asentamiseen

Luo työhakemisto omille rakennuksillesi:

```bash
mkdir -p ~/src
cd ~/src
```

Lataa ja asenna libsodium:

```bash
git clone https://github.com/input-output-hk/libsodium
```

```bash
cd libsodium
git checkout 66f017f1
```

```bash
./autogen.sh
```

```bash
./configure
```

```bash
make
sudo make install
```

Lisää seuraava .bashrc tiedostoon ja aseta lähde:

```bash
echo "export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"" >> ~/.bashrc

echo "export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"" >> ~/.bashrc

source ~/.bashrc
```

Ne, jotka suorittavat cardano-nodea järjestelmäpalveluna, käyttävät seuraavia toimintoja:

```
sudo ldconfig
```

Tämä varmistaa, että järjestelmä on tietoinen libsodiumista (ei vain käyttäjätasolla).

## Lataa cardano-node & -cli

Dynaamiset binaarit ja Cardano node asetustiedostot tarjoaa [SRN pool ](https://armada-alliance.com/stake-pools/cc1b1c03798884c636703443a23b8d9e827d6c0417921600394198a0)🙏 ja ovat saatavissa [Github repositoryssamme](https://github.com/armada-alliance/cardano-node-binaries).

```bash
wget https://github.com/armada-alliance/cardano-node-binaries/blob/main/dynamic-binaries/1.30.1/cardano-1_30_1-aarch64-ubuntu_2004.zip?raw=true
```

Pura zip tiedoston sisältö.

```bash
unzip cardano-1_30_1-aarch64-ubuntu_2004.zip?raw=true
```

### Tarkista, onko cardano-node jo käynnissä

{% hint style="warning" %}
**Nyt meidän on varmistettava, ettei meidän cardano-node ole jo käynnissä. Jos näin on, meidän on suljettava se ennen jatkamista.**
{% endhint %}

Voit tarkistaa, jos sinulla on cardano-node prosessi jo käynnissä muutamalla eri tavalla kuten käyttämällä `htop` -komentoa tai tarkistamalla systemd palvelu.

Jos olet seurannut [Pi-Node -opasta](../pi-pool-tutorial/) voit tarkistaa cardano-noden tilan ja lopettaa sen käyttämällä seuraavia komentoja.

```bash
cardano-service status
cardano-service stop
```

{% hint style="info" %}
Jos käytät Linuxin `htop` -komentoa, tarkista vain prosessi, joka alkaa `cardano-node run` ja käytä `SIGINT` lopettaaksesi prosessin.
{% endhint %}

## Korvaa vanhat binäärit ja asetustiedostot uusilla

Jos käytät [Pi-Node -opasta](../pi-pool-tutorial/) ja cardano-node & -cli ovat kansiossa `~/.local/bin`

```bash
mv cardano-1_30_1-aarch64-ubuntu_2004/cardano-node cardano-1_30_1-aarch64-ubuntu_2004/cardano-cli ~/.local/bin
```

### Tarkista cardano-noden versio

```bash
cardano-node --version
```

#### Tuloste:

```bash
cardano-node 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0fb43f4e3da8b225f4f86557aed90a183981a64f
```

### Tarkista cardano-cli versio

```bash
cardano-cli --version
```

#### Tuloste:

```bash
cardano-cli 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0fb43f4e3da8b225f4f86557aed90a183981a64f
```

### Korvaa cardano noden asetustiedostot

Olemme jo ladanneet kolmeen eri verkkoon (mainnet, testnet ja alonzo-purple testnet) liittyvät konfiguraatiotiedostot.

{% tabs %}
{% tab title="Mainnet Config" %}
```bash
mv cardano-1_30_1-aarch64-ubuntu_2004/files/mainnet/* ~/pi-pool/files
```
{% endtab %}

{% tab title="Testnet Config" %}
```bash
mv cardano-1_30_1-aarch64-ubuntu_2004/files/testnet/* ~/pi-pool/files
```
{% endtab %}

{% tab title="Alonzo Purple Config" %}
```bash
mv cardano-1_30_1-aarch64-ubuntu_2004/files/alonzo-purple/* ~/pi-pool/files
```
{% endtab %}
{% endtabs %}

## Käynnistä Cardano Node uudelleen

Nyt meidän täytyy vain käynnistää uudelleen cardano-node palvelu, jos käytät meidän [Pi-Node opasta](../pi-pool-tutorial/) käytä tätä komentoa

```bash
cardano-service start
```

Odota muutama sekunti tai tarkista sitten prosessin status

```bash
cardano-service status
```

