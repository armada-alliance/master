---
description: Luo Stake Pool lompakko ja Staking avain.
---

# Pool lompakko & staking avaimet

{% hint style="danger" %}
Tämä osio ei ole vielä valmis, pahoittelumme.
{% endhint %}

{% hint style="Huomaa" %}
On erittäin kätevää luoda lompakon avaimet mnemonic seed:istä. Näin voit hallita poolin lompakkoa Yoroin tai Daedaluksen avulla ja helposti palauttaa poolin lompakon mihin tahansa yhteensopivaan lompakkoon sekä hallinnoida palkkioita.

cardano-lompakkoa ei voi rakentaa ARM laitteeseen. Alla on ohjeistus miten se tehdään x86 koneessa jossa on cardano-lompakko ja sitten siirretään avaimet kylmään offline kone.

On suositeltavaa ladata cardano-lompakko binaari, tarkista sen sha256 hash ja siirry offline tilaan kunnes avaimet ovat turvallisesti kylmässä koneessa.
{% endhint %}

## Luo

{% hint style="Huomaa" %}
🔥 **Kriittinen turvallisuusneuvo:** `payment` ja `stake` avaimet on tuotettava ja niitä on käytettävä tapahtumien rakentamiseen kylmässä ympäristössä. Toisin sanoen, vain **ilma-sillatussa offline kylmässä koneessasi**. Ainoat vaiheet jotka toteutetaan verkossa kuumassa ympäristössä ovat ne jotka vaativat reaaaliaikaisia live-tietoja. Näitä ovat seuraavat vaiheet:

-   lohkoketjun kärjen kysely
-   osoitteen saldoa koskeva pyyntö
-   siirtotapahtuman lähettäminen
    {% endhint %}

Luo 15-sanan tai 24-sanan pituinen Shelley yhteensopiva mnemonic [Daedalus](https://daedaluswallet.io/) tai [Yoroi](https://yoroi-wallet.com) mieluiten offline-koneella.

### Nouda x86 Cardano-lompakko binääri

{% hint style="info" %}
Jos et käytä Linuxia paikallisessa koneessasi, voit asentaa Ubuntun usb-tikkulle ja käynnistää koneen uudelleen siitä.
{% endhint %}

Lataa cardano-lompakko paikalliseen koneeseen.

<https://github.com/input-output-hk/cardano-wallet/releases>

{% hint style="info" %}
Kiitokset [ilap:lle](https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894) & [Coin Cashew:lle](https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node#10-setup-payment-and-stake-keys) tämän prosessin luomisesta.
{% endhint %}

Luo skripti.

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

Pura avaimesi. Päivitä komento mnemonisen fraasin avulla.

```bash
###
### On air-gapped offline machine,
###
./extractPoolStakingKeys.sh extractedPoolKeys/ <15|24-word length mnemonic>
```

{% hint style="danger" %}
**Tärkeää**: **base.addr** ja **base.addr_candidate** täytyy olla sama. Tarkastele tuotosta näytöllä.
{% endhint %}
