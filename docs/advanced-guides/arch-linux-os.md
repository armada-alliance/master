Install Asahi Arch

log in to both alarm and root. Change the passwords.

Update as root

```
pacman -Syu
```

Start and enable sshd, pw auth is disabled for root, login with alarm user.

```
systemctl start sshd.service
systemctl enable sshd.service
```
Install the sudo package and open the sudoers file with visudo and enable the wheel group

```
pacman -S sudo git curl wget htop rsync
sudo EDITOR=nano visudo
```

Add a new user to the wheel group, give it a password.

```
useradd -m -G wheel -s /bin/bash ada
passwd ada
```

Log out and back in as your new user with SSH. Test sudo by upgrading the system again.

{% hint style="info" %}
The Arch Bash shell is boring. Optionally install [Bash-it](https://bash-it.readthedocs.io/en/latest/installation/) for a fancy shell.
{% endhint %}

{% hint style="warning" %}
Remember to copy your ssh key and disable password aurthentication in sshd_config.
{% endhint %}

## Bash completion
Add 'complete -cf sudo' to the bottom of .bash_profile and source.

```
echo complete -cf sudo >> ${HOME}/.bash_profile; . $HOME/.bash_profile
```

## Locales

Generate the [locales](https://wiki.archlinux.org/title/locale) you need by uncommenting what you want(en_US.UTF-8 UTF-8 for example) and generating.

```
sudo nano /etc/locale.gen
sudo locale-gen
sudo localectl set-locale LANG=en_US.UTF-8
```

## Time

Set your timezone

```
sudo timedatectl set-timezone America/New_York
```

No more daylight savings, possible to set RTC to local? testing, might not want to do this.

```
sudo timedatectl set-local-rtc 1
# set to 0 for UTC
```

## Chrony

While we are messing with time.. Install and open chrony.conf and replace contents with below (use ctrl+k to cut whole lines).


```
sudo pacman -S chrony
sudo nano /etc/chrony.conf
```

{% hint style="warning" %}
Note: systemd-timesyncd.service is in conflict with chronyd, so you need to disable it first if you want to enable chronyd properly.
{% endhint %}


```
sudo systemctl stop systemd-timesyncd.service
sudo systemctl disable systemd-timesyncd.service
# enable and start chrony
sudo systemctl start chronyd.service
sudo systemctl enable chronyd.service
```

## Packages

Add the following packages to build and run cardano-node.

```
sudo pacman -S --needed base-devel
sudo pacman -S openssl libtool unzip jq
```

## zram swap

Install and create a conf file with following.

```
sudo pacman -S zram-generator
sudo nano /usr/lib/systemd/zram-generator.conf
```

Add and save.

```
[zram0]
zram-size =  min(ram / 1)
```
Reboot and check htop to confirm.

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

```
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

## Grafana

```
cd ~/git

git clone https://aur.archlinux.org/snapd.git
cd snapd
makepkg -si

sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install grafana --channel=rock/edge

```

## Wiregaurd
```
sudo pacman -S wireguard-tools
```

## Static ip

[netctl](https://wiki.archlinux.org/title/netctl)

Copy the ethernet-static template into place with your interface's name and edit.

```
sudo cp /etc/netctl/examples/ethernet-static /etc/netctl/enp3s0
sudo nano /etc/netctl/enp3s0
```

Edit the interface name to match and add one static ip like below for Mac Mini

```
Description='A basic static ethernet connection'
Interface=enp3s0
Connection=ethernet
IP=static
Address=('192.168.1.152/24')
Gateway='192.168.1.1'
DNS=('192.168.1.1')
```

Enable/start the profile and disable/stop dhcp. It will complain when you try and start it. Don't worry, be happy.

```
sudo netctl enable enp3s0
sudo netctl start enp3s0
sudo systemctl stop dhcpcd
sudo systemctl disable dhcpcd
sudo reboot
```









