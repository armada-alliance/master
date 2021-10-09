---
description: >-
  Tämä osio auttaa ymmärtämään relayn ja block producer noden topologioita.
---

# Stake Pool -verkosto: Perusteita

## Oletukset

Tässä tutoriaalissa oletamme, että Raspberry Pi nodesi pyörivät kotonasi, liitettyinä routeriin ja ovat yhteydessä internetiin jonkin palveluntuottajalta ostetun yhteyden kautta. Nodien pitäisi olla määritetyn palomuurin takana, ja avoimia portteja tulisi olla mahdollisimman vähän. Lisäksi palomuuriasetusten tulisi olla mahdollisimman speifisiä. Lisäksi ISP:n internetreitittimessä tulee olla palomuuriasetukset määritettynä. Jos et tunne palomuuriasetuksia, niiden jättäminen ISP:n määrittämiin oletusarvoihin on yleisesti ottaen OK.

Jos sinulla on core node käynnissä, vähimmäis palomuurisäännöt ovat sellaiset, joissa portti on määritettynä auki vain relayden IP-osoitteille ja lisäksi yksi avoin portti ssh:n käyttöön. Jos haluat seurata lohkon tuottajan metriikkaa Grafanan avulla, sinun täytyy avata myös Grafanan portti. Sama juttu, jos haluat seurata relaytasi.

Muistathan, että emme ole verkostoasiantuntijoita. Edellä mainittu on tarkoitettu antamaan yleiskäsitys siitä, miten noden topologia ja verkko ovat vuorovaikutuksessa keskenään. Edistyneempiä verkkokeskusteluja varten voit käyttää NASEC discord kanavaa.

## Yleiskatsaus

Sinun **relay nodesi** tulisi saavuttaa muilta relay nodeja sekä oma lohkon tuottajasi. Sinun **lohkon tuottajasi** pitäisi olla yhteydessä vain omiin relay nodeihisi.

{% tabs %}
{% tab title="Relay Node" %}
{% code title="mainnet-topology.json" %}
```bash
{
  "Producers": [
    {
      "addr": "block producers private IPv4",
      "port": 6000,
      "valency": 1
    },
    {
      "addr": "138.197.71.216",
      "port": 3001,
      "valency": 1
    },
    {
      "addr": "107.23.17.23",
      "port": 3001,
      "valency": 1
    },
    {
      "addr": "3.140.154.176",
      "port": 6002,
      "valency": 1
    }
  ]
}
```
{% endcode %}
{% endtab %}

{% tab title="Block Producer" %}
{% code title="mainnet-topology.json" %}
```bash
{
  "Producers": [
    {
      "addr": "10.20.30.1",
      "port": 6001,
      "valency": 1
    },
    {
      "addr": "10.20.30.2",
      "port": 6002,
      "valency": 1
    }
  ]
}
```
{% endcode %}

Yllä olevat **addr** ja **port** -merkintöjen pitäisi olla relay nodiesi IP-osoitteita. Se on siinä. Lohkon tuottajasi palomuuriasetusten pitäisi rajoittaa pääsy muuaalle kuin näihin IP-osoitteisiin portissa, jossa käytät lohkon tuottajaa. Alla on esimerkki palomuurin tilasta lohkon tuottajassa joka kuuntelee porttia 6000.

{% code title="> sudo ufw status" %}
```text
To                         Action      From
--                         ------      ----
6000/tcp                   ALLOW       10.20.30.1
6000/tcp                   ALLOW       10.20.30.2
```
{% endcode %}
{% endtab %}
{% endtabs %}

Esimerkin ensimmäinen **addr** rivi  **10.20.30.** on lohkon tuottajan IP-osoite ja **port 6000** on portti, jonka olet määrittänyt lohkon tuottajalle. Tämän kohdan pitäisi olla täsmälleen sama kaikissa relay nodeissasi.

Muut kolme objektia ovat verkoston muiten käyttäjien osoitteita. Voit asettaa ne manuaalisesti tai voit käyttää **topologyUpdater.sh** skriptiä Guild-operaattoreilta. Jos päätät käyttää topologyUpdater.sh skriptiä varmista, että asetat **CUSTOM\_PEERS** -rivin oikein ennen kuin suoritat sen. Tämä on pipe-erotettu joukko addr:port:valency pareja vertaisnodeista, joita haluat komentosarjan lisävän lopulliseen topology.json tiedostoon. Tämän rivin pitää sisältää myös oma lohkon tuottajasi. Valenssin oletusarvo on 1 \(yksi\), jos sitä ei ole määritelty. Tässä esimerkki, jossa kaksi ensimmäistä objektia yllä olevasta mainnet-topology.json tiedostosta:

CUSTOM\_PEERS="10.20.30.3**:**6000**\|**138.197.71.216**:**6000"

{% hint style="info" %}
Aseta **valenssi** arvoon 0 \(nolla\) poistaaksesi etäkäyttäjän käytöstä, jos et halua poistaa käyttäjää kokonaan tiedostosta.
{% endhint %}

## Poolin Rekisteröinti

Kun luot stake poolisi **pool.json** metadatatiedoston huomaat osion nimeltä **poolRelays**. Tässä kohtaa voit lisätä **julkiset** relayt, jotka näkyvät muille. Voit lisätä ne staattisina IP-osoitteina tai verkkotunnuksena, kuten **north.acme.com**. Jos sinulla on käynnissä useampi kuin yksi relay sisäisessä verkossasi, sinun täytyy määrittää niille eri portit, kuten 6001 ja 6002.

{% code title="pool.json" %}
```bash
"poolRelays": [
    {
      "relayType": "dns",
      "relayEntry": "north.acme.com",
      "relayPort": "6001"
    },
    {
      "relayType": "dns",
      "relayEntry": "north.acme.com",
      "relayPort": "6002"
    }
  ],
```
{% endcode %}

Tyypillinen kotiverkko altistaa maailmalle vain yhden ulkoisen IP-osoitteen, joka on dynaamisesti määritetty Internet-palveluntarjoajasi kautta \(Internet Service Provider\). Dynaamisesti määritetyt ulkoiset IP-osoitteet voivat olla suhteellisen staattisia pitkäänkin, mutta tämä ei ole taattu ja sinun kannattaa harkita DNS tunnuksen rekisteröimistä, jotta voit käyttää dns merkintöjä pool.json tiedostossa. Muuten, joka kerta kun ulkoinen IP-osoitteesi muuttuu sinun täytyy uudelleen rekisteröidä poolisi ja päivittää uudet relay IP-osoitteet.

## DNS Client

Ellei sinulla ole ISP:n osoittamaa staattista IP-osoitetta, jossain vaiheessa sinun täytyy harkita dynaamisen DNS:n rekisteröimistä. DNS palvelu toimii sisäisessä verkossa ja lähettää ulkoisen,ISP:n määrittämän, IP-osoitteesi dynaamisen DNS verkkotunnuksen tarjoajalle, kuten Google domains. Siten, aina kun ISP muuttaa ulkoisen dynaaminen IP-osoitteesi, DNS palvelu näkee sen ja välittää uuden IP-osoitteen verkkotunnuksen tarjoajalle. Tällä prosessilla ei ole juuri mitään vaikutusta verkkotunnusosoitteisiin.

### DNS Client Esimerkkejä

* [ddclient](https://support.google.com/domains/answer/6147083?hl=en)
* no-ip
* namecheap.com openwrt ddns-scripts

Oliko tämä tieto hyödyllistä? Ansaitse palkintoja kanssamme! [Harkitse ADA: n delegoimista pooleihimme](../cardano-developer-guides/delegate.md).

