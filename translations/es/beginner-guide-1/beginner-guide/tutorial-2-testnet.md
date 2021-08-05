---
description: >-
  Después de completar la configuración de la Raspberry Pi, ahora estamos listos para descargar los archivos necesarios para Testnet (red de pruebas).
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
2. Downloading the binaries needed to build a Cardano node relay
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

### Create our bash variables

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

### Download the Cardano-node static build

| Provided By                                                                                                                      | Link to Cardano Static Build                                                                           |
|:-------------------------------------------------------------------------------------------------------------------------------- |:------------------------------------------------------------------------------------------------------ |
| [**ZW3RK**](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) **1PCT Haskell CI Support Pool** | \*\*\*\*[**https://ci.zw3rk.com/build/1755**](https://ci.zw3rk.com/build/1755)\*\*\*\* |

* A[ **static build**](https://en.wikipedia.org/wiki/Static_build) is a ****[**compiled**](https://en.wikipedia.org/wiki/Compiler) ****version of a program that has been statically linked against libraries.

Now we need to simply download the zip file above to our Pi's home directory and then move it to the right location so we can call on it later to start the node.

```bash
# First change to the home directory
cd $HOME

# Now we can download the cardano-node 
wget https://ci.zw3rk.com/build/1755/download/1/aarch64-unknown-linux-musl-cardano-node-1.26.2.zip
```

* Use [**unzip**](https://linux.die.net/man/1/unzip) command on the downloaded zip file and extract its contents.

  ```bash
  unzip aarch64-unknown-linux-musl-cardano-node-1.26.1.zip
  ```

* Next, we need to make sure the newly downloaded "cardano-node" folder and its contents are present.

{% hint style="info" %}
If you are unsure if the file downloaded properly or need the name of the folder/files, we can use the Linux [**ls**](https://www.man7.org/linux/man-pages/man1/ls.1.html) command.
{% endhint %}

Now we need to move the cardano-node folder into our local binary directory.

```bash
mv cardano-node/* ~/.local/bin
```

Before we proceed let's make sure the cardano-node and cardano-cli is in our $PATH

```bash
cardano-node version
cardano-cli version
```

Now we can move in to our files folder, and download the four Cardano node configuration files we need from the official [IOHK website](https://hydra.iohk.io/build/5822084/download/1/index.html) and or [documentation](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html). We will be using our "wget" command to download the files.

```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
```

* **Use the nano bash editor to change a few things in our "testnet-config.json" file**
* [ ] Change the **"TraceBlockFetchDecisions"** line from "**false**" to "**true**"
* [ ] Change the **"hasEKG"** to **12600**
* [ ] Change  the **"hasPrometheus"** address/port to 12700

```text
sudo nano testnet-config.json
```

### Create the systemd files

We will use the linux systemd service manager to handle the starting, stoping, and restarting of our Cardano node relay.

{% hint style="info" %}
If you'd like to find out more about Linux systemd go to the Linux manual page.[https://www.man7.org/linux/man-pages/man1/systemd.1.html](https://www.man7.org/linux/man-pages/man1/systemd.1.html)
{% endhint %}

```bash
sudo nano $HOME/.local/bin/cardano-service
```

**Now we need to make the cardano-node startup script**

{% hint style="info" %}
How to start the cardano-node can be found here on the Cardano documentation.[https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html)
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

**Now we must give access permission to our new systemd service script**

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

We now should reload our systemd service to make sure it picks up our cardano-service

```bash
sudo systemctl daemon-reload
```

**If we don't want to call "sudo systemctl" everytime we want to start, stop, or restart the cardano-node service we can create a "function" that will be added into our .bashrc shell script that will do this for us** [https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/](https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/)

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

## Download a snapshot of the blockchain to speed the sync process

{% hint style="info" %}
We have been provided a snapshot of the testnet database thanks to Star Forge Pool \[OTG\]. If you don't want to download a database, **you may skip this step**. Beware, if you skip downloading our snapshot it may take up to 8 hours to get the node fully synced.
{% endhint %}

{% hint style="danger" %}
**Make sure you have not started a Cardano node before proceeding.** 🛑
{% endhint %}

First, make sure the cardano-service we created earlier is stopped, then we download the database in our testnet-relay/files. You can run the following commands to begin our download.

```bash
# Make sure you do not have the cardano-node running in the background
cardano-service stop
cd $NODE_HOME
# Remove old db and its contents if present
rm -r db/ 
#Download testnet db snapshot
wget -r -np -nH -R "index.html*" -e robots=off https://test-db.adamantium.online/db/
```

{% hint style="info" %}
This download will take anywhere from 25 min to 2 hours depending on your internet speeds.
{% endhint %}

* After the database has finished downloading, it is a good idea to add a clean file to it before we start the relay. Copy/paste the following command into your terminal window.

```bash
touch db/clean
```

## Finish syncing to the blockchain

* Now we can start the "passive" relay node to begin syncing to the blockchain.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## Setting up gLiveView to monitor the node during its syncing process

#### Now you can change to the $NODE\_FILES folder and then download the gLiveView monitor service

```bash
cd $NODE_Files
curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
```

* Need to change the "**CNODE\_PORT**" to the port you set on your cardano-node, in our case let's change it to **3001.**

```bash
sudo nano env
```

* Finally, we can exit the nano editor and just run the gLiveView script.

```bash
./gLiveView.sh
```

{% hint style="success" %}
If you want to monitor your Raspberry Pi performance you can use the following commands.
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

## References:

{% tabs %}
{% tab title="📚" %}
{% embed url="https://github.com/wcatz/pi-pool" caption="" %}

{% embed url="https://github.com/alessandrokonrad/Pi-Pool" caption="" %}

{% embed url="https://github.com/angerman" caption="" %}

{% embed url="https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html" caption="" %}

{% embed url="https://cardano-community.github.io/guild-operators/\#/Scripts/gliveview" caption="" %}
{% endtab %}
{% endtabs %}

