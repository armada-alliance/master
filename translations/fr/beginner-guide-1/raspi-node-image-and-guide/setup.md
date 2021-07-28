---
description: 'Retrieve Raspbian image, write image to target drive, create user.'
---

# Download & Flash

## Installing the Raspberry Pi Debian "buster" OS

Download the latest official release of 64 bit Raspbian OS.

**1. Download Raspbian 64bit OS image** [**here**](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-05-28/2021-05-07-raspios-buster-arm64.zip) **and save it in an accessible location for now on your computer.**

**2. Next, download the Raspberry Pi Imager software that we will use in order to write the OS image onto our target drive. Ce logiciel est situé sur le site** [**Raspberry Pi**](https://www.raspberrypi.org/software/)**. Veuillez télécharger la version correcte pour votre ordinateur.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Insert the target drive\(your SSD or NVMe with usb3 adapter\) into your computer and open the "Raspberry Pi Imager".**

* **Click on "CHOOSE OS"  then "Use custom" choose the Raspbian image file you downloaded.**
* **Next, click on the "CHOOSE SD" and choose the target drive you inserted into the computers usb port.**
* **The "WRITE" button will appear and you can click on it to begin writing/verifying the OS onto the target drive.**
* **Finally, once it has finished the writing/verifying process, you will see a pop-up window saying that the OS was successfully written to the drive, click "CONTINUE" and remove your drive from the computer.**

![](../../.gitbook/assets/image-2-.png)

## Boot & Configure

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

### Create the ada user

```text
sudo adduser ada && sudo adduser ada sudo
```

Go ahead and reboot, log in as your new ada user.

```text
sudo reboot
```

