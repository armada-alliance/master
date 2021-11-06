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

**Choose OS -> Misc utility images -> Raspberry Pi 4 EEPROM boot recovery** [https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md)
{% endhint %}

![](../../../../.gitbook/assets/otgpoltut (1) (1) (3) (3).png)

{% tabs %}
{% tab title="Pre configured Pi-Node.img.gz" %}
### Obtain Pi-Node.img.gz file

| [Pi-Node](https://mainnet.adamantium.online/Pi-Node.img.gz) |
| ----------------------------------------------------------- |
|                                                             |

### Within Raspberry Pi Imager

**Choose OS -> Use custom**

Locate the .img.gz file you downloaded & wish to flash.

Locate your target drive & write it to disk.

![](../../../.gitbook/assets/custom_os.png)
{% endtab %}

{% tab title="Fresh Ubuntu 21.10 installation" %}
### Within Raspberry Pi Imager

### Select  Ubuntu Server 21.10 (RPI 3/4/400)

**Choose OS -> Other general purpose OS -> Ubuntu -> Ubuntu Server 21.10 (RPI 3/4/400)**. The 64 bit server option.

Locate your target drive & write it to disk.

{% endtab %}
{% endtabs %}
