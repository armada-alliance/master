---
description: Cree la billetera del Stake Pool y delegar las claves.
---

# Billetera del Pool y Delegar Claves

{% hint style="danger" %}
no hecho todavía lo sentimos.
{% endhint %}

{% hint style="warning" %}
Es muy conveniente crear las claves de tu billetera desde unas semillas mnemotécnicas. Esto te permitirá administrar la billetera del pool desde Yoroi o Daedalus & fácilmente restaurando la billetera del pool en cualquier billetera compatible y administrar tus recompensas.

cardano-wallet no se construirá sobre arm. A continuación se muestra cómo hacerlo con una máquina x86 ejecutando cardano-wallet y luego transfiere las claves a su máquina fría fuera de línea.

Se recomienda descargar el binario de la billetera de cardano, comprueba su hash sha256 & luego desconecta hasta que las llaves estén seguras en la máquina en frío.
{% endhint %}

## Generar

{% hint style="danger" %}
🔥 **Consejo crítico de seguridad operativa:** `pago` y `apuesta` claves deben ser generadas y utilizadas para construir transacciones en entorno frío. En otras palabras, tu **máquina fría sin conexión** totalmente desconectada. Los únicos pasos realizados en línea en un ambiente caliente son aquellos pasos que requieren datos en vivo. A saber, el siguiente tipo de pasos:

-   consultar el final del slot actual
-   consultar el saldo de una dirección
-   enviar una transacción
    {% endhint %}

Crea 15 o 24 palabras mnemótécnicas compatibles con Shelley de [Daedalus](https://daedaluswallet.io/) o [Yoroi](https://yoroi-wallet.com) en una máquina sin conexión preferentemente.

### Recuperar binario cardano-wallet x86

{% hint style="info" %}
If you are not using Linux on your local machine you can write Ubuntu to a usb stick and boot from it.
{% endhint %}

Descargue cardano-wallet a tu máquina local.

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
