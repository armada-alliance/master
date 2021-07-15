---
description: Flash image
---

# Download & Flash

## Flash Image

Download, install & open [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager/releases/latest). Plug in your target USB drive.

{% tabs %}
{% tab title="Local Machine \(Ubuntu\)" %}
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

**Choose OS -&gt; Misc utility images -&gt; Raspberry Pi 4 EEPROM boot recovery** [https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md)
{% endhint %}

![](../../../.gitbook/assets/otgpoltut%20%281%29%20%281%29%20%281%29.png)

{% tabs %}
{% tab title="Pre configured Pi-Node.img.gz" %}
### Obtain Pi-Pool .img.gz file

| [Pi-Node](https://db.adamantium.online/Pi-Node.img.gz) |
|:------------------------------------------------------ |
|                                                        |


### Within Raspberry Pi Imager

**Choose OS -&gt; Use custom**

Locate the .img.gz file you downloaded & wish to flash.

Locate your target drive & write it to disk.

![](../../../.gitbook/assets/image-2-.png)
{% endtab %}

{% tab title="Fresh Ubuntu 21.04 installation" %}
### Within Raspberry Pi Imager

### Select  Ubuntu Server 21.04 \(RPI 3/4/400\)

**Choose OS -&gt; Other general purpose OS -&gt; Ubuntu -&gt; Ubuntu Server 21.04 \(RPI 3/4/400\)**. The 64 bit server option.

Locate your target drive & write it to disk.

![](../../../.gitbook/assets/21.04-rpi-imager.png)
{% endtab %}
{% endtabs %}

