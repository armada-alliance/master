# Download & Flash

## Installing the RaspiNode OS

**1. Laden Sie das vorkonfigurierte Armada Alliance 64bit OS Cardano-Node Bild ** [**hier herunter**](https://db.adamantium.online/RasPi-Node.img.gz) **und speichern Sie es an einem zugänglichen Ort auf Ihrem Computer.**

**2. Als nächstes laden Sie die Raspberry Pi Imager Software herunter, die wir verwenden werden, um das OS-Image auf unser Ziel-Laufwerk zu schreiben. Diese Software befindet sich auf der** [**Raspberry Pi Webseite**](https://www.raspberrypi.org/software/)**. Bitte laden Sie die korrekte Version für Ihren Computer herunter.**

![](../../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Legen Sie das Ziellaufwerk\(Ihr SSD oder NVMe mit usb3-Adapter\) in Ihren Computer ein und öffnen Sie den "Raspberry Pi Imager".**

![](../../.gitbook/assets/custom_os.png)

* **Klicken Sie auf "CHOOSE OS" und dann "Use custom" wählen Sie die Raspbian Image-Datei, die Sie heruntergeladen haben.**
* **Als nächstes klicken Sie auf "CHOOSE SD" und wählen das Laufwerk aus, das Sie in die USB Schnittstelle ihres Computers eingelegt haben.**
* **Die Schaltfläche "WRITE" erscheint und Sie können nun darauf klicken, um das Betriebssystem auf das Ziellaufwerk zu schreiben/zu überprüfen.**
* **Schließlich, sobald der Schreib-/Überprüfungsprozess beendet ist, erscheint ein Pop-Up-Fenster, das sagt, dass das Betriebssystem erfolgreich auf das Laufwerk geschrieben wurde, klicken Sie auf "CONTINUE" und entfernen Sie Ihr Laufwerk vom Computer.**

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
