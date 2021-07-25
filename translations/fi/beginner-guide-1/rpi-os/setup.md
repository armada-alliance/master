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

**1. Download the Debian “buster” Raspberry Pi 64bit OS image** [**here**](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-05-28/2021-05-07-raspios-buster-arm64.zip) **and save it in an accessible location for now on your computer.**

**2. Seuraavaksi, lataa Raspberry Pi Imager ohjelma, jota käytetään asentamaan yllä mainittu käyttöjärjestelmä Raspberry Pi:lle. Tämä ohjelmisto on saatavilla** [**Raspberry Pi verkkosivuilla**](https://www.raspberrypi.org/software/)**. Tarkasta, että lataat koneellesi oikean version.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Aseta SD kortti tietokoneeseesi ja avaa "Raspberry Pi Imager".**

* **Click on "CHOOSE OS"  then find the** _**raspios-buster-arm64.zip**_ **file you have downloaded in step \(1\) of this tutorial and select it.**
* **Seuraavaksi klikkaa "CHOOSE SD" ja etsi SD-kortti, jonka asetit tietokoneeseen**
* **Nyt, "WRITE" painike ilmestyy ja voit klikata sitä ja aloittaa OS:n kirjoittamisen/todentamisen SD-kortille.**
* **Lopuksi, kun kirjoitusprosessi on valmis, näet pop-up ikkunan, joka kertoo että käyttöjärjestelmä on asennettu onnistuneesti SD kortille. Klikkaa "CONTINUE" ja poista SD kortti tietokoneesta.**

{% hint style="info" %}
#### **If you still have issues following the written instructions, watch the video below.**
{% endhint %}

{% embed url="https://www.youtube.com/watch?v=ntaXWS8Lk34" %}



### Osa 2:

### Raspberry Pi:n Määrittäminen

The first thing that we want to do is get the Raspberry Pi booted up and configured for our use.

To do this we will need to insert the SD card we flashed earlier with the Raspberry Pi OS into the bottom of the Raspberry Pi. Then we can insert our HDMI, Keyboard, Mouse, and power supply.

Once the Raspberry Pi startup screen is finished and you have booted into the Raspberry Pi OS Desktop screen we can now begin to set up our Raspberry Pi configuration and settings.

{% hint style="info" %}
If this is your first time booting up the Raspberry Pi OS you will have to follow some initial configurations listed below
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

**This is the final step in this tutorial. We are going to first insert our external SSD into one of the USB 3.0 slots marked blue.**

![](../../.gitbook/assets/pi4.jpeg)

Open the Raspberry Pi applications menu and then click on the **SD Card Copier** application.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-9.11.39-pm%20%281%29.png)

Then we want to select **COPY FROM DEVICE** - **\(mmcblk0\) SD CARD.**

Next, select **COPY TO DEVICE - \(sda\) SSD Device.**

Once the copy process is complete open a new terminal window and enter the following command.

```text
sudo raspi-config
```

This will bring you to the Raspberry Pi's system configuration settings where you can access the **Advanced Options.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.19-pm.png)

Next select **Boot Order.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.40-pm%20%281%29.png)

Valitse sitten **USB Boot**.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.14.05-pm%20%281%29.png)

Nyt voit valita **&lt;Ok&gt;** sitten **&lt;Finish&gt;**, sulje Raspberry Pi:n järjestelmän asetusvalikko ja käynnistä Pi uudelleen.

Sinun pitäisi nyt pystyä sulkemaan Pi kun se käynnistyy uudelleen, poista SD-kortti, sitten voit käynnistää Pi ja sen pitäisi käynnistyä ulkoisesta USB-tallennuslaitteesta.

{% hint style="success" %}
#### Nyt kun olemme suorittaneet suurimman osan käyttöönottoasetuksista, voimme jatkaa Pi:n valmistelua ja siirtyä seuraavaan [ohjeeseen](tutorial-2-relaynode.md).
{% endhint %}

