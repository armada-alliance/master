# Staattinen Versio

{% hint style="info" %}
Tämä opas noudattaa samoja asetuksia kuin meidän [Pi-Node opas ja image](../pi-pool-tutorial/) joten sinun täytyy ehkä säätää osioita noden ympäristön ja asetusten perusteella.
{% endhint %}

{% hint style="success" %}
### Nykyinen Virallinen Cardano Node Versio: 1.30.1
{% endhint %}

### Yleiskatsaus 🗒

* [ ] Lataa Cardano Noden Dynaaminen versio & konfiguraatiotiedosto
* [ ] Pura tiedoston sisältö
* [ ] Tarkista, jos sinulla on jo Cardano Node -palvelu käynnissä
  * Sammuta turvallisesti Cardano node, jos se on käynnissä
* [ ] Korvaa vanhat binaarit uudella cardano-nodella ja cardano-cli:llä
* [ ] Tarkista, että cardano-node ja -cli versio on päivitetty nykyiseen versioon
* [ ] Korvaa vanhat asetustiedostot uusilla \(jos tarpeen\)
* [ ] Käynnistä Cardano node uudelleen
* [ ] Tarkista, että palvelin on käynnistynyt oikein

## Lataa cardano-node & cli

### Staattiset binäärit ja Cardano node -konfiguraatiotiedostot toimittaa [\[ZW3RK\]](https://armada-alliance.com/identities/zw3rk) pool🙏 ja ne löytyvät [Github repositorystamme](https://github.com/armada-alliance/cardano-node-binaries/tree/main/static-binaries).

```bash
wget https://github.com/armada-alliance/cardano-node-binaries/raw/main/static-binaries/1_30_1.zip
```

Pura zip tiedoston sisältö.

```bash
unzip 1_30_1.zip
```

### Tarkista, onko kardano-solmu jo käynnissä

{% hint style="warning" %}
**Nyt meidän on varmistettava, ettei meidän kardano-node ole jo käynnissä. Jos näin on, meidän on suljettava se ennen jatkamista.**
{% endhint %}

You can check if you have a cardano-node process already running a few ways like using`htop` or by checking your systemd service.

If you have been following our [Pi-Node guide](../pi-pool-tutorial/) you can check your cardano-node status and stop it using the following commands.

```bash
cardano-service status
cardano-service stop
```

{% hint style="info" %}
If you use Linux's `htop` service just check for a processing running starting with `cardano-node run` and use `SIGINT` to terminate the process.
{% endhint %}

## Replace the old binaries and config files with the new ones

If you are using the [Pi-Node guide](../pi-pool-tutorial/) and your cardano-node & cli in `~/.local/bin`

```bash
mv cardano-node/* ~/.local/bin
```

### Check your cardano-node version

```bash
cardano-node --version
```

#### Output:

```bash
cardano-node 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

### Check your cardano-cli version

```bash
cardano-cli --version
```

#### Output:

```bash
cardano-cli 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

### Download & Replace the Cardano node configuration files

{% tabs %}
{% tab title="Mainnet" %}
```bash
cd ~/pi-pool/files
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-config.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-topology.json
```
{% endtab %}

{% tab title="Testnet" %}
```bash
cd ~/pi-pool/files
wget https://hydra.iohk.io/build/7370192/download/1/testnet-config.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-topology.json
```
{% endtab %}

{% tab title="Alonzo-Purple" %}
```bash
cd ~/pi-pool/files
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-config.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-topology.json
```
{% endtab %}
{% endtabs %}

## Restart the Cardano Node

Now we just need to restart our cardano-node service if you are using our [Pi-Node guide](../pi-pool-tutorial/) use this command

```bash
cardano-service start
```

Wait a few seconds or so then check the status

```bash
cardano-service status
```

