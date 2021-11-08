# Alpine Linux OS 🗻

![](../.gitbook/assets/image%20%281%29.png)

### Warum das Alpine Betriebsystem auf dem Raspberry Pi verwenden? Hier einige Gründe:

* **Sehr geringer Speicherverbrauch \(~50MB im Leerlauf gegenüber ~350MB bei Ubuntu 20.04\).**
* **Niedrigerer CPU overhead** **\(27 tasks/ 31 threads aktiv bei Alpine gegenüber 57 tasks / 111 threads bei Ubuntu wenn der cardano-node läuft\).**
* **Kühler Pi 😎 \(Kein Schärz, die CPU läuft kühler, weil der CPU Overhead geringer ist\).**
* **Und zuletzt, wieso nicht? Wenn Sie statische Binärdateien verwenden wollen, können Sie auch die Vorteile von AlpineOS nutzen**

## Ersteinrichtung für AlpineOS auf Raspberry Pi 4B 8GB:

1\) Laden Sie das AlpineOS für RPi 4 aarch64 hier herunter: [https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz](https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/aarch64/alpine-rpi-3.13.5-aarch64.tar.gz)

2\) Dekomprimieren Sie die .tar.gz Datei und kopieren Sie ihren Inhalt auf eine SSD/SD-Karte

3\) Schliessen Sie eine Tastatur und einen Monitor an.

4\) Melden Sie sich mit dem Benutzernamen 'root' an.

5\) Führen Sie den Befehl `setup-alpine` aus folgen Sie den Anweisungen.

{% hint style="info" %}
Wenn Sie in `setup-alpine`  sind, werden Sie aufgefordert, das Systemlaufwerk auszuwählen. Wenn Sie an diesem Punkt sind, geben Sie `y` ein, um die Festplatte einzurichten und die Partition für `sys` zu erstellen.
{% endhint %}



6\) Führen Sie einen Neustart aus.

7\) Fügen Sie einen neuen Benutzer namens cardano über den Befehl `adduser cardano` hinzu und setzten Sie ein Passwort. \(Für andere Benutzernamen als **cardano**, siehe **Allgemeine Fehlerbehebung**\)

8\) Führen Sie die folgenden Befehle aus, um dem neuen Benutzer Root Rechte zu gewähren

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

9\) Beenden Sie entweder root über den Befehl `exit` oder starten Sie neu und melden Sie sich als cardano an

10\) Installieren Sie Bash, um die Kompatibilität von Bash Skripten zu gewährleisten

```text
    sudo apk add bash
```

11\) Installieren Sie auch git und wget, denn wir werden sie später brauchen.

```text
    sudo apk add git wget
```

### Installation der statischen Binärdateien 'cardano-node' und 'cardano-cli' \(AlpineOS verwendet fast ausschliesslich statische Binärdateien, daher sollten Sie nicht-statische Builds vermeiden\)

{% hint style="info" %}
**Die statischen Binärdateien für Version 1.27.0 erhalten Sie über diesen** [**link**](https://ci.zw3rk.com/build/1758) **der freundlicherweise von Moritz Angermann, der SPO von ZW3RK pool, zur Verfügung gestellt wurde🙏**
{% endhint %}

**Führen Sie die folgenden Befehle aus, um die Binärdateien zu installieren und sie in das richtige Verzeichnis zu verschieben.**

* Herunterladen der Binärdateien

```text
    wget -O ~/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip https://ci.zw3rk.com/build/1758/download/1/aarch64-unknown-linux-musl-cardano-node-1.27.0.zip
```

* Entpacken und installieren Sie die Binärdateien mit den Befehlen

```text
    unzip -d ~/ aarch64-unknown-linux-musl-cardano-node-1.27.0.zip

    sudo mv ~/cardano-node/* /usr/local/bin/
```

## Installieren Sie den Armada Alliance Alpine Linux Cardano Node Dienst

{% hint style="success" %}
#### Wenn Sie sich entschieden haben, AlpineOS für Ihre Cardano-Stake-Pool-Operationen zu verwenden, könnten Sie diese Sammlung von Skripten und Diensten nützlich finden.
{% endhint %}

{% hint style="info" %}
#### Um die Skripte und Dienste korrekt zu installieren, sollten Sie diese Schritte auf keinen Fall überspringen🏴‍☠️😎
{% endhint %}

1\) Klonen Sie dieses Repo, um die notwendigen Ordner und Skripte zu erhalten, um Ihren Cardano Knoten schnell zu starten. Verwenden Sie den Befehl:

```text
    git clone https://github.com/armada-alliance/alpine-rpi-os
```

2\) Führen Sie die folgenden Befehle aus, um anschliessend den Ordner **cnode**, die Skripte und die Dienste in den richtigen Ordnern zu installieren. Der **cnode** Ordner enthält alles was ein **Cardano node** braucht, um als funktionsfähiger Relay Knoten zu starten:

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

3\) Um die Synchronisierung zu beschleunigen, sollten Sie diesen optionalen Befehl zum Herunterladen des neuesten db Ordners, der von einem unserer Alliance-Mitglieder gehostet wird, in Betracht ziehen.

```text
    wget -r -np -nH -R "index.html*" -e robots=off https://mainnet.adamantium.online/db/ -P ~/cnode
```

4\) Folgen Sie der Anleitung in **README.txt** im **$HOME** Verzeichnis nach erfolgreicher Installation von **cnode**, Skripten und Dienste.

```text
    more ~/README.txt
```

## Einrichten von Prometheus und Node Exporter

1\) Laden Sie Prometheus und node-exporter in das Home Verzeichnis herunter

```text
    wget -O ~/prometheus.tar.gz https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-arm64.tar.gz
```

```text
    wget -O ~/node_exporter.tar.gz https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-arm64.tar.gz
```

2\) Extrahieren der tarballs

```text
tar -xzvf prometheus.tar.gz
```

```text
tar -xzvf node_exporter.tar.gz
```

3\) Benenne die Ordner mit den folgenden Befehlen um

```text
    mv prometheus-2.27.1.linux-arm64 Prometheus
```

```text
    mv node_exporter-1.1.2.linux-arm64 node_exporter
```

4\) Folgen Sie der Anleitung in README.txt im $HOME Verzeichnis nach erfolgreicher Installation von cnode, Skripten und Dienste.

```text
    more ~/README.txt
```

## Allgemeine Fehlerbehebung

* Wenn Sie einen anderen als einen anderen Benutzernamen als Cardano verwenden, verwenden Sie die folgenden Befehle und ersetzen Sie den `-Benutzernamen` mit Ihrem gewählten Benutzernamen.

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

* Wenn Sie Probleme mit der Port-Weiterleitung über SSH haben, führen Sie folgenden Befehl aus

```text
sudo nano /etc/ssh/sshd_config
```

* Bearbeiten Sie die Zeile `AllowTcpForwarding no` to `AllowTcpForwarding yes`

{% hint style="info" %}
  Stellen Sie sicher, dass diese Zeile nicht mit einem`#` auskommentiert ist
{% endhint %}

{% hint style="success" %}
Herzlichen Dank an unseren [alliance member](https://armada-alliance.com) Sayshar, operator of [\[SRN\] Pool](https://www.adasrn.com/), der dieses Tutorial möglich gemacht hat 🏴‍☠️ 🙏 😎
{% endhint %}



