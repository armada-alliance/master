---
description: Create the stake pools wallet and and staking key.
---

# Pool wallet & staking keys

{% hint style="danger" %}
not done yet sorry.
{% endhint %}

{% hint style="warning" %}
It is very convenient to create your wallet keys from a mnemonic seed. This will allow you to manage the pools wallet from Yoroi or Daedalus & easily restore the pools wallet into any compatible wallet and manage rewards.

cardano-wallet will not build on arm. Below is how to do it with an x86 machine running cardano-wallet and then transfer the keys to your cold offline machine.

It is recommended to download the cardano-wallet binary, check it's sha256 hash & then go offline until keys are safely on the cold machine.
{% endhint %}

## Generate

{% hint style="danger" %}
🔥 **Critical Operational Security Advice:** `payment` and `stake` keys must be generated and used to build transactions in cold environment. In other words, your **air-gapped offline Cold machine**. The only steps performed online in a hot environment are those steps that require live data. Namely the follow type of steps:

-   querying the current slot tip
-   querying the balance of an address
-   submitting a transaction
    {% endhint %}

Create a 15-word or 24-word length Shelley compatible mnemonic with [Daedalus](https://daedaluswallet.io/) or [Yoroi](https://yoroi-wallet.com) on a offline machine preferred.

### Retrieve x86 cardano-wallet binary

{% hint style="info" %}
If you are not using Linux on your local machine you can write Ubuntu to a usb stick and boot from it.
{% endhint %}

Download cardano-wallet to your local machine.

<https://github.com/input-output-hk/cardano-wallet/releases>

{% hint style="info" %}
Credits to [ilap](https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894) & [Coin Cashew](https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node#10-setup-payment-and-stake-keys) for creating this process.
{% endhint %}

Create the script.

```bash
nano extractPoolStakingKeys.sh
```

```bash
#!/bin/bash

CADDR=\${CADDR:=\$( which cardano-address )}
[[ -z "\$CADDR" ]] && ( echo "cardano-address cannot be found, exiting..." >&2 ; exit 127 )

CCLI=\${CCLI:=\$( which cardano-cli )}
[[ -z "\$CCLI" ]] && ( echo "cardano-cli cannot be found, exiting..." >&2 ; exit 127 )

OUT_DIR="\$1"
[[ -e "\$OUT_DIR"  ]] && {
           echo "The \"\$OUT_DIR\" is already exist delete and run again." >&2
           exit 127
} || mkdir -p "\$OUT_DIR" && pushd "\$OUT_DIR" >/dev/null

shift
MNEMONIC="\$*"

# Generate the master key from mnemonics and derive the stake account keys
# as extended private and public keys (xpub, xprv)
echo "\$MNEMONIC" |\
"\$CADDR" key from-recovery-phrase Shelley > root.prv

cat root.prv |\
"\$CADDR" key child 1852H/1815H/0H/2/0 > stake.xprv

cat root.prv |\
"\$CADDR" key child 1852H/1815H/0H/0/0 > payment.xprv

TESTNET=0
MAINNET=1
NETWORK=\$MAINNET

cat payment.xprv |\
"\$CADDR" key public | tee payment.xpub |\
"\$CADDR" address payment --network-tag \$NETWORK |\
"\$CADDR" address delegation \$(cat stake.xprv | "\$CADDR" key public | tee stake.xpub) |\
tee base.addr_candidate |\
"\$CADDR" address inspect
echo "Generated from 1852H/1815H/0H/{0,2}/0"
cat base.addr_candidate
echo

# XPrv/XPub conversion to normal private and public key, keep in mind the
# keypars are not a valind Ed25519 signing keypairs.
TESTNET_MAGIC="--testnet-magic 42"
MAINNET_MAGIC="--mainnet"
MAGIC="\$MAINNET_MAGIC"

SESKEY=\$( cat stake.xprv | bech32 | cut -b -128 )\$( cat stake.xpub | bech32)
PESKEY=\$( cat payment.xprv | bech32 | cut -b -128 )\$( cat payment.xpub | bech32)

cat << EOF > stake.skey
{
    "type": "StakeExtendedSigningKeyShelley_ed25519_bip32",
    "description": "",
    "cborHex": "5880\$SESKEY"
}
EOF

cat << EOF > payment.skey
{
    "type": "PaymentExtendedSigningKeyShelley_ed25519_bip32",
    "description": "Payment Signing Key",
    "cborHex": "5880\$PESKEY"
}
EOF

"\$CCLI" shelley key verification-key --signing-key-file stake.skey --verification-key-file stake.evkey
"\$CCLI" shelley key verification-key --signing-key-file payment.skey --verification-key-file payment.evkey

"\$CCLI" shelley key non-extended-key --extended-verification-key-file payment.evkey --verification-key-file payment.vkey
"\$CCLI" shelley key non-extended-key --extended-verification-key-file stake.evkey --verification-key-file stake.vkey


"\$CCLI" shelley stake-address build --stake-verification-key-file stake.vkey \$MAGIC > stake.addr
"\$CCLI" shelley address build --payment-verification-key-file payment.vkey \$MAGIC > payment.addr
"\$CCLI" shelley address build \
    --payment-verification-key-file payment.vkey \
    --stake-verification-key-file stake.vkey \
    \$MAGIC > base.addr

echo "Important the base.addr and the base.addr_candidate must be the same"
diff base.addr base.addr_candidate
popd >/dev/null
```

```bash
###
### On air-gapped offline machine,
###
chmod +x extractPoolStakingKeys.sh
export PATH="$(pwd)/cardano-wallet-shelley-2020.7.28:$PATH"
```

Extract your keys. Update the command with your mnemonic phrase.

```bash
###
### On air-gapped offline machine,
###
./extractPoolStakingKeys.sh extractedPoolKeys/ <15|24-word length mnemonic>
```

{% hint style="danger" %}
**Important**: The **base.addr** and the **base.addr_candidate** must be the same. Review the screen output.
{% endhint %}
