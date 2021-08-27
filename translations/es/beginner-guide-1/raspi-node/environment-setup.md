---
description: Instalar los paquetes necesarios para ejecutar cardano-node y configurar nuestro entorno
---

# Configuración del entorno

![](../../.gitbook/assets/download-10-%20%281%29%20%282%29.jpeg)

## Instalar paquetes

Instala los paquetes que necesitaremos.

```bash
sudo apt install build-essential libssl-dev tcptraceroute python3-pip \
         jq make automake unzip net-tools nginx ssl-cert pkg-config \
         libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev \
         zlib1g-dev g++ libncursesw5 libtool autoconf -y
```

## Entorno

Crear algunos directorios.

```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/pi-pool/files
mkdir -p $HOME/pi-pool/scripts
mkdir -p $HOME/pi-pool/logs
mkdir $HOME/git
mkdir $HOME/tmp
```

### Crear variables de bash & añade ~/.local/bin a nuestro $PATH🏃

{% hint style="info" %}
[Variables de entorno en Linux/Unix](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells/247769#247769).
{% endhint %}

{% hint style="warning" %}
Los cambios a este archivo requieren recargar .bashrc o cerrar sesión y volver a entrar.
{% endhint %}

{% tabs %}
{% tab title="Testnet" %}
```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
echo export NODE_HOME=$HOME/pi-pool >> $HOME/.bashrc
echo export NODE_CONFIG=mainnet >> $HOME/.bashrc
echo export NODE_FILES=$HOME/pi-pool/files >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$HOME/pi-pool/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
echo export NODE_HOME=$HOME/pi-pool >> $HOME/.bashrc
echo export NODE_CONFIG=mainnet >> $HOME/.bashrc
echo export NODE_FILES=$HOME/pi-pool/files >> $HOME/.bashrc
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> $HOME/.bashrc
echo export CARDANO_NODE_SOCKET_PATH="$HOME/pi-pool/db/socket" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}
{% endtabs %}

### Recuperar archivos del nodo

```bash
cd $NODE_FILES
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
```

Ejecutar lo siguiente para modificar testnet-config.json y actualizar TraceBlockFetchDecisions a "true"

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g"
```

{% hint style="info" %}
**Consejo para nodos Relay**: Es posible reducir el uso de memoria y cpu estableciendo "TraceMemPool" en "false" en **mainnet-config.json.** Esto desactivará los datos de mempool en Grafana y gLiveView.sh.
{% endhint %}

### Recuperar binarios aarch64

{% hint style="info" %}
Los **binarios de cardano** no oficiales & cardano-cli disponibles para nosotros están siendo construidos por un ingeniero de IOHK en su **tiempo libre**. Visita el grupo '[Arming Cardano](https://t.me/joinchat/FeKTCBu-pn5OUZUz4joF2w)' Telegram para más información.
{% endhint %}

```bash
cd $HOME/tmp
wget -O cardano_node_$(date +"%m-%d-%y").zip https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
unzip *.zip
mv cardano-node/* $HOME/.local/bin
rm -r cardano*
cd $HOME
```

{% hint style="warning" %}
Si ya existen binarios tendrás que confirmar la sobreescritura de los antiguos.
{% endhint %}

Confirma que los binarios están en ada $PATH.

```bash
cardano-node version
cardano-cli version
```

### Creación de Systemd

Vamos a crear ahora el archivo systemd y su script de arranque para que systemd pueda gestionar cardano-node.

```bash
nano $HOME/.local/bin/cardano-service
```

Pega lo siguiente, guardar & salir.

{% tabs %}
{% tab title="Testnet" %}
```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3003
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/testnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/testnet-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node +RTS -N4 --disable-delayed-os-memory-return -qg -qb -c -RTS run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3003
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node +RTS -N4 --disable-delayed-os-memory-return -qg -qb -c -RTS run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}
```
{% endtab %}
{% endtabs %}

Permitir la ejecución de nuestro nuevo script de arranque.

```bash
chmod +x $HOME/.local/bin/cardano-service
```

Abre /etc/systemd/system/cardano-node.service

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

Pega lo siguiente, guardar & salir.

```bash
# The Cardano Node Service (part of systemd)
# file: /etc/systemd/system/cardano-node.service

[Unit]
Description     = Cardano node service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = ada
Type            = simple
WorkingDirectory= /home/ada/pi-pool
ExecStart       = /bin/bash -c "PATH=/home/ada/.local/bin:$PATH exec /home/ada/.local/bin/cardano-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=3
LimitNOFILE=32768
Restart=always
RestartSec=5
#EnvironmentFile=-/home/ada/.pienv

[Install]
WantedBy= multi-user.target
```

Establezca los permisos y vuelva a cargar systemd para que recoja nuestro nuevo archivo del servicio.

```bash
sudo systemctl daemon-reload
```

Vamos a añadir una función al fondo de nuestro archivo .bashrc para hacer nuestra vida un poco más fácil.

```bash
nano $HOME/.bashrc
```

```bash
cardano-service() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-node.service
}
```

Guardar y salir

```bash
source $HOME/.bashrc
```

Lo que acabamos de hacer fue añadir una función para controlar nuestro cardano-service sin tener que escribir:

> > sudo systemctl enable cardano-node.service sudo systemctl start cardano-node.service sudo systemctl stop cardano-node.service sudo systemctl status cardano-node.service

Ahora sólo tenemos que hacer:

* cardano-service enable \\(habilita cardano-node.service con autoejecución al arrancar\\)
* cardano-service start      \(inicia cardano-node.service\)
* cardano-service stop       \(para cardano-node.service\)
* cardano-service status    \(muestra el estado de cardano-node.service\)

## ⛓ Sincronización de la cadena ⛓

Ahora estás listo para empezar a usar cardano-node. Hacer esto iniciará el proceso de "sincronización de la cadena". Esto tomará alrededor de 30 horas y porque la carpeta db tiene unos 8,5GB de tamaño en este momento. Si tenemos un nodo y sincronizado podemos copiar de ese nodo la carpeta db a nuestros nuevo nodo para ahorrar tiempo.

### Descargar la instantánea

{% hint style="danger" %}
No intente esto con una tarjeta sd de 8GB. ¡No hay suficiente espacio! [Crea tu archivo de la imagen](https://app.gitbook.com/@wcatz/s/pi-pool-guide/create-.img-file) y flashealo en tu ssd.
{% endhint %}

He empezado a tomar instantáneas de mi carpeta db de backup y la he alojado en un directorio web. Con este método tarda unos 15 minutos en tirar la última instantánea y tal vez otros 30 minutos en sincronizar hasta el final de la cadena. El servicio se presta según lo establecido. Depende de ti. Si quieres sincronizar la cadena por tu cuenta simplemente:

```bash
cardano-service enable
cardano-service start
cardano-service status
```

De lo contrario, asegúrate de que tu nodo **no** se está ejecutando & borra la carpeta db si existe y descargue db.

```bash
cardano-service stop
cd $NODE_HOME
rm -r db/
```

Descargar la instantánea de la base de datos.

{% tabs %}
{% tab title="Testnet" %}
```bash
wget -r -np -nH -R "index.html*" -e robots=off https://test-db.adamantium.online/db/
```
{% endtab %}

{% tab title="Mainnet" %}
```bash
wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/
```
{% endtab %}
{% endtabs %}

Una vez que wget se haya completado habilitar & iniciar cardano-nodo.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## gLiveView.sh

Los scripts de Guild Operators tienen un par de herramientas útiles para operar en el Pool. No queremos que el proyecto como un todo, aunque hay un par de scripts que vamos a usar.

{% embed url="https://github.com/cardano-community/guild-operators/tree/master/scripts/cnode-helper-scripts" caption="" %}

```bash
cd $NODE_HOME/scripts
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
```

Tenemos que editar el archivo env para trabajar con nuestro entorno. El número de puerto aquí tendrá que ser actualizado para que coincida con el puerto en el que se está ejecutando el nodo cardano-nodo. Para el **Pi-Node** es el puerto 3003. A medida que construyamos el Pool iremos adaptándolo. Por ejemplo Pi-Relay\(2\) se ejecutará en el puerto 3002, Pi-Relay\(1\) en 3001 y Pi-Core en el puerto 3000.

{% hint style="info" %}
Puedes cambiar la ejecución del puerto de cardano-node en /home/ada/.local/bin/cardano-service.
{% endhint %}

```bash
sed -i env \
    -e "s/\#CNODE_HOME=\"\/opt\/cardano\/cnode\"/CNODE_HOME=\"\home\/ada\/pi-pool\"/g" \
    -e "s/"6000"/"3001"/g" \
    -e "s/\#CONFIG=\"\${CNODE_HOME}\/files\/config.json\"/CONFIG=\"\${NODE_FILES}\/${NODE_CONFIG}-config.json\"/g" \
    -e "s/\#SOCKET=\"\${CNODE_HOME}\/sockets\/node0.socket\"/SOCKET=\"\${NODE_HOME}\/db\/socket\"/g"
```

Permitir la ejecución de gLiveView.sh.

```bash
chmod +x gLiveView.sh
```

## topologyUpdater.sh

Hasta que el peer to peer no esté habilitado en los operadores de red se necesita una forma de obtener una lista de relays/peers a los que conectarse. El servicio de actualizador de topología (topology updater) se ejecuta en segundo plano con cron. Cada hora el script se ejecutará y le dirá al servicio que eres un relay y quieres ser parte de la red. Añadirá tu relay a su directorio después de cuatro horas y comenzará a generar una lista de relays en un archivo json en el directorio $NODE\_HOME/logs. Un segundo script, relay-topology\_pull.sh puede ser usado manualmente para generar un archivo mainnet-topolgy con relays/peers que sean relacionados directamente entre tú y ellos.

{% hint style="info" %}
La lista generada te mostrará la distancia en millas &  para saber dónde se encuentra ubicado el relay.
{% endhint %}

Abrir un archivo llamado topologyUpdater.sh

```bash
cd $NODE_HOME/scripts
nano topologyUpdater.sh
```

Pega lo siguiente, guardar & salir.

{% hint style="warning" %}
El número de puerto aquí tendrá que ser actualizado para que coincida con el puerto en el que se está ejecutando el nodo cardano-nodo. Si estás utilizando registros dns puede agregar el FQDN que coincide con la línea 6\(línea 6 solamente\). Déjalo como si no estuviera usando dns. El servicio recogerá la IP pública y la utilizará.
{% endhint %}

```bash
#!/bin/bash
# shellcheck disable=SC2086,SC2034

USERNAME=ada
CNODE_PORT=3003 # must match your relay node port as set in the startup command
CNODE_HOSTNAME="CHANGE ME"  # optional. must resolve to the IP you are requesting from
CNODE_BIN="/home/ada/.local/bin"
CNODE_HOME="/home/ada/pi-pool"
LOG_DIR="${CNODE_HOME}/logs"
GENESIS_JSON="${CNODE_HOME}/files/testnet-shelley-genesis.json"
NETWORKID=$(jq -r .networkId $GENESIS_JSON)
CNODE_VALENCY=1   # optional for multi-IP hostnames
NWMAGIC=$(jq -r .networkMagic < $GENESIS_JSON)
[[ "${NETWORKID}" = "Mainnet" ]] && HASH_IDENTIFIER="--mainnet" || HASH_IDENTIFIER="--testnet-magic ${NWMAGIC}"
[[ "${NWMAGIC}" = "1097911063" ]] && NETWORK_IDENTIFIER="--mainnet" || NETWORK_IDENTIFIER="--testnet-magic ${NWMAGIC}"

export PATH="${CNODE_BIN}:${PATH}"
export CARDANO_NODE_SOCKET_PATH="${CNODE_HOME}/db/socket"

blockNo=$(/home/ada/.local/bin/cardano-cli query tip ${NETWORK_IDENTIFIER} | jq -r .block )

# Note:
# if you run your node in IPv4/IPv6 dual stack network configuration and want announced the
# IPv4 address only please add the -4 parameter to the curl command below  (curl -4 -s ...)
if [ "${CNODE_HOSTNAME}" != "CHANGE ME" ]; then
  T_HOSTNAME="&hostname=${CNODE_HOSTNAME}"
else
  T_HOSTNAME=''
fi

if [ ! -d ${LOG_DIR} ]; then
  mkdir -p ${LOG_DIR};
fi

curl -s -f -4 "https://api.clio.one/htopology/v1/?port=${CNODE_PORT}&blockNo=${blockNo}&valency=${CNODE_VALENCY}&magic=${NWMAGIC}${T_HOSTNAME}" | tee -a "${LOG_DIR}"/topologyUpdater_lastresult.json
```

Guardar, salir y hacerlo ejecutable.

```bash
chmod +x topologyUpdater.sh
```

{% hint style="warning" %}
No podrá ejecutar con éxito ./topologyUpdater.sh hasta que esté completamente sincronizado hasta el final de la cadena.
{% endhint %}

{% hint style="info" %}
Elegir nano cuando se te pida que editor quieres usar.
{% endhint %}

Crea una tarea de cron (cron job) que ejecutará el script cada hora.

```bash
crontab -e
```

Añadir lo siguiente en la parte inferior, guardar & salir.

{% hint style="info" %}
La imagen de Pi-Node tiene esta línea de cron deshabilitada por defecto. Puedes activarla eliminando el \#.
{% endhint %}

```bash
33 * * * * /home/ada/pi-pool/scripts/topologyUpdater.sh
```

Después de 4 horas corriendo se añadirá al servicio y podrá llevar su nueva lista de peers al archivo de topología principal.

Crear otro archivo relay-topology\_pull.sh y pegar en lo siguiente.

```bash
nano relay-topology_pull.sh
```

```bash
#!/bin/bash
BLOCKPRODUCING_IP=<BLOCK PRODUCERS PRIVATE IP>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o /home/ada/pi-pool/files/testnet-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&magic=1097911063&customPeers=${BLOCKPRODUCING_IP}:${BLOCKPRODUCING_PORT}:1"
```

Guardar, salir y hacerlo ejecutable.

```bash
chmod +x relay-topology_pull.sh
```

{% hint style="danger" %}
Al arrastrar una nueva lista se sobrescribirá tu archivo de topología existente. Ten esto en cuenta.
{% endhint %}

Después de 4 horas puede instalar su nueva lista y reiniciar el servicio de cardano.

```bash
cd $NODE_HOME/scripts
./relay-topology_pull.sh
```

{% hint style="info" %}
relay-topology\_pull.sh añadirá 15 peers a tu archivo mainnet-topology. Normalmente retiro los 5 relays más lejanos y uso los 10 más cercanos.
{% endhint %}

```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```

{% hint style="info" %}
Puedes usar gLiveView.sh para ver qué tiempos de ping tienes en relación a los peers de tu archivo mainnet-topology. Usa Ping para resolver el hostname de la IP.
{% endhint %}

Changes to this file will take affect upon restarting the cardano-service.

{% hint style="warning" %}
Don't forget to remove the last comma in your topology file!
{% endhint %}

Status should show as enabled & running.

Once your node syncs past epoch 208\(shelley era\) you can use gLiveView.sh to monitor.

{% hint style="danger" %}
It can take up to an hour for cardano-node to sync to the tip of the chain. Use ./gliveView.sh, htop and log outputs to view process. Be patient it will come up.
{% endhint %}

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

![](../../.gitbook/assets/pi-node-glive%20%282%29.png)

## Prometheus, Node Exporter & Grafana

Prometheus connects to cardano-nodes backend and serves metrics over http. Grafana in turn can use that data to display graphs and create alerts. Our Grafana dashboard will be made up of data from our Ubuntu system & cardano-node. Grafana can display data from other sources as well, like [adapools.org](https://adapools.org/).

{% hint style="info" %}
You can connect a Telegram bot to Grafana which can alert you of problems with the server. Much easier than trying to configure email alerts.
{% endhint %}

{% embed url="https://github.com/prometheus" caption="" %}

![](../../.gitbook/assets/pi-pool-grafana%20%282%29%20%282%29%20%282%29%20%282%29%20%281%29%20%282%29.png)

### Install Prometheus & Node Exporter.

{% hint style="info" %}
Prometheus can scrape the http endpoints of other servers running node exporter. Meaning Grafana and Prometheus does not have to be installed on your core and relays. Only the package prometheus-node-exporter is required if you would like to build a central Grafana dashboard for the pool, freeing up resources.
{% endhint %}

```bash
sudo apt-get install -y prometheus prometheus-node-exporter
```

Disable them in systemd for now.

```bash
sudo systemctl disable prometheus.service
sudo systemctl disable prometheus-node-exporter.service
```

### Configure Prometheus

Open prometheus.yml.

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Replace the contents of the file with.

{% hint style="warning" %}
Indentation must be correct YAML format or Prometheus will fail to start.
{% endhint %}

```yaml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label job=<job_name> to any timeseries scraped from this config.
  - job_name: 'Prometheus' # To scrape data from the cardano node
    scrape_interval: 5s
    static_configs:
#      - targets: ['<CORE PRIVATE IP>:12798']
#        labels:
#          alias: 'C1'
#          type:  'cardano-node'
#      - targets: ['<RELAY PRIVATE IP>:12798']
#        labels:
#          alias: 'R1'
#          type:  'cardano-node'
      - targets: ['localhost:12798']
        labels:
          alias: 'N1'
          type:  'cardano-node'

#      - targets: ['<CORE PRIVATE IP>:9100']
#        labels:
#          alias: 'C1'
#          type:  'node'
#      - targets: ['<RELAY PRIVATE IP>:9100']
#        labels:
#          alias: 'R1'
#          type:  'node'
      - targets: ['localhost:9100']
        labels:
          alias: 'N1'
          type:  'node'
```

Save & exit.

Edit mainnet-config.json so cardano-node exports traces on all interfaces.

```bash
cd $NODE_FILES
sed -i ${NODE_CONFIG}-config.json -e "s/127.0.0.1/0.0.0.0/g"
```

### Install Grafana

{% embed url="https://github.com/grafana/grafana" caption="" %}

Add Grafana's gpg key to Ubuntu.

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Add latest stable repo to apt sources.

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Update your package lists & install Grafana.

```bash
sudo apt update
sudo apt install grafana
```

Change the port Grafana listens on so it does not clash with cardano-node.

```bash
sudo sed -i /etc/grafana/grafana.ini \
-e "s/;http_port/http_port/" \
-e "s/3000/5000/"
```

### cardano-monitor bash function

Open .bashrc.

```bash
cd $HOME
nano .bashrc
```

Down at the bottom add.

```bash
cardano-monitor() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" prometheus.service
    sudo systemctl "$1" prometheus-node-exporter.service
    sudo systemctl "$1" grafana-server.service
}
```

Save, exit & source.

```bash
source .bashrc
```

Here we tied all three services under one function. Enable Prometheus.service, prometheus-node-exporter.service & grafana-server.service to run on boot and start the services.

```bash
cardano-monitor enable
cardano-monitor start
```

{% hint style="warning" %}
At this point you may want to start cardano-service and get synced up before we continue to configure Grafana. Skip ahead to [syncing the chain section](https://app.gitbook.com/@wcatz/s/pi-pool-guide/~/drafts/-MYFtFDZp-rTlybgAO71/pi-node/environment-setup/@drafts#syncing-the-chain). Choose whether you want to wait 30 hours or download my latest chain snapshot. Return here once gLiveView.sh shows you are at the tip of the chain.
{% endhint %}

### Configure Grafana

On your local machine open your browser and got to [http://&lt;Pi-Node's](http://<Pi-Node's) private ip&gt;:5000

Log in and set a new password. Default username and password is **admin:admin**.

#### Configure data source

In the left hand vertical menu go to **Configure** &gt; **Datasources** and click to **Add data source**. Choose Prometheus. Enter [http://localhost:9090](http://localhost:9090) where it is grayed out, everything can be left default. At the bottom save & test. You should get the green "Data source is working" if cardano-monitor has been started. If for some reason those services failed to start issue **cardano-service restart**.

#### Import dashboards

Save the dashboard json files to your local machine.

{% embed url="https://github.com/armada-alliance/dashboards" caption="" %}

In the left hand vertical menu go to **Dashboards** &gt; **Manage** and click on **Import**. Select the file you just downloaded/created and save. Head back to **Dashboards** &gt; **Manage** and click on your new dashboard.

![](../../.gitbook/assets/pi-pool-grafana%20%282%29%20%282%29%20%282%29%20%282%29%20%281%29.png)

### Configure poolDataLive

Here you can use the poolData api to bring your pools data into Grafana.

{% embed url="https://api.pooldata.live/dashboard" caption="" %}

Follow the instructions to install the Grafana plugin, configure your datasource and import the dashboard.

Follow log output to journal.

```bash
sudo journalctl --unit=cardano-node --follow
```

Follow log output to stdout.

```bash
sudo tail -f /var/log/syslog
```

## Grafana, Nginx proxy\_pass & snakeoil

Let's put Grafana behind Nginx with self signed\(snakeoil\) certificate. The certificate was generated when we installed the ssl-cert package.

You will get a warning from your browser. This is because ca-certificates cannot follow a trust chain to a trusted \(centralized\) source. The connection is however encrypted and will protect your passwords flying around in plain text.

```bash
sudo nano /etc/nginx/sites-available/default
```

Replace contents of the file with below.

```bash
# Default server configuration
#
server {
        listen 80 default_server;
        return 301 https://$host$request_uri;
}

server {
        # SSL configuration
        #
        listen 443 ssl default_server;
        #listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        include snippets/snakeoil.conf;

        add_header X-Proxy-Cache $upstream_cache_status;
        location / {
          proxy_pass http://127.0.0.1:5000;
          proxy_redirect      off;
          include proxy_params;
        }
}
```

Check that Nginx is happy with our changes and restart it.

```bash
sudo nginx -t
## if ok do
sudo service nginx restart
```

You can now visit your pi-nodes ip address without any port specification, the connection will be upgraded to SSL/TLS and you will get a scary message\(not really scary at all\). Continue through to your dashboard.

![](../../.gitbook/assets/snakeoil.png)

From here you have a pi-node with tools to build a stake pool from the following pages. Best of Luck and please join the [armada-alliance](https://armada-alliance.com), together we are stronger!

