---
description: 'Nouda Raspbian image, kirjoita image kohdeasemalle, luo käyttäjä.'
---

# Lataa & Polta

## RaspiNode OS -järjestelmän asentaminen

**1. Lataa Armada Alliancen esikonfiguroitu Raspbian 64bit OS Cardano-node kuva** [**tästä**](https://db.adamantium.online/RasPi-Node.img.gz) **ja tallenna se toistaiseksi tietokoneellesi.**

**2. Seuraavaksi, lataa Raspberry Pi Imager ohjelma, jota käytetään asentamaan yllä mainittu käyttöjärjestelmä Raspberry Pi:lle. Tämä ohjelmisto on saatavilla** [**Raspberry Pi verkkosivuilla**](https://www.raspberrypi.org/software/)**. Tarkasta, että lataat koneellesi oikean version.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Aseta kohdeasema\(SSD tai NVMe usb3-sovittimen avulla\) tietokoneeseesi ja avaa "Raspberry Pi Imager".**

* **Klikkaa "CHOOSE OS" ja sitten "Use custom" valitse Raspbian image tiedosto, jonka latasit.**
* **Seuraavaksi klikkaa "CHOOSE SD" ja etsi SD-kortti, jonka asetit tietokoneeseen.**
* **Nyt, "WRITE" painike ilmestyy ja voit klikata sitä ja aloittaa OS:n kirjoittamisen/todentamisen SD-kortille.**
* **Lopuksi, kun kirjoitusprosessi on valmis, näet pop-up ikkunan, joka kertoo että käyttöjärjestelmä on asennettu onnistuneesti SD kortille. Klikkaa "CONTINUE" ja poista SD kortti tietokoneesta.**

![](../../.gitbook/assets/image-2-.png)

## Käynnistys & asetukset

Aseta SSD yhteen sinisestä usb3-porteista. Seuraavaksi kiinnitetään HDMI kaapeli, näppäimistö, hiiri ja virtalähde.

{% hint style="danger" %}
Ensimmäiset Pi4:t eivät ole oletusarvoisesti käynnistyneet USB3:sta, mutta nykyään niiden pitäisi. Jos imagesi ei käynnisty kaksi yleisintä ongelmaa ovat vanhemmat laiteohjelmistot Pi:ssäsi tai yhteensopimaton USB3 sovitin.
{% endhint %}

![](../../.gitbook/assets/pi4.jpeg)

{% hint style="info" %}
Kaikki mitä todella tarvitsee tehdä on poistaa automaattinen kirjautuminen & luoda ada käyttäjä jolla on sudo oikeudet. Kun kirjaudumme takaisin, poistamme oletus Pi käyttäjän ja määrittämme palvelin ympäristön sekä cardan-noden & cardano-clin.
{% endhint %}

![Avaa Raspberry Pi Configuration apuohjelma.](../../.gitbook/assets/raspberrypi-configuration.png)

![Aseta automaattinen kirjautuminen pois käytöstä](../../.gitbook/assets/disable-auto-login.png)

### Luo ada käyttäjä

```text
sudo adduser ada && sudo adduser ada sudo
```

Sitten vaan uudelleenkäynnistys ja kirjaudu sisään uutena ada käyttäjänä.

```text
sudo reboot
```

