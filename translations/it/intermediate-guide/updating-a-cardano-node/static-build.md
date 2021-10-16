# Static Build

{% hint style="info" %}
This guide follows the same setup as our [Pi-Node guide and image](../pi-pool-tutorial/) so you may need to adjust sections based on your node's environment and setup.
{% endhint %}

{% hint style="success" %}
### Current Official Cardano Node Version: 1.30.1
{% endhint %}

### Overview 🗒

* [ ] Download Cardano Node Static build & configuration file
* [ ] Extract the file's content
* [ ] Check if you already have Cardano Node service running
  * Safely shutdown your Cardano node if it is running
* [ ] Replace the old binaries with the new cardano-node and cardano-cli
* [ ] Check cardano-node and cli version is updated to the current version
* [ ] Replace old configuration files with new ones (if needed)
* [ ] Restart your Cardano Node
* [ ] Check that node has started properly

## Download the cardano-node & cli

### Static binaries and Cardano node configuration files are provided by [\[ZW3RK\]](https://armada-alliance.com/identities/zw3rk) pool🙏 and can be found at our [Github repository](https://github.com/armada-alliance/cardano-node-binaries/tree/main/static-binaries).

```bash
wget https://github.com/armada-alliance/cardano-node-binaries/raw/main/static-binaries/1_30_1.zip
```

Extract the content from the zip file.

```bash
unzip 1_30_1.zip
```

### Check if cardano-node is running already

{% hint style="warning" %}
**Now we need to make sure we do not have a cardano-node already running. If we do we must shut it down before proceeding.**
{% endhint %}

You can check if you have a cardano-node process already running a few ways like using`htop` or by checking your systemd service.

If you have been following our [Pi-Node guide](../pi-pool-tutorial/) you can check your cardano-node status and stop it using the following commands.

```bash
cardano-service status
cardano-service stop
```

{% hint style="info" %}
If you use Linux's `htop` service just check for a processing running starting with `cardano-node run` and use `SIGINT` to terminate the process.
{% endhint %}

## Replace the old binaries and config files with the new ones

If you are using the [Pi-Node guide](../pi-pool-tutorial/) and your cardano-node & cli in `~/.local/bin`

```bash
mv cardano-node/* ~/.local/bin
```

### Check your cardano-node version

```bash
cardano-node --version
```

#### Output:

```bash
cardano-node 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

### Check your cardano-cli version

```bash
cardano-cli --version
```

#### Output:

```bash
cardano-cli 1.30.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

### Download & Replace the Cardano node configuration files

{% tabs %}
{% tab title="Mainnet" %}
```bash
cd ~/pi-pool/files
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-config.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/mainnet-topology.json
```
{% endtab %}

{% tab title="Testnet" %}
```bash
cd ~/pi-pool/files
wget https://hydra.iohk.io/build/7370192/download/1/testnet-config.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/testnet-topology.json
```
{% endtab %}

{% tab title="Alonzo-Purple" %}
```bash
cd ~/pi-pool/files
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-config.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-byron-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-shelley-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-alonzo-genesis.json
wget https://hydra.iohk.io/build/7370192/download/1/alonzo-purple-topology.json
```
{% endtab %}
{% endtabs %}

## Restart the Cardano Node

Now we just need to restart our cardano-node service if you are using our [Pi-Node guide](../pi-pool-tutorial/) use this command

```bash
cardano-service start
```

Wait a few seconds or so then check the status

```bash
cardano-service status
```
