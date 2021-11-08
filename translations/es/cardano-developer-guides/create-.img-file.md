---
description: Como crear una imagen que puedas flashear en otras Pi
---

# Crear archivo .img

## Hacer el archivo base .img.gz Pi-Node para su reutilización

Coloca tu tarjeta micro sd en tu máquina local y localiza lo que se llama en /dev. Para mi portátil es /dev/mmcblk0. Es probable que el tuyo sea diferente.

```text
sudo fdisk -l
```

Después de localizarlo muevelo al directorio en el que quieres guardar y crear la imagen.

```bash
# example
# sudo cat /dev/mmcblk0 > pi-node.img
sudo cat /dev/<your sd card> > pi-node.img
```

{% hint style="info" %}
el comando cat es mejor que dd para esto. cat usará todos los núcleos cpu de sus sistemas, mientras que dd usa un núcleo. cat es más rápido 🙀
{% endhint %}

Una vez que finalice usaremos [PiShrink.sh](https://github.com/Drewsif/PiShrink) para simplificar particiones y comprimir \\(entre algunos otros trucos\\).

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

¡Y ahí esta! 🧙♂

Descargar [Pi-Node.img.gz](https://mainnet.adamantium.online/Pi-Node.img.gz)

