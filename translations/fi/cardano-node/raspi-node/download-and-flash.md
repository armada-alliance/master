# Lataa & Polta

## RaspiNode OS -järjestelmän asentaminen

**1. Lataa Armada Alliancen esikonfiguroitu Raspbian 64bit OS Cardano-node kuva** [**tästä**](https://db.adamantium.online/RasPi-Node.img.gz) **ja tallenna se toistaiseksi tietokoneellesi.**

**2. Seuraavaksi, lataa Raspberry Pi Imager ohjelma, jota käytetään asentamaan yllä mainittu käyttöjärjestelmä Raspberry Pi:lle. Tämä ohjelmisto on saatavilla** [**Raspberry Pi verkkosivuilla**](https://www.raspberrypi.org/software/)**. Tarkasta, että lataat koneellesi oikean version.**

![](../../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Insert the target drive(your SSD with usb3 adapter) into your computer and open the "Raspberry Pi Imager".**

![](../../.gitbook/assets/custom\_os.png)

* **Click on "CHOOSE OS"  then "Use custom" choose the Raspbian image file you downloaded.**&#x20;
* **Seuraavaksi klikkaa "CHOOSE SD" ja etsi SD-kortti, jonka asetit tietokoneeseen.**
* **The "WRITE" button will appear and you can click on it to begin writing/verifying the OS onto the target drive.** &#x20;
* **Finally, once it has finished the writing/verifying process, you will see a pop-up window saying that the OS was successfully written to the drive, click "CONTINUE" and remove your drive from the computer.**&#x20;

## Käynnistys & asetukset

Aseta SSD yhteen sinisestä usb3-porteista. Seuraavaksi kiinnitetään HDMI kaapeli, näppäimistö, hiiri ja virtalähde.

{% hint style="danger" %}
Ensimmäiset Pi4:t eivät ole oletusarvoisesti käynnistyneet USB3:sta, mutta nykyään niiden pitäisi. Jos imagesi ei käynnisty kaksi yleisintä ongelmaa ovat vanhemmat laiteohjelmistot Pi:ssäsi tai yhteensopimaton USB3 sovitin.
{% endhint %}

![](../../../.gitbook/assets/pi4 (1).jpeg)

{% hint style="info" %}
Kaikki mitä todella tarvitsee tehdä on poistaa automaattinen kirjautuminen & luoda ada käyttäjä jolla on sudo oikeudet. Kun kirjaudumme takaisin, poistamme oletus Pi käyttäjän ja määrittämme palvelin ympäristön sekä cardan-noden & cardano-clin.
{% endhint %}

![](../../../.gitbook/assets/raspberrypi-configuration.png)

![](../../../.gitbook/assets/disable-auto-login.png)

## Luo ada käyttäjä

```
sudo adduser ada && sudo adduser ada sudo
```

Sitten vaan uudelleenkäynnistys ja kirjaudu sisään uutena ada käyttäjänä.

```
sudo reboot
```
