---
description: 'Descargar imagen Raspbian, escribir imagen en unidad de destino, crear usuario.'
---

# Descargar y Flashear

## Instalar el sistema operativo RaspiNode

**1. Descargue la imagen Raspbian del nodo Cardano-node de 64 bits de la Armada Alliance** [**aquí**](https://mainnet. adamantium. online/RasPi-Node.img. gz) **y guárdelo en una ubicación accesible por ahora en su ordenador.**

**2. A continuación, descarga el software de Raspberry Pi Imager que utilizaremos para grabar la imagen del sistema operativo en nuestro disco de destino. Este software se encuentra en el sitio web** [**Raspberry Pi**](https://www.raspberrypi.org/software/)**. Por favor, descargue la versión correcta para su computadora.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Inserta la unidad de destino\(su SSD o NVMe con el adaptador usb3\) en su ordenador y abra el "Raspberry Pi Imager".**

* **Haz clic en "CHOOSE OS" y luego en "Use custom" elija el archivo de imagen Raspbian que haya descargado.**
* **Luego, haz clic en el "CHOSE SD" y busca la tarjeta SD que has introducido en la computadora.**
* **Ahora, el botón "WRITE" aparecerá y puedes hacer clic en él para comenzar a escribir/verificar el sistema operativo en la tarjeta SD.**
* **Finalmente, una vez haya terminado el proceso de escritura/verificación, verás una ventana emergente que dice que el sistema operativo se ha escrito con éxito en la tarjeta SD, haz clic en "CONTINUE" y retira tu tarjeta SD de la computadora.**

![](../../.gitbook/assets/image-2-.png)

## Arranque & Configuración

Inserta la SSD en uno de los puertos azules de usb3. Luego inserta el HDMI, Keyboard, Mouse, Ethernet y fuente de energía.

{% hint style="danger" %}
Los primeros Pi4's a enviar no arrancan desde USB3 por defecto, hoy en día ya lo hacen. Si tu imagen no arranca los dos problemas más comunes son que el firmware es antiguo en tu Raspberry Pi o un adaptador USB3 incompatible.
{% endhint %}

![](../../.gitbook/assets/pi4.jpeg)

{% hint style="info" %}
Todo lo que realmente necesitamos hacer aquí es desactivar el inicio de sesión automático & crear el usuario ada con privilegios sudo. Después de volver a iniciar sesión, eliminaremos el usuario predeterminado de Pi y configuraremos el servidor & entorno para el nodo cardano & cardano-cli.
{% endhint %}

![Abra la utilidad de configuración de Raspberry Pi.](../../.gitbook/assets/raspberrypi-configuration.png)

![Establecer inicio automático deshabilitado](../../.gitbook/assets/disable-auto-login.png)

### Crear el usuario ada

```text
sudo adduser ada && sudo adduser ada sudo
```

Avanza y reinicia, inicia sesión como tu nuevo usuario ada.

```text
sudo reboot
```

