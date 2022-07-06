# cardano-node on Asahi Arch Linux, Apple Silicon

## Prerequisites 

Asahi Arch, minimal or desktop installed.

log in to both alarm and root. Change the passwords.

Update the system as root.

```bash
pacman -Syu
```

Start and enable sshd. pw auth is disabled for root, login with alarm user.

```bash
systemctl start sshd.service
systemctl enable sshd.service
```

Install pacman-contrib which includes sudo and some other useful packages and open the sudoers file with visudo and enable the wheel group.

```bash
pacman -S pacman-contrib sudo git curl wget htop rsync numactl
sudo EDITOR=nano visudo
```

Like below.

```bash
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL:ALL) ALL

```

Add a new user to the wheel group, give it a password.

```bash
useradd -m -G wheel -s /bin/bash ada
passwd ada
```

Log out and back in as your new user with SSH. Test sudo by upgrading the system again.

```bash
sudo pacman -Syu
```

{% hint style="info" %}
The Arch Bash shell is boring. Optionally install [Bash-it](https://bash-it.readthedocs.io/en/latest/installation/) for a fancy shell.
{% endhint %}

{% hint style="warning" %}
Remember to copy your ssh key and disable password aurthentication in sshd_config.
{% endhint %}

## Bash completion
Add 'complete -cf sudo' to the bottom of .bash_profile and source.

```bash
echo complete -cf sudo >> ${HOME}/.bash_profile; . $HOME/.bash_profile
```

## Locales

Generate the [locales](https://wiki.archlinux.org/title/locale) by uncommenting (en_US.UTF-8 UTF-8 for example) and generating.

```bash
sudo nano /etc/locale.gen
sudo locale-gen
sudo localectl set-locale LANG=en_US.UTF-8
```

## Time

Set your timezone

```bash
sudo timedatectl set-timezone America/New_York
```

No more daylight savings, possible to set RTC to local? testing, might not want to do this.

```bash
sudo timedatectl set-local-rtc 1
# set to 0 for UTC
```

## Chrony

While we are messing with time.. Install and open chrony.conf and replace contents with below (use ctrl+k to cut whole lines).


```bash
sudo pacman -S chrony
sudo nano /etc/chrony.conf
```

```bash
pool time.google.com       iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.euro.apple.com   iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.apple.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool ntp.ubuntu.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3

# This directive specify the location of the file containing ID/key pairs for
# NTP authentication.
keyfile /etc/chrony/chrony.keys

# This directive specify the file into which chronyd will store the rate
# information.
driftfile /var/lib/chrony/chrony.drift

# Uncomment the following line to turn logging on.
#log tracking measurements statistics

# Log files location.
logdir /var/log/chrony

# Stop bad estimates upsetting machine clock.
maxupdateskew 5.0

# This directive enables kernel synchronisation (every 11 minutes) of the
# real-time clock. Note that it can’t be used along with the 'rtcfile' directive.
rtcsync

# Step the system clock instead of slewing it if the adjustment is larger than
# one second, but only in the first three clock updates.
makestep 0.1 -1

# Get TAI-UTC offset and leap seconds from the system tz database
leapsectz right/UTC

# Serve time even if not synchronized to a time source.
local stratum 10
```

{% hint style="warning" %}
Note: systemd-timesyncd.service is in conflict with chronyd, so you need to disable it first if you want to enable chronyd properly.
{% endhint %}


```bash
sudo systemctl stop systemd-timesyncd.service
sudo systemctl disable systemd-timesyncd.service
# enable and start chrony
sudo systemctl start chronyd.service
sudo systemctl enable chronyd.service
```

## Packages

Add the following packages to build and run cardano-node.

```bash
sudo pacman -S --needed base-devel
sudo pacman -S openssl libtool unzip jq bc xz numactl
```

## zram swap

Install and create a conf file with following.

```bash
sudo pacman -S zram-generator
sudo nano /usr/lib/systemd/zram-generator.conf
```

You may want to read up on zram. I always set 1.5 times the amount of system ram. [github](https://github.com/systemd/zram-generator/blob/main/man/zram-generator.conf.md) | [Hayden James](https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters/) |
[lubuntu mailing list](https://lists.ubuntu.com/archives/lubuntu-users/2013-October/005831.html)

```bash
[zram0]
zram-size = min(24 * 1024)
```
This will give you 24gb of zram swap and will absorb the brunt of running the built in leaderlogs. Reboot and check htop to confirm.


## Prometheus

Install Prometheus and Prometheus-node-exporter

```bash
sudo pacman -S prometheus prometheus-node-exporter
```

## Grafana

Two ways to install Grafana. From AUR or with snap. Pros and cons. Cannot install additional plugins with AUR version (looking into it). Snap is controversial security wise. I need additional plugins so built snap and installed grafana with it.

### With Snap

```bash
mkdir ~/git
cd ~/git
git clone https://aur.archlinux.org/snapd.git
cd snapd/
makepkg -si
reboot
sudo snap install grafana --channel=rock/edge
```

### Grafan-bin AUR

```bash
mkdir ~/git
cd ~/git

git clone https://aur.archlinux.org/grafana-bin.git
cd grafana-bin
makepkg -si
sudo systemctl start grafana.service
sudo systemctl enable grafana.service

```

## Wireguard

```bash
sudo pacman -S wireguard-tools

```

## Static ip

[systemd](https://wiki.archlinux.org/title/Systemd-networkd#Wired_adapter_using_a_static_IP)

Create a network service file, make sure edit it to your needs.

```bash
sudo nano /etc/systemd/network/20-wired.network
```

```bash
[Match]
Name=enp3s0

[Network]
Address=192.168.1.151/24
Gateway=192.168.1.1
DNS=192.168.1.1
```

Make sure no other network service is running like netctl, then enable and start the service. If there is a network service running stop and disable it.

```bash
sudo systemctl --type=service
```

```bash
sudo systemctl enable systemd-networkd.service
sudo systemctl start systemd-networkd.service
```

Disable/stop dhcp.

```bash

sudo systemctl stop dhcpcd
sudo systemctl disable dhcpcd
sudo reboot
```

## Hostname

[Set the Hostname](https://wiki.archlinux.org/title/Network_configuration#Set_the_hostname)

Edit /etc/hostname

```bash
sudo nano /etc/hostname
```
and /etc/hosts

```bash
sudo nano /etc/hosts
```

```bash
127.0.0.1        localhost
::1              localhost
127.0.1.1        myhostname
```

## Server setup

Tweak/Harden system to our needs.

## sysctl
[sysctl](https://wiki.archlinux.org/title/sysctl)

```bash
sudo nano /etc/sysctl.d/98-cardano-node.conf
```

Add the following.

```bash
## Asahi Node ##

# swap more to zram
vm.vfs_cache_pressure = 500
vm.swappiness = 100
vm.dirty_background_ratio = 1
vm.dirty_ratio = 50

fs.file-max = 10000000
fs.nr_open = 10000000

# enable forwarding if using wireguard
net.ipv4.ip_forward = 0

# ignore ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

net.ipv4.icmp_ignore_bogus_error_responses = 1

# disable IPv6
#net.ipv6.conf.all.disable_ipv6 = 1
#net.ipv6.conf.default.disable_ipv6 = 1

# block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 3
#net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_recv=45

# in progress tasks
net.ipv4.tcp_keepalive_time = 240
net.ipv4.tcp_keepalive_intvl = 4
net.ipv4.tcp_keepalive_probes = 5

# reboot if we run out of memory
vm.panic_on_oom = 1
kernel.panic = 10
```

### Disable the root user

Use sudo to become root..
```
sudo passwd -l root
```

### Secure shared memory

Mount shared memory as read only. Open /etc/fstab.

```
sudo nano /etc/fstab
```

Add this line at the bottom, save & exit.

```
tmpfs    /run/shm    tmpfs    ro,noexec,nosuid    0 0
```

### Increase open file limit for $USER

Add a couple lines to the bottom of /etc/security/limits.conf

```bash
sudo bash -c "echo -e '${USER} soft nofile 800000\n${USER} hard nofile 1048576\n' >> /etc/security/limits.conf"
```

Confirm it was added to the bottom.

```bash
cat /etc/security/limits.conf
```
## Choose testnet or mainnet.

{% hint style="danger" %}
There is a 500 ₳ Registration deposit and another 5 ₳ in registration costs to start a pool on mainnet. First time users are strongly reccomended to use testnet. You can get tada (test ada) from the testnet faucet. [tada faucet link](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/)
{% endhint %}

Create the directories for our project.

```bash
mkdir -p ${HOME}/.local/bin
mkdir -p ${HOME}/pi-pool/files
mkdir -p ${HOME}/pi-pool/scripts
mkdir -p ${HOME}/pi-pool/logs
mkdir ${HOME}/git
mkdir ${HOME}/tmp
```

Create an .adaenv file, choose which network you want to be on and source the file. This file will hold the variables/settings for operating a Pi-Node. /home/ada/.adaenv

```shell
echo -e NODE_CONFIG=testnet >> ${HOME}/.adaenv; source ${HOME}/.adaenv
```

### Create bash variables & add \~/.local/bin to our $PATH 🏃

{% hint style="info" %}
[Environment Variables in Linux/Unix](https://askubuntu.com/questions/247738/why-is-etc-profile-not-invoked-for-non-login-shells/247769#247769).
{% endhint %}

{% hint style="warning" %}
You must reload environment files after updating them. Same goes for cardano-node, changes to the topology or config files require a cardano-service restart.
{% endhint %}

```bash
echo . ~/.adaenv >> ${HOME}/.bashrc
cd .local/bin; echo "export PATH=\"$PWD:\$PATH\"" >> $HOME/.adaenv
echo export NODE_HOME=${HOME}/pi-pool >> ${HOME}/.adaenv
echo export NODE_PORT=3003 >> ${HOME}/.adaenv
echo export NODE_FILES=${HOME}/pi-pool/files >> ${HOME}/.adaenv
echo export TOPOLOGY='${NODE_FILES}'/'${NODE_CONFIG}'-topology.json >> ${HOME}/.adaenv
echo export DB_PATH='${NODE_HOME}'/db >> ${HOME}/.adaenv
echo export CONFIG='${NODE_FILES}'/'${NODE_CONFIG}'-config.json >> ${HOME}/.adaenv
echo export NODE_BUILD_NUM=$(curl https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/index.html | grep -e "build" | sed 's/.*build\/\([0-9]*\)\/download.*/\1/g') >> ${HOME}/.adaenv
echo export CARDANO_NODE_SOCKET_PATH="${HOME}/pi-pool/db/socket" >> ${HOME}/.adaenv
source ${HOME}/.bashrc; source ${HOME}/.adaenv
```

### Retrieve node files

```bash
cd $NODE_FILES
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-config.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-byron-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-shelley-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-alonzo-genesis.json
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/${NODE_CONFIG}-topology.json
wget -N https://raw.githubusercontent.com/input-output-hk/cardano-node/master/cardano-submit-api/config/tx-submit-mainnet-config.yaml
```

Run the following to modify ${NODE\_CONFIG}-config.json and update TraceBlockFetchDecisions to "true" & listen on all interfaces with Prometheus Node Exporter.

```bash
sed -i ${NODE_CONFIG}-config.json \
    -e "s/TraceBlockFetchDecisions\": false/TraceBlockFetchDecisions\": true/g" \
    -e "s/127.0.0.1/0.0.0.0/g"
```

Save & exit.

```bash
source ${HOME}/.adaenv
```

## Build Libsodium

This is IOHK's fork of Libsodium. It is needed for the dynamic build binary of cardano-node.

```bash
cd; cd git/
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install
```

Add library path to ldconfig.

```bash
sudo touch /etc/ld.so.conf.d/local.conf 
echo "/usr/local/lib" | sudo tee -a /etc/ld.so.conf.d/local.conf 
```
Echo library paths into .bashrc file and source it.

```bash
echo "export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"" >> ~/.bashrc
echo "export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"" >> ~/.bashrc
. ~/.bashrc
```

Update link cache for shared libraries and confirm.

```bash
sudo ldconfig; ldconfig -p | grep libsodium
```

## Build secp256k1

```
git clone https://github.com/bitcoin-core/secp256k1.git

cd secp256k1

git reset --hard $SECP256K1_REF

./autogen.sh

./configure  --enable-module-schnorrsig --enable-experimental

make

sudo make install
```

For those who run cardano-node as a systemd service, run the following:


Update link cache for shared libraries and confirm.

```bash
sudo ldconfig; ldconfig -p | grep secp256k1
```


## LLVM 9.0.1

```bash
cd ~/git
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz
tar -xf clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz
export PATH=~/git/clang+llvm-9.0.1-aarch64-linux-gnu/bin/:$PATH
```

## ncurses5 compat libs

```bash
cd ~/git
git clone https://aur.archlinux.org/ncurses5-compat-libs.git
cd ncurses5-compat-libs/
gpg --recv-key CC2AF4472167BE03
## If this fails it is probly due to your DNS service(Google). 
## Use https://www.quad9.net/
```

Change target architecture to aarch64 in the build file.

```bash
nano PKGBUILD
arch=(aarch64)
```
and build it.

```bash
makepkg -si
```

Confirm.

```bash
clang --version
```

## GHCUP, GHC & Cabal

Install ghcup use defaults when asked.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

```bash
. ~/.bashrc
ghcup upgrade
ghcup install cabal 3.4.0.0
ghcup set cabal 3.4.0.0

ghcup install ghc 8.10.4
ghcup set ghc 8.10.4
```

### Obtain cardano-node

```bash
cd $HOME/git
git clone https://github.com/input-output-hk/cardano-node.git
cd cardano-node
git fetch --all --recurse-submodules --tags
git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)
```

Configure with 8.10.4 set libsodium

```bash
cabal configure -O0 -w ghc-8.10.4

echo -e "package cardano-crypto-praos\n flags: -external-libsodium-vrf" > cabal.project.local
sed -i $HOME/.cabal/config -e "s/overwrite-policy:/overwrite-policy: always/g"
rm -rf dist-newstyle/build/aarch64-linux/ghc-8.10.4

```

Build them.

```bash
cabal build cardano-cli cardano-node cardano-submit-api
```

Add them to your PATH.

```bash
cp $(find $HOME/git/cardano-node/dist-newstyle/build -type f -name "cardano-cli") $HOME/.local/bin/
cp $(find $HOME/git/cardano-node/dist-newstyle/build -type f -name "cardano-node") $HOME/.local/bin/
cp $(find $HOME/git/cardano-node/dist-newstyle/build -type f -name "cardano-submit-api") $HOME/.local/bin/
```

Check

```bash
cardano-node version
cardano-cli version
```

### Systemd unit startup scripts

Create the systemd unit file and startup script so systemd can manage cardano-node.

```bash
nano ${HOME}/.local/bin/cardano-service
```

Paste the following, save & exit.

```bash
#!/bin/bash
. /home/ada/.adaenv

## +RTS -N6 -RTS = Multicore(4)
cardano-node run +RTS -N6 -RTS \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${CARDANO_NODE_SOCKET_PATH} \
  --port ${NODE_PORT} \
  --config ${CONFIG}
```

Allow execution of our new cardano-node service file.

```bash
chmod +x ${HOME}/.local/bin/cardano-service
```

Open /etc/systemd/system/cardano-node.service.

```bash
sudo nano /etc/systemd/system/cardano-node.service
```

Paste the following, You will need to edit the username here if you chose to not use ada. Save & exit.

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
TimeoutStopSec=10
LimitNOFILE=32768
Restart=always
RestartSec=10
EnvironmentFile=-/home/ada/.adaenv

[Install]
WantedBy= multi-user.target
```

Create the systemd unit file and startup script so systemd can manage cardano-submit-api.

```bash
nano ${HOME}/.local/bin/cardano-submit-service
```

```bash
#!/bin/bash
. /home/ada/.adaenv

cardano-submit-api \
  --socket-path ${CARDANO_NODE_SOCKET_PATH} \
  --port 8090 \
  --config /home/ada/pi-pool/files/tx-submit-mainnet-config.yaml \
  --listen-address 0.0.0.0 \
  --mainnet
```

Allow execution of our new cardano-submit-api service script.

```bash
chmod +x ${HOME}/.local/bin/cardano-submit-service
```

Create /etc/systemd/system/cardano-submit.service.

```bash
sudo nano /etc/systemd/system/cardano-submit.service
```

Paste the following, You will need to edit the username here if you chose to not use ada. save & exit.

```bash
# The Cardano Submit Service (part of systemd)
# file: /etc/systemd/system/cardano-submit.service

[Unit]
Description     = Cardano submit service
Wants           = network-online.target
After           = network-online.target

[Service]
User            = ada
Type            = simple
WorkingDirectory= /home/ada/pi-pool
ExecStart       = /bin/bash -c "PATH=/home/ada/.local/bin:$PATH exec /home/ada/.local/bin/cardano-submit-service"
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=10
LimitNOFILE=32768
Restart=always
RestartSec=10
EnvironmentFile=-/home/ada/.adaenv

[Install]
WantedBy= multi-user.target
```

Reload systemd so it picks up our new service files.

```bash
sudo systemctl daemon-reload
```

Let's add a couple functions to the bottom of our .adaenv file to make life a little easier.

```bash
nano ${HOME}/.adaenv
```

```bash
cardano-service() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-node.service
}

cardano-submit() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" cardano-submit.service
}

cardano-monitor() {
    #do things with parameters like $1 such as
    sudo systemctl "$1" prometheus.service
    sudo systemctl "$1" prometheus-node-exporter.service
    sudo systemctl "$1" grafana-server
}
```

What we just did there was add a couple functions to control our cardano-service and cardano-submit without having to type out

> > sudo systemctl enable cardano-node.service
> >
> > sudo systemctl start cardano-node.service
> >
> > sudo systemctl stop cardano-node.service
> >
> > sudo systemctl status cardano-node.service

Now we just have to:

* cardano-service enable (enables cardano-node.service auto start at boot)
* cardano-service start (starts cardano-node.service)
* cardano-service stop (stops cardano-node.service)
* cardano-service status (shows the status of cardano-node.service)

Or

* cardano-submit enable (enables cardano-submit.service auto start at boot)
* cardano-submit start (starts cardano-submit.service)
* cardano-submit stop (stops cardano-submit.service)
* cardano-submit status (shows the status of cardano-submit.service)

The submit service listens on port 8090. You can connect your Nami wallet like below to submit tx's yourself in Nami's settings.

```bash
http://<node lan ip>:8090/api/submit/tx
```

## gLiveView.sh

Guild operators scripts has a couple useful tools for operating a pool. We do not want the project as a whole, though there are a couple scripts we are going to use.

{% embed url="https://github.com/cardano-community/guild-operators/tree/master/scripts/cnode-helper-scripts" %}

```bash
cd $NODE_HOME/scripts
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
```

{% hint style="info" %}
You can change the port cardano-node runs on in the .adaenv file in your home directory. Open the file edit the port number. Load the change into your shell & restart the cardano-node service.

```bash
nano /home/ada/.adaenv
source /home/ada/.adaenv
cardano-service restart
```
{% endhint %}

Add a line sourcing our .adaenv file to the top of the env file and adjust some paths.

```bash
sed -i env \
    -e "/#CNODEBIN/i. ${HOME}/.adaenv" \
    -e "s/\#CNODE_HOME=\"\/opt\/cardano\/cnode\"/CNODE_HOME=\"\${HOME}\/pi-pool\"/g" \
    -e "s/\#CNODE_PORT=6000"/CNODE_PORT=\"'${NODE_PORT}'\""/g" \
    -e "s/\#CONFIG=\"\${CNODE_HOME}\/files\/config.json\"/CONFIG=\"\${NODE_FILES}\/"'${NODE_CONFIG}'"-config.json\"/g" \
    -e "s/\#TOPOLOGY=\"\${CNODE_HOME}\/files\/topology.json\"/TOPOLOGY=\"\${NODE_FILES}\/"'${NODE_CONFIG}'"-topology.json\"/g" \
    -e "s/\#LOG_DIR=\"\${CNODE_HOME}\/logs\"/LOG_DIR=\"\${CNODE_HOME}\/logs\"/g"
```

Allow execution of gLiveView.sh.

```bash
chmod +x gLiveView.sh
```

## Install Cronie

Arch does not use cron. You can set up a systemd timer or install some other cron like scheduler.


```
sudo pacman -S cronie
```

Enable & start.

```bash
sudo systemctl enable cronie.service
sudo systemctl start cronie.service
```

## topologyUpdater.sh (not needed on block producer)

Until peer to peer is enabled on the network operators need a way to get a list of relays/peers to connect to. The topology updater service runs in the background with cron. Every hour the script will run and tell the service you are a relay and want to be a part of the network. It will add your relay to it's directory after four hours you should see in connections in gLiveView.

{% hint style="info" %}
The list generated will show you the distance & a clue as to where the relay is located.
{% endhint %}

Download the topologyUpdater script and have a look at it. Here is where you will enter your block producer or any other custom peers you would like to always be connected to.

```bash
wget https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/topologyUpdater.sh
```

```bash
nano topologyUpdater.sh
```

Save, exit and make it executable.

```bash
chmod +x topologyUpdater.sh
```

{% hint style="warning" %}
You will not be able to successfully execute ./topologyUpdater.sh until you are fully synced up to the tip of the chain.
{% endhint %}

Create a cron job that will run once an hour.

```bash
EDITOR=nano crontab -e
```

### Add save and exit

```bash
SHELL=/bin/bash
33 * * * * . $HOME/.adaenv; $HOME/pi-pool/scripts/topologyUpdater.sh
```

```bash
nano $NODE_FILES/${NODE_CONFIG}-topology.json
```

{% hint style="info" %}
You can use gLiveView.sh to view ping times in relation to the peers in your {NODE\_CONFIG}-topology file. Use Ping to resolve hostnames to IP's.
{% endhint %}

Changes to this file will take affect upon restarting the cardano-service.

## Display inbound connections in Grafana

```bash
mkdir -p $HOME/custom-metrics/tmp
nano $HOME/custom-metrics/peers_in.sh
```
Add following, update port # to match cardano-node port.

```bash
INCOMING_PEERS="$(ss -tnp state established | grep "cardano-node" | awk -v port=":3003" '$3 ~ port {print}' | wc -l)"
echo "peers_in ${INCOMING_PEERS}" > /home/ada/custom-metrics/tmp/peers_in.prom.tmp
mv /home/ada/custom-metrics/tmp/peers_in.prom.tmp /var/lib/node_exporter/peers_in.prom
```
Make it executable.

```bash
chmod +x $HOME/custom-metrics/peers_in.sh
```
Open node-exporter configuration file and add..

```bash
sudo nano /etc/conf.d/prometheus-node-exporter
```
Make it look like this.

```bash
NODE_EXPORTER_ARGS='--collector.textfile.directory=/var/lib/node_exporter'
```

Create that directory.

```bash
sudo mkdir /var/lib/node_exporter
```
Create a cron job as root to run our script every minute.

```bash
sudo EDITOR=nano crontab -e
```

```bash
SHELL=/bin/bash
* * * * * /home/ada/custom-metrics/peers_in.sh
```
Restart prometheus-node-xporter.

```bash
sudo systemctl restart prometheus-node-exporter
```

After a minute you should be able to find a metric in Grafana called 'peers_in'.

## System updates

Track available system upgrades(pacman)

```bash
nano $HOME/custom-metrics/pacman_upgrades.sh
```

Add following.

```bash
UPDATES="$(/usr/bin/checkupdates | wc -l)"
echo "pacman_upgrades_pending ${UPDATES}" > /home/ada/custom-metrics/tmp/pacman_upgrades_pending.prom.tmp
mv /home/ada/custom-metrics/tmp/pacman_upgrades_pending.prom.tmp /var/lib/node_exporter/pacman_upgrades_pending.prom
```

Make it executable.

```bash
chmod +x $HOME/custom-metrics/pacman_upgrades.sh
```

Create a cron job as root to run our script once a day at 1 am.

```bash
sudo EDITOR=nano crontab -e
```

```bash
0 1 * * * /home/ada/custom-metrics/pacman_upgrades.sh
```

Restart prometheus-node-xporter.

```bash
sudo systemctl restart prometheus-node-exporter
```

In Grafana find the 'pacman_upgrades_pending' metric. It will not be available until you fire off the script or cron runs it.


## Usefull links

[custom-metrics](https://nsrc.org/workshops/2021/sanog37/nmm/netmgmt/en/prometheus/ex-custom-metrics.htm)

[arch prometheus monitoring](https://vdwaa.nl/arch-prometheus-monitoring.html)












