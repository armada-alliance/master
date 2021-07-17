---
description: Miten lisätä adapools.org summary.json tiedot Grafana tapahtumaksi.
---

# Lisää adapoolien mittareita Grafanaan

## Oletukset

Olet määrittänyt solmun käyttäen yhtä täällä esitetyistä opetusohjelmista. Jos näin on, sinulla pitäisi olla tarvittavat riippuvuudet asennettuna, joita alla olevat ohjeet käyttävät. Jos näin ei ole, katso apt install [Environment Setup](intermediate-guide/pi-pool-tutorial/pi-node/environment-setup.md#install-packages) -osio Pi-pool-tutorialissa.

Ei kun menoksi!

## Luo uusi hakemisto

Aloittaaksesi, valitse sijainti koneessa, jossa on Grafana. Täällä voit luoda uuden hakemiston node exporterin käyttöön. Solmun viejä sijaitsee todennäköisesti /opt/cardano/monitoring/**node\_exporter** pi-poolin oletussijainnin vuoksi. __Jos tämä ei pidä paikkansa, koita löydätkö sen käyttämällä komentoa "which node\_exporter" Jos tämä ei löydä sitä, hakemisto, jossa se sijaitsee, ei ole sinun $PATH ja sinun täytyy kaivaa syvemmälle. [Tarkista tämä git](https://github.com/prometheus/node_exporter) saadaksesi lisätietoja.

Muuta uuden hakemiston sijaintia, tässä olen valinnut paikallisen bin käyttäjälleni.

```text
> cd $HOME/.local/bin
```

Nyt tee uusi hakemisto, täällä voimme tallentaa mukautetun tekstitiedoston tilastot joita node\_exporter jäsentää. Kutsun hakemistoa **customStats**, mutta voit nimetä sen haluamallasi tavalla.

```text
> mkdir customStats
```

## Hae adapoolien Yhteenvetotiedosto

adapools.org sivusto tarjoaa **summary.json** tiedoston jokaiselle rekisteröidylle poolille. Käytämme tätä tiedostoa jäsentääksemme haluamamme tiedot ja tallentaaksemme sen juuri luomaamme hakemistoon. Voimme luoda bash skriptin, joka käsittelee tämän meille. Olen $HOME/.local/bin hakemistossa:

```text
> nano getAdaPoolsSummary.sh
```

Lisää tämä sisältö alla, korvaa **POOLIDI** oman poolisi ID-tunnuksella, tallenna ja poistu. Pohjimmiltaan tämä vetää kopion poolisi **summary. json** tiedostosta, poistaa joitakin asioita, joita node exporter ei voi jäsentää \(string values\) ja tallentaa kopion uuteen hakemistoon.

```text
curl https://js.adapools.org/pools/<YOUR POOL ID>/summary.json 2>/dev/null \
| jq '.data | del(.direct, .hist_bpe, .handles, .hist_roa, .db_ticker, .db_name, .db_description, .db_url, .ticker_orig, .pool_id, .pool_id_bech32, .group_basic)' \
| tr -d \"{},: \
| awk NF \
| sed -e 's/^[ \t]*/adapools_/' > $HOME/.local/bin/customStats/adapools.prom
```

Nyt kun **getAdaPoolsSummary.sh** on suoritettu, se päivittää tiedoston nimeltä **adapools.prom** uudessa hakemistossamme. Tämä tiedosto sisältää mittareita, jotka alkavat termillä **adapools** ja näkyvät Grafana kyselyn rakentajan mittariosiossa sellaisenaan.

{% hint style="Huomaa" %}
On tärkeää, että tiedoston tulokset eivät sisällä merkkijonon arvoja. Node exporter ilmoittaa virheestä etkä näe adapoolsin metriikkaa.
{% endhint %}

If you discover string values, you can remove them by adding a new key to the "del" section in the script above. For example, to remove the **adapools\_db\_description** metric \(has a string value\), you'd add **.db\_description** to the **del\( \)** section.

## Create crontab Entry

Depending on how often you want to refresh a copy of these stats, you can create a local crontab entry to pull a fresh copy of the adapools.prom file.

```text
> crontab -e
```

The following line **runs the script we created every 5 minutes**. Add the line, save and exit. Since this data doesn't change that often, you shouldn't need to pull it that often. Don't piss off the adapools.org folks by pulling this data every 5 seconds - it's not necessary. For other examples of crontab run times, [see this lovely link](https://crontab.tech/examples).

```text
*/5 * * * * $HOME/.local/bin/getAdaPoolsSummary.sh
```

## Run node exporter Command

Now that we are generating the **adapools.prom** file, we need to tell the node exporter where to find our custom text file. Depending on how you are running your node exporter instance, you'll need to add the following command line parameters. This might be found in the **startMonitor** script included with the pi-pool default build.

```text
> node_exporter --collector.textfile.directory=$HOME/.local/bin/customStats --collector.textfile
```

If all goes as planned, you should be able to pull up this URL in your browser and see the new **adapools** metrics. If this worked, your new metrics should be visible in the Grafana query builder.

```text
http://<YOUR GRAFANA NODE IP>:9100/metrics
```

{% hint style="info" %}
There are other methods you could use to implement this approach. Basically, if you can create a text file with key/value pairs and put it into this new directory, the node exporter should pull the data into Grafana. It opens up a vast array of possibilities. Just ensure you prefix the label names with a unique value \(the **adapools\_** __part in the adapools.prom file above\) per file.
{% endhint %}

Was this information helpful? Earn rewards with us! [Consider delegating some ADA](delegate/).

