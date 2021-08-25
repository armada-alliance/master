# Stake pool keys

## Raspberry Pi & entropy

Before we start generating keys with a headless server we should create a safe amount of entropy.

{% hint style="info" %}
[https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/](https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/)

[https://github.com/nhorman/rng-tools](https://github.com/nhorman/rng-tools)
{% endhint %}

> But consider the fate of a standalone, headless server \(or a microcontroller for that matter\) with no human typing or mousing around, and no spinning iron drive providing mechanical irregularity. Where does _it_ get entropy after it starts up? What if an attacker, or bad luck, forces periodic reboots? This is a [real problem](http://www.theregister.co.uk/2015/12/02/raspberry_pi_weak_ssh_keys/).

```bash
## install rng-tools on both core & cold
sudo apt-get install rng-tools
sudo reboot
```

{% hint style="warning" %}
The block-producer node requires only these 3 files as defined in the [Shelley ledger specs](https://hydra.iohk.io/build/2473732/download/1/ledger-spec.pdf):

1. stake pool cold key \(node.cert\)
2. stake pool hot key \(kes.skey\)
3. stake pool VRF key \(vrf.skey\)
{% endhint %}

Generate a KES keypair.

{% hint style="info" %}
KES \(key evolving signature\) keys are created to secure your stake pool against hackers who might compromise your keys.

**On mainnet, you will need to regenerate the KES key every 90 days.**
{% endhint %}

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

Make a directory to store your cold keys in your offline machine.

{% tabs %}
{% tab title="Offline Cold" %}
```bash
mkdir $HOME/cold-keys
cd $HOME/cold-keys
```
{% endtab %}
{% endtabs %}

Make a set of cold keys and create the cold counter file.

{% tabs %}
{% tab title=" Offline Cold" %}
```bash
cardano-cli node key-gen \
    --cold-verification-key-file node.vkey \
    --cold-signing-key-file node.skey \
    --operational-certificate-issue-counter node.counter
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
\*\*\*\*🔥 **Cold keys** **must be generated and stored on your air-gapped offline machine.** The cold keys are the files stored in `$HOME/cold-keys.`
{% endhint %}

Determine the number of slots per KES period from the genesis file.

{% tabs %}
{% tab title="Core" %}
```bash
slotsPerKESPeriod=$(cat $NODE_FILES/${NODE_CONFIG}-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
echo slotsPerKESPeriod: ${slotsPerKESPeriod}
```
{% endtab %}
{% endtabs %}

Query chain to determine current slot number or 'tip' of the chain.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slotNo')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Find the kesPeriod by dividing the slot tip by the slotsPerKESPeriod.

{% tabs %}
{% tab title="Core" %}
```bash
kesPeriod=$((${slotNo} / ${slotsPerKESPeriod}))
echo kesPeriod: ${kesPeriod}
startKesPeriod=${kesPeriod}
echo startKesPeriod: ${startKesPeriod}
```
{% endtab %}
{% endtabs %}

Move **kes.vkey** to your **cold environment**.

{% hint style="warning" %}
Replace &lt;startKesPeriod&gt; below to match.
{% endhint %}

Create your operational certificate \(node.crt\).

{% tabs %}
{% tab title="Offline Cold" %}
```bash
cardano-cli node issue-op-cert \
    --kes-verification-key-file kes.vkey \
    --cold-signing-key-file $HOME/cold-keys/node.skey \
    --operational-certificate-issue-counter $HOME/cold-keys/node.counter \
    --kes-period <startKesPeriod> \
    --out-file node.cert
```
{% endtab %}
{% endtabs %}

Move **node.cert** to your **hot environment**.

Generate the verification \(VRF\) key pair.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-VRF \
    --verification-key-file vrf.vkey \
    --signing-key-file vrf.skey
```
{% endtab %}
{% endtabs %}

Update VRF key permission to read-only.

```bash
chmod 400 vrf.skey
```

Stop cardano-node.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service stop
```
{% endtab %}
{% endtabs %}

Update the cardano-service startup script to include your KES, VRF and operational certificate.

```bash
nano $HOME/.local/bin/cardano-service
```

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
  --port ${PORT} \
  --config ${CONFIG} \
  --shelley-kes-key ${KES} \
  --shelley-vrf-key ${VRF} \
  --shelley-operational-certificate ${CERT}
```
{% endtab %}
{% endtabs %}

Update your cores mainnet-topology.json file.

```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```

```bash
 {
    "Producers": [
      {
        "addr": "<relays private ip address>",
        "port": 3002,
        "valency": 1
      }
    ]
  }
```

Restart your block producer as a core node.

```bash
cardano-service start
```

{% hint style="warning" %}
Wait for the core node to sync back up to the tip of the chain.

gLiveView.sh
{% endhint %}

