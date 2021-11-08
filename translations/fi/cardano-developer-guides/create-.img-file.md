---
description: Näin voit luoda imagen, jonka voit siirtää uuteen Raspberry Pi:hin
---

# Luo .img-tiedosto

## Tee Pi-Node base .img.gz tiedosto uudelleenkäyttöä varten

Laita micro SD-kortti koneeseen ja etsi mikä se on nimeltään /dev. Minun kannettavassani se on /dev/mmcblk0. Sinulla todennäköisesti kuitenkin eri nimi.

```text
sudo fdisk -l
```

Kun olet löytänyt SD-kortin nimen, siirry hakemistoon johon haluat tallentaa imagen ja luo image.

```bash
# esimerkki
# sudo cat /dev/mmcblk0 > pi-node.img
sudo cat /dev/<your sd card> > pi-node.img
```

{% hint style="info" %}
cat-komento on parempi kuin dd tässä tapauksessa. cat käyttää kaikkia järjestelmän cpu ytimiä, kun taas dd käyttää vain yhtä ydintä. cat on siis nopeampi 🙀
{% endhint %}

Kun .img on valmis, käytämme [PiShrink.sh](https://github.com/Drewsif/PiShrink) skriptiä, joka pakkaa osiot \(sekä muutamia muita temppuja\).

{% code title="install pishrinks.sh" %}
```bash
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/local/bin
```
{% endcode %}

```bash
sudo pishrink.sh -az pi-node.img Pi-Node.img.gz
```

> pishrink.sh: Shrunk Pi-Node.img.gz from 7.5G to 1.3G ...

Ja siellä on se! 🧙♂

Download [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz)

