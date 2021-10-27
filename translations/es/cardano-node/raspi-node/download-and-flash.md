# Descargar y Flashear

## Instalar el sistema operativo RaspiNode

**1. Download the Armada Alliance's pre-configured Raspbian 64bit OS Cardano-node image** [**here** ](https://mainnet.adamantium.online/Raspi-Node.img.gz)**and save it in an accessible location for now on your computer.**

{% hint style="warning" %}
There is a testnet image you can download [here](https://testnet.adamantium.online/Test-Raspi-node.img.gz). May need binaries upgraded.
{% endhint %}

**2. A continuación, descarga el software de Raspberry Pi Imager que utilizaremos para grabar la imagen del sistema operativo en nuestro disco de destino. Este software se encuentra en el sitio web** [**Raspberry Pi**](https://www.raspberrypi.org/software/)**. Por favor, descargue la versión correcta para su computadora.**

![](../../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Insert the target drive(your SSD with usb3 adapter) into your computer and open the "Raspberry Pi Imager".**

![](../../.gitbook/assets/custom\_os.png)

* **Click on "CHOOSE OS"  then "Use custom" choose the Raspbian image file you downloaded.**&#x20;
* **Luego, haz clic en el "CHOSE SD" y busca la tarjeta SD que has introducido en la computadora.**
* **The "WRITE" button will appear and you can click on it to begin writing/verifying the OS onto the target drive.** &#x20;
* **Finally, once it has finished the writing/verifying process, you will see a pop-up window saying that the OS was successfully written to the drive, click "CONTINUE" and remove your drive from the computer.**&#x20;

## Arranque & Configuración

Inserta la SSD en uno de los puertos azules de usb3. Luego inserta el HDMI, Keyboard, Mouse, Ethernet y fuente de energía.

{% hint style="danger" %}
Los primeros Pi4's a enviar no arrancan desde USB3 por defecto, hoy en día ya lo hacen. Si tu imagen no arranca los dos problemas más comunes son que el firmware es antiguo en tu Raspberry Pi o un adaptador USB3 incompatible.
{% endhint %}

![](../../../.gitbook/assets/pi4 (1).jpeg)

{% hint style="info" %}
Todo lo que realmente necesitamos hacer aquí es desactivar el inicio de sesión automático & crear el usuario ada con privilegios sudo. Después de volver a iniciar sesión, eliminaremos el usuario predeterminado de Pi y configuraremos el servidor & entorno para el nodo cardano & cardano-cli.
{% endhint %}

![](../../../.gitbook/assets/raspberrypi-configuration.png)

![](../../../.gitbook/assets/disable-auto-login.png)

## Crear el usuario ada

```
sudo adduser ada && sudo adduser ada sudo
```

Avanza y reinicia, inicia sesión como tu nuevo usuario ada.

```
sudo reboot
```
