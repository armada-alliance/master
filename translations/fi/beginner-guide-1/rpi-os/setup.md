---
description: Tässä tutoriaalissa käymme läpi Raspberry Pin ja Linuxin asentamisen perusteita
---

# Raspberry Pin asennus

## Yhteenveto <a id="h.vrhvb96nxxe9"></a>

1. Lataa käyttöjärjestelmä \(OS\). Tässä ohjeessa käytämme Raspberry Pi OS -käyttöjärjestelmää.
2. Asenna Raspberry Pi OS käyttäen Raspberry Pi Imager ohjelmaa
3. Asenna käyttöjärjestelmä SD-kortille
4. Käynnistä Pi ja määritä asetukset
5. Aseta ulkoinen SSD ja kopioi SD-kortti siihen
6. Sammuta ja käynnistä uudelleen SSD-laitteesta

{% hint style="info" %}
### ÄLÄ OHITA OHJEEN KOHTIA
{% endhint %}

### **Osa 1:**

### Asennetaan Raspberry Pi Debian "buster" OS <a id="h.lpv6ciisjqp3"></a>

Nyt lataamme uusimman virallisen version Raspberry Pi 64bit Debian OS. Tämä on virallinen Raspberry Pi:lle ja ARM64 CPU:lle suunniteltu 64bit Linux käyttöjärjestelmä. Tämä tekee järjestelmästä hyvin vakaan ja helpottaa Raspberry Pin käyttöönottoa.

**1. Lataa Debian “buster” Raspberry Pi 64bit OS image** [**täältä**] ja tallenna se toistaiseksi tietokoneellesi kätevästi saataville</p>

**2. Seuraavaksi, lataa Raspberry Pi Imager ohjelma, jota käytetään asentamaan yllä mainittu käyttöjärjestelmä Raspberry Pi:lle. Tämä ohjelmisto on saatavilla** [**Raspberry Pi verkkosivuilla**](https://www.raspberrypi.org/software/)**. Tarkasta, että lataat koneellesi oikean version.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Aseta SD kortti tietokoneeseesi ja avaa "Raspberry Pi Imager".**

* **Klikkaa "CHOOSE OS"-painiketta ja etsi sitten** _**raspios-buster-arm64. ip**_ **tiedosto jonka latasit tämän oppitunnin vaiheessa \(1\) ja valitse se.**
* **Seuraavaksi klikkaa "CHOOSE SD" ja etsi SD-kortti, jonka asetit tietokoneeseen**
* **Nyt, "WRITE" painike ilmestyy ja voit klikata sitä ja aloittaa OS:n kirjoittamisen/todentamisen SD-kortille.**
* **Lopuksi, kun kirjoitusprosessi on valmis, näet pop-up ikkunan, joka kertoo että käyttöjärjestelmä on asennettu onnistuneesti SD kortille. Klikkaa "CONTINUE" ja poista SD kortti tietokoneesta.**

{% hint style="info" %}
#### **Jos sinulla on ongelmia seurata kirjallisia ohjeita, voit katsoa alla olevan videon.**
{% endhint %}

{% upotettu url="https://www.youtube.com/watch?v=ntaXWS8Lk34" %}



### Osa 2:

### Raspberry Pi:n Määrittäminen

Ensimmäinen asia, jonka haluamme tehdä, on saada Raspberry Pi käynnistymään ja määritettyä käyttöömme.

Tätä varten valmistelemamme SD kortti asetetaan Raspberry Pin pohjassa olevaan asemaan. Seuraavaksi kiinnitetään HDMI kaapeli, näppäimistö, hiiri ja virtalähde.

Kun Raspberry Pin käynnistys on valmis ja Raspberry Pi OS työpöytä on näkyvillä voimme aloittaa asetusten määrittämisen.

{% hint style="info" %}
Mikäli tämä on ensimmäinen kerta kun käynnistät Raspberry Pi koneesi, seuraa käyttöönoton ohjeita alla.
{% endhint %}

* [ ] Ensin, vaihda koneen käyttäjätunnuksesi ja salasanasi. Näin turvaat laitteesi, evätkä oletustunnukset ole enää käytössä.
* [ ] Syötä WiFi-kirjautumistiedot (**voit** **ohittaa tämän, jos käytät Ethernet-yhteyttä**)
* [ ] Määritä paikallinen aikavyöhyke.
* [ ] Valitse kieli ja näppäimistön asetukset.
* [ ] Päivitä Raspberry Pi (ohita tämä, jos haluat päivittää laitteen terminaalin kautta)

{% hint style="success" %}
#### Näiden ensiasetusten jälkeen on aika asettaa Raspberri Pi käynnistymään USB portin kautta, jotta voit käyttää ulkoista SSD kovalevyäsi.
{% endhint %}

### Osa 3:

### Pi:n käynnistäminen USB:n kautta

**Tämä on tutoriaalin viimeinen vaihe. Ensin ulkoinen SSD kovalevy liitetään sinisellä merkittyyn USB 3.0 porttiin.**

![](../../.gitbook/assets/pi4.jpeg)

Avaa Raspberry Pi sovellukset valikko ja valitse sitten **SD-kortti kopioija** sovellus.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-9.11.39-pm%20%281%29.png)

Sitten haluamme valita **KOPIOI LAITTEESTA** - **\(mmcblk0\) SD CARD.**

Valitse seuraavaksi **KOPIOI LAITTEELLE –\(sda\) SSD-laitteelle.**

Kun kopiointiprosessi on valmis, avaa uusi pääteikkuna ja kirjoita seuraava komento.

```text
sudo raspi-config
```

Tämä tuo sinut Raspberry Pi:n järjestelmäasetuksiin, jossa voit muokata **Advanced Options**-asetuksia.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.19-pm.png)

Valitse seuraavaksi **Boot Order.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.40-pm%20%281%29.png)

Valitse sitten **USB Boot**.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.14.05-pm%20%281%29.png)

Nyt voit valita **&lt;Ok&gt;** sitten **&lt;Finish&gt;**, sulje Raspberry Pi:n järjestelmän asetusvalikko ja käynnistä Pi uudelleen.

Sinun pitäisi nyt pystyä sulkemaan Pi kun se käynnistyy uudelleen, poista SD-kortti, sitten voit käynnistää Pi ja sen pitäisi käynnistyä ulkoisesta USB-tallennuslaitteesta.

{% hint style="success" %}
#### Nyt kun olemme suorittaneet suurimman osan käyttöönottoasetuksista, voimme jatkaa Pi:n valmistelua ja siirtyä seuraavaan [ohjeeseen](tutorial-2-relaynode.md).
{% endhint %}

