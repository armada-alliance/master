---
description: This is how you create an image you can flash to other Pi's
---

# Create .img file

## Make the Pi-Node base .img.gz file for reuse

Put your micro sd card in your local machine and locate what it's called in /dev. For my laptop it is /dev/mmcblk0. Yours will likely be different.

```text
sudo fdisk -l
```

After locating move into the directory you wish to save the image to and create the image.

```bash
# example
# sudo cat /dev/mmcblk0 > pi-node.img
sudo cat /dev/<your sd card> > pi-node.img
```

{% hint style="info" %}
cat is better than dd for this. cat will use all of your systems cpu cores, whereas dd uses one core. cat is faster 🙀
{% endhint %}

Once that completes we will use [PiShrink.sh](https://github.com/Drewsif/PiShrink) to deflate partitions and compress \(among a few other tricks\).

{% code title="install pishrinks.sh" %}
```bash
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/local/bin
```
{% endcode %}

```bash
sudo pishrink.sh -aZ pi-node.img Pi-Node.img.gz
```

> pishrink.sh: Shrunk Pi-Node.img.gz from 7.5G to 1.3G ...

And there you have it! 🧙♂

Download [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz)

