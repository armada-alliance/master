---
description: Luo toiminnalliset avaimet & sertifikaatit. Luo lompakko & rekisteröi stakepool
---

# Pi-Core/Kylmä kone

{% hint style="danger" %}
Tarvitset Pi-Noden, joka on konfiguroitu uudella staattisella IP-osoitteella omassa lähiverkossasi. Täysin pätevä verkkotunnus ja cardano-service tiedosto on asetettu käyttämään porttia 3000. Sinun täytyy myös päivittää env-tiedosto, jota gLiveView.sh käyttää osoitteessa $NODE\_HOME/skripts.

Et ota käyttöön topologian päivityspalvelua core nodessa, joten voit poistaa nämä kaksi komentosarjaa ja poistaa kommentoidun cron työn cron-taulukosta.

Varmista, että core node on synkronoitu lohkoketjun kärkeen saakka.
{% endhint %}

{% hint style="warning" %}
On olemassa tapa luoda poolin lompakon **payment keypair** luomalla Yoroi lompakko ja käyttämällä cardano-lompakkoa poimimaan avainpari Yoroin mnemonic seed:istä. Näin saat varmuuskopion lompakosta avainsanojen muodoss ja voit helposti siirtää palkintoja tai lähettää varoja muualle. Voit tehdä tämän millä tahansa Shelley aikakauden mnemonisella seedillä. Itse pidän Yoroista, koska se on nopea.

[https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894​](https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894)

Cardano-lompakko ei rakennu Arm koneisiin riippuvuuden epäonnistumisen vuoksi. @ZW3RK yritti rakentaa sen meille, mutta se ei onnistunut. Haluat ehkä asentaa cardano-lompakon offline x86 koneeseen ja käydä läpi tämän prosessin. Näin minä sen tein. Löydät cardano-lompakon binäärin alla.

[https://hydra.iohk.io/build/3770189](https://hydra.iohk.io/build/3770189)
{% endhint %}

### Ota blockfetch seuranta käyttöön

```text
sed -i ${NODE_FILES}-mainnet-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g"
```

## Luo avaimet & Myönnä käyttötodistus

{% hint style="warning" %}
#### KES-avainten kierrätys

KES avaimet on uudistettava ja uusi **pool.cert** on myönnettävä ja toimitettava ketjulle 90 päivän välein. Tiedosto **node.counter** pitää kirjaa siitä, kuinka monta kertaa tämä on tehty.
{% endhint %}

Luo a KES avainpari: **kes.vkey** & **kes.skey**

{% tabs %}
{% tab title="Core" %}
```bash
cd $NODE_HOME
cardano-cli node key-gen-KES \
  --verification-key-file kes.vkey \
  --signing-key-file kes.skey
```
{% endtab %}
{% endtabs %}

Luo noden kylmä avainpari: **node.vkey**, **node.skey** ja **node.counter** tallenna avaintiedostot kylmään Offline koneeseen.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
mkdir $HOME/cold-keys
cd cold-keys
cardano-cli node key-gen \
  --cold-verification-key-file node.vkey \
  --cold-signing-key-file node.skey \
  --operational-certificate-issue-counter node.counter
```
{% endtab %}
{% endtabs %}

Luo muuttujat, joissa on KES-jakson slottien määrä. Tämä määritetään genesis-tiedostosta ja ketjun nykyisestä kärjestä.

{% tabs %}
{% tab title="Core" %}
```bash
slotsPerKesPeriod=$(cat $NODE_FILES/mainnet-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slot')
echo slotsPerKesPeriod: ${slotsPerKesPeriod}
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Aseta **startKesPeriod** muuttuja jakamalla **slotNo** / **slotsPerKESPeriod**.

{% tabs %}
{% tab title="Core" %}
```bash
startKesPeriod=$((${slotNo} / ${slotsPerKesPeriod}))
echo startKesPeriod: ${startKesPeriod}
```
{% endtab %}
{% endtabs %}

Kirjoita **startKesPeriod** arvo ylös & kopioi **kes.vkey** kylmään offline koneeseen.

Myönnä **node.cert** sertifikaatti käyttäen: **kes.vkey**, **node.skey**, **node.counter** ja **startKesPeriod** arvoa.

Korvaa **&lt;startKesPeriod&gt;** arvolla, jonka kirjoitit ylös.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli node issue-op-cert \
  --kes-verification-key-file kes.vkey \
  --cold-signing-key-file $HOME/cold-keys/node.skey \
  --operational-certificate-issue-counter $HOME/cold-keys/node.counter
  --kes-period <startKesPeriod> \
  --out-file node.cert
```
{% endtab %}
{% endtabs %}

Kopioi **node.cert** block producer koneeseesi.

Luo VRF avainpari.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-VRF \
  --verification-key-file vrf.vkey \
  --signing-key-file vrf.skey
```
{% endtab %}
{% endtabs %}

Turvallisuussyistä **vrf.skey** **tarvitsee** vain lukuoikeudet muuten cardano-node ei käynnisty.

{% tabs %}
{% tab title="Core" %}
```bash
chmod 400 vrf.skey
```
{% endtab %}
{% endtabs %}

Muokkaa cardano-service startup skriptiä lisäämällä **kes.skey**, **vrf. avain** ja **node.cert** kardano-noden run komentoon ja muuta portti jota se kuuntelee.

{% tabs %}
{% tab title="Core" %}
```bash
nano $HOME/.local/bin/cardano-service
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3000
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json
KES=${DIRECTORY}/kes.skey
VRF=${DIRECTORY}/vrf.skey
CERT=${DIRECTORY}/node.cert
## +RTS -N4 -RTS = Multicore(4)
cardano-node run +RTS -N4 -RTS \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG} \
  --shelley-kes-key ${KES} \
  --shelley-vrf-key ${VRF} \
  --shelley-operational-certificate ${CERT}
```
{% endtab %}
{% endtabs %}

Lisää relaysi mainnet-topolgy.jsoniin.

{% tabs %}
{% tab title="Core" %}
```bash
nano $NODE_FILES/mainnet-topology.json
```
{% endtab %}
{% endtabs %}

Käytä LAN IPv4:ää addr -kenttään, jos et käytä verkkotunnuksen DNS. Varmista, että sinulla on asianmukaiset tietueet asetettuna rekisteriin tai DNS palveluun. Alla on muutamia esimerkkejä.

Valency suurempi kuin yksi käytetään vain DNS round robin srv tietueiden kanssa.

{% tabs %}
{% tab title="1 Relay DNS" %}
```text
{
  "Producers": [
    {
      "addr": "r1.example.com",
      "port": 3001,
      "valency": 1
    }
  ]
}
```
{% endtab %}

{% tab title="2 Relays DNS" %}
```text
{
  "Producers": [
    {
      "addr": "r1.example.com",
      "port": 3001,
      "valency": 1
    },
    {
      "addr": "r2.example.com",
      "port": 3002,
      "valency": 1
    }
  ]
}
```
{% endtab %}

{% tab title="1 Relay IPv4" %}
```text
{
  "Producers": [
    {
      "addr": "192.168.1.151",
      "port": 3001,
      "valency": 1
    }
  ]
}
```
{% endtab %}

{% tab title="2 Relays IPv4" %}
```text
{
  "Producers": [
    {
      "addr": "192.168.1.151",
      "port": 3001,
      "valency": 1
    },
    {
      "addr": "192.168.1.152",
      "port": 3002,
      "valency": 1
    }
  ]
}
```
{% endtab %}
{% endtabs %}

Käynnistä uudelleen ja node on nyt core eikä relay.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service restart
```
{% endtab %}
{% endtabs %}

## Luo pool-lompakko, maksu & staking avainparit

{% hint style="danger" %}
**Kylmä offlline kone.** Käytä aikaa visualisoidaksesi toiminnot täällä.

1. _**Luo**_ lompakon avain pari nimeltä payment. = **payment.vkey** & **payment.skey**
2. _**Luo**_ staking avainpari. = **stake.vkey** & **stake.skey**
3. _**Rakenna**_ stake osoite juuri luodusta **stake.vkey -avaimesta**. = **stake.addr**
4. _**Rakenna**_ lompakon osoite **payment.vkey** & delegoi **stake.vkey**. = **payment.addr**
5. Lisää varoja lompakkoon lähettämällä ada **payment.addr**
6. Tarkista saldo.
{% endhint %}

### 1. Luo lompakon avainpari

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cd $NODE_HOME
cardano-cli address key-gen \
  --verification-key-file payment.vkey \
  --signing-key-file payment.skey
```
{% endtab %}
{% endtabs %}

### 2. Luo staking avainpari

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address key-gen \
  --verification-key-file stake.vkey \
  --signing-key-file stake.skey
```
{% endtab %}
{% endtabs %}

### 3. Koosta staking osoite

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address build \
  --stake-verification-key-file stake.vkey \
  --out-file stake.addr \
  --mainnet
```
{% endtab %}
{% endtabs %}

### 4. Koosta maksuosoite

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli address build \
  --payment-verification-key-file payment.vkey \
  --stake-verification-key-file stake.vkey \
  --out-file payment.addr \
  --mainnet
```
{% endtab %}
{% endtabs %}

### 5. Siirrä varoja lompakkoon

```text
cat payment.addr
```

Kopioi **payment.addr** Usb-asemaan ja siirrä se core-noden pi-pool kansioon.

Lisää varoja lompakkoon. Tämä on ainoa lompakko, jota stake poolisi käyttää, joten pledge summa menee myös tänne. On 2 ada:n staking rekisteröintimaksu ja 500 ada:n poolin rekisteröinti talletukset. Nämä ovat pantteja, jotka saat takaisin, kun lopetat poolisi.

{% hint style="Huomaa" %}
Testaa lompakko lähettämällä pieni määrä adaa, odota muutama minuutti ja tarkasta lompakkosi saldo.
{% endhint %}

{% hint style="danger" %}
Core noden on oltava synkronoitu lohkoketjun kärjen kanssa.
{% endhint %}

### 6. Tarkista saldo

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --mainnet
```
{% endtab %}
{% endtabs %}

## Rekisteröi staking osoite

Myönnä staking rekisteröintisertifikaatti: **stake.cert**

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address registration-certificate \
  --stake-verification-key-file stake.vkey \
  --out-file stake.cert
```
{% endtab %}
{% endtabs %}

Kopioi **stake.cert** core noden pi-pool -kansioon.

Kysy ketjun nykyinen slotti eli kärki.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slotNo')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Hae lompakon utxo tai saldo.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --mainnet > fullUtxo.out

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out
cat balance.out
tx_in=""
total_balance=0

while read -r utxo; do
  in_addr=$(awk '{ print $1 }' <<< "${utxo}")
  idx=$(awk '{ print $2 }' <<< "${utxo}")
  utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
  total_balance=$((${total_balance}+${utxo_balance}))
  echo TxHash: ${in_addr}#${idx}
  echo Lovelace: ${utxo_balance}
  tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out
txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: $((${total_balance} / 1000000))
echo Number of UTXOs: ${txcnt}
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Jos sinulle tulee

`cardano-cli: Network.Socket.connect: : does not exist (No such file or directory)`

Se johtuu siitä, että ydin ei ole lopettanut synkronointia lohkoketjun kärkeen. Tämä voi kestää pitkän aikaa uudelleenkäynnistyksen jälkeen. Jos katsot db/ kansioon cardano-palvelun pysähdyksen jälkeen, näet tiedoston nimeltä 'puhdas'. Se on vahvistustiedosto tietokannan puhtaasta sammutuksesta. Kestää yleensä 5-10 minuuttia synkronoida takaisin ketjun kärkeen Raspberry Pi :lla (näin ainakin epochin 267 kohdalla).

Jos cardano-nodea ei kuitenkaan sammutettu 'puhtaasti', mistä tahansa syystä, voi kestää jopa tunnin tarkistaa tietokanta \(ketju \) ja luoda uusi socket tiedosto. Socket tiedosto luodaan, kun synkronointi on valmis.
{% endhint %}

Kysy mainnet protokollan parametrit.

```text
cardano-cli query protocol-parameters \
    --mainnet \
    --out-file params.json
```

Nouda **stakeAddressDeposit** arvo **params.json**.

{% tabs %}
{% tab title="Core" %}
```bash
stakeAddressDeposit=$(cat $NODE_HOME/params.json | jq -r '.stakeAddressDeposit')
echo stakeAddressDeposit : ${stakeAddressDeposit}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Staking osoitteen rekisteröinti on 2000 000 lovelacea tai 2 adaa.
{% endhint %}

{% hint style="Huomaa" %}
Take note of the invalid-hereafter input. We are taking the current slot number\(tip of the chain\) and adding 1,000 slots. If we do not issue the signed transaction before the chain reaches this slot number the tx will be invalidated. A slot is one second so you have 16.666666667 minutes to get this done. 🐌
{% endhint %}

Build **tx.tmp** file to hold some information.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+0 \
  --invalid-hereafter $(( ${slotNo} + 1000)) \
  --fee 0 \
  --certificate stake.cert \
  --out-file tx.tmp
```
{% endtab %}
{% endtabs %}

Calculate the minimal fee.

{% tabs %}
{% tab title="Core" %}
```bash
fee=$(cardano-cli transaction calculate-min-fee \
  --tx-body-file tx.tmp \
  --tx-in-count ${txcnt} \
  --tx-out-count 1 \
  --mainnet \
  --witness-count 2 \
  --byron-witness-count 0 \
  --protocol-params-file params.json | awk '{ print $1 }')
echo fee: $fee
```
{% endtab %}
{% endtabs %}

Calculate txOut.

{% tabs %}
{% tab title="Core" %}
```bash
txOut=$((${total_balance}-${stakeAddressDeposit}-${fee}))
echo Change Output: ${txOut}
```
{% endtab %}
{% endtabs %}

Build the full transaction to register your staking address.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+${txOut} \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee ${fee} \
  --certificate-file stake.cert \
  --out-file tx.raw
```
{% endtab %}
{% endtabs %}

Transfer **tx.raw** to your Cold offline machine and sign the transaction with the **payment.skey** and **stake.skey**.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file payment.skey \
  --signing-key-file stake.skey \
  --mainnet \
  --out-file tx.signed
```
{% endtab %}
{% endtabs %}

Move **tx.signed** transaction file back to the core nodes pi-pool folder.

Submit the transaction to the blockchain.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction submit \
  --tx-file tx.signed \
  --mainnet
```
{% endtab %}
{% endtabs %}

## Register the pool 🏊

Create a **poolMetaData.json** file. It will contain important information about your pool. You will need to host this file somewhere online forevermore. It must be online and you cannot edit it without resubmitting/updating your pool.cert. In the next couple steps we will hash

{% hint style="Huomaa" %}
metadata-url must be less than 64 characters.
{% endhint %}

{% embed url="https://pages.github.com/" caption="Hosting your poolMetaData.json on github is popular choice" %}

I say host it on your Pi with NGINX.

{% tabs %}
{% tab title="Core" %}
```bash
cd $NODE_HOME
nano poolMetaData.json
```
{% endtab %}
{% endtabs %}

{% hint style="Huomaa" %}
The **extendedPoolMetaData.json** file is used by adapools and others to scrape information like where to find your pool logo and social media links. Unlike the **poolMetaData.json** this files hash is not stored in your registration certificate and can be edited without having to rehash and resubmit **pool.cert**.
{% endhint %}

Add the following and customize to your metadata.

{% tabs %}
{% tab title="Core" %}
```text
{
"name": "Pool Name",
"description": "Pool description, no longer than 255 characters.",
"ticker": "AARCH",
"homepage": "https://example.com/",
"extended": "https://example.com/extendedPoolMetaData.json"
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli stake-pool metadata-hash \
  --pool-metadata-file poolMetaData.json > poolMetaDataHash.txt
```
{% endtab %}
{% endtabs %}

Copy poolMetaData.json to [https://pages.github.io](https://pages.github.io) or host it yourself along with your website.

{% hint style="info" %}
Here is my **poolMetaData.json** & **extendedPoolMetaData.json** as a reference and shameless links back to my site. 😰

[https://adamantium.online/poolMetaData.json](https://adamantium.online/poolMetaData.json)

[https://adamantium.online/extendedPoolMetaData.json​](https://adamantium.online/extendedPoolMetaData.json)
{% endhint %}

{% tabs %}
{% tab title="Core" %}
```bash
minPoolCost=$(cat $NODE_HOME/params.json | jq -r .minPoolCost)
echo minPoolCost: ${minPoolCost}
```
{% endtab %}
{% endtabs %}

Use the format below to register single or multiple relays.

{% tabs %}
{% tab title="DNS Relay\(1\)" %}
```text
--single-host-pool-relay <r1.example.com> \
--pool-relay-port <R1 NODE PORT> \
```
{% endtab %}

{% tab title="IPv4 Relay\(1\)" %}
```text
--pool-relay-ipv4 <RELAY NODE PUBLIC IP> \
--pool-relay-port <R1 NODE PORT> \
```
{% endtab %}

{% tab title="DNS Relay\(2\)" %}
```text
--single-host-pool-relay <r1.example.com> \
--pool-relay-port <R1 NODE PORT> \
--single-host-pool-relay <r2.example.com> \
--pool-relay-port <R2 NODE PORT> \
```
{% endtab %}

{% tab title="IPv4 Relay\(2\)" %}
```text
--pool-relay-ipv4 <R1 NODE PUBLIC IP> \
--pool-relay-port <R1 NODE PORT> \
--pool-relay-ipv4 <R2 NODE PUBLIC IP> \
--pool-relay-port <R2 NODE PORT> \
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Edit the information below to match your pools desired configuration.
{% endhint %}

Issue a stake pool registration certificate.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-pool registration-certificate \
  --cold-verification-key-file $HOME/cold-keys/node.vkey \
  --vrf-verification-key-file vrf.vkey \
  --pool-pledge 10000000000 \
  --pool-cost 340000000 \
  --pool-margin 0.01 \
  --pool-reward-account-verification-key-file stake.vkey \
  --pool-owner-stake-verification-key-file stake.vkey \
  --mainnet \
  --single-host-pool-relay <r1.example.com> \
  --pool-relay-port 3001 \
  --metadata-url <https://example.com/poolMetaData.json> \
  --metadata-hash $(cat poolMetaDataHash.txt) \
  --out-file pool.cert
```
{% endtab %}
{% endtabs %}

Issue a delegation certificate from **stake.skey** & **node.vkey**.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address delegation-certificate \
  --stake-verification-key-file stake.vkey \
  --cold-verification-key-file $HOME/cold-keys/node.vkey \
  --out-file deleg.cert
```
{% endtab %}
{% endtabs %}

Retrieve your stake pool id.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-pool id --cold-verification-key-file $HOME/cold-keys/node.vkey --output-format hex > stakePoolId.txt
cat stakePoolId.txt
```
{% endtab %}
{% endtabs %}

Move **pool.cert**, **deleg.cert** & **stakePoolId.txt** to your online core machine.

Query the current slot number or tip of the chain.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slotNo')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Hae lompakon utxo tai saldo.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --mainnet > fullUtxo.out

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out
cat balance.out
tx_in=""
total_balance=0

while read -r utxo; do
  in_addr=$(awk '{ print $1 }' <<< "${utxo}")
  idx=$(awk '{ print $2 }' <<< "${utxo}")
  utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
  total_balance=$((${total_balance}+${utxo_balance}))
  echo TxHash: ${in_addr}#${idx}
  echo Lovelace: ${utxo_balance}
  tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out
txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: $((${total_balance} / 1000000))
echo Number of UTXOs: ${txcnt}
```
{% endtab %}
{% endtabs %}

Parse **params.json** for stake pool registration deposit value. Spoiler: it's 500 ada but that could change in the future.

{% tabs %}
{% tab title="Core" %}
```bash
stakePoolDeposit=$(cat $NODE_HOME/params.json | jq -r '.stakePoolDeposit')
echo stakePoolDeposit: ${stakePoolDeposit}
```
{% endtab %}
{% endtabs %}

Build temporary **tx.tmp** to hold information while we build our raw transaction file.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+$(( ${total_balance} - ${stakePoolDeposit}))  \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee 0 \
  --certificate-file pool.cert \
  --certificate-file deleg.cert \
  --out-file tx.tmp
```
{% endtab %}
{% endtabs %}

Calculate the transaction fee.

{% tabs %}
{% tab title="Core" %}
```bash
fee=$(cardano-cli transaction calculate-min-fee \
  --tx-body-file tx.tmp \
  --tx-in-count ${txcnt} \
  --tx-out-count 1 \
  --mainnet \
  --witness-count 3 \
  --byron-witness-count 0 \
  --protocol-params-file params.json | awk '{ print $1 }')
  echo fee: ${fee}
```
{% endtab %}
{% endtabs %}

Calculate your change output.

{% tabs %}
{% tab title="Core" %}
```bash
txOut=$((${total_balance}-${stakePoolDeposit}-${fee}))
echo txOut: ${txOut}
```
{% endtab %}
{% endtabs %}

Build your **tx.raw** \(unsigned\) transaction file.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+${txOut} \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee ${fee} \
  --certificate-file pool.cert \
  --certificate-file deleg.cert \
  --out-file tx.raw
```
{% endtab %}
{% endtabs %}

Move **tx.raw** to your cold offline machine.

Sign the transaction with your **payment.skey**, **node.skey** & **stake.skey**.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file payment.skey \
  --signing-key-file $HOME/cold-keys/node.skey \
  --signing-key-file stake.skey \
  --mainnet \
  --out-file tx.signed
```
{% endtab %}
{% endtabs %}

Move **tx.signed** back to your core node & submit the transaction to the blockchain.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction submit \
  --tx-file tx.signed \
  --mainnet
```
{% endtab %}
{% endtabs %}

## Confirm successful registration

### pool.vet

pool.vet is a website for pool operators to check the validity of their stake pools on chain data. You can check this site for problems and clues as to how to fix them.

{% embed url="https://pool.vet/" caption="" %}

### adapools.org

You should create an account and claim your pool here.

{% embed url="https://adapools.org/" caption="" %}

### pooltool.io

You should create an account and claim your pool here.

{% embed url="https://pooltool.io/" caption="" %}

## Backups

Get a couple small usb sticks and backup all your files and folders\(except the db/ folder\). Backup your online Core first then the Cold offline files and folders. **Do it now**, not worth the risk! **Do not plug the USB stick into anything online after Cold files are on it!**

![https://twitter.com/insaladaPool/status/1380087586509709312?s=19](../../.gitbook/assets/insalada%20%281%29.png)

