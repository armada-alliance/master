---
description: Miten saada tietää Stake Poolin Slot varaukset seuraavalle Epochille
---

# CNCLI Leader Lokit📑

## Rakenna CNCLI \(kiitos [@AndrewWestberg](https://github.com/AndrewWestberg)\)

{% hint style="Huomaa" %}
Cncli:n suorittaminen lohko-tuotanto/Core nodella on kätevä tapa, mutta resurssien säästämiseksi voit rakentaa ja ajaa cncli:n myös toisella \(monitorointi\) laitteella. Siksi sinun täytyy saada stake-snapshot.json yhdestä käynnissä nodestasi ja kopioida genesis tiedostot ja vrf.skey Core nodestasi monitoroivaan laitteeseen.
{% endhint %}

### Valmista Rust ympäristö ja asenna Rustup

```bash
mkdir -p $HOME/.cargo/bin
```

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Valitse Vaihtoehto 1 \(oletus\)

```bash
source $HOME/.cargo/env

rustup install stable

rustup default stable

rustup update

rustup component add clippy rustfmt
```

Asenna tarvittavat paketit. Järjestelmässäsi saattaa jo olla suurin osa näistä.

{% tabs %}
{% tab title="Monitor" %}
```bash
sudo apt update -y && sudo apt install -y automake \ 
build-essential pkg-config libffi-dev libgmp-dev \ 
libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev \ 
make g++ tmux git jq wget libncursesw5 libtool autoconf
```
{% endtab %}

{% tab title="Core" %}
```bash
sudo apt update -y
```
{% endtab %}
{% endtabs %}

### Rakenna cncli

```bash
# If you don't have a $HOME/git folder you can create one using:
# mkdir $HOME/git

cd $HOME/git

git clone --recurse-submodules https://github.com/AndrewWestberg/cncli

cd cncli
```

Tarkista [https://github.com/AndrewWestberg/cncli](https://github.com/AndrewWestberg/cncli) saadaksesi viimeisimmän tunnisteen ja säädä komentoa alla. Tätä kirjoitettaessa se on v3.1.4

```bash
git checkout <latest_tag_name>
```

```bash
# This will take some time on a Raspberry Pi - be patient, it'll git r dun.
# Grab some coffee, check the strawberries, whatever.

cargo install --path . --force
```

Tarkista, onko asennus onnistunut ja etsi `cncli`

```bash
cncli --version

command -v cncli

echo $PATH
```

Komennon `-v` pitäisi näyttää missä `cncli` suoritettava tällä hetkellä sijaitsee, `.cargo/bin`. Komento `echo` näyttää `PATH`.

Sinun pitäisi olla `.local/bin` `PATH`, jos ei ole \(Core pitäisi olla \), tee se nyt ja lisää se `PATH`:

{% tabs %}
{% tab title="Monitor" %}
```bash
mkdir -p $HOME/.local/bin
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}
{% endtabs %}

Siirrä `cncli` nykyisestä sijainnista `.local/bin`

```bash
mv <path/to>/cncli $HOME/.local/bin/cncli
```

## Suorita cncli synkronointi ja ota se käyttöön palveluna

{% hint style="info" %}
CNCLI sync luo sqlite3 tietokannan \(cncli.db\), ja se on kytkettävä käynnissä olevaan core-nodeesi. Oppaassa oletetaan, että olet seurannut armada-allianssin tutoriaaleja tähän asti ja käytä samaa kansiorakennetta.
{% endhint %}

```bash
mkdir -p $HOME/pi-pool/cncli

sudo nano /etc/systemd/system/cncli-sync.service
```

Liitä seuraavat, säädä ip ja portti, tallenna ja poistu.

{% tabs %}
{% tab title="Monitor" %}
```bash
[Unit]
Description=CNCLI Sync
After=multi-user.target

[Service]
Type=simple
Restart=always
RestartSec=5
LimitNOFILE=131072
ExecStart=/home/ada/.local/bin/cncli sync --host <your_core_ip> --port <your_core_port> --db /home/ada/pi-pool/cncli/cncli.db
KillSignal=SIGINT
SuccessExitStatus=143
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cncli-sync

[Install]
WantedBy=multi-user.target
```
{% endtab %}

{% tab title="Core" %}
```bash
[Unit]
Description=CNCLI Sync
After=multi-user.target

[Service]
Type=simple
Restart=always
RestartSec=5
LimitNOFILE=131072
ExecStart=/home/ada/.local/bin/cncli sync --host 127.0.0.1 --port <cardano_node_port> --db $HOME/pi-pool/cncli/cncli.db
KillSignal=SIGINT
SuccessExitStatus=143
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cncli-sync

[Install]
WantedBy=multi-user.target
```
{% endtab %}
{% endtabs %}

Ota palvelu käyttöön

```bash
sudo systemctl daemon-reload

sudo systemctl enable cncli-sync.service

sudo systemctl start cncli-sync.service
```

Tee cncli.db kirjoitettavaksi \(tarvitaan seuraavassa komentosarjassa\)

```bash
cd $HOME/pi-pool/cncli

sudo chmod a+w cncli.db
```

### Luo leaderlog-stake-snapshot-v4.sh skripti \(kiitos [@sayshar](https://github.com/sayshar)\)

{% tabs %}
{% tab title="Monitor" %}
```bash
    mkdir -p $HOME/pi-pool/scripts
  sudo nano $HOME/pi-pool/scripts/leaderlog-stake-snapshot-v4.sh
```
{% endtab %}

{% tab title="Core" %}
```bash
    sudo nano $HOME/pi-pool/scripts/leaderlog-stake-snapshot-v4.sh
```
{% endtab %}
{% endtabs %}

Liitä seuraavat, säädä parametreja, tallenna ja poistu.

{% tabs %}
{% tab title="Monitor" %}
```bash
#!/bin/bash

##############################################################
###################   To be filled  ##########################
##############################################################

POOLID="type pool ID"

VRFSKEY=$HOME/pi-pool/cncli/vrf.skey

BYRON=$HOME/pi-pool/cncli/mainnet-byron-genesis.json

SHELLEY=$HOME/pi-pool/cncli/mainnet-shelley-genesis.json

CNCLIDB=$HOME/pi-pool/cncli/cncli.db #Ensure you point to the correct folder after running cncli sync.

TZ="America/Los_Angeles" #https://en.wikipedia.org/wiki/List_of_tz_database_time_zones [default: America/Los_Angeles].

EPOCH="current" #prev or next for last and next epoch respectively. Default is current.

##############################################################


if [ "$EPOCH" = "current" ] || [ "$EPOCH" = "prev" ] || [ "$EPOCH" = "next" ]; then
    if [ "$EPOCH" = "current" ]; then
               echo ""
                echo "Please be patient. Generating leaderlogs for the current epoch."
            echo ""
        POOLSTAKE=`cat stake-snapshot.json | awk '$1 ~ /poolStakeSet/ {print $NF+0}'`
                ACTIVESTAKE=`cat stake-snapshot.json | awk '$1 ~ /activeStakeSet/ {print $NF+0}'`
    fi
    if [ "$EPOCH" = "next" ]; then
                echo ""
                echo "Please be patient. Generating leaderlogs for the next epoch."
                echo ""
                POOLSTAKE=`cat stake-snapshot.json | awk '$1 ~ /poolStakeMark/ {print $NF+0}'`
                ACTIVESTAKE=`cat stake-snapshot.json | awk '$1 ~ /activeStakeMark/ {print $NF+0}'`
    fi
    if [ "$EPOCH" = "prev" ]; then
                echo ""
                echo "Please be patient. Generating leaderlogs for the previous epoch."
                echo ""
                POOLSTAKE=`cat stake-snapshot.json | awk '$1 ~ /poolStakeGo/ {print $NF+0}'`
                ACTIVESTAKE=`cat stake-snapshot.json | awk '$1 ~ /activeStakeGo/ {print $NF+0}'`
    fi
    cncli leaderlog --pool-id $POOLID --pool-vrf-skey $VRFSKEY --byron-genesis $BYRON --shelley-genesis $SHELLEY --pool-stake $POOLSTAKE --active-stake $ACTIVESTAKE --db $CNCLIDB --tz $TZ --ledger-set $EPOCH > slot.json
else
        echo ""
          echo "Invalid EPOCH entry"
        echo ""
fi

if [ -f ./slot.json ]; then
    epoch=`cat slot.json | awk '$1 ~ /"epoch":/ {print $NF+0}'`
    mv slot.json slot_$epoch.json
    echo ""
    echo "Previewing leaderlogs slots for epoch $epoch"
    echo ""
    cat slot_$epoch.json
    echo ""    
    if [ -f ./slot_.json ]; then
        rm slot_.json
    fi
    else
    echo ""
    echo "Leaderlogs could not be generated. Please check parameters and try again. Also ensure system has adequate RAM if failure repeats."
    echo ""
fi
```
{% endtab %}

{% tab title="Core" %}
```bash
#!/bin/bash

##############################################################
###################   To be filled  ##########################
##############################################################

POOLID="type pool ID"

VRFSKEY=$HOME/pi-pool/vrf.skey

BYRON=$HOME/pi-pool/files/mainnet-byron-genesis.json

SHELLEY=$HOME/pi-pool/files/mainnet-shelley-genesis.json

CNCLIDB=$HOME/pi-pool/cncli/cncli.db #Ensure you point to the correct folder after running cncli sync.

TZ="America/Los_Angeles" #https://en.wikipedia.org/wiki/List_of_tz_database_time_zones [default: America/Los_Angeles].

EPOCH="current" #prev or next for last and next epoch respectively. Default is current.

##############################################################

if [ ! -f stake-snapshot.json ];then
    cardano-cli query stake-snapshot --stake-pool-id $POOLID --mainnet > stake-snapshot.json
    echo ""
    cat stake-snapshot.json
    echo ""
else
    ANS="N"
    echo ""
    echo "The file stake-snapshot.json is detected. Would you like to recreate it? y/N"
    echo ""
    read ANS
fi

if [ $ANS = "y" ] || [ $ANS = "Y" ]; then
    echo ""
        echo "Generating new stake-snapshot.json."
        echo ""
        cardano-cli query stake-snapshot --stake-pool-id $POOLID --mainnet > stake-snapshot.json
    echo ""
    echo "Previewing stake-snapshot.json"
    echo ""
        cat stake-snapshot.json
    echo ""
else
        echo ""
        echo "Previewing stake-snapshot.json"
    echo ""
    cat stake-snapshot.json
    echo ""
fi

if [ "$EPOCH" = "current" ] || [ "$EPOCH" = "prev" ] || [ "$EPOCH" = "next" ]; then
    if [ "$EPOCH" = "current" ]; then
               echo ""
                echo "Please be patient. Generating leaderlogs for the current epoch."
            echo ""
        POOLSTAKE=`cat stake-snapshot.json | awk '$1 ~ /poolStakeSet/ {print $NF+0}'`
                ACTIVESTAKE=`cat stake-snapshot.json | awk '$1 ~ /activeStakeSet/ {print $NF+0}'`
    fi
    if [ "$EPOCH" = "next" ]; then
                echo ""
                echo "Please be patient. Generating leaderlogs for the next epoch."
                echo ""
                POOLSTAKE=`cat stake-snapshot.json | awk '$1 ~ /poolStakeMark/ {print $NF+0}'`
                ACTIVESTAKE=`cat stake-snapshot.json | awk '$1 ~ /activeStakeMark/ {print $NF+0}'`
    fi
    if [ "$EPOCH" = "prev" ]; then
                echo ""
                echo "Please be patient. Generating leaderlogs for the previous epoch."
                echo ""
                POOLSTAKE=`cat stake-snapshot.json | awk '$1 ~ /poolStakeGo/ {print $NF+0}'`
                ACTIVESTAKE=`cat stake-snapshot.json | awk '$1 ~ /activeStakeGo/ {print $NF+0}'`
    fi
    cncli leaderlog --pool-id $POOLID --pool-vrf-skey $VRFSKEY --byron-genesis $BYRON --shelley-genesis $SHELLEY --pool-stake $POOLSTAKE --active-stake $ACTIVESTAKE --db $CNCLIDB --tz $TZ --ledger-set $EPOCH > slot.json
else
        echo ""
          echo "Invalid EPOCH entry"
        echo ""
fi

if [ -f ./slot.json ]; then
    epoch=`cat slot.json | awk '$1 ~ /"epoch":/ {print $NF+0}'`
    mv slot.json slot_$epoch.json
    echo ""
    echo "Previewing leaderlogs slots for epoch $epoch"
    echo ""
    cat slot_$epoch.json
    echo ""    
    if [ -f ./slot_.json ]; then
        rm slot_.json
    fi
    else
    echo ""
    echo "Leaderlogs could not be generated. Please check parameters and try again. Also ensure system has adequate RAM if failure repeats."
    echo ""
fi
```
{% endtab %}
{% endtabs %}

Tee skriptistä suoritettava

```bash
sudo chmod +x leaderlog-stake-snapshot-v4.sh
```

{% hint style="warning" %}
Jos asensit cncli Core jatka kohdasta "Suorita leaderlog skripti", muuten sinun täytyy tehdä vielä muutamia lisävaiheita:
{% endhint %}

Suorita seuraava komento ydin nodessasi. Varmista, että lisäät pool id:si.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query stake-snapshot --stake-pool-id <your_pool_id> --mainnet > stake-snapshot.json
```
{% endtab %}
{% endtabs %}

Sitten kopioi `vrf.skey`, `mainnet-byron-genesis.json`, `mainnet-shelley-genesis.json` `stake-snapshot.json` Coresta cncli laitteellesi. \(USB-tikulla, scp:llä tai rsync...\) Siirrä ne oikeaan kansioon:

{% tabs %}
{% tab title="Monitor" %}
```bash
mv /path/to/vrf.skey $HOME/pi-pool/cncli/vrf.skey
mv /path/to/mainnet-byron-genesis.json $HOME/pi-pool/cncli/mainnet-byron-genesis.json
mv /path/to/mainnet-shelley-genesis.json $HOME/pi-pool/cncli/mainnet-shelley-genesis.json
mv /path/to/stake-snapshot.json $HOME/pi-pool/scripts/stake-snapshot.json
```
{% endtab %}
{% endtabs %}

### Suorita leaderlog skripti

{% hint style="warning" %}
Joka kerta kun suoritat skriptin tarvitset uuden stakesnapshot.json:in, paitsi jos stake ei ole muuttunut viime epochien aikana.
{% endhint %}

```bash
cd $HOME/pi-pool/scripts
./leaderlog-stake-snapshot-v4.sh
```

Aikataulu on tallennettu slot\_`luku-of-epoch`.json.

{% hint style="warning" %}
Skripti laskee oletusarvoisesti aikataulun nykyiselle epochille. Voit ajaa skriptin seuraavalle epochille 1,5 päivää ennen epochin alkua. \(Tai 70 % nykyisen epochin alusta.\) Vain muuttaa aika-parametria skriptissä arvosta "current" arvoon "next".
{% endhint %}

{% hint style="danger" %}
Ole varovainen ja pidä lohkojohtajan aikataulu yksityisenä, koska hyökkääjät voivat käyttää näitä tietoja strategisesti hyväkseen kohdistaessaan hyökkäyksen pooliisi.
{% endhint %}

