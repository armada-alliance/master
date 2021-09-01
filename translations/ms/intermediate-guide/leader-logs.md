---
description: How to get your Stake Pools Slot Assignments for next Epoch
---

# CNCLI Leader Logs📑

## Build CNCLI \(thanks to [@AndrewWestberg](https://github.com/AndrewWestberg)\)

{% hint style="warning" %}
Running it on your block-producing/Core node is the convenient way, but to save resources you may build and run cncli on another \(i.e. your monitoring\) device. Therefore you will need to get the stake-snapshot.json from one of your running nodes and copy the genesis files and the vrf.skey from your Core to the particular device.
{% endhint %}

### Prepare Rust Environment and install Rustup

```bash
mkdir -p $HOME/.cargo/bin
```

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Choose Option 1 \(default\)

```bash
source $HOME/.cargo/env

rustup install stable

rustup default stable

rustup update

rustup component add clippy rustfmt
```

Install any necessary packages. Your system may already have most to all of these.

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

### Build cncli

```bash
# If you don't have a $HOME/git folder you can create one using:
# mkdir $HOME/git

cd $HOME/git

git clone --recurse-submodules https://github.com/AndrewWestberg/cncli

cd cncli
```

Check [https://github.com/AndrewWestberg/cncli](https://github.com/AndrewWestberg/cncli) for the latest tag name and adjust the command below. For the time of writing this, it's v3.1.4

```bash
git checkout <latest_tag_name>
```

```bash
# This will take some time on a Raspberry Pi - be patient, it'll git r dun.
# Grab some coffee, check the strawberries, whatever.

cargo install --path . --force
```

Check if the installation was successful and locate `cncli`

```bash
cncli --version

command -v cncli

echo $PATH
```

The `command -v` should show you where the `cncli` executable currently lives, `.cargo/bin`. The `echo` command will show what's on your `PATH`.

You should have `.local/bin` on your `PATH`, but in case you don't \(Core should have it\), do it now and add it to your `PATH`:

{% tabs %}
{% tab title="Monitor" %}
```bash
mkdir -p $HOME/.local/bin
echo PATH="$HOME/.local/bin:$PATH" >> $HOME/.bashrc
source $HOME/.bashrc
```
{% endtab %}
{% endtabs %}

Move `cncli` from it's current location to `.local/bin`

```bash
mv <path/to>/cncli $HOME/.local/bin/cncli
```

## Run cncli sync and deploy it as a service

{% hint style="info" %}
CNCLI sync creates an sqlite3 database \(cncli.db\), and needs to be connected to your running core-node. The guide assumes you have followed the armada-alliance guide so far and use the same folder structure.
{% endhint %}

```bash
mkdir -p $HOME/pi-pool/cncli

sudo nano /etc/systemd/system/cncli-sync.service
```

Paste the following, adjust ip and port, save and exit.

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

Enable the service

```bash
sudo systemctl daemon-reload

sudo systemctl enable cncli-sync.service

sudo systemctl start cncli-sync.service
```

Make the cncli.db writable \(needed for the following script\)

```bash
cd $HOME/pi-pool/cncli

sudo chmod a+w cncli.db
```

### Create the leaderlog-stake-snapshot-v4.sh script \(thanks to [@sayshar](https://github.com/sayshar)\)

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

Paste the following, adjust parameters, save and exit.

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

Make it executable

```bash
sudo chmod +x leaderlog-stake-snapshot-v4.sh
```

{% hint style="warning" %}
If you installed cncli on your Core continue with "Run leaderlog script", otherwise you have to do some more steps:
{% endhint %}

Run the following command on your Core. Make sure to add your pool id.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query stake-snapshot --stake-pool-id <your_pool_id> --mainnet > stake-snapshot.json
```
{% endtab %}
{% endtabs %}

Then copy `vrf.skey`, `mainnet-byron-genesis.json`, `mainnet-shelley-genesis.json` `stake-snapshot.json` from your Core to your cncli device. \(via USB-stick, scp or rsync...\) Move them to the right directory:

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

### Run leaderlog script

{% hint style="warning" %}
Every time you run the script you need a fresh stake-snapshot.json, except your stake didn't change for the last few epochs.
{% endhint %}

```bash
cd $HOME/pi-pool/scripts
./leaderlog-stake-snapshot-v4.sh
```

The schedule is saved to slot\_`number-of-epoch`.json.

{% hint style="warning" %}
The script calculates the schedule for the current epoch by default. You can run it for the next epoch 1.5 days before. \(Or at 70% into the current epoch.\) Just change the epoch parameter in the script from "current" to "next".
{% endhint %}

{% hint style="danger" %}
Be careful to keep your block leader schedule private, as attackers could use this information to strategically attack your pool.
{% endhint %}

