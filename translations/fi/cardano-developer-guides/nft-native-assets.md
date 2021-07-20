---
description: Tehdään uusia Cardano alkuperäisresursseja ❤️✨
---

# Cardano alkuperäisresurssit (Native Assets) \(NFT\) 💰

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
Jos käytät Raspberry Pi konetta [h](../beginner-guide-1/beginner-guide/tutorial-2-relaynode.md)[tässä](../beginner-guide-1/beginner-guide/tutorial-2-relaynode.md) on helposti seurattava tutoriaali, jonka teimme Cardano Relay Node:n rakentamiseen ja käynnistämiseen.
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
cardano-cli 1.26.2 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
cardano-node 1.26.2 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

#### Varmista että node.js versio on oikein ja on v14.16.1

```bash
#Kopioi/liitä pääteikkunaan
palvelin -v
```

```bash
v14.16.1
```

#### Video Walk-through:

{% embed url="https://youtu.be/oP3jZyPxB-I" caption="" %}

## Luo projektihakemisto ja aloitusasetukset

```bash
# check for $NODE_HOME
echo $NODE_HOME

# if the above echo didn't return anything, you need to set a $NODE_HOME
# or use a static path for the CARDANO_NODE_SOCKET_PATH location

export NODE_HOME="/home/ada/pi-pool"
# Change this to where cardano-node creates socket
export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket"

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
{% tab title="MAINNET" %}
```bash
#NODE_BUILD_NUM voi olla eri
NODE_BUILD_NUM=6198010
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/mainnet-shelley-genesis.json
```
{% endtab %}

{% tab title="TESTNET" %}
```bash
NODE_BUILD_NUM=6198010
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/testnet-shelley-genesis.json
```
{% endtab %}
{% endtabs %}

**Nyt meidän täytyy antaa käyttöoikeudet meidän uudelle skriptille ja sitten ajamme skriptin ja lataamme genesis tiedostot.**

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
Jos käytät testiverkkoa varmista, että sinulla on oikea testnet-magic versionumero. Löydät nykyisen testnet-version [täältä](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html).
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

{% tab title="Fetch Genesis File" %}

{% tab title="Create Cardano Client" %}
{% embed url="https://youtu.be/-fnaF3FWL3k" caption="" %}
{% endtab %}
{% endtabs %}

## Luo uusi lompakko

```bash
nano create-wallet.js
```

```javascript
const cardano = require('./cardano')

const createWallet = (account) => {
  cardano.addressKeyGen(account);
  cardano.stakeAddressKeyGen(account);
  cardano.stakeAddressBuild(account);
  cardano.addressBuild(account);
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

```text
$ cd ..
node src/get-balance.js
```

* Voimme nyt lähettää joitakin varoja \(ADA\) luomaamme lompakkoon, odota muutama minuutti, ja sitten tarkista saldo uudelleen varmistaaksesi, että tapahtuma onnistui.

{% hint style="info" %}
Jos käytät testnetiä, sinun täytyy saada tADA testnet-hanasta [täältä](https://developers.cardano.org/en/testnets/cardano/tools/faucet/).
{% endhint %}

#### _Video Walk-through_ :

{% tabs %}
{% tab %}

{% endtab %}
{% endtabs %}

## Paina (Mint) uudet Native-Assetit/NFT:t Cardano lohkoketjuun

Ennen kuin ryhdymme lyömään meidän alkuperäis (native) assetteja, meillä on oltava muutamia asioita hoidettu. Meidän täytyy ensin saada "asset" meidän [IPFS](https://ipfs.io/#install) node:en ja luoda IPFS linkki. Jos et tiedä IPFS-järjestelmästä tai mitä se todella tekee, suosittelemme lukemaan dokumentaation kautta [täällä](https://docs.ipfs.io/) tai katsomaan tämän [videon](https://www.youtube.com/watch?v=5Uj6uR3fp-U).

Koska käytämme kuvatiedostoa assettinamme, meidän pitää ladata pienempi kuvake-kokoinen versio kuvastamme \(mieluiten alle 1MB\). Tätä käytetään nättiin visualisointiin sivustoilla, kuten [pool.pm](https://pool.pm) ja lompakoissamme. Sitten lataamme koko lähdekuvan NFT assetistamme.

* [ ] Lataa [IPFS](https://ipfs.io/#install)
* [ ] Lataa assetisi tiedostot IPFS:ään
* [ ] Hae meidän kuvakkeen IPFS linkki
* [ ] Hae src IPFS-linkki

#### Viitteeksi:

* **image \(thumbnail version\) - ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE**
* **src \(full-size version\) - ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N**

### Luo mint-asset.js skripti

Tässä skriptissä on kolme pääosaa:

1. **Luo policy id**
2. **Määrittele metatiedot**
3. **Määritä painatustapahtuma**

```javascript
nano mint-asset.js
```

```javascript
const fs = require("fs");
const cardano = require("./cardano");

// 1. Get the wallet
// 2. Define mint script
// 3. Create POLICY_ID
// 4. Define ASSET_NAME
// 5. Create ASSET_ID
// 6. Define metadata
// 7. Define transaction
// 8. Build transaction
// 9. Sign transaction
// 10. Submit transaction

const buildTransaction = (tx) => {
  const raw = cardano.transactionBuildRaw(tx);
  const fee = cardano.transactionCalculateMinFee({
    ...tx,
    txBody: raw,
  });
  tx.txOut[0].amount.lovelace -= fee;
  return cardano.transactionBuildRaw({ ...tx, fee });
};

const signTransaction = (wallet, tx, script) => {
  return cardano.transactionSign({
    signingKeys: [wallet.payment.skey, wallet.payment.skey],
    scriptFile: script,
    txBody: tx,
  });
};

const wallet = cardano.wallet("ADAPI");

const mintScript = {
  keyHash: cardano.addressKeyHash(wallet.name),
  type: "sig",
};

const POLICY_ID = cardano.transactionPolicyid(mintScript);
const ASSET_NAME = "BerrySpaceGreen";
const ASSET_ID = POLICY_ID + "." + ASSET_NAME;

const metadata = {
  721: {
    [POLICY_ID]: {
      [ASSET_NAME]: {
        name: "token name",
        image: "ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE",
        description: "Super Fancy Berry Space Green NFT",
        type: "image/png",
        src: "ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N",
        authors: ["PIADA", "SBLYR"],
      },
    },
  },
};

const tx = {
  txIn: wallet.balance().utxo,
  txOut: [
    {
      address: wallet.paymentAddr,
      amount: { ...wallet.balance().amount, [ASSET_ID]: 1 },
    },
  ],
  mint: [{ action: "mint", amount: 1, token: ASSET_ID }],
  metadata,
  witnessCount: 2,
};

const raw = buildTransaction(tx);
const signed = signTransaction(wallet, raw, mintScript);
const txHash = cardano.transactionSubmit(signed);
console.log(txHash);
```

* **Suorita minting script, odota hetki ja tarkista lompakkomme saldo**

```text
$ cd ..
node src/mint-asset.js
```

## Sending your NFT back to Daedulus or Yoroi wallet

Now we must create a new script to send our newly minted NFT to a wallet.

```javascript
cd cardaon-minter/src
nano send-back-asset-to-wallet.js
```

There are few main parts we have to this script in order to send the asset:

1. Get the wallet
2. Define the transaction
3. Build the transaction
4. Calculate the fee
5. Pay the fee by subtracting it from the sender's utxo
6. Build the final transaction
7. Sign the transaction
8. Submit the transaction

```javascript
const cardano = require("./cardano");

const sender = cardano.wallet("ADAPI");

console.log(
  "Balance of Sender wallet: " +
    cardano.toAda(sender.balance().amount.lovelace) +
    " ADA"
);

const receiver =
  "addr1qym6pxg9q4ussr96c9e6xjdf2ajjdmwyjknwculadjya488pqap23lgmrz38glvuz8qlzdxyarygwgu3knznwhnrq92q0t2dv0";

const txInfo = {
  txIn: cardano.queryUtxo(sender.paymentAddr),
  txOut: [
    {
      address: sender.paymentAddr,
      amount: {
        lovelace: sender.balance().amount.lovelace - cardano.toLovelace(1.5),
      },
    },
    {
      address: receiver,
      amount: {
        lovelace: cardano.toLovelace(1.5),
        "ad9c09fa0a62ee42fb9555ef7d7d58e782fa74687a23b62caf3a8025.BerrySpaceGreen": 1,
      },
    },
  ],
};

const raw = cardano.transactionBuildRaw(txInfo);

const fee = cardano.transactionCalculateMinFee({
  ...txInfo,
  txBody: raw,
  witnessCount: 1,
});

//pay the fee by subtracting it from the sender utxo
txInfo.txOut[0].amount.lovelace -= fee;

//create final transaction
const tx = cardano.transactionBuildRaw({ ...txInfo, fee });

//sign the transaction
const txSigned = cardano.transactionSign({
  txBody: tx,
  signingKeys: [sender.payment.skey],
});

//subm transaction
const txHash = cardano.transactionSubmit(txSigned);
console.log("TxHash: " + txHash);
```

```javascript
$ cd ..
node src/send-back-asset-to-wallet.js
```

### Final Steps to view your NFT

1. View your nft in your wallet
2. View your asset on cardanoassets.com
3. View your asset on pool.pm \(see the actual picture\)
4. Show the original minting metadata
5. Open the src and image ipfs links in your browser to prove that it worked

#### _Video Walk-through:_

{% embed url="https://youtu.be/awxVkFbWoKM" caption="" %}

{% hint style="success" %}
**If you liked this tutorial and want to see more like it please consider staking your ADA with our** [**PIADA**](https://adapools.org/pool/b8d8742c7b7b512468448429c776b3b0f824cef460db61aa1d24bc65) **Stake Pool, or giving a one-time donation to our Alliance** [**https://cointr.ee/armada-alliance**](https://cointr.ee/armada-alliance)**.**
{% endhint %}

\*\*\*\*

