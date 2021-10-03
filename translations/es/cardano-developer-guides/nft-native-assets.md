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
cardano-cli 1.26.2 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
cardano-node 1.26.2 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

#### Verifica que nuestra versión de node.js es correcta y está en v14.16.1

```bash
#Copia/pega esto en la terminal
node -v
```

```bash
v14.16.1
```

#### Vídeo explicativo:

{% embed url="https://youtu.be/oP3jZyPxB-I" caption="" %}

## Crear nuestro directorio del proyecto y la configuración inicial

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

1. **Copia el último Cardano node genesis compilado en el sitio web de hydra de IOHK**
   * [https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html](https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html)
2. **Crea un script de shell de bash para descargar el último archivo de configuración de Génesis necesario**

```bash
nano fetch-config.sh
```

{% tabs %}
{% tab title="MAINNET" %}
```bash
#NODE_BUILD_NUM podría ser diferente
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

**Ahora necesitamos dar permisos a nuestro nuevo script para ejecutar entonces ejecutaremos nuestro script y descargaremos los archivos de génesis.**

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
Si estás usando testnet asegúrate de que tienes el número de versión de testnet-magic correcto. Puedes encontrar la versión actual de testnet [aquí](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html).
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



{% embed url="https://youtu.be/Xkx9vdibbq0" caption="" %}



{% embed url="https://www.youtube.com/watch?v=X5cRGA0qyQE" caption="" %}

{% embed url="https://youtu.be/-fnaF3FWL3k" caption="" %}

## Crea una billetera local

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

```text
cd ..
node src/get-balance.js
```

* Podemos seguir adelante y enviar algunos fondos \(ADA\) a la billetera que creamos, espera unos minutos, y luego revisa el saldo de nuevo para asegurarnos de que la transacción fue exitosa.

{% hint style="info" %}
Si está usando testnet debe obtener su tADA del grifo testnet [aquí](https://developers.cardano.org/en/testnets/cardano/tools/faucet/).
{% endhint %}

#### _Video Explicativo_ :

{% tabs %}
{% tab %}

{% endtab %}
{% endtabs %}

## Acuña nuestro Activo-Nativo/NFT en Cardano

Antes de proceder a acuñar nuestro Activo Nativo debemos tener algunas cosas claras. Primero necesitamos colocar nuestro "activo" en nuestro nodo [IPFS](https://ipfs.io/#install) y generar el enlace IPFS. Si no sabes sobre IPFS o lo que realmente hace, te recomendamos que leas la documentación [aquí](https://docs.ipfs.io/) o que veas este [video](https://www.youtube.com/watch?v=5Uj6uR3fp-U).

Puesto que estamos usando un archivo de imagen para ser nuestro activo, debemos subir una versión más pequeña de nuestra imagen \(idealmente menos de 1MB\). Esto se utilizará en sitios como [pool.pm](https://pool.pm) para mostrar nuestros activos de forma agradable en nuestras carteras. A continuación, cargamos la imagen de tamaño completo como nuestra imagen fuente.

* [ ] Descarga [IPFS](https://ipfs.io/#install)
* [ ] Sube los archivos de tu activo a IPFS
* [ ] Obtén nuestro enlace IPFS de la imagen en miniatura
* [ ] Obtén el enlace src IPFS

#### Como referencia:

* **image \(thumbnail version\) - ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE**
* **src \(full-size version\) - ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N**

### Crea nuestro script mint-asset.js

Este script tiene tres componentes principales:

1. **Generar policy id**
2. **Definir tus metadatos**
3. **Crear una transacción de acuñación**

```javascript
nano mint-asset.js
```

```javascript
const fs = require("fs");
const cardano = require("./cardano");

// 1. Obtener la billetera
// 2. Definir el script de acuñación
// 3. Crear POLICY_ID
// 4. Definir ASSET_NAME
// 5. Crear ASSET_ID
// 6. Definir metadatos
// 7. Definir la transacción
// 8. Construir la transacción
// 9. Firmar la transacción
// 10. Enviar la transacción

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

* **Ejecute el script de acuñación, luego espere unos segundos para comprobar el saldo de nuestra cartera**

```text
cd ..
node src/mint-asset.js
```

_**Vídeo explicativo:**_

{% tabs %}

{% embed url="https://youtu.be/qTzLgMyJC7s" caption="" %}

## Enviando tu NFT de vuelta a la billetera de Daedalus o de Yoroi

Ahora debemos crear un nuevo script para enviar nuestro NFT recién acuñado a una billetera.

```javascript
cd cardano-minter/src
nano send-back-asset-to-wallet.js
```

Hay algunas partes importantes que tenemos en este script para enviar el activo:

1. Obtener la billetera
2. Definir la transacción
3. Construir la transacción
4. Calcular la comisión.
5. Paga la comisión restándola del utxo del remitente
6. Construir la transacción final
7. Firmar la transacción
8. Enviar la transacción

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
cd ..
node src/send-back-asset-to-wallet.js
```

### Pasos finales para ver tu NFT

1. Ver tu nft en tu billetera
2. Ver tu activo en cardanoassets.com
3. Ver tu activo en pool.pm \(ver la imagen real\)
4. Mostrar los metadatos originales de acuñación
5. Abre los enlaces src e imagen ipfs en tu navegador para comprobar que ha funcionado

#### _Vídeo explicativo:_

{% embed url="https://youtu.be/awxVkFbWoKM" caption="" %}

{% hint style="success" %}
**Si te ha gustado este tutorial y quieres ver más, por favor considera delegar tus ADA en cualquiera de la Alianza** [**Stake Pools**](https://armada-alliance.com/stake-pools)**, o dona una sola donación a nuestra Alianza** [**https://cointr. e/armada-alianza**](https://cointr.ee/armada-alliance)**.**
{% endhint %}

