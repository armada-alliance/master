---
description: 'Optimoi laitteistot, suojaa (harden) Ubuntu'
---

# Palvelimen Asetukset

## Konfiguroi Laitteisto

Säästetään hieman energiaa, nostetaan CPU:n hallintaa pikkuisen ja asetetaan GPU:n RAM mahdollisimman alas.

{% hint style="warning" %}
Tässä muutamia linkkejä ylikellotukseen ja aseman nopeuksien testaamiseen. Jos sinulla on lämpöaltaita, voit turvallisesti asettaa 2000. Kiinnitä huomiota yli-voltitus suosituksiin jotta nekin sopivat valitsemaasi kellotusnopeuteen.

* [https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)
* [https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/](https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/)
* [https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock](https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock)
* [https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/](https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/)
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)

Huomaa, että Ubuntu tallentaa config.txt -tiedoston eri paikkaan kuin Raspbian.
{% endhint %}

### Aseman nopeustesti

#### Kirjoitusnopeus

```text
sudo dd if=/dev/zero of=/tmp/output conv=fdatasync bs=384k count=1k; sudo rm -f /tmp/output
```

#### Lukunopeus

```text
sudo hdparm -Tt /dev/sda
```

### Ylikellotus, muisti & radiot

Muokkaa /boot/firmware/config.txt. Liitä Pi Pool lisäykset tiedoston loppuun.

```bash
sudo nano /boot/firmware/config.txt
```

```text
[pi4]
max_framebuffers=2

[all]
kernel=vmlinuz
cmdline=cmdline.txt
initramfs initrd.img followkernel

# Enable the audio output, I2C and SPI interfaces on the GPIO header
dtparam=audio=on
dtparam=i2c_arm=on
dtparam=spi=on

# Enable the serial pins
enable_uart=1

# Comment out the following line if the edges of the desktop appear outside
# the edges of your display
disable_overscan=1

# If you have issues with audio, you may try uncommenting the following line
# which forces the HDMI output into HDMI mode instead of DVI (which doesn't
# support audio output)
#hdmi_drive=2

# If you have a CM4, uncomment the following line to enable the USB2 outputs
# on the IO board (assuming your CM4 is plugged into such a board)
#dtoverlay=dwc2,dr_mode=host

# Config settings specific to arm64
arm_64bit=1
dtoverlay=dwc2

## Pi Pool ##
over_voltage=6
arm_freq=2000
gpu_mem=16
disable-wifi
disable-bt
```

Tallenna ja käynnistä uudelleen.

```text
sudo reboot
```

## Määritä Ubuntu

### Poista root käyttäjä käytöstä

```text
sudo passwd -l root
```

### Suojaa jaettu muisti

Avaa /etc/fstab.

```text
sudo nano /etc/fstab
```

Lisää seuraava tiedoston loppuun omalle riville, tallenna & sulje nano.

```text
tmpfs    /run/shm    tmpfs    ro,noexec,nosuid    0 0
```

### Lisää avoimen tiedoston rajaa

Avaa /etc/security/limits.conf.

```text
sudo nano /etc/security/limits.conf
```

Lisää seuraava tiedoston loppuun omalle riville, tallenna & sulje nano.

```text
ada soft nofile 800000
ada hard nofile 1048576
```

### Optimoi suorituskyky & tietoturva

Lisää seuraava /etc/sysctl.conf tiedoston loppuun. Tallenna ja sulje.

{% hint style="info" %}
[https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3)
{% endhint %}

{% hint style="Huomaa" %}
Olen poistamassa IPv6 ja IPv4 siirtoa käytöstä. Saatat haluta pitää nämä. Olen nähnyt väitteitä, että IPv6 on hitaampi ja saattaa häiritä toimintaa.
{% endhint %}

```text
sudo nano /etc/sysctl.conf
```

```text
## Pi Pool ##

# swap more to zram                     
vm.vfs_cache_pressure=500
vm.swappiness=100
vm.dirty_background_ratio=1
vm.dirty_ratio=50

fs.file-max = 10000000
fs.nr_open = 10000000

# enable forwarding if using wireguard
net.ipv4.ip_forward=0

# ignore ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

net.ipv4.icmp_ignore_bogus_error_responses = 1

# disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 3
net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_recv=45

# in progress tasks
net.ipv4.tcp_keepalive_time = 240
net.ipv4.tcp_keepalive_intvl = 4
net.ipv4.tcp_keepalive_probes = 5

# reboot if we run out of memory
vm.panic_on_oom = 1
kernel.panic = 10

# Use Google's congestion control algorithm
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

#### Muutoksemme otetaan käyttöön uudelleen käynnistyksen jälkeen

Luo uusi tiedosto. Liitä seuraava, tallenna & sulje.

```text
sudo nano /etc/rc.local
```

```text
#!/bin/bash

# Anna suorittimen käynnistyksen rutiinien aika asettua.
sleep 120

sysctl -p /etc/sysctl.conf

exit 0
```

### Poista IRQ-balance käytöstä

{% hint style="info" %}
[**http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/**](http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/)
{% endhint %}

Sinun pitäisi poistaa IRQ Balance käytöstä varmistaaksesi, ettet saa laitteistokatkoksia säikeissäsi. IRQ Balance -laitteen kytkeminen pois päältä optimoi virransäästön ja suorituskyvyn välisen tasapainon jakamalla laitteiston keskeytyksiä useiden prosessorien välillä.

Avaa /etc/default/irqbalance ja lisää alareunaan. Tallenna, poistu ja käynnistä uudelleen.

```text
sudo nano /etc/default/irqbalance
```

```text
ENABLED="0"
```

### Chrony

Meidän täytyy saada aikamme synkronoitua niin tarkasti kuin mahdollista. Avaa /etc/chrony/chrony.conf

```text
sudo apt install chrony
```

```bash
sudo nano /etc/chrony/chrony.conf
```

Korvaa tiedoston sisältö alla olevalla tekstillä, Tallenna ja poistu.

```bash
pool time.google.com       iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
#pool time.facebook.com     iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
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

```bash
sudo service chrony restart
```

### Zram swap

{% hint style="info" %}
Olemme havainneet, että kardano-node voi turvallisesti käyttää tätä pakattua swapia RAM:ina periaatteessatämä antaa meille noin 20 gb RAM:ia. Olemme jo asettaneet ytimen parametrit zram:ia varten /etc/sysctl.conf tiedostossa
{% endhint %}

Vaihto levylle on hidasta, vaihtaminen pakattuun ram tilaan on nopeampaa ja antaa meille jonkin verran enemmän marginaalia ennen muistin loppumista \(oom\).

{% embed url="https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters" caption="" %}

{% embed url="https://lists.ubuntu.com/archives/lubuntu-users/2013-October/005831.html" caption="" %}

```text
sudo apt install zram-config
```

```bash
sudo nano /usr/bin/init-zram-swapping
```

Kerro oletusasetukset kolmella. Tämä antaa sinulle 12,5 Gt virtuaalista pakattua swapia RAM:ksi.

{% hint style="info" %}
mem=$\(\(\(totalmem / 2 / ${NRDEVICES}\) \* 1024 \* 3\)\)
{% endhint %}

```bash
#!/bin/sh
# load dependency modules
NRDEVICES=$(grep -c ^processor /proc/cpuinfo | sed 's/^0$/1/')
if modinfo zram | grep -q ' zram_num_devices:' 2>/dev/null; then
  MODPROBE_ARGS="zram_num_devices=${NRDEVICES}"
elif modinfo zram | grep -q ' num_devices:' 2>/dev/null; then
  MODPROBE_ARGS="num_devices=${NRDEVICES}"
else
  exit 1
fi
modprobe zram $MODPROBE_ARGS
# Calculate memory to use for zram (1/2 of ram)
totalmem=`LC_ALL=C free | grep -e "^Mem:" | sed -e 's/^Mem: *//' -e 's/  *.*//'`
mem=$(((totalmem / 2 / ${NRDEVICES}) * 1024 * 3))
# initialize the devices
for i in $(seq ${NRDEVICES}); do
  DEVNUMBER=$((i - 1))
  echo zstd > /sys/block/zram${DEVNUMBER}/comp_algorithm
  echo $mem > /sys/block/zram${DEVNUMBER}/disksize
  mkswap /dev/zram${DEVNUMBER}
  swapon -p 5 /dev/zram${DEVNUMBER}
done
```

{% hint style="info" %}
Katso, kuinka paljon zram swap:ia cardano-node käyttää.

```text
CNZRAM=$(pidof cardano-node)
grep --color VmSwap /proc/$CNZRAM/status
```
{% endhint %}

### Raspberry Pi & entropia

Ennen kuin alamme tuottaa avaimia palvelimella meidän pitää luoda turvallinen määrä entropiaa.

{% hint style="info" %}
[https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/](https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/)

[https://github.com/nhorman/rng-tools](https://github.com/nhorman/rng-tools)
{% endhint %}

> Mietitäänpä yksittäisen, "headless" palvelimen \(tai mikro-ohjaimenkin\) kohtaloa, ilman ihmisen syöttämiä kirjoituksia tai hiiren liikkeitä, eikä kehräävää asemaa tarjoamassa mekaanista epäsäännöllisyyttä. Mistä _se_ saa entropiaa käynnistyttyään? Entä jos hyökkääjä tai huono onni, pakottaa säännöllisiä uudelleenkäynnistyksiä? Tämä on [todellinen ongelma](http://www.theregister.co.uk/2015/12/02/raspberry_pi_weak_ssh_keys/).

```text
sudo apt-get install rng-tools
```

```text
sudo reboot
```

