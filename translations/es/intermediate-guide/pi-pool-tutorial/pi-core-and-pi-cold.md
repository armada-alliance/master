---
description: Create operational keys & certificates. Create wallet & register stake pool
---

# Pi-Core/Frío

{% hint style="danger" %}
You need to have a Pi-Node configured with a new static ip address on your LAN. A fully qualified domain name and cardano-service file set to start on port 3000. You also need to update the env file used by gLiveView.sh located in $NODE_HOME/scripts.

You do not enable the topology updater service on a core node so feel free to delete those two scripts and remove the commented out cron job.

Make sure your core node is synced to the tip of the blockchain.
{% endhint %}

{% hint style="warning" %}
There exists a way to create your pool wallets **payment keypair** by creating a wallet in Yoroi and using cardano-wallet to extract the key pair from the mnemonic seed from Yoroi. This allows you to have a seed backup of the wallet and to also easily extract rewards or send funds elsewhere. You can do this with any shelley era mnemonic seed. I prefer Yoroi because it is quick.

[https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894​](https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894)

Cardano-wallet will not build on arm due to dependency failure. @ZW3RK tried to build it for us and it would not. You may want to install cardano-wallet on an offline x86 machine and go through this process. That is how I did it. You can get cardano-wallet binary below.

[https://hydra.iohk.io/build/3770189](https://hydra.iohk.io/build/3770189)
{% endhint %}

### Enable blockfetch tracing

```
sed -i ${NODE_FILES}/${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g"
```

## Generate Keys & Issue Operational Certificate

{% hint style="warning" %}
#### Rotating the KES keys

KES keys need to be regenerated and a new **pool.cert** needs to be issued and submitted to the chain every 90 days. The **node.counter** file keeps track of how many times this has been done.
{% endhint %}

Generate a KES key pair: **kes.vkey** & **kes.skey**

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

Generate a node cold key pair: **node.vkey**, **node.skey** and **node.counter** file on your Cold Offline machine.

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

Create variables with the number of slots per KES period from the genesis file and current tip of the chain.

{% tabs %}
{% tab title="Core" %}
```bash
slotsPerKesPeriod=$(cat $NODE_FILES/${NODE_CONFIG}-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
slotNo=$(cardano-cli query tip --${NODE_CONFIG} | jq -r '.slot')
echo slotsPerKesPeriod: ${slotsPerKesPeriod}
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Set the **startKesPeriod** variable by dividing **slotNo** / **slotsPerKESPeriod**.

{% tabs %}
{% tab title="Core" %}
```bash
startKesPeriod=$((${slotNo} / ${slotsPerKesPeriod}))
echo startKesPeriod: ${startKesPeriod}
```
{% endtab %}
{% endtabs %}

Write **startKesPeriod** value down & copy the **kes.vkey** to your cold offline machine.

Issue a **node.cert** certificate using: **kes.vkey**, **node.skey**, **node.counter** and **startKesPeriod** value.

Replace **\<startKesPeriod>** with the value you wrote down.

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

Copy **node.cert** to your Core machine.

Generate a VRF key pair.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-VRF \
  --verification-key-file vrf.vkey \
  --signing-key-file vrf.skey
```
{% endtab %}
{% endtabs %}

For security purposes the **vrf.skey** **needs** read only permissions or cardano-node will not start.

{% tabs %}
{% tab title="Core" %}
```bash
chmod 400 vrf.skey
```
{% endtab %}
{% endtabs %}

Edit the cardano-service startup script by adding **kes.skey**, **vrf.skey** and **node.cert** to the cardano-node run command and changing the port it listens on.

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
TOPOLOGY=${FILES}/${NODE_CONFIG}-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/${NODE_CONFIG}-config.json
KES=${DIRECTORY}/kes.skey
VRF=${DIRECTORY}/vrf.skey
CERT=${DIRECTORY}/node.cert
## +RTS -N4 -RTS = Multicore(4)
cardano-node run +RTS -N4 -RTS \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --port ${PORT} \
  --config ${CONFIG} \
  --shelley-kes-key ${KES} \
  --shelley-vrf-key ${VRF} \
  --shelley-operational-certificate ${CERT}
```
{% endtab %}
{% endtabs %}

Add your relay(s) to ${NODE_CONFIG}-topolgy.json.

{% tabs %}
{% tab title="Core" %}
```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```
{% endtab %}
{% endtabs %}

Use your LAN IPv4 for addr field if you are not using domain DNS. Be sure to have proper records set with your registrar or DNS service. Below are some examples.

Valency greater than one is only used with DNS round robin srv records.

{% tabs %}
{% tab title="1 Relay DNS" %}
```
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
```
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
```
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
```
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

Restart and your node is now running as a core.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service restart
```
{% endtab %}
{% endtabs %}

## Create the pool wallet payment & staking key pairs

{% hint style="danger" %}
**Cold offlline machine.** Take the time to visualize the operations here.

1. _**Generate**_ a wallet key pair named payment. = **payment.vkey** & **payment.skey**
2. _**Generate**_ staking key pair. = **stake.vkey** & **stake.skey**
3. _**Build**_ a stake address from the newly created **stake.vkey**. = **stake.addr**
4. _**Build**_ a wallet address from the **payment.vkey** & delegate with **stake.vkey**. = **payment.addr**
5. Add funds to the wallet by sending ada to **payment.addr**
6. Check balance.
{% endhint %}

### 1. Generate wallet key pair

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

### 2. Generate staking key pair

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address key-gen \
  --verification-key-file stake.vkey \
  --signing-key-file stake.skey
```
{% endtab %}
{% endtabs %}

### 3. Build stake address

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address build \
  --stake-verification-key-file stake.vkey \
  --out-file stake.addr \
  --${NODE_CONFIG}
```
{% endtab %}
{% endtabs %}

### 4. Build payment address

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli address build \
  --payment-verification-key-file payment.vkey \
  --stake-verification-key-file stake.vkey \
  --out-file payment.addr \
  --${NODE_CONFIG}
```
{% endtab %}
{% endtabs %}

### 5. Fund wallet

```
cat payment.addr
```

Copy **payment.addr** to a thumb drive and move it to the core nodes pi-pool folder.

Add funds to the wallet. This is the only wallet the pool uses so your pledge goes here as well. There is a 2 ada staking registration fee and a 500 ada pool registration deposit that you can get back when retiring your pool.

{% hint style="warning" %}
Test the wallet by sending a small amount waiting a few minutes and querying it's balance.
{% endhint %}

{% hint style="danger" %}
Core node needs to be synced to the tip of the blockchain.
{% endhint %}

### 6. Check balance

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --${NODE_CONFIG}
```
{% endtab %}
{% endtabs %}

## Register stake address

Issue a staking registration certificate: **stake.cert**

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address registration-certificate \
  --stake-verification-key-file stake.vkey \
  --out-file stake.cert
```
{% endtab %}
{% endtabs %}

Copy **stake.cert** to your core node's pi-pool folder.

Query current slot number or tip of the chain.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --${NODE_CONFIG} | jq -r '.slot')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Get the utxo or balance of the wallet.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --${NODE_CONFIG} > fullUtxo.out

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
If you get

`cardano-cli: Network. Socket.connect: : does not exist (No such file or directory)`

It is because the core has not finished syncing to the tip of the blockchain. This can take a long time after a reboot. If you look in the db/ folder after cardano-service stop you will see a file named 'clean'. That is confirmation file of a clean database shutdown. It usually takes 5 to 10 minutes to sync back to the tip of the chain on Raspberry Pi as of epoch 267.

If however the cardano-node does not shutdown 'cleanly' for whatever reason it can take up to an hour to verify the database(chain) and create the socket file. Socket file is created once your synced.
{% endhint %}

Query ${NODE_CONFIG} for protocol parameters.

```
cardano-cli query protocol-parameters \
    --${NODE_CONFIG} \
    --out-file params.json
```

Retrieve **stakeAddressDeposit** value from **params.json**.

{% tabs %}
{% tab title="Core" %}
```bash
stakeAddressDeposit=$(cat $NODE_HOME/params.json | jq -r '.stakeAddressDeposit')
echo stakeAddressDeposit : ${stakeAddressDeposit}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Stake address registration is 2,000,000 lovelace or 2 ada.
{% endhint %}

{% hint style="warning" %}
Take note of the invalid-hereafter input. We are taking the current slot number(tip of the chain) and adding 1,000 slots. If we do not issue the signed transaction before the chain reaches this slot number the tx will be invalidated. A slot is one second so you have 16.666666667 minutes to get this done. 🐌
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
  --${NODE_CONFIG} \
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
  --${NODE_CONFIG} \
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
  --${NODE_CONFIG}
```
{% endtab %}
{% endtabs %}

## Register the pool 🏊

Create a **poolMetaData.json** file. It will contain important information about your pool. You will need to host this file somewhere online forevermore. It must be online and you cannot edit it without resubmitting/updating your pool.cert. In the next couple steps we will hash

{% hint style="warning" %}
metadata-url must be less than 64 characters.
{% endhint %}

{% embed url="https://pages.github.com/" %}
Hosting your poolMetaData.json on github is popular choice
{% endembed %}

I say host it on your Pi with NGINX.

{% tabs %}
{% tab title="Core" %}
```bash
cd $NODE_HOME
nano poolMetaData.json
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The **extendedPoolMetaData.json** file is used by adapools and others to scrape information like where to find your pool logo and social media links. Unlike the **poolMetaData.json** this files hash is not stored in your registration certificate and can be edited without having to rehash and resubmit **pool.cert**.
{% endhint %}

Add the following and customize to your metadata.

{% tabs %}
{% tab title="Core" %}
```
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

Copy poolMetaData.json to [https://pages.github.io](https://pages.github.io) or host it yourself along with your website. Be careful not to accidentally insert a space or a new line, which would result in a different hash.

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
{% tab title="DNS Relay(1)" %}
```
--single-host-pool-relay <r1.example.com> \
--pool-relay-port <R1 NODE PORT> \
```
{% endtab %}

{% tab title="IPv4 Relay(1)" %}
```
--pool-relay-ipv4 <RELAY NODE PUBLIC IP> \
--pool-relay-port <R1 NODE PORT> \
```
{% endtab %}

{% tab title="DNS Relay(2)" %}
```
--single-host-pool-relay <r1.example.com> \
--pool-relay-port <R1 NODE PORT> \
--single-host-pool-relay <r2.example.com> \
--pool-relay-port <R2 NODE PORT> \
```
{% endtab %}

{% tab title="IPv4 Relay(2)" %}
```
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

Copy vrf.vkey and poolMetaDataHash.txt to your cold machine and issue a stake pool registration certificate.

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
  --${NODE_CONFIG} \
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
slotNo=$(cardano-cli query tip --${NODE_CONFIG} | jq -r '.slot')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Get the utxo or balance of the wallet.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --${NODE_CONFIG} > fullUtxo.out

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
  --${NODE_CONFIG} \
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

Build your **tx.raw** (unsigned) transaction file.

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
  --${NODE_CONFIG} \
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
  --${NODE_CONFIG}
```
{% endtab %}
{% endtabs %}

## Confirm successful registration

### pool.vet

pool.vet is a website for pool operators to check the validity of their stake pools on chain data. You can check this site for problems and clues as to how to fix them.

{% embed url="https://pool.vet/" %}

### adapools.org

You should create an account and claim your pool here.

{% embed url="https://adapools.org/" %}

### pooltool.io

You should create an account and claim your pool here.

{% embed url="https://pooltool.io/" %}

## Backups

Get a couple small usb sticks and backup all your files and folders(except the db/ folder). Backup your online Core first then the Cold offline files and folders. **Do it now**, not worth the risk! **Do not plug the USB stick into anything online after Cold files are on it!**

![https://twitter.com/insaladaPool/status/1380087586509709312?s=19](../../../.gitbook/assets/insalada (2).png)
