# Rotate KES

{% hint style="info" %}
It is best to rename the old **kes.vkey**, **kes.skey** & **node.cert** files beforehand. Append the date. I tend to use mv instead of cp. This way I do not start creating copies of files.
{% endhint %}

{% hint style="warning" %}
You only need **kes.skey**, **node.cert** and **vrf.skey** on your Core node.
{% endhint %}

Determine KES period by querying current slot number divided by slots per KES period found in genesis file.

{% tabs %}
{% tab title="Core" %}
```bash
cd $NODE_HOME
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slot')
slotsPerKESPeriod=$(cat $NODE_FILES/mainnet-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
kesPeriod=$((${slotNo} / ${slotsPerKESPeriod}))
startKesPeriod=${kesPeriod}
echo startKesPeriod: ${startKesPeriod}
```
{% endtab %}
{% endtabs %}

Generate a new KES key pair.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-KES \
  --verification-key-file kes.vkey \
  --signing-key-file kes.skey
```
{% endtab %}
{% endtabs %}

Move **kes.vkey** to your **Cold Offline** machine & issue a new node.cert.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cd $NODE_HOME
chmod u+rwx $HOME/cold-keys
cardano-cli node issue-op-cert \
  --kes-verification-key-file kes.vkey \
  --cold-signing-key-file $HOME/cold-keys/node.skey \
  --operational-certificate-issue-counter $HOME/cold-keys/node.counter \
  --kes-period <startKesPeriod> \
  --out-file node.cert
chmod a-rwx $HOME/cold-keys
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The cold.counter in your cold-keys folder keeps track of how many times you have rotated your kes pair.
{% endhint %}

Move **node.cert** back to Core & restart the cardano-service.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service restart
```
{% endtab %}
{% endtabs %}

