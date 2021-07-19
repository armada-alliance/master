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

Jos huomaat merkkijonon arvoja, voit poistaa ne lisäämällä uuden avaimen "del" osioon skriptin yllä. Esimerkiksi poistaaksesi **adapools\_db\_description** mittarin \(sisältää merkkijonoarvon\), lisäisit **.db\_description** **del\( \)** -osioon.

## Luo crontab Sääntö

Riippuen siitä, kuinka usein haluat päivittää kopion näistä tilastoista, voit luoda paikallisen crontab merkinnän ja vetää tuoreen kopion adapools.prom tiedostosta.

```text
> crontab -e
```

Seuraava rivi **ajaa luomamme skriptin 5 minuutin välein**. Lisää rivi, tallenna ja poistu. Koska nämä tiedot eivät muutu kovin usein, sinun ei pitäisi myöskään vetää päivitystä kovin usein. Älä suututa adapools.org:n väkeä vetämällä tätä tietoa 5 sekunnin välein - se ei ole tarpeen. Muita esimerkkejä crontab ajoajoista, [katso tämä ihana linkki](https://crontab.tech/examples).

```text
*/5 * * * * $HOME/.local/bin/getAdaPoolsSummary.sh
```

## Suorita node exporter käsky

Nyt kun olemme luomassa **adapools.prom** tiedostoa, meidän täytyy kertoa node exporterille mistä se löytää mukautetun tekstitiedostomme. Riippuen siitä, miten käytät node exporteria, sinun täytyy lisätä seuraavat komentoriviparametrit. Tämä saattaa löytyä **startMonitor** skriptistä, joka sisältyy pi-poolin oletusversioon.

```text
> node_exporter --collector.textfile.directory=$HOME/.local/bin/customStats --collector.textfile
```

Jos kaikki menee suunnitelmien mukaisesti, sinun pitäisi pystyä nostamaan tämä URL selaimessasi ja nähdä uusia **adapools** mittareita. Jos tämä toimi, uusien mittareidesi pitäisi näkyä Grafana kyselyn rakentajassa.

```text
http://<YOUR GRAFANA NODE IP>:9100/metrics
```

{% hint style="info" %}
On olemassa muitakin menetelmiä, joita voit käyttää tämän lähestymistavan toteuttamiseen. Periaatteessa, jos luot tekstitiedoston avaimen/arvon pareilla ja laitat sen tähän uuteen kansioon, node exporterin pitäisi vetää tiedot Grafanaan. Se avaa laajan valikoiman mahdollisuuksia. Varmista vain, että lisäät yksilöllisen etuliitteen tunnisteen nimeen \( **adapools\_** __part in the adapools.prom file above\) per tiedosto.
{% endhint %}

Oliko tämä tieto hyödyllistä? Ansaitse palkintoja kanssamme! [Harkitse ADA: n delegoimista pooleihimme](delegate/).

