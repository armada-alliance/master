---
description: 'Nouda Raspbian image, kirjoita image kohdeasemalle, luo käyttäjä.'
---

# Lataa & Polta

## Installing the RaspiNode OS

**1. Download the Armada Alliance's pre-configured Raspbian 64bit OS Cardano-node image** [**here**](https://db.adamantium.online/RasPi-Node.img.gz) **and save it in an accessible location for now on your computer.**

**2. Next, download the Raspberry Pi Imager software that we will use in order to write the OS image onto our target drive. Tämä ohjelmisto on saatavilla** [**Raspberry Pi verkkosivuilla**](https://www.raspberrypi.org/software/)**. Tarkasta, että lataat koneellesi oikean version.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Insert the target drive\(your SSD or NVMe with usb3 adapter\) into your computer and open the "Raspberry Pi Imager".**

* **Klikkaa "CHOOSE OS" ja sitten "Use custom" valitse Raspbian image tiedosto, jonka latasit.**
* **Seuraavaksi klikkaa "CHOOSE SD" ja etsi SD-kortti, jonka asetit tietokoneeseen.**
* **Nyt, "WRITE" painike ilmestyy ja voit klikata sitä ja aloittaa OS:n kirjoittamisen/todentamisen SD-kortille.**
* **Lopuksi, kun kirjoitusprosessi on valmis, näet pop-up ikkunan, joka kertoo että käyttöjärjestelmä on asennettu onnistuneesti SD kortille. Klikkaa "CONTINUE" ja poista SD kortti tietokoneesta.**

![](../../.gitbook/assets/image-2-.png)

## Käynnistys & asetukset

Insert the SSD into one of the blue usb3 ports. Then insert the HDMI, Keyboard, Mouse, Ethernet and power supply.

{% hint style="danger" %}
The first Pi4's to ship do not boot from USB3 by default, nowadays they do. If your image does not boot the two most common issues are older firmware on your Pi or an incompatible USB3 adaptor.
{% endhint %}

![](../../.gitbook/assets/pi4.jpeg)

{% hint style="info" %}
All we really need to do here is disable auto login & create the ada user with sudo privileges. After we log back in we will delete the default Pi user and configure the server & environment for cardan-node & cardano-cli.
{% endhint %}

![Open the Raspberry Pi Configuration utility.](../../.gitbook/assets/raspberrypi-configuration.png)

![Set Auto Login to Disabled](../../.gitbook/assets/disable-auto-login.png)

### Luo ada käyttäjä

```text
sudo adduser ada && sudo adduser ada sudo
```

Go ahead and reboot, log in as your new ada user.

```text
sudo reboot
```

