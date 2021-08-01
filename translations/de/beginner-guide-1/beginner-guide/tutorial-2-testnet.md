---
description: >-
  Nach Abschluss des Raspberry Pi Setups sind wir nun bereit, die für das Testnetz benötigten Dateien herunterzuladen.
---

# Einrichtung eines Knoten im Testnet

{% hint style="warning" %}
**Dieses Tutorial beschreibt die Synchronisation eines einzelnen Knotens mit der Cardano Blockchain! Wir haben einige Schritte und Sicherheitsvorkehrungen übersprungen, um dieses Tutorial so einfach wie möglich zu machen - BENUTZEN Sie dieses Tutorial NICHT, um einen Mainnet-Pool einzurichten. Bitte verwenden Sie unsere**[ **intermediate Guides**](../../intermediate-guide/pi-pool-tutorial/pi-node/) **für das Mainnet.**
{% endhint %}

{% hint style="warning" %}
**Dieses Tutorial ist nur für Raspberry Pi OS 64bit geeignet und ist ausschließlich für Bildungszwecke gedacht, um die Synchronisierung einens Cardano-Knoten zur Blockchain zu bekommen.**
{% endhint %}

## Übersicht

1. Einrichtung der Umgebung
2. Laden Sie die Binärdateien herunter, die zum Aufbau eines Cardano-Knoten-Relays benötigt werden
3. Konfigurationsdateien vom IOHK/Cardano-Knoten herunterladen
4. Konfigurationseinstellungen bearbeiten
5. Herunterladen eines Datenbank-Snapshots, um den Synchronisierungsprozess zu beschleunigen
6. Starten des passiven Relaisknoten, um sich mit dem Testnet zu verbinden
7. Überwachung des Relays Knoten [**Guild Operators gLiveView** ](https://cardano-community.github.io/guild-operators/#/)

![](../../.gitbook/assets/download-10-%20%281%29.jpeg)

{% hint style="info" %}
Dieses Tutorial könte auch für **mainnet** verwendet werden. Ersetzen Sie einfach alle Instanzen des Wortes "**testnet**" durch "**mainne**t" innerhalb dieses Tutorials.
{% endhint %}

## Einrichtung der Umgebung

* Wir müssen zuerst unser Betriebssystem aktualisieren und benötigte Upgrades installieren, falls verfügbar.

{% hint style="info" %}
Es wird dringend empfohlen, das Betriebssystem jedes Mal, wenn Sie aufstarten und sich bei Ihrem **Raspberry Pi** einloggen, zu aktualisieren, um Sicherheitslücken zu vermeiden.
{% endhint %}

```text
# Wir verwenden den sudo-Präfix um Befehle als nicht-root-Benutzer auszuführen  

sudo apt update
sudo apt upgrade -y
```

* Wir können nun den Pi neu starten und die Aktualisierungen wirksam machen, indem wir diesen Befehl im Terminal ausführen.

```text
sudo reboot
```

### Verzeichnisse erstellen

```bash
mkdir -p $HOME/.local/bin
mkdir -p $HOME/testnet-relay/files
```

### ~/.local/bin zu unserem $PATH hinzufügen

{% hint style="info" %}
[Wie man ein Verzeichnis in $PATH hinzufügt](https://www.howtogeek.com/658904/how-to-add-a-directory-to-your-path-in-linux/)
{% endhint %}

```bash
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
```

### Bash-Variablen erstellen

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

### Cardano-node static build herunterladen

| Bereitgestellt durch                                                                                                             | Link zu Cardano Static-Build                                                                           |
|:-------------------------------------------------------------------------------------------------------------------------------- |:------------------------------------------------------------------------------------------------------ |
| [**ZW3RK**](https://adapools.org/pool/e2c17915148f698723cb234f3cd89e9325f40b89af9fd6e1f9d1701a) **1PCT Haskell CI Support Pool** | \*\*\*\*[**https://ci.zw3rk.com/build/1755**](https://ci.zw3rk.com/build/1755)\*\*\*\* |

* Ein[ **Static-Build**](https://en.wikipedia.org/wiki/Static_build) ist eine ****[**kompilierte**](https://en.wikipedia.org/wiki/Compiler) ****Version eines Programms, das statisch gegen Bibliotheken gelinkt wurde.

Jetzt müssen wir einfach die Zip-Datei oben in das Home-Verzeichnis unseres Pi herunterladen und sie dann an die richtige Stelle verschieben, damit wir sie später aufrufen können, um den Knoten zu starten.

```bash
# Als erstes wechseln Sie zum Home Verzeichniss
cd $HOME

# Jetzt können wir den cardano-node herunterladen 
wget https://ci.zw3rk.com/build/1755/download/1/aarch64-unknown-linux-musl-cardano-node-1.26.2.zip
```

* Verwenden Sie [**unzip**](https://linux.die.net/man/1/unzip) Befehl für die heruntergeladene Zip Datei und entpacken Sie deren Inhalt.

  ```bash
  unzip aarch64-unknown-linux-musl-cardano-node-1.26.1.zip
  ```

* Als nächstes müssen wir sicherstellen, dass der neu heruntergeladene "cardano-node" Ordner und seine Inhalte vorhanden sind.

{% hint style="info" %}
Wenn Sie sich nicht sicher sind, ob die Datei richtig heruntergeladen wurde oder ob Sie den Namen des Ordners/der Dateien benötigen wir können den Linux Befehl [**ls**](https://www.man7.org/linux/man-pages/man1/ls.1.html) verwenden.
{% endhint %}

Jetzt müssen wir cardano-node in unser lokales Binärverzeichnis verschieben.

```bash
mv cardano-node/* ~/.local/bin
```

Bevor wir fortfahren, wollen wir sicherstellen, dass cardano-node und cardano-cli in unserem $PATH sind

```bash
cardano-node version
cardano-cli version
```

Jetzt können wir in unseren Dateiordner wechseln, und laden die vier Cardano-Knoten Konfigurationsdateien herunter, die wir von der offiziellen [IOHK-Website](https://hydra.iohk.io/build/5822084/download/1/index.html) oder [Dokumentation](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html) benötigen. Wir verwenden den "wget" Befehl, um die Dateien herunterzuladen.

```bash
cd $NODE_FILES
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
```

* **Verwende den nano Bash-Editor um ein paar Einstellungen in unserer "testnet-config.json" Datei zu ändern**
* [ ] Ändere die **"TraceBlockFetchDecisions"** Zeile von "**false**" auf "**true**"
* [ ] Ändere **"hasEKG"** zu **12600**
* [ ] Ändere **"hasPrometheus"** Adresse/Port zu 12700

```text
sudo nano testnet-config.json
```

### Erstelle die systemd Dateien

Wir werden den Linux systemd Service Manager benutzen, um das Starten, Stoppen und Neustarten unseres Cardano-Knotenrelais auszuführen.

{% hint style="info" %}
Wenn Sie mehr über Linux systemd erfahren möchten, öffnen Sie die Linux-Handbuchseite.[https://www.man7.org/linux/man-pages/man1/systemd.1.html](https://www.man7.org/linux/man-pages/man1/systemd.1.html)
{% endhint %}

```bash
sudo nano $HOME/.local/bin/cardano-service
```

**Jetzt müssen wir den cardano-node start-up Skript erstellen**

{% hint style="info" %}
Wie man den cardano-node startet, findet man auch hier in der Cardano-Dokumentation.[https://docs.cardano.org/projects/cardano-node/de/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html)
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

**Jetzt müssen wir die Zugriffsrechte auf unser neues systemd Service Skript erteilen**

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

Wir sollten nun unseren systemd Service neu laden, um sicherzustellen, dass er unseren cardano-service richtig lädt

```bash
sudo systemctl daemon-reload
```

**Wenn wir nicht jedes Mal "sudo systemctl" aufrufen wollen wenn wir den cardano-node Service stoppen oder starten, können wir eine "Funktion" erstellen und in unseren .bashrc Shell-Skript hinzufügen, welches dies für uns tun wird** [https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/](https://www.routerhosting.com/knowledge-base/what-is-linux-bashrc-and-how-to-use-it-full-guide/)

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

## Herunterladen eines Snapshots der Blockchain, um den Sync-Prozess zu beschleunigen

{% hint style="info" %}
Dank Star Forge Pool \[OTG\] wurde uns einen Snapshot der Testnet Blockchain zur Verfügung gestellt. Wenn Sie keinen Snapshot herunterladen wollen, **können Sie diesen Schritt** überspringen. Seien Sie sich bewusst, wenn Sie den Snapshot Schritt überspringen, kann es bis zu 8 Stunden dauern, bis der Knoten vollständig synchronisiert wird.
{% endhint %}

{% hint style="danger" %}
**Stellen Sie sicher, dass Sie den cardano-node noch nicht gestartet haben, bevor Sie hier fortfahren.**🛑
{% endhint %}

Stellen Sie zuerst sicher, dass der cardano-service, den wir vorher erstellt haben, gestoppt ist, dann laden wir die Blockchain in unseren testnet-relay/-files herunter. Sie können die folgenden Befehle ausführen, um unseren Download zu starten.

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
Dieser Download dauert je nach Internetgeschwindigkeit zwischen 25 Minuten und 2 Stunden.
{% endhint %}

* Nachdem die Blockchain fertig heruntergeladen wurde, ist es eine gute Idee, eine saubere Datei hinzuzufügen, bevor wir den Relais-Knoten starten. Kopieren/Einfügen des folgenden Befehls in Ihr Terminalfenster.

```bash
touch db/clean
```

## Synchronisierung mit der Blockchain vervollständigen

* Jetzt können wir den "passiven" Relay-Knoten starten, um die Synchronisierung mit der Blockchain zu beginnen.

```bash
cardano-service enable
cardano-service start
cardano-service status
```

## Einrichtung von gLiveView zur Überwachung des Knotens während des Synchronisierungsprozesses

#### Jetzt können Sie in den $NODE\_FILES Ordner wechseln und dann den gLiveView Monitor Service herunterladen

```bash
cd $NODE_Files
curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
```

* Wir müssen nun den "**CNODE\_PORT**" auf den Port ändern, den Sie auf Ihrem Cardano-Knoten gesetzt haben. In unserem Fall ändern wir es auf **3001.**

```bash
sudo nano env
```

* Letzlich können wir den nano Editor verlassen und einfach den gLiveView-Skript ausführen.

```bash
./gLiveView.sh
```

{% hint style="success" %}
Wenn Sie Ihre Raspberry Pi Performance überwachen möchten, können Sie die folgenden Befehle verwenden.
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

## Referenzen:

{% tabs %}
{% tab title="📚" %}
{% embed url="https://github.com/wcatz/pi-pool" caption="" %}

{% embed url="https://github.com/alessandrokonrad/Pi-Pool" caption="" %}

{% embed url="https://github.com/angerman" caption="" %}

{% embed url="https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html" caption="" %}

{% embed url="https://cardano-community.github.io/guild-operators/\#/Scripts/gliveview" caption="" %}
{% endtab %}
{% endtabs %}

