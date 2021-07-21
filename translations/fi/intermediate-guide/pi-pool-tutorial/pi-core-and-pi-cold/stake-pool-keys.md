# Stake pool avaimet

## Raspberry Pi & entropia

Ennen kuin alamme tuottaa avaimia palvelimella meidän pitää luoda turvallinen määrä entropy.

{% hint style="info" %}
[https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/](https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/)

[https://github.com/nhorman/rng-tools](https://github.com/nhorman/rng-tools)
{% endhint %}

> Mietitäänpä yksittäisen, "headless" palvelimen \(tai mikro-ohjaimenkin\) kohtaloa, ilman ihmisen syöttämiä kirjoituksia tai hiiren liikkeitä, eikä kehräävää asemaa tarjoamassa mekaanista epäsäännöllisyyttä. Mistä _se_ saa entropiaa käynnistyttyään? Entä jos hyökkääjä tai huono onni, pakottaa säännöllisiä uudelleenkäynnistyksiä? Tämä on [todellinen ongelma](http://www.theregister.co.uk/2015/12/02/raspberry_pi_weak_ssh_keys/).

```bash
## install rng-tools on both core & cold
sudo apt-get install rng-tools
sudo reboot
```

{% hint style="warning" %}
Lohkontuottaja node vaatii vain nämä kolme tiedostoa, jotka on määritelty [Shelley ledgerin ominaisuuksissa](https://hydra.iohk.io/build/2473732/download/1/ledger-spec.pdf):

1. stake pool kylmä avain \(node.cert\)
2. stake pool kuuma avain \(kes.skey\)
3. stake pool VRF avain \(vrf.skey\)
{% endhint %}

Luo KES avainpari.

{% hint style="info" %}
KES \(key evolving signature\) avaimet on luotu suojaamaan stake pooliasi hakkereita vastaan, jotka saattaisivat vaarantaa avaintesi turvallisuuden.

**Pääverkossa (mainnet), KES avaimet täytyy uudistaa 90 päivän välein.**
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

Tee offline-koneeseen hakemisto, johon voit tallentaa kylmät avaimet.

{% tabs %}
{% tab title="Offline Cold" %}
```bash
mkdir $HOME/cold-keys
cd $HOME/cold-keys
```
{% endtab %}
{% endtabs %}

Tee kylmä avainpari ja luo kylmä laskuri-tiedosto.

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
\*\*\*\*🔥 **Kylmät avaimet** **on tuotettava ja tallennettava ilma-sillatussa offline-tilassa.** Kylmät avaintiedostot, tallennetaan hakemistoon `$HOME/cold-keys.`
{% endhint %}

Määritä slottien lukumäärä KES-jaksoa kohti genesis-tiedostosta.

{% tabs %}
{% tab title="Core" %}
```bash
slotsPerKESPeriod=$(cat $NODE_FILES/${NODE_CONFIG}-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
echo slotsPerKESPeriod: ${slotsPerKESPeriod}
```
{% endtab %}
{% endtabs %}

Selvitä ketjun nykyisen slotin numero tai ketjun 'kärki'.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slotNo')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Selvitä kesPeriod jakamalla ketjun kärki slotsPerKESPeriod arvolla.

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

Siirrä **kes.vkey** **kylmään ympäristöön**.

{% hint style="warning" %}
Korvaa &lt;startKesPeriod&gt; alapuolella vastaamaan määrittlemääsi arvoa.
{% endhint %}

Luo toiminnallinen sertifikaatti \(node.crt\).

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

Siirrä **node.cert** **kuumaan ympäristöön**.

Luo tarkistus\(VRF\) avainpari.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-VRF \
    --verification-key-file vrf.vkey \
    --signing-key-file vrf.skey
```
{% endtab %}
{% endtabs %}

Päivitä VRF avainten oikeudet arvoon "vain luku".

```bash
chmod 400 vrf.skey
```

Pysäytä cardano-node.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service stop
```
{% endtab %}
{% endtabs %}

Päivitä cardano-service startup skripti sisältämään KES, VRF ja käyttösertifikaatin.

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

Päivitä blockproducer noden mainnet-topology.json -tiedosto.

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

Käynnistä lohkon tuottaja node uudelleen ydinpalvelimena.

```bash
cardano-service start
```

{% hint style="warning" %}
Odota, että lohkon tuottaja node synkronoituu uudelleen ketjun kärkeen saakka.

gLiveView.sh
{% endhint %}

