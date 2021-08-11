# Cardano NFT-kokoelma opas

## Edellytykset

* cardano-node / cardano-cli asennettu paikalliseen koneeseen \([https://docs.cardano.org/projects/cardano-node/en/latest](https://docs.cardano.org/projects/cardano-node/en/latest)\)
* Node.js versio 14 asennettu
* cardano-cli-js paketti asennettu
* cardano-minter repo edellisestä tutoriaalista

{% hint style="info" %}
**Jos et ole jo katsonut, katso meidän edellisen NFT opetusohjelman video 😎**
{% endhint %}

{% embed url="https://youtu.be/OeOliguGn7Y" caption="" %}

### Kloonaa cardano-minter repo jos et ole jo...

```text
git clone https://github.com/armada-alliance/cardano-minter
cd cardano-minter
```

### Riippuvuuksien asennus

```text
npm install form-data dotenv axios lodash sharp promise-parallel-throttle --save
```

## Aloitetaan nyt tutoriaalilla 😊

### 1. Luo alkuperäiset assetit

* "cardano-minter" hakemistossa, tee skripti, joka luo assetit nätisti muotoiltuun JSON tiedostoon nimeltä "assets.json".

```text
nano create-initial-assets-json.js
```

```javascript
/**
 * Tämä skripti on vastuussa alkuperäisen
 * assets.json luomisesta, joka voidaan myöhemmin säätää vastaamaan erityisiä tarpeitasi
 *
 * Voit määritellä:
 * 1. assettien määrän
 * 2. haluatko aloittaa kokoelman joko arvolla 1 tai 0
 * 3. mikä on mimeType (jpeg, png tai gif)
 */

const times = require('lodash/times')
const fs = require("fs"). romisit

const AMOUNT_OF_ASSETS = 15
const START_WITH_ZERO = true
const MIME_TYPE = 'image/png'

async function main() {

    const assets = times(AMOUNT_OF_ASSETS). ap(i => {

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

* Assets.json tiedoston pitäisi näyttää [tältä](https://github.com/armada-alliance/cardano-minter-collection/blob/master/src/assets.json).

### 2. Lataa satunnaisia kuvia testausta varten

* Tee hakemisto nimeltä 'kuvat', johon voit ladata testikuvat
* Luo skripti, joka menee ja nappaa kuvia internetistä ja lataa ne kuvat-kansioon

```text
cd src
nano download-test-images.js
```

```javascript
/**
 * Tämä skripti olettaa assets.json tiedoston olemassaoloon
 * src hakemistossa. Paikallisessa tiedostojärjestelmässä pitäisi olla
 * viite tiedostopolkuun `src` hakemistoon
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

### 3. Laajenna metadata.json kuvakkeilla \(valinnainen\)

* luo kuvakkeet, jotka perustuvat metadata.jsonin kuviin ja anna niille sama nimi `_thumbnail` -tunnisteella, joka on lisätty nimeen

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

### 4. Luo [pinata.cloud](https://pinata.cloud/) -tili saadaksesi API-avaimemme

1. Luo käyttäjätili
2. Luo API-avaimet

### 5. API-avaimia on säilytettävä turvallisesti

* luo .env tiedosto ja liitä siihen avaimet

{% hint style="info" %}
Varmista, että **.env** tiedosto on **cardano-minter** hakemistossa, **eikä** **** **src** kansiossa
{% endhint %}

```text
nano .env
```

```text
PINATA_API_KEY='Enter Your API Key'
PINATA_API_SECRET='Enter Your API Secret Key'
```

### 6. Lataa ja kiinnitä data IPFS-järjestelmään

{% hint style="info" %}
Lue [tämä artikkeli ](https://docs.ipfs.io/how-to/pin-files/#three-kinds-of-pins)saadaksesi lisää tietoa siitä, miksi haluamme kiinnittää NFT: mme IPFS:lle.
{% endhint %}

* **Ensinnäkin, meidän täytyy tehdä skripti nimeltään pin-to-ipfs.js, tämä skripti "vie" ja kiinnittää kuvamme IPFS:ään käyttäen pinata.cloud APIa.**

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
$ cd ..
node src/pin-to-ipfs.js
```

* Seuraavaksi voimme luoda skriptin nimeltä pin-images-to-ipfs.js, tämä käy läpi kuvia/resursseja ja "kiinnittää" kuvia IPFS:ään käyttämällä paikallista nodea.

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

{% hint style="Huomaa" %}
### Ennen kuin jatkat painoprosessia (minting process), on hyvä ymmärtää painotoiminnan säännösten ja niiden skriptien tärkeys!
{% endhint %}

**Lue Cardano Dokumentaatio osoitteesta "**[**Skriptit**](https://docs.cardano.org/projects/cardano-node/en/latest/reference/simple-scripts.html#Step-1---construct-the-tx-body)**" ja/tai katso video, jonka teimme aiheesta:**

{% embed url="https://youtu.be/v6q66zcFqew" caption="" %}

### 7. Luo "open" tai "unlocked" minting käytäntö ja skripti \(Valinnainen\)

* Luomme avoimen painatussääntö (minting policy) -skriptin ja tallennamme sen JSON ja TXT muodossa.

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

### 8. Luo "aika-lukittu" minting käytäntö ja skripti \(Suositeltu\)

* Luo "aika-lukittu" minting policy skripti ja tallenna se JSON ja TXT muodoissa.

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

### 9. Luo skripti, jotta saat policy ID:n

* Haluamme tehdä skriptin, joka hakee meidän policy ID:n, jotta voimme käyttää sitä tutoriaalimme muissa osissa

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

### 9. Määritä painatustapahtuma

1. luo painatustapahtuma metadata.jsonin kanssa
2. laske kustannus
3. luo uudelleen
4. allekirjoita
5. lähetä

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

### 10. Lähetä assetit takaisin lompakkoon

* Tee skripti lähettääksesi useita assetteja takaisin lompakkoon yhdessä tapahtumassa.

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

