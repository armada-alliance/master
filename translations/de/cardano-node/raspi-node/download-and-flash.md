# Download & Flash

## Installing the RaspiNode OS

**1. Download the Armada Alliance's pre-configured Raspbian 64bit OS Cardano-node image** [**here** ](https://mainnet.adamantium.online/Raspi-Node.img.gz)**and save it in an accessible location for now on your computer.**

{% hint style="warning" %}
There is a testnet image you can download [here](https://testnet.adamantium.online/Test-Raspi-node.img.gz). May need binaries upgraded.
{% endhint %}

**2. Als nächstes laden Sie die Raspberry Pi Imager Software herunter, die wir verwenden werden, um das OS-Image auf unser Ziel-Laufwerk zu schreiben. Diese Software befindet sich auf der** [**Raspberry Pi Webseite**](https://www.raspberrypi.org/software/)**. Bitte laden Sie die korrekte Version für Ihren Computer herunter.**

![](../../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Insert the target drive(your SSD with usb3 adapter) into your computer and open the "Raspberry Pi Imager".**

![](../../.gitbook/assets/custom\_os.png)

* **Click on "CHOOSE OS"  then "Use custom" choose the Raspbian image file you downloaded.**&#x20;
* **Als nächstes klicken Sie auf "CHOOSE SD" und wählen das Laufwerk aus, das Sie in die USB Schnittstelle ihres Computers eingelegt haben.**
* **The "WRITE" button will appear and you can click on it to begin writing/verifying the OS onto the target drive.** &#x20;
* **Finally, once it has finished the writing/verifying process, you will see a pop-up window saying that the OS was successfully written to the drive, click "CONTINUE" and remove your drive from the computer.**&#x20;

## Booten & konfigurieren

Legen Sie die SSD in einen der blauen usb3-Ports ein. Dann verbinden Sie die HDMI, Tastatur, Maus, Ethernet und Stromversorgung.

{% hint style="danger" %}
Die ersten Pi4's booteten standardmäßig nicht von der USB3 Schnittstelle, heutzutage tun sie es jedoch. Wenn Ihr Image nicht booten sollte dann sind die beiden häufigsten Fehlerquellen eine ältere Firmware auf Ihrem Pi oder ein inkompatibler USB3-Adapter.
{% endhint %}

![](../../../.gitbook/assets/pi4 (1).jpeg)

{% hint style="info" %}
Alles, was wir hier wirklich tun müssen, ist die automatische Anmeldung zu deaktivieren & einen ada Benutzer mit sudo Privilegien erstellen. Nachdem wir uns wieder eingeloggt haben, werden wir den Standard-Pi-Benutzer löschen und die Server- & -Umgebung für cardan-node & cardano-cli konfigurieren.
{% endhint %}

![](../../../.gitbook/assets/raspberrypi-configuration.png)

![](../../../.gitbook/assets/disable-auto-login.png)

## ada Benutzer erstellen

```
sudo adduser ada && sudo adduser ada sudo
```

Starten Sie neu und melden Sie sich als Ihr neuer ada-Benutzer an.

```
sudo reboot
```
