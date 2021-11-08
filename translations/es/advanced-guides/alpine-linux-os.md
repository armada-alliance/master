# Alpine Linux OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

### ¿Por qué usar AlpineOS en el Raspberry Pi? Estas son algunas razones:

* **Consumo de memoria muy bajo \(~50MB utilizado durante inactividad vs ~350MB para Ubuntu 20.04\).**
* **Baja sobrecarga de la CPU** **\(27 tareas/ 31 hilos activos para Alpine vs 57 tareas / 111 hilos para Ubuntu cuando se está ejecutando un nodo de Cardano\).**
* **Pi más frío 😎 \(Literalmente, la CPU se mantiene más fría debido a la menor sobrecraga de la misma\).**
* **Y por último, ¿por qué no? Si usas binarios estáticos, también podrías aprovechar AlpineOS 😜**

## Configuración inicial para AlpineOS en Raspberry Pi 4B 8GB:

1\) Descargue AlpineOS para RPi 4 aarch64 aquí: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)

2\) Descomprime el archivo .tar.gz y copia su contenido en una tarjeta SSD/SD

3\) Conecta un teclado y un monitor.

4\) Iniciar sesión con el nombre de usuario 'root'.

5\) Ejecuta el comando `setup-alpine` y sigue las instrucciones.

{% hint style="info" %}
Cuando esté en `configurar-alpine`  se le pedirá que elija el disco del sistema. Una vez que esté en este punto, ingrese, `y`, para configurar el disco y crear la partición para `sys`.
{% endhint %}



6\) Reinicie el sistema.

7\) Añada un nuevo usuario llamado cardano mediante el comando `adduser cardano` y su contraseña como se indica. \(Para un nombre de usuario distinto de **cardano**, consulte **la solución general de problemas**\)

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

### Instalando los binarios estáticos 'cardano-node' y 'cardano-cli' \(AlpineOS utiliza binarios estáticos casi exclusivamente así que debes evitar las compilaciones no estáticas\)

{% hint style="info" %}
**Puede obtener los binarios estáticos para la versión 1.27. a través de este** [**enlace**](https://ci.zw3rk.com/build/1758) **cortesía de Moritz Angermann el SPO de ZW3RK pool 🙏**
{% endhint %}

**Ejecuta los siguientes comandos para instalar los binarios y colocarlos en el directorio correcto.**

* Descargar los binarios

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
```

* Descomprimir e instalar los binarios a través de los comandos

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Instalar el servicio de nodos Cardano Alpine Linux de Armada Alliance

{% hint style="success" %}
#### Si usted ha decidido usar AlpineOS para las operaciones del stake pool de Cardano, encontrará esta colección de scripts y servicios útiles.
{% endhint %}

{% hint style="info" %}
#### Para instalar correctamente los scripts y servicios no omitir los siguientes pasos 🏴‍☠️😎
{% endhint %}

1\) Clona este repositorio para obtener la carpeta y los scripts necesarios para iniciar rápidamente tu nodo Cardano. Ejecuta el comando:

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

2\) Ejecuta los siguientes comandos para instalar la carpeta **cnode**, scripts y servicios en las carpetas correctas. La carpeta **cnode** contiene todo lo que un nodo **Cardano** necesita iniciar como un nodo relay funcional:

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
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4\) Siga la guía escrita en **README.txt** contenida en el directorio **$HOME** después de instalar **cnode**, scripts y servicios.

```text
    more ~/README.txt
```

## Configurar prometheus y el exportador de nodos

1\) Descargar Prometheus y node-exporter en el directorio de inicio

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
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
    mv prometheus-2.27.1.linux-arm64 prometheus
```

```text
    mv node_exporter-1.1.2.linux-arm64 node_exporter
```

4\) Siga la guía escrita en README.txt contenida en el directorio $HOME después de instalar cnode, scripts y servicios para iniciar los servicios en correctamente.

```text
    more ~/README.txt
```

## Solución general de problemas

* Si utilizas otro nombre de usuario distinto de cardano, usa los siguientes comandos y reemplaza `username` con el nombre de usuario elegido.

```text
    sed -i 's@/home/cardano@/home/<username>@g' ~/cnode_env
```

```text
    sudo sed -i 's@/home/cardano@/home/<username>@g' /etc/init.d/cardano-node
```

```text
    sudo sed -i 's@/home/cardano@/home/<username>@g' /etc/init.d/prometheus
```

```text
    sudo sed -i 's@/home/cardano@/home/<username>@g' /etc/init.d/node-export
```

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



