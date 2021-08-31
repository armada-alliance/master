# Alpine Linux OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

## ¿Por qué usar AlpineOS en el Raspberry Pi? Estas son algunas razones:

* **Consumo de memoria muy bajo \(~50MB utilizado durante inactividad vs ~350MB para Ubuntu 20.04\).**
* **Baja sobrecarga de la CPU** **\(27 tareas/ 31 hilos activos para Alpine vs 57 tareas / 111 hilos para Ubuntu cuando se está ejecutando un nodo de Cardano\).**
* **Pi más frío 😎 \(Literalmente, la CPU se mantiene más fría debido a la menor sobrecraga de la misma\).**
* **Y por último, ¿por qué no? Si usas binarios estáticos, también podrías aprovechar AlpineOS 😜**

## If you have previously used this guide and intend to update the scripts. Follow these steps. Then follow the rest of the steps outlined in this guide accordingly 🙂.

1\) Update the git local repo.

```text
cd ~/alpine-rpi-os
```

```text
git fetch --recurse-submodules --tags --all
```

2\) Identify the latest tag.

```text
git tag
```

3\) Replace `<tag>` in this step with the latest tag such as `v1.2.1`.

```text
git checkout tags/<tag>
```

## Upgrading to Alpine v3.14 from Alpine v3.13:

1\) Update your current version of AlpineOS.

```text
sudo apk update
```

```text
sudo apk upgrade
```

2\) Edit the repository to reflect Alpine v3.14.

```text
sudo sed -i 's@v3.13@v3.14@g' /etc/apk/repositories
```

3\) Update the package list.

```text
sudo apk update
```

4\) Upgrading packages to v3.14

```text
sudo apk add --upgrade apk-tools
```

```text
sudo apk upgrade --available
```

```text
sudo sync
```

```text
sudo reboot now
```

5\) Now you should have AlpineOS upgraded to v3.14 🍷.

```text
cat /etc/alpine-release
```

6\) To troubleshoot the upgrade, refer to the link: [https://wiki.alpinelinux.org/wiki/Upgrading\_Alpine](https://wiki.alpinelinux.org/wiki/Upgrading_Alpine)

## Configuración inicial para AlpineOS en Raspberry Pi 4B 8GB:

1\) Download the AlpineOS for RPi 4 aarch64 here: [https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/aarch64/alpine-rpi-3.14.2-aarch64.tar.gz)

2\) Descomprime el archivo .tar.gz y copia su contenido en una tarjeta SSD/SD

3\) Conecta un teclado y un monitor.

4\) Iniciar sesión con el nombre de usuario 'root'.

5\) Ejecuta el comando `setup-alpine` y sigue las instrucciones.

{% hint style="info" %}
When you are in `setup-alpine` you will be prompted to choose the system disk. Una vez que esté en este punto, ingrese, `y`, para configurar el disco y crear la partición para `sys`.
{% endhint %}

6\) Reinicie el sistema.

7\) Añada un nuevo usuario llamado cardano mediante el comando `adduser cardano` y su contraseña como se indica.

8\) Ejecute los siguientes comandos para otorgar privilegios de root al nuevo usuario creado

```text
apk add sudo
echo '%wheel ALL=(ALL) ALL' > /etc/sudoers.d/wheel
addgroup cardano wheel
addgroup cardano sys
addgroup cardano adm
addgroup cardano root
addgroup cardano bin
addgroup cardano daemon
addgroup cardano disk
addgroup cardano floppy
addgroup cardano dialout
addgroup cardano tape
addgroup cardano video
```

9\) Salir del root mediante el comando `exit` o reiniciar el sistema e iniciar sesión en cardano

10\) Instalar bash para asegurar la compatibilidad de scripts bash

```text
    sudo apk add bash
```

11\) También instale git y wget, lo necesitaremos más adelante.

```text
    sudo apk add git wget
```

12\) By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. To use the ondemand governor which scales CPU frequency according to system load, `cpufreq.start` is included in this repo which should be added to /etc/local.d/. You may run the following commands to do this for you.

```text
    cd ~
```

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```text
    git tag
```

Replace `<tag>` with the latest tag in the next command.

```text
    git checkout tags/<tag>
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```text
    sudo chmod +x /etc/local.d/cpufreq.start
```

```text
    sudo rc-update add local default
```

12\) **\[CPU Governor - Optional\]** By default, AlpineOS uses the powersave governor which sets CPU frequency at the lowest. To use the ondemand governor which scales CPU frequency according to system load, `cpufreq.start` is included in this repo which should be added to /etc/local.d/. You may run the following commands to do this for you.

```text
    cd ~
```

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```text
    cd alpine-rpi-os
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/cpufreq.start /etc/local.d/
```

```text
    sudo chmod +x /etc/local.d/cpufreq.start
```

```text
    sudo rc-update add local default
```

13\) **\[ZRAM - Optional\]** To alleviate RAM limitation on RPi, ZRAM is recommended to enable RAM compression. Use the following steps to install zram-init and install the scripts. The scripts provided will enable a 50% boost in useable RAM capacity. This step assumes you have followed step 12.

```text
    sudo apk add zram-init
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/local.d/zram.* /etc/local.d/
```

```text
    sudo chmod +x /etc/local.d/zram.*
```

14\) Reboot the system. For the Raspberry Pi 4B 8GB, you should expect around 3.81GB of swap via ZRAM when checking with `htop` \(`sudo apk add htop` if htop is unavailable\).

## Instalando los binarios estáticos 'cardano-node' y 'cardano-cli' \(AlpineOS utiliza binarios estáticos casi exclusivamente así que debes evitar las compilaciones no estáticas\)

{% hint style="info" %}
**You can obtain the static binaries for version 1.29.0 via this** [**link**](https://ci.zw3rk.com/build/1758) **courtesy of Moritz Angermann, the SPO of ZW3RK pool 🙏**
{% endhint %}

**Ejecuta los siguientes comandos para instalar los binarios y colocarlos en el directorio correcto.**

* Descargar los binarios

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.29.0.zip https://ci.zw3rk.com/build/1771/download/1/aarch64-unknown-linux-musl-cardano-node-1.29.0.zip
```

* Descomprimir e instalar los binarios a través de los comandos

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.29.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Instalar el servicio de nodos Cardano Alpine Linux de Armada Alliance

{% hint style="success" %}
### Si usted ha decidido usar AlpineOS para las operaciones del stake pool de Cardano, encontrará esta colección de scripts y servicios útiles.
{% endhint %}

{% hint style="info" %}
### Para instalar correctamente los scripts y servicios no omitir los siguientes pasos 🏴‍☠️😎
{% endhint %}

1\) Clone this repo to obtain the neccessary folder and scripts to quickly start your cardano node. You may skip this step if you have already clonned this repo from step 12 when setting up AlpineOS.

```text
    cd ~
```

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

```text
    git tag
```

Replace `<tag>` with the latest tag in the next command.

```text
    git checkout tags/<tag>
```

2\) Ejecuta los siguientes comandos para instalar la carpeta **cnode**, scripts y servicios en las carpetas correctas. The **cnode** folder contains everything a **Cardano node** needs to start as a functional relay node.

```text
    cd ~
```

```text
    cp -r alpine-rpi-os/alpine_cnode_scripts_and_services/home/cardano/* ~/
```

```text
    sudo cp alpine-rpi-os/alpine_cnode_scripts_and_services/etc/init.d/* /etc/init.d/
```

```text
    chmod +x ~/start_stop_cnode_service.sh ~/cnode/autorestart_cnode.sh
```

```text
    sudo chmod +x /etc/init.d/cardano-node /etc/init.d/prometheus /etc/init.d/node-exporter
```

3\) Para una sincronización más rápida, considere este comando opcional para descargar la última carpeta db alojada por uno de nuestros miembros.

```text
    wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/ -P ~/cnode
```

4\) Siga la guía escrita en **README.txt** contenida en el directorio **$HOME** después de instalar **cnode**, scripts y servicios.

```text
    more ~/README.txt
```

## Configurar prometheus y el exportador de nodos

1\) Descargar Prometheus y node-exporter en el directorio de inicio

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.29.2/prometheus-2.29.2.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-arm64.tar.gz
```

2\) Extraer los tarballs

```text
tar -xzvf prometheus.tar.gz
```

```text
tar -xzvf node_exporter.tar.gz
```

3\) Renombrar las carpetas con los siguientes comandos

```text
    mv prometheus-2.29.2.linux-arm64 prometheus
```

```text
    mv node_exporter-1.2.2.linux-arm64 node_exporter
```

4\) Siga la guía escrita en README.txt contenida en el directorio $HOME después de instalar cnode, scripts y servicios para iniciar los servicios en correctamente.

```text
    more ~/README.txt
```

## Solución general de problemas

* Si tienes problemas con el reenvío de puertos a través de SSH, ejecuta el siguiente comando

```text
sudo nano /etc/ssh/sshd_config
```

* Editar la línea `AllowTcpForwarding no` a `AllowTcpForwarding sí`

{% hint style="info" %}
Asegúrate de que esta línea no esté comentada con un `#`
{% endhint %}

{% hint style="success" %}
Nos gustaría dar una mención especial a nuestro [miembro de la alianza](https://armada-alliance.com) Sayshar, operador de [\[SRN\] Pool](https://www.adasrn.com/), por proporcionar este tutorial 🏴‍☠️ 🙏 😎
{% endhint %}

