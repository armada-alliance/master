---
description: Vamos a hacer algunos activos nativos en Cardano ❤️✨
---

# NFT (Tokens no fungibles) en Cardano 💰

## ¿Para quién es esta guía?

* Para las personas que quieren hacer NFT o Activos Nativos en Cardano
* Para personas que conocen Cardano

## Beneficios de los NFT en Cardano

* Bajo costo por transacción
* Nativos en la cadena de bloques

## Prerrequisitos

{% hint style="danger" %}
Hicimos este tutorial para usarlo con máquinas **Raspberry-Pi-ARM** ejecutándose en **Linux OS** así que asegúrate de descargar el **nodo** correcto para tu **máquina local/CPU y sistema operativo**. Actualmente, el Cardano-node y Cardano-cli están pensados para ser construidos a partir de código fuente en máquinas Linux. Cualquier otro sistema operativo tendrá sus propias complejidades de construcción, y no las cubrimos en ninguno de nuestros tutoriales. [Cómo construir un Nodo de Cardano desde el código fuente](https://docs.cardano.org/projects/cardano-node/en/latest/getting-started/install.html)
{% endhint %}

{% hint style="info" %}
Si estás usando una máquina Raspberry Pi [aquí](https://docs.armada-alliance.com/learn/beginner-guide-1/raspi-node) tienes un tutorial fácil de seguir que hicimos para tener un Nodo Relay de Cardano en funcionamiento.
{% endhint %}

* cardano-node / cardano-cli configurado en la máquina local
* Asegúrate de que tienes un nodo Cardano corriendo y sincronizado completamente con la base de datos
* Asegúrate de que node.js está instalado.

```bash
#Copia/Pega esto en tu terminar si no tienes node.js instalado
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Verifica que todo está configurado correctamente en nuestra máquina ⚙️

```bash
#Copia/pega en la terminal
cardano-cli version; cardano-node version
```

El resultado debería parecerse a esto 👇

```bash
cardano-cli 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
cardano-node 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

#### Verifica que nuestra versión de node.js es correcta y está en v14.16.1

```bash
#Copia/pega esto en la terminal
node -v
```

```bash
v14.18.1
```

#### Vídeo explicativo:

{% embed url="https://youtu.be/oP3jZyPxB-I" %}

## Crear nuestro directorio del proyecto y la configuración inicial

Make sure our `$NODE_HOME` environment variable exists

```bash
# check for $NODE_HOME
echo $NODE_HOME
```

If the above command didn't return anything, you need to set the`$NODE_HOME`bash environment variable or use a static path for the Cardano node's socket location in`db`C in your Cardano node directory.

```
export NODE_HOME="/home/ada/pi-pool"
# Change this to where cardano-node creates socket
export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket"
```

Now let's make our projects directory then create our <mark style="color:blue;">package.json</mark> file and install the <mark style="color:blue;">cardanocli-js</mark> package.

```bash
mkdir cardano-minter
cd cardano-minter
npm init -y #creates package.json)
npm install cardanocli-js --save
```

1. **Copia el último Cardano node genesis compilado en el sitio web de hydra de IOHK**
   * [https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html](https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html)
2. **Crea un script de shell de bash para descargar el último archivo de configuración de Génesis necesario**

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

**Now we need to give permissions to our new script to execute then we will run our script and download the genesis files.**

```bash
sudo chmod +x fetch-config.sh
./fetch-config.sh
```

### A continuación, creamos nuestra carpeta src y luego creamos el cliente Cardano.

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

#### _Video Explicativo_ :

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





## Crea una billetera local

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
cd ..
node src/create-wallet.js
```

#### Verifica que el saldo de la billetera es cero, si es así enviaremos fondos a la billetera

* **Primero, necesitamos crear un script get-balance.js**

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

* **Ahora, comprueba el saldo de nuestra billetera.**

```
cd ..
node src/get-balance.js
```

* We can go ahead and send some funds (ADA) into our wallet we created, wait a few minutes, and then check the balance again to make sure the transaction was successful.

{% hint style="info" %}
If you are using testnet you must get your tADA from the testnet faucet [here](https://developers.cardano.org/en/testnets/cardano/tools/faucet/).
{% endhint %}

#### _Video Explicativo_ :

{% tabs %}
{% tab title="undefined" %}

{% endtab %}
{% endtabs %}

## Acuña nuestro Activo-Nativo/NFT en Cardano

Before we proceed to mint our Native Asset we must have a few things taken care of. We need to first get our "asset" onto our [IPFS](https://ipfs.io/#install) node and generate the IPFS link. If you do not know about IPFS or what it actually does we recommend having a read through the documentation [here](https://docs.ipfs.io) or watching this [video](https://www.youtube.com/watch?v=5Uj6uR3fp-U).

Since we are using an image file to be our asset we should upload a smaller thumbnail-sized version of our image (ideally less than 1MB). This will be used on sites like [pool.pm](https://pool.pm) to display our assets nicely in our wallets. We then upload the full-size image as our source image.

* [ ] Descarga [IPFS](https://ipfs.io/#install)
* [ ] Sube los archivos de tu activo a IPFS
* [ ] Obtén nuestro enlace IPFS de la imagen en miniatura
* [ ] Obtén el enlace src IPFS

#### Como referencia:

* **image (thumbnail version) - ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE**
* **src (full-size version) - ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N**

### Crea nuestro script mint-asset.js

This script has three main components:

1. **Generar policy id**
2. **Definir tus metadatos**
3. **Crear una transacción de acuñación**

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

* **Ejecute el script de acuñación, luego espere unos segundos para comprobar el saldo de nuestra cartera**

```
cd ..
node src/mint-asset.js
```

_**Vídeo explicativo:**_

{% tabs %}
{% tab title="" %}
{% embed url="https://youtu.be/qTzLgMyJC7s" %}


{% endtab %}
{% endtabs %}

## Enviando tu NFT de vuelta a la billetera de Daedalus o de Yoroi

Now we must create a new script to send our newly minted NFT to a wallet.

```javascript
cd cardaon-minter/src
nano send-back-asset-to-wallet.js
```

There are few main parts we have to this script in order to send the asset:

1. Obtener la billetera
2. Definir la transacción
3. Construir la transacción
4. Calcular la comisión.
5. Paga la comisión restándola del utxo del remitente
6. Construir la transacción final
7. Firmar la transacción
8. Enviar la transacción

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
cd ..
node src/send-back-asset-to-wallet.js
```

### Pasos finales para ver tu NFT

1. Ver tu nft en tu billetera
2. Ver tu activo en cardanoassets.com
3. View your asset on pool.pm (see the actual picture)
4. Mostrar los metadatos originales de acuñación
5. Abre los enlaces src e imagen ipfs en tu navegador para comprobar que ha funcionado

#### _Vídeo explicativo:_

{% embed url="https://youtu.be/awxVkFbWoKM" %}

{% hint style="success" %}
**If you liked this tutorial and want to see more like it please consider staking your ADA with any of our Alliance's** [**Stake Pools**](https://armada-alliance.com/stake-pools)**, or giving a one-time donation to our Alliance** [**https://cointr.ee/armada-alliance**](https://cointr.ee/armada-alliance)**.**
{% endhint %}
