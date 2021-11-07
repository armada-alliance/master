---
description: Tehdään uusia Cardano alkuperäisresursseja ❤️✨
---

# Cardano Native Asset (NFT) 💰

## Kenelle tämä opas on tarkoitettu?

* Ihmisille, jotka haluavat tehdä NFT:n tai alkuperäisresursseja (native assets/tokens) Cardano lohkoketjuun
* Ihmisille, jotka tietävät Cardanosta

## NFT:n edut Cardanossa lohkoketjussa

* Alhaiset käsittelymaksut
* Alkuperäinen lohkoketjussa

## Edellytykset

{% hint style="danger" %}
Teimme tämän tutoriaalin käytettäväksi **Raspberry-Pi-ARM** koneiden kanssa, jotka toimivat **Linux käyttöjärjestelmällä** joten muista ladata **oikea** node.js **paikalliseen koneeseen/suorittimeen ja OS**. Tällä hetkellä Cardano-node ja Cardano-cli on tarkoitus rakentaa lähteestä Linux-koneilla. Missä tahansa muussa käyttöjärjestelmässä on omat monimuotoisuutensa, emmekä kata niitä toistaiseksi missään meidän tutoriaaleissamme. [Kuinka rakentaa Cardano node lähteestä](https://docs.cardano.org/projects/cardano-node/en/latest/getting-started/install.html)
{% endhint %}

{% hint style="info" %}
Jos käytät Raspberry Pi konetta h[tässä](https://docs.armada-alliance.com/learn/beginner-guide-1/raspi-node) on helposti seurattava tutoriaali, jonka teimme Cardano Relay Node:n rakentamiseen ja käynnistämiseen.
{% endhint %}

* cardano-node / cardano-cli perustettu paikalliseen koneeseen
* Varmista, että sinulla on Cardano node käynnissä ja täysin synkronoitu tietokantaan
* Varmista, että node.js asennettu

```bash
#Kopioi/Liitä tämä päätelaitteeseesi, jos node.js ei ole vielä asennettu
curl -sL https://deb.nodesource.com/setup_14.x ·sudo -E bash -
sudo apt-get install -y nodejs
```

### Varmista, että kaikki on asennettu oikein koneellemme ⚙️

```bash
#Kopioi/liitä pääteikkunaan
cardano-cli versio; cardano-node versio
```

Tulostesi pitäisi näyttää tältä 👇

```bash
cardano-cli 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
cardano-node 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

#### Varmista että node.js versio on oikein ja on v14.16.1

```bash
#Kopioi/liitä pääteikkunaan
palvelin -v
```

```bash
v14.18.1
```

#### Video Walk-through:

{% embed url="https://youtu.be/oP3jZyPxB-I" %}

## Luo projektihakemisto ja aloitusasetukset

Varmista, että ympäristömuuttujamme `$NODE_HOME` on olemassa

```bash
# check for $NODE_HOME
echo $NODE_HOME
```

Jos yllä oleva komento ei palauta mitään, sinun täytyy asettaa `$NODE_HOME` bash ympäristömuuttuja tai käyttää staattista polkua Cardano noden socket-tiedoston sijaintiin `/db` Cardano node hakemistossa.

```
export NODE_HOME="/home/ada/pi-pool"
# Change this to where cardano-node creates socket
export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket"
```

Tehdään nyt projektihakemistoon ja luodaan sitten <mark style="color:blue;">package.json</mark> tiedosto ja asennetaan <mark style="color:blue;">cardanocli-js</mark> paketti.

```bash
mkdir cardano-minter
cd cardano-minter
npm init -y #creates package.json)
npm install cardanocli-js --save
```

1. **Kopioi Cardano noden genesis uusin versio numero IOHK hydra verkkosivuilla**
   * [https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html](https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html)
2. **Luo bash komentosarja lataamaan tarvittava uusin Genesis config tiedosto**

```bash
nano fetch-config.sh
```

{% tabs %}
{% tab title="TESTNET" %}
```bash
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/testnet-shelley-genesis.json
```
{% endtab %}

{% tab title="MAINNET" %}
```bash
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/mainnet-shelley-genesis.json
```
{% endtab %}
{% endtabs %}

**Nyt meidän täytyy antaa käyttöoikeudet meidän uudelle skriptillemme, sitten ajamme skriptin ja lataamme genesis tiedostot.**

```bash
sudo chmod +x fetch-config.sh
./fetch-config.sh
```

### Seuraavaksi teemme src kansion / hakemiston ja sitten luomme Cardano tilauksen.

```bash
mkdir src
cd src
nano cardano.js
```

{% hint style="info" %}
If you are using testnet make sure you have the correct testnet-magic version number. You can find the current testnet version [here](https://hydra.iohk.io/build/7926804/download/1/testnet-shelley-genesis.json) or simply look in your <mark style="color:blue;">testnet-shelley-genesis.json</mark> file in your cardano node directory.

<mark style="color:blue;"></mark>
{% endhint %}

{% tabs %}
{% tab title="MAINNET" %}
```javascript
const Cardano = require("cardanocli-js");

const cardano = new Cardano({
    network: "mainnet",
    dir: __dirname + "/../",
    shelleyGenesisPath: __dirname + "/../mainnet-shelley-genesis.json",
});

module.exports = cardano;
```
{% endtab %}

{% tab title="TESTNET" %}
```javascript
const Cardano = require("cardanocli-js")

const cardano = new Cardano({
    network: "testnet-magic 1097911063",
    dir: __dirname + "/../",
    shelleyGenesisPath: __dirname + "/../testnet-shelley-genesis.json"
});

module.exports = cardano;
```
{% endtab %}
{% endtabs %}

#### _Video Walk-through_ :

{% tabs %}
{% tab title="Create Project" %}
{% embed url="https://youtu.be/Xkx9vdibbq0" %}


{% endtab %}

{% tab title="Get Cardano genisis files" %}
{% embed url="https://www.youtube.com/watch?v=X5cRGA0qyQE" %}


{% endtab %}

{% tab title="Setup Cardano js client" %}
{% embed url="https://youtu.be/-fnaF3FWL3k" %}


{% endtab %}
{% endtabs %}





## Luo uusi lompakko

```bash
nano create-wallet.js
```

```javascript
const cardano = require('./cardano')

const createWallet = (account) => {
  const payment = cardano.addressKeyGen(account);
  const stake = cardano.stakeAddressKeyGen(account);
  cardano.stakeAddressBuild(account);
  cardano.addressBuild(account, {
    paymentVkey: payment.vkey,
    stakeVkey: stake.vkey,
  });
  return cardano.wallet(account);
};

createWallet("ADAPI")
```

```bash
$ cd ..
node src/create-wallet.js
```

#### Vahvista lompakon saldo, saldo on nolla, sitten rahoitamme lompakon

* **Ensinnäkin meidän on luotava get-balance.js skripti**

```bash
cd src
nano get-balance.js
```

```javascript
// create get-balance.js
const cardano = require('./cardano')

const sender = cardano.wallet("ADAPI");

console.log(
    sender.balance()
)
```

* **Tarkista nyt lompakon saldo.**

```
$ cd ..
node src/get-balance.js
```

* We can go ahead and send some funds (ADA) into our wallet we created, wait a few minutes, and then check the balance again to make sure the transaction was successful.

{% hint style="info" %}
If you are using testnet you must get your tADA from the testnet faucet [here](https://developers.cardano.org/en/testnets/cardano/tools/faucet/).
{% endhint %}

#### _Video Walk-through_ :

{% tabs %}
{% tab title="undefined" %}

{% endtab %}
{% endtabs %}

## Paina (Mint) uudet Native-Assetit/NFT:t Cardano lohkoketjuun

Before we proceed to mint our Native Asset we must have a few things taken care of. We need to first get our "asset" onto our [IPFS](https://ipfs.io/#install) node and generate the IPFS link. If you do not know about IPFS or what it actually does we recommend having a read through the documentation [here](https://docs.ipfs.io) or watching this [video](https://www.youtube.com/watch?v=5Uj6uR3fp-U).

Since we are using an image file to be our asset we should upload a smaller thumbnail-sized version of our image (ideally less than 1MB). This will be used on sites like [pool.pm](https://pool.pm) to display our assets nicely in our wallets. We then upload the full-size image as our source image.

* [ ] Lataa [IPFS](https://ipfs.io/#install)
* [ ] Lataa assetisi tiedostot IPFS:ään
* [ ] Hae meidän kuvakkeen IPFS linkki
* [ ] Hae src IPFS-linkki

#### Viitteeksi:

* **image (thumbnail version) - ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE**
* **src (full-size version) - ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N**

### Luo mint-asset.js skripti

This script has three main components:

1. **Luo policy id**
2. **Määrittele metatiedot**
3. **Määritä painatustapahtuma**

```javascript
nano mint-asset.js
```

```javascript
const cardano = require("./cardano")

// 1. Get the wallet

const wallet = cardano.wallet("ADAPI")

// 2. Define mint script

const mintScript = {
    keyHash: cardano.addressKeyHash(wallet.name),
    type: "sig"
}

// 3. Create POLICY_ID

const POLICY_ID = cardano.transactionPolicyid(mintScript)

// 4. Define ASSET_NAME

const ASSET_NAME = "BerrySpaceGreen"

// 5. Create ASSET_ID

const ASSET_ID = POLICY_ID + "." + ASSET_NAME

// 6. Define metadata

const metadata = {
    721: {
        [POLICY_ID]: {
            [ASSET_NAME]: {
                name: ASSET_NAME,
                image: "ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE",
                description: "Super Fancy Berry Space Green NFT",
                type: "image/png",
                src: "ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N",
                // other properties of your choice
                authors: ["PIADA", "SBLYR"]
            }
        }
    }
}

// 7. Define transaction

const tx = {
    txIn: wallet.balance().utxo,
    txOut: [
        {
            address: wallet.paymentAddr,
            value: { ...wallet.balance().value, [ASSET_ID]: 1 }
        }
    ],
    mint: {
        actions: [{ type: "mint", quantity: 1, asset: ASSET_ID }],
        script: [mintScript]
    },
    metadata,
    witnessCount: 2
}

// 8. Build transaction

const buildTransaction = (tx) => {

    const raw = cardano.transactionBuildRaw(tx)
    const fee = cardano.transactionCalculateMinFee({
        ...tx,
        txBody: raw
    })

    tx.txOut[0].value.lovelace -= fee

    return cardano.transactionBuildRaw({ ...tx, fee })
}

const raw = buildTransaction(tx)

// 9. Sign transaction

const signTransaction = (wallet, tx) => {

    return cardano.transactionSign({
        signingKeys: [wallet.payment.skey, wallet.payment.skey],
        txBody: tx
    })
}

const signed = signTransaction(wallet, raw)

// 10. Submit transaction

const txHash = cardano.transactionSubmit(signed)

console.log(txHash)
```

* **Suorita minting script, odota hetki ja tarkista lompakkomme saldo**

```
$ cd ..
node src/mint-asset.js
```

_**Video Walk-through:**_

{% tabs %}
{% tab title="" %}
{% embed url="https://youtu.be/qTzLgMyJC7s" %}


{% endtab %}
{% endtabs %}

## NFT:n lähettäminen takaisin Daedalus tai Yoroi lompakkoon

Now we must create a new script to send our newly minted NFT to a wallet.

```javascript
cd cardaon-minter/src
nano send-back-asset-to-wallet.js
```

There are few main parts we have to this script in order to send the asset:

1. Hae lompakko
2. Määrittele tapahtuma
3. Rakenna tapahtuma
4. Laske käsittelymaksu
5. Maksa käsittelymaksu vähentämällä se lähettäjän utxosta
6. Rakenna lopullinen tapahtuma
7. Allekirjoita tapahtuma
8. Lähetä tapahtuma

```javascript
const cardano = require("./cardano")

// 1. get the wallet

const sender = cardano.wallet("ADAPI")

// 2. define the transaction

console.log(
    "Balance of Sender wallet: " +
    cardano.toAda(sender.balance().value.lovelace) + " ADA"
)

const receiver = "addr1qym6pxg9q4ussr96c9e6xjdf2ajjdmwyjknwculadjya488pqap23lgmrz38glvuz8qlzdxyarygwgu3knznwhnrq92q0t2dv0"

const txInfo = {
    txIn: cardano.queryUtxo(sender.paymentAddr),
    txOut: [
        {
            address: sender.paymentAddr,
            value: {
                lovelace: sender.balance().value.lovelace - cardano.toLovelace(1.5)
            }
        },
        {
            address: receiver,
            value: {
                lovelace: cardano.toLovelace(1.5),
                "ad9c09fa0a62ee42fb9555ef7d7d58e782fa74687a23b62caf3a8025.BerrySpaceGreen": 1
            }
        }
    ]
}

// 3. build the transaction

const raw = cardano.transactionBuildRaw(txInfo)

// 4. calculate the fee

const fee = cardano.transactionCalculateMinFee({
    ...txInfo,
    txBody: raw,
    witnessCount: 1
})

// 5. pay the fee by subtracting it from the sender utxo

txInfo.txOut[0].value.lovelace -= fee

// 6. build the final transaction

const tx = cardano.transactionBuildRaw({ ...txInfo, fee })

// 7. sign the transaction

const txSigned = cardano.transactionSign({
    txBody: tx,
    signingKeys: [sender.payment.skey]
})

// 8. submit the transaction

const txHash = cardano.transactionSubmit(txSigned)

console.log(txHash)
```

```javascript
$ cd ..
node src/send-back-asset-to-wallet.js
```

### Lopulliset vaiheet NFT:n katseluun

1. Tarkastele NFT:tä lompakossasi
2. Tarkastele assettiasi cardanoassets.com -palvelussa
3. View your asset on pool.pm (see the actual picture)
4. Näytä alkuperäinen mintingin metatiedot
5. Avaa src ja kuvan ipfs linkit selaimessasi varmistaaksesi, että prosessi toimi

#### _Video Walk-through:_

{% embed url="https://youtu.be/awxVkFbWoKM" %}

{% hint style="success" %}
**If you liked this tutorial and want to see more like it please consider staking your ADA with any of our Alliance's** [**Stake Pools**](https://armada-alliance.com/stake-pools)**, or giving a one-time donation to our Alliance** [**https://cointr.ee/armada-alliance**](https://cointr.ee/armada-alliance)**.**
{% endhint %}
