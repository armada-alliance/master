# KES-avainten uusiminen

{% hint style="info" %}
On parasta uudelleennimetä vanha **kes.vkey**, **kes.skey** & **node.cert** tiedostot etukäteen. Lisää päivämäärä. Minulla on tapana käyttää mv-komentoa cp-komennon sijaan. Näin en ala luoda kopioita tiedostoista.
{% endhint %}

{% hint style="Huomaa" %}
Tarvitset vain **kes.skey**, **node.cert** ja **vrf.skey** Core nodellasi.
{% endhint %}

Määritä KES-aika hakemalla nykyisen slotin numero ja jakamalla se KES-jakson slottimäärällä, joka löytyy genesis tiedostosta.

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

Luo uusi KES avainpari.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-KES \
  --verification-key-file kes.vkey \
  --signing-key-file kes.skey
```
{% endtab %}
{% endtabs %}

Siirrä **kes.vkey** **Kylmään Offline** koneeseen & myönnä uusi node.cert.

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

{% hint style="Huomaa" %}
Cold.counter tiedosto pitää kirjaa kuinka monta kertaa olet kierrättänyt Kes avainparisi.
{% endhint %}

Siirrä **node.cert** takaisin Coreen & käynnistä cardano-palvelu.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service restart
```
{% endtab %}
{% endtabs %}

