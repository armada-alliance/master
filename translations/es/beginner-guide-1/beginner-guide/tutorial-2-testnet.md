---
description: >-
  Después de completar la configuración de la Raspberry Pi, ya estamos listos para descargar los archivos necesarios para Testnet (red de pruebas).
---

# Configurar un nodo en Testnet

{% hint style="warning" %}
**¡Este tutorial está diseñado para conseguir una sincronización de un nodo completo con el blockchain de Cardano! Hemos omitido ciertos pasos de seguridad para hacer este tutorial lo más fácil posible - NO UTILICE este tutorial para crear un Stake Pool de Mainnet. Por favor, utilice nuestras**[ **guías intermedias**](../../intermediate-guide/pi-pool-tutorial/pi-node/) **para la Mainnet.**
{% endhint %}

{% hint style="warning" %}
**Este tutorial es sólo para usar con Raspberry Pi OS 64bit y está pensado sólo para propósitos educativos para conseguir una sincronización de un nodo completo con la cadena de bloques de Cardano.**
{% endhint %}

## Resumen

1. Configuración de entorno
2. Descargando los binarios necesarios para construir un nodo Relay de Cardano
3. Descargar archivos de configuración de IOHK/Cardano-node
4. Editar los ajustes de configuración
5. Descargar una instantánea (snapshot) de la base de datos para acelerar el proceso de sincronización
6. Ejecuta el nodo Relay básico pasivo para conectarse a  Testnet (red de pruebas)
7. Monitorea el nodo Relay con [**Guild Operators gLiveView** ](https://cardano-community.github.io/guild-operators/#/)

![](../../.gitbook/assets/download-10-%20%281%29.jpeg)

{% hint style="info" %}
Este tutorial puede ser utilizado en **mainnet** si lo desea. Simplemente reemplace en todas las instancias la palabra "**testnet**" por "**mainne**t" durante este tutorial.
{% endhint %}

## Configurando tu entorno

* Primero debemos actualizar nuestro sistema operativo e instalar las actualizaciones necesarias si están disponibles.

{% hint style="info" %}
Es altamente recomendable actualizar el sistema operativo cada vez que arranca e inicie sesión en su **Raspberry Pi** para prevenir vulnerabilidades de seguridad.
{% endhint %}

```text
# Estamos usando el prefijo sudo para ejecutar comandos como no-root-user  

sudo apt update
sudo apt upgrade -y
```

* Ahora podemos reiniciar el Pi y dejar que las actualizaciones surtan efecto ejecutando este comando en una terminal.

```text
sudo reboot
```

### Crear nuestros directorios

```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/testnet-relay/files
```

### Añadir ~/.local/bin a nuestro $PATH

{% hint style="info" %}
[Cómo añadir un directorio a su $PATH en Linux](https://www.howtogeek.com/658904/how-to-add-a-directory-to-your-path-in-linux/)
{% endhint %}

```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
```

### Crear nuestras variables bash

```bash
echo export NODE_HOME=$HOME/testnet-relay >> $HOME/.bashrc
echo export NODE_FILES=$HOME/testnet-relay/files >> $HOME/.bashrc
echo export NODE_CONFIG=testnet >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```

```bash
sudo reboot
```

### Descarga la versión estática de Cardano-node

| Proporcionado por                                                                                                                | Enlace a Cardano Static Build                                                                          |
|:-------------------------------------------------------------------------------------------------------------------------------- |:------------------------------------------------------------------------------------------------------ |
| [**ZW3RK**](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) **1PCT Haskell CI Support Pool** | \*\*\*\*[**https://ci.zw3rk.com/build/1755**](https://ci.zw3rk.com/build/1755)\*\*\*\* |

* Una[ **Static Build (compilación estática)**](https://en.wikipedia.org/wiki/Static_build) es una versión ****[**compilada**](https://en.wikipedia.org/wiki/Compiler) **** de un programa que ha sido enlazado estáticamente con las bibliotecas.

Ahora necesitamos simplemente descargar el archivo zip de arriba al directorio de inicio de nuestro Pi y luego moverlo a la ubicación correcta para que podamos ejecutarlo más adelante para iniciar el nodo.

```bash
# Primero cambie al directorio de inicio
cd $HOME

# Ahora podemos descargar el nodo de cardano- 
con https://ci.zw3rk.com/build/1755/download/1/aarch64-unknown-linux-musl-cardano-node-1.26.2.zip
```

* Usa el comando[**unzip**](https://linux.die.net/man/1/unzip) para el archivo zip descargado y extrae su contenido.

  ```bash
  unzip aarch64-unknown-linux-musl-cardano-node-1.26.1.zip
  ```

* A continuación, tenemos que asegurarnos de que la carpeta "cardano-node" y su contenido están correctos.

{% hint style="info" %}
Si no estás seguro si el archivo se ha descargado correctamente o necesita el nombre de la carpeta/archivos, podemos usar el comando de Linux [**ls**](https://www.man7.org/linux/man-pages/man1/ls.1.html).
{% endhint %}

Ahora necesitamos mover la carpeta cardano-node a nuestro directorio binario local.

```bash
mv cardano-node/* ~/.local/bin
```

Antes de continuar, asegurémonos de que el nodo cardano y el cli estén en nuestro $PATH

```bash
cardano-node version
cardano-cli version
```

Ahora podemos entrar a nuestra carpeta de archivos, y descargue los cuatro archivos de configuración del nodo Cardano que necesitamos desde el sitio web oficial [IOHK](https://hydra.iohk.io/build/5822084/download/1/index.html) y o [documentación](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html). Utilizaremos el comando "wget" para descargar los archivos.

```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
```

* **Utiliza el editor nano bash para cambiar algunas cosas en nuestro archivo "testnet-config.json"**
* [ ] Cambie la línea **"TraceBlockFetchDecisions"** de "**false**" a "**true**"
* [ ] Cambie el **"hasEKG"** por **12600**
* [ ] Cambiar en **"hasPrometheus"** por la dirección/puerto a 12700

```text
sudo nano testnet-config.json
```

### Crear los archivos del sistema

Utilizaremos el gestor de servicios del sistema linux para manejar el inicio, parado y reinicio nuestro del nodo Relay de Cardano.

{% hint style="info" %}
Si quieres saber más sobre el sistema Linux ve a la página del manual de Linux.[https://www.man7.org/linux/man-pages/man1/systemd.1.html](https://www.man7.org/linux/man-pages/man1/systemd.1.html)
{% endhint %}

```bash
sudo nano $HOME/.local/bin/cardano-service
```

**Ahora necesitamos hacer el script de inicio de cardano-node**

{% hint style="info" %}
Cómo iniciar el cardano-node se puede encontrar aquí en la documentación de Cardano.[https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html)
{% endhint %}

```bash
#!/bin/bash
DIRECTORY=/home/pi/testnet-relay
FILES=/home/pi/testnet-relay/files
PORT=3001
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/testnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/testnet-config.json

cardano-node run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```

**Ahora debemos dar permiso de acceso a nuestro nuevo script de servicio de systemd**

```bash
sudo chmod +x $HOME/.local/bin/cardano-service
```

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

```bash
# The Cardano Node Service (part of systemd)
# file: /etc/systemd/system/cardano-node.service 

[Unit]
Description     = Cardano node service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = pi
Type            = simple
WorkingDirectory= /home/pi/testnet-relay
ExecStart       = /bin/bash -c "PATH=/home/pi/.local/bin:$PATH exec /home/pi/.local/bin/cardano-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=3
LimitNOFILE=32768
Restart=always
RestartSec=5

[Install]
WantedBy= multi-user.target
```

Ahora deberíamos recargar nuestro servicio systemd para asegurarnos de que recoge nuestro nuevo cardano-service

```bash
sudo systemctl daemon-reload
```

**Si no queremos llamar a "sudo systemctl" cada vez que queremos empezar, parar, o reinicie el servicio cardano-node podemos crear una "función" que se añadirá a nuestro .bashrc shell script que hará eso por nosotros** [https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/](https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/)

```bash
nano $HOME/.bashrc
```

```bash
cardano-service() {
    sudo systemctl "$1" cardano-node.service
}
```

```bash
source $HOME/.bashrc
```

## Descargar una instantánea (snapshot) del blockchain para acelerar el proceso de sincronización

{% hint style="info" %}
Se nos ha proporcionado una instantánea (snapshot) de la base de datos de testnet gracias a Star Forge Pool \[OTG\]. Si no desea descargar la base de datos, **puedes saltar este paso**. Tenga cuidado, si se salta la descarga de nuestra instantánea, puede tardar hasta 8 horas en sincronizar completamente el nodo.
{% endhint %}

{% hint style="danger" %}
**Asegúrate de que no has iniciado un nodo de Cardano antes de continuar.**🛑
{% endhint %}

Primero, asegúrate de que cardano-service que creamos anteriormente está parado y luego ya podemos descargar la base de datos en nuestro Relay de pruebas/archivos. Puedes ejecutar los siguientes comandos para iniciar nuestra descarga.

```bash
# Asegúrate de que no tienes el cardano-node ejecutándose en segundo plano:
cardano-service stop
cd $NODE_HOME
# Elimina el antiguo db y su contenido si está presente
rm -r db/ 
#Descargar testnet db snapshot
wget -r -np -nH -R "index. tml*" -e robots=off https://test-db.adamantium.online/db/
```

{% hint style="info" %}
Esta descarga tomará entre 25 minutos y 2 horas dependiendo de tu velocidad de Internet.
{% endhint %}

* Después de que la base de datos haya terminado de descargar, es una buena idea añadir un archivo limpio a él antes de iniciar el Relay. Copia/pega el siguiente comando en la ventana de tu terminal.

```bash
touch db/clean
```

## Terminar de sincronizar con el blockchain

* Ahora podemos iniciar el nodo Relay "pasivo" para comenzar a sincronizar con la cadena de bloques.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## Configurando gLiveView para monitorear el nodo durante su proceso de sincronización

#### Ahora puedes irte a la carpeta $NODE\_FILES y luego descargar el servicio de monitoreo de gLiveView

```bash
cd $NODE_Files
curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
```

* Necesitas cambiar el "**CNODE\_PORT**al puerto que configuraste en tu cardano-node, en nuestro caso, vamos a cambiarlo a **3001.**

```bash
sudo nano env
```

* Por último, podemos salir del nano editor y ejecutar el script gLiveView.

```bash
./gLiveView.sh
```

{% hint style="success" %}
Si deseas controlar el rendimiento de tu Raspberry Pi puedes utilizar los siguientes comandos.
{% endhint %}

{% tabs %}
{% tab title="Get Cpu Temp" %}
```bash
vcgencmd measure_temp
```
{% endtab %}

{% tab title="Use htop for CPU and RAM Performance" %}
```bash
htop
```
{% endtab %}
{% endtabs %}

## Referencias:

{% tabs %}
{% tab title="📚" %}
{% embed url="https://github.com/wcatz/pi-pool" caption="" %}

{% embed url="https://github.com/alessandrokonrad/Pi-Pool" caption="" %}

{% embed url="https://github.com/angerman" caption="" %}

{% embed url="https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html" caption="" %}

{% embed url="https://cardano-community.github.io/guild-operators/\#/Scripts/gliveview" caption="" %}
{% endtab %}
{% endtabs %}

