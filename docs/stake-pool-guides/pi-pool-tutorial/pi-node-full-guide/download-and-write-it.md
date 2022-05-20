---
description: Flash image
---

# Download & Flash

## Flash Image

Download, install & open [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager/releases/latest). Plug in your target USB drive.

{% tabs %}
{% tab title="Local Machine (Ubuntu)" %}
```bash
# Ubuntu users can download and install with snapd
sudo apt update
sudo apt install snapd
sudo snap install rpi-imager
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Older models of the Pi4B 8GB need to have their boot loader updated to boot from USB. If your image won't boot remove the USB3 drive and use rpi-imager to flash Pi 4 EEPROM boot recovery to an sd card.

Plug the Pi into a monitor, insert the sd card and power up. Once you see a green screen you should be good to boot from your USB3 drive. Newer versions are shipping with a USB boot capable boot loader. **Feeling lucky?**

**Choose OS -> Misc utility images -> Bootloader -> USB Boot** [https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md)
{% endhint %}

![](<../../../../.gitbook/assets/otgpoltut (1) (1) (3) (1) (4).png>)

{% tabs %}
{% tab title="Pre configured Pi-Node.img.gz" %}
**Obtain Pi-Node.img.gz file**

\| [Pi-Node](https://mainnet.adamantium.online/Pi-Node.img.gz)

**Within Raspberry Pi Imager**

**Choose OS -> Use custom**

Locate the .img.gz file you downloaded & wish to flash.

Locate your target drive & write it to disk.

![](<../../../.gitbook/assets/image-2- (1) (3).png>)
{% endtab %}

{% tab title="Fresh Ubuntu 22.04 LTS installation" %}
**Within Raspberry Pi Imager**

**Download Ubuntu Server 22.04 (RPI 3/4/400)**

[Raspberry Pi Generic (64-bit ARM) preinstalled server image](https://cdimage.ubuntu.com/ubuntu-server/jammy/daily-preinstalled/current/jammy-preinstalled-server-arm64+raspi.img.xz)

**Choose OS -> Use custom** Locate the .img.gz file you downloaded & wish to flash. Locate your target drive & write it to disk.
{% endtab %}
{% endtabs %}
