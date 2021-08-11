# Cardano NFT Collection Tutorial 👛

## Prerequisites

* cardano-node / cardano-cli set up on local machine \([https://docs.cardano.org/projects/cardano-node/en/latest](https://docs.cardano.org/projects/cardano-node/en/latest)\)
* Node.js installed version 14
* cardano-cli-js package installed
* cardano-minter repo from the previous tutorial

{% hint style="info" %}
**If you haven't already, please watch our video from the previous NFT tutorial 😎**
{% endhint %}

{% embed url="https://youtu.be/OeOliguGn7Y" caption="" %}

### Clone the cardano-minter repo if you haven't already...

```text
git clone https://github.com/armada-alliance/cardano-minter
cd cardano-minter
```

### Install additional dependencies

```text
npm install form-data dotenv axios lodash sharp promise-parallel-throttle --save
```

## Now, let's start with the tutorial 😊

### 1. Create our initial assets

* While in the "cardano-minter" directory create a script that will generate our assets in a nicely formatted JSON file called "assets.json".

```text
nano create-initial-assets-json.js
```

```javascript
/**
 * This script is responsible for generating the initial
 * assets.json that can later be adjusted to fit your specific needs
 *
 * You can define:
 * 1. amount of assets
 * 2. whether you want to start the collection with either 1 or 0
 * 3. amount of assets
 * 2. whether you want to start the collection with either 1 or 0
 * 3. /**
 * This script is responsible for generating the initial
 * assets.json that can later be adjusted to fit your specific needs
 *
 * You can define:
 * 1. amount of assets
 * 2. whether you want to start the collection with either 1 or 0
 * 3. what the mimeType is (jpeg, png or gif)
 */

const times = require('lodash/times')
const fs = require("fs").promises

const AMOUNT_OF_ASSETS = 15
const START_WITH_ZERO = true
const MIME_TYPE = 'image/png'

async function main() {

    const assets = times(AMOUNT_OF_ASSETS).map(i => {

        const number = START_WITH_ZERO ? i : i + 1
        const id = `PIADA${number}` // PIADA0

        const [extension] = MIME_TYPE.split("/").reverse() // png

        return {
            id,
            name: `PIADA #${number}`,
            // description: "", 
            image: `images/${id}_thumbnail.${extension}`, // images/PIADA0_thumbnail.png
            src: `images/${id}.${extension}`, // images/PIADA0.png
            type: MIME_TYPE,
            // add whatever like below
            authors: ["PIADA", "SBLYR"],
            website: "https://ada-pi.io"
        }
    })

    await fs.writeFile(__dirname + '/assets.json', JSON.stringify(assets, null, 2))
}

main()
```

```text
node src/create-initial-assets-json.js
```

* Your assets.json file should look like [this](https://github.com/armada-alliance/cardano-minter-collection/blob/master/src/assets.json).

### 2. Download random images for testing

* Make a folder called images to download the test images into
* Create a script that will go and grab the images from the internet and download them into the images folder

```text
cd src
nano download-test-images.js
```

```javascript
/**
 * This script expect the assets.json to exist
 * inside the src directory there should be a reference
 * to a filepath on the local file system relative to the `src` dir
 */

const random = require('lodash/random')
const axios = require('axios')
const fs = require('fs').promises

const assets = require("./assets.json")

async function main() {

    await Promise.all(
        assets.map(async asset => {

            const { data } = await axios.get(`https://source.unsplash.com/640x400?cat&v=${random()}`, { responseType: 'arraybuffer' })
            console.log(`[${asset.name}] downloaded random cat image`)

            await fs.writeFile(__dirname + '/' + asset.src, data)
            console.log(`[${asset.name}] image saved to "${asset.src}"`)
        })
    )
}

main()
```

```text
node src/download-test-images.js
```

### 3. Extend metadata.json with thumbnails \(optional\)

* generate thumbnails based on images from the metadata.json and give them the same name with `_thumbnail` tag added to the name

```text
cd src
nano generate-thumbnails.js
```

```javascript
const fs = require('fs').promises
const sharp = require("sharp")

const generateThumbnail = (filePath, data) => new Promise(resolve => {

    sharp(data)
        .resize(300) // 640x400 (640 -> 200)  
        .toFile(filePath, resolve)
})

const assets = require('./assets.json')

async function main() {

    await Promise.all(
        assets.map(async asset => {

            const data = await fs.readFile(__dirname + '/' + asset.src)

            await generateThumbnail(__dirname + "/" + asset.image, data)
            console.log(`[${asset.name}] thumbnail generated at "${asset.image}"`)
        })
    )
}

main()
```

```text
node src/generate-thumbnails.js
```

### 4. Create our [pinata.cloud](https://pinata.cloud/) account to get our API keys

1. Create an account
2. Create API keys

### 5. Need to safely store our API keys

* create .env file and paste in our keys

{% hint style="info" %}
Make sure the **.env** file is in the **cardano-minter** directory but **not in** **the** **src** folder
{% endhint %}

```text
nano .env
```

```text
PINATA_API_KEY='Enter Your API Key'
PINATA_API_SECRET='Enter Your API Secret Key'
```

### 6. Upload and pin our data to IPFS

{% hint style="info" %}
Read [this article ](https://docs.ipfs.io/how-to/pin-files/#three-kinds-of-pins)to learn more about why we want to Pin our NFTs to IPFS.
{% endhint %}

* **First, we need to make a script called pin-to-ipfs.js, this script will "upload" and Pin our images to IPFS using the pinata.cloud API.**

```text
nano pin-to-ipfs.js
```

```javascript
const dotenv = require('dotenv')
dotenv.config()
const axios = require("axios")
const FormData = require('form-data')
const fs = require('fs')

const pinata = axios.create({
    baseURL: 'https://api.pinata.cloud',
    headers: {
        pinata_api_key: process.env.PINATA_API_KEY,
        pinata_secret_api_key: process.env.PINATA_API_SECRET
    }
})

module.exports = async (name, filePath) => {

    let data = new FormData()
    data.append('file', fs.createReadStream(filePath))

    const metadata = JSON.stringify({
        name
    })

    data.append('pinataMetaData', metadata)

    const pinataOptions = JSON.stringify({
        cidVersion: 0,
        customPinPolicy: {
            regions: [
                {
                    id: 'FRA1',
                    desiredReplicationCount: 1
                },
                {
                    id: 'NYC1',
                    desiredReplicationCount: 1
                }
            ]
        }
    })

    data.append('pinataOptions', pinataOptions)

    const response = await pinata.post('/pinning/pinFileToIPFS', data, {
        maxBodyLength: 'Infinity', // this is needed to prevent axios from erroring out with large files
        headers: {
            'Content-Type': `multipart/form-data; boundary=${data._boundary}`
        }
    }).catch(e => {

        if (e.response) {
            console.log(e.response.error)
        } else {
            console.log(e.message)
        }
    })

    const hash = response.data.IpfsHash

    return {
        hash,
        ipfsLink: `ipfs://${hash}`,
        httpLink: `https://ipfs.io/ipfs/${hash}`
    }
}
```

```bash
cd ..
node src/pin-to-ipfs.js
```

* Next, we can create a script called pin-images-to-ipfs.js, this will run through our images/assets and "pin" the images to IPFS using our local node.

```bash
cd src
nano pin-images-to-ipfs.js
```

```javascript
const fs = require("fs").promises
const pinToIpfs = require("./pin-to-ipfs")
const Throttle = require("promise-parallel-throttle")

const assets = require("./assets.json")

async function main() {

    const updated_assets = await Throttle.sync(
        assets.map(asset => async () => {

            const { ipfsLink: image, httpLink: imageLink } = await pinToIpfs(`${asset.id}_image`, __dirname + "/" + asset.image)
            console.log(`[${asset.name}] pinned image to ipfs (${imageLink})`)

            const { ipfsLink: src, httpLink: srcLink } = await pinToIpfs(`${asset.id}_src`, __dirname + "/" + asset.src)
            console.log(`[${asset.name}] pinned image to ipfs (${srcLink})`)

            return {
                ...asset,
                image,
                src
            }
        })
    )

    // write updated assets to assets.json
    await fs.writeFile(__dirname + '/assets.json', JSON.stringify(updated_assets, null, 2))

    console.log('written updates to assets.json')
}

main()
```

```bash
node src/pin-images-to-ipfs.js
```

{% hint style="warning" %}
### Before you continue to the minting process, please understand the importance of minting policies and their scripts!
{% endhint %}

**Read the Cardano Documentation on "**[**Scripts**](https://docs.cardano.org/projects/cardano-node/en/latest/reference/simple-scripts.html#Step-1---construct-the-tx-body)**" and/or watch a video we made discussing the subject:**

{% embed url="https://youtu.be/v6q66zcFqew" caption="" %}

### 7. Create an "open" or "unlocked" minting policy and script \(Optional\)

* We will create an open minting policy script and export it in JSON and TXT format.

```bash
cd src
nano create-mint-policy.js
```

```javascript
const fs = require("fs")
const cardano = require("./cardano")

const wallet = cardano.wallet("PIADA")

const mintScript = {
    keyHash: cardano.addressKeyHash(wallet.name),
    type: "sig"
}

fs.writeFileSync(__dirname + "/mint-policy.json", JSON.stringify(mintScript, null, 2))
fs.writeFileSync(__dirname + "/mint-policy-id.txt", cardano.transactionPolicyid(mintScript))
```

```text
node src/create-mint-policy.js
```

### 8. Create a "time-locked" minting policy and script \(Recommended\)

* Create a "time-locked" minting policy script and export it in JSON and TXT format.

```text
cd src
nano create-time-locked-mint-policy.js
```

```javascript
const fs = require("fs")
const cardano = require("./cardano")

const wallet = cardano.wallet("PIADA")

const { slot } = cardano.queryTip()

const SLOTS_PER_EPOCH = 5 * 24 * 60 * 60 // 432000

const mintScript = {
    type: "all",
    scripts: [
        {
            slot: slot + (SLOTS_PER_EPOCH * 5),
            type: "before"
        },
        {
            keyHash: cardano.addressKeyHash(wallet.name),
            type: "sig"
        }
    ]
}

fs.writeFileSync(__dirname + "/mint-policy.json", JSON.stringify(mintScript, null, 2))
fs.writeFileSync(__dirname + "/mint-policy-id.txt", cardano.transactionPolicyid(mintScript))
```

```text
node src/create-time-locked-mint-policy.js
```

### 9. Create a script to get our policy ID

* We want to make a script that can get our Policy ID to be used in other parts of our program

```bash
cd src
nano get-policy-id.js
```

```javascript
const cardano = require("./cardano")
const mintScript = require("./mint-policy.json")

module.exports = () => {

    const policyId = cardano.transactionPolicyid(mintScript)

    return {
        policyId,
        mintScript
    }
}
```

```text
node src/get-policy-id.js
```

### 9. Define the mint transaction

1. build mint transaction with metadata.json
2. calc fee
3. rebuild
4. sign
5. submit

```bash
cd src
nano mint-multiple-assets.js
```

```javascript
const cardano = require("./cardano")
const getPolicyId = require("./get-policy-id")
const assets = require("./assets.json")

const wallet = cardano.wallet("PIADA")

const { policyId: POLICY_ID, mintScript } = getPolicyId()

const metadata_assets = assets.reduce((result, asset) => {

    const ASSET_ID = asset.id // PIADA0

    // remove id property from the asset metadata
    const asset_metadata = {
        ...asset
    }

    delete asset_metadata.id

    return {
        ...result,
        [ASSET_ID]: asset_metadata
    }
}, {})

const metadata = {
    721: {
        [POLICY_ID]: {
            ...metadata_assets
        }
    }
}

const txOut_value = assets.reduce((result, asset) => {

    const ASSET_ID = POLICY_ID + "." + asset.id
    result[ASSET_ID] = 1
    return result

}, {
    ...wallet.balance().value
})

const mint_actions = assets.map(asset => ({ action: "mint", amount: 1, token: POLICY_ID + "." + asset.id }))

const tx = {
    txIn: wallet.balance().utxo,
    txOut: [
        {
            address: wallet.paymentAddr,
            amount: txOut_value
        }
    ],
     mint: {
        actions: mint_actions,
        script: [mintScript]
    },
    metadata,
    witnessCount: 2
}

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

const signTransaction = (wallet, tx, script) => {

     return cardano.transactionSign({
        signingKeys: [wallet.payment.skey, wallet.payment.skey],
        txBody: tx
    })
}

const signed = signTransaction(wallet, raw, mintScript)

// 10. Submit transaction

const txHash = cardano.transactionSubmit(signed)

console.log(txHash)
```

```bash
node src/mint-multiple-assets.js
```

### 10. Send assets back to wallet

* Make a script to send multiple assets back to a wallet in a single transaction.

```bash
cd src
nano send-multiple-assets-back-to-wallet.js
```

```javascript
const cardano = require("./cardano")
const assets = require("./assets.json")
const getPolicyId = require('./get-policy-id')

const sender = cardano.wallet("PIADA")

console.log(
    "Balance of Sender address" +
    cardano.toAda(sender.balance().amount.lovelace) + " ADA"
)

const { policyId: POLICY_ID } = getPolicyId()

function sendAssets({ receiver, assets }) {

    const txOut_amount_sender = assets.reduce((result, asset) => {

        const ASSET_ID = POLICY_ID + "." + asset
        delete result[ASSET_ID]
        return result
    }, {
        ...sender.balance().amount
    })

    const txOut_amount_receiver = assets.reduce((result, asset) => {

        const ASSET_ID = POLICY_ID + "." + asset
        result[ASSET_ID] = 1
        return result
    }, {})

    // This is depedent at the network, try to increase this amount of ADA
    // if you get an error saying: OutputTooSmallUTxO
    const MIN_ADA = 3

    const txInfo = {
        txIn: cardano.queryUtxo(sender.paymentAddr),
        txOut: [
            {
                address: sender.paymentAddr,
                amount: {
                    ...txOut_amount_sender,
                    lovelace: txOut_amount_sender.lovelace - cardano.toLovelace(MIN_ADA)
                }
            },
            {
                address: receiver,
                amount: {
                    lovelace: cardano.toLovelace(MIN_ADA),
                    ...txOut_amount_receiver
                }
            }
        ]
    }

    const raw = cardano.transactionBuildRaw(txInfo)

    const fee = cardano.transactionCalculateMinFee({
        ...txInfo,
        txBody: raw,
        witnessCount: 1
    })

    txInfo.txOut[0].amount.lovelace -= fee

    const tx = cardano.transactionBuildRaw({ ...txInfo, fee })

    const txSigned = cardano.transactionSign({
        txBody: tx,
        signingKeys: [sender.payment.skey]
    })

    const txHash = cardano.transactionSubmit(txSigned)

    console.log(txHash)
}

sendAssets({
    receiver: "addr1qylm539axczhyvdh90f6c09ptrz8asa4hgq8u5shkw3v9vjae9ftypmc8tmd2rrwngdxm4sr3tpzmxw4zyg3z7vttpwsl0alww",
    assets: assets.map(asset => asset.id)
})
```

```bash
node src/send-multiple-assets-back-to-wallet.js
```

