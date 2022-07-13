# burn 13.1
https://download.freebsd.org/ftp/snapshots/arm64/aarch64/ISO-IMAGES/13.1/FreeBSD-13.1-STABLE-arm64-aarch64-RPI-20220527-e5b204542f3-250927.img.xz

# login
root
root

freebsd-update fetch install

```
pkg update

adduser  # ada

pw group mod wheel -m ada
```
## SSH
is listening, set password auth to yes in sshd_config.


## Install packages for this
```
pkg install sudo git nano htop bash bash-completion chrony libedit jq wget tmux
```

## Overclock
```
sudo nano /boot/msdos/config.txt
```

add to bottom
```
over_voltage=6
arm_freq=2000
```

## dynamic cpu scaling

```
sudo nano /etc/rc.conf
# add following
powerd_enable="YES"
# Then restart the daemon:
/etc/rc.d/powerd restart
```

reboot and check

```
sudo sysctl dev.cpu.0.freq_levels
```



## Swap

4gb swap file

```
sudo dd if=/dev/zero of=/usr/swap0 bs=1m count=4096
sudo chmod 0600 /usr/swap0
```
Add to bottom of /etc/fstab

```
md	none	swap	sw,file=/usr/swap0,late	0	0
```

reboot.


## Better bash

First source .bashrc at login, create and add the following to .bash_profile

```
# Startup file for login instances of the bash(1) shell.

# First of all, run a .bashrc file if it exists.
test -f ~/.bashrc && . ~/.bashrc

# The following section should be pretty minimal, if present at all.
mesg y >/dev/null 2>&1
/usr/bin/true

```

https://bash-it.readthedocs.io/en/latest/installation/

```
git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it

~/.bash_it/install.sh
```
### use it

```
cd
nano .bashrc
```

Add to bottom

```
[[ $PS1 && -f /usr/local/share/bash-completion/bash_completion.sh ]] && \
	. /usr/local/share/bash-completion/bash_completion.sh
```
Then
```
chsh -s /usr/local/bin/bash ada
chsh -s bash
```

## Chrony

```
sudo nano /etc/rc.conf

```
Add to bottom

```
chronyd_enable="YES"
```
Then open

```
sudo nano /usr/local/etc/chrony.conf
```
Replace everything in there with following.
```
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

Reboot to get time right.

## Build portsnap db

Portsnap is being deprecated?
https://marc.info/?l=freebsd-ports&m=159656662608767&w=2

```
sudo portsnap fetch
sudo portsnap extract
```

## Tmux
let's not build this over ssh
ctrl+b then d to exit tmux terminal
reattatch to session with:
tmux attach -t cardano

```
tmux new -s cardano
```

Continue

```
mkdir git
cd git
git clone -b cardano https://github.com/freebsd/freebsd-ports-haskell.git
```
```
sudo su
cp -r freebsd-ports-haskell/net-p2p/cardano-node /usr/ports/net-p2p/cardano-node
 
cd /usr/ports/ports-mgmt/portmaster && make install clean
 
portmaster -fR /usr/ports/net-p2p/cardano-node
```

exit tmux ctrl+b then d

Let it run..
 
```
cardano-node --version
```

Edit /usr/local/etc/rc.d/cardano_node 

if needed
Also, read documentation in that file on how to 
start cardano-node on boot, etc 

Example /etc/rc.conf lines:

cardano_node_enable="YES"

cardano_node_port="3001"








