---
description: En este tutorial recorreremos la configuración básica de Raspberry Pi y Linux
---

# Configurando el Raspberry Pi

## Resumen <a id="h.vrhvb96nxxe9"></a>

1. Descargue un sistema operativo \(OS\). Para este tutorial, usaremos el sistema operativo de Raspberry Pi.
2. Instalar Raspberry Pi OS usando Raspberry Pi Imager
3. Flashear el sistema operativo en la tarjeta SD
4. Inicia el Pi y configura los ajustes
5. Inserte el disco SSD externo y copie la tarjeta SD a él
6. Apague y reinicie desde el disco SSD

{% hint style="info" %}
### NO SALTAR NINGÚN PASO
{% endhint %}

### **Parte Uno:**

### Instalando el sistema operativo «buster» de Raspberry Pi Debian <a id="h.lpv6ciisjqp3"></a>

Ahora vamos a descargar la última versión oficial de Debian OS de Raspberry Pi de 64 bits. Esta es la distribución oficial del sistema operativo Linux 64bit que está diseñada para Raspberry Pi y su CPU ARM64. Esto hace que sea estable y muy fácil empezar con el Raspberry Pi.

**1. Descarga la imagen del sistema operativo Raspberry Pi de 64 bits** [**aquí**](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-05-28/2021-05-07-raspios-buster-arm64.zip) **y guárdalo en una ubicación accesible, por ahora puede ser en su ordenador.**

**2. A continuación, descargue el software Raspberry Pi Imager que utilizaremos para instalar el sistema operativo en nuestro Raspberry Pi. Este software se encuentra en el sitio web** [**Raspberry Pi**](https://www.raspberrypi.org/software/)**. Por favor, descargue la versión correcta para su computadora.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Inserta la tarjeta SD en tu computadara y abre "Raspberry Pi Imager".**

* **Haz clic en "CHOSE OS" y luego encuentra la** _**raspios-buster-arm64. ip**_ **archivo que ha descargado en el paso \(1\) de este tutorial y selecciónelo.**
* **Luego, haz clic en el "CHOSE SD" y encuentra la tarjeta SD que has introducido en la computadora**
* **Ahora, el botón "Escribir" aparecerá y puedes hacer clic en él para comenzar a escribir o verificar el sistema operativo en la tarjeta SD.**
* **Finalmente, una vez que haya terminado el proceso de escritura/verificación, verás una ventana emergente que dice que el sistema operativo se ha escrito con éxito en la tarjeta SD, haz clic en "CONTINUE" y retira tu tarjeta SD de la computadora.**

{% hint style="info" %}
#### **Si todavía tiene problemas siguiendo las instrucciones escritas, vea el vídeo que está a continuación (en inglés).**
{% endhint %}

{% embed url="https://www.youtube.com/watch?v=ntaXWS8Lk34" %}



### Parte 2:

### Configurando el Raspberry Pi

Lo primero que queremos hacer es arrancar y configurar la Raspberry Pi para nuestro uso.

Para hacer esto, necesitaremos insertar la tarjeta SD, que flasheamos anteriormente con el sistema operativo Raspberry Pi, en la parte inferior de la Raspberry Pi. Luego podemos insertar nuestro HDMI, teclado, ratón y fuente de alimentación.

Una vez que arranca el sistema operativo Raspberry Pi, tendrá que seguir algunas configuraciones iniciales que se enumeran a continuación.

{% hint style="info" %}
Si esta es la primera vez que inicias el sistema operativo Raspberry Pi, tendrás que seguir algunas configuraciones iniciales listadas a continuación
{% endhint %}

* [ ] Primero, necesitas cambiar el nombre de host y contraseña de Raspberry Pi, esto te asegurará de que no sólo está ejecutando la información básica de acceso.
* [ ] Introduce la información de inicio de sesión Wifi \(**usted** **puede** **omitir esto si está usando Ethernet**\)
* Ajusta tu zona horaria local.
* [ ] Elige la configuración del idioma y el teclado.
* [ ] Actualiza la Raspberry Pi \(omite esto si desea actualizar a través de línea de comandos\)

{% hint style="success" %}
#### Después de haber terminado con estos pasos de configuración iniciales, es hora de proceder a que la Rasberry Pi arranque desde su USB de modo que podamos usar nuestra SSD externa.
{% endhint %}

### Parte 3:

### Configurando la Pi para Arrancar desde USB

**Este es el paso final de este tutorial. Primero vamos a insertar nuestra SSD externa en una de las entradas USB 3.0 marcadas en azul.**

![](../../.gitbook/assets/pi4.jpeg)

Abra el menú de aplicaciones Raspberry Pi y haga clic en la aplicación **SD Card Copier**.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-9.11.39-pm%20%281%29.png)

Luego seleccionamos **COPY FROM DEVICE** - **\(mmcblk0\) SD CARD.**

A continuación, selecciona **COPY TO DEVICE - \(sda\) SSD Device.**

Una vez completado el proceso de copia, abra una nueva ventana de la terminal y ejecuta el siguiente comando.

```text
sudo raspi-config
```

Esto te llevará a la configuración del sistema de Raspberry Pi donde podrás acceder a las **Opciones Avanzadas.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.19-pm.png)

Después selecciona **Boot Order.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.40-pm%20%281%29.png)

A continuación, elija **USB Boot**.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.14.05-pm%20%281%29.png)

Ahora puedes seleccionar **&lt;Ok&gt;** y luego **&lt;Finish&gt;**, para cerrar el menú de configuración del sistema de Raspberry Pi y reiniciar el Pi.

Ahora deberías ser capaz de apagar la Pi después de reiniciar, quitar la tarjeta SD y después encender la Pi, ya debería arrancar desde su dispositivo de almacenamiento USB externo.

{% hint style="success" %}
#### Ahora que hemos terminado la mayor parte de la configuración inicial, podemos continuar con la preparación de la Pi y pasar al siguiente [tutorial](tutorial-2-relaynode.md).
{% endhint %}

