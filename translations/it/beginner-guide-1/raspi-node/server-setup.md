---
description: 'optimize hardware, harden Ubuntu'
---

# Server Setup

## Configure Hardware

Let's save some power, raise the governor on the CPU a bit, and set GPU ram as low as we can.

{% hint style="warning" %}
Here are some links for overclocking and testing your drive speeds. If you have heat sinks you can safely go to 2000. Just pay attention to over volt recommendations to go with your chosen clock speed.

* [https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)
* [https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/](https://www.seeedstudio.com/blog/2020/02/12/how-to-safely-overclock-your-raspberry-pi-4-to-2-147ghz/)
* [https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock](https://www.tomshardware.com/how-to/raspberry-pi-4-23-ghz-overclock)
* [https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/](https://dopedesi.com/2020/11/24/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-by-alex-ellis-nov-2020/)
* [Legendary Technology: New Raspberry Pi 4 Bootloader USB](https://jamesachambers.com/new-raspberry-pi-4-bootloader-usb-network-boot-guide/)
{% endhint %}

### Test drive speed

#### Write speed

```text
sudo dd if=/dev/zero of=/tmp/output conv=fdatasync bs=384k count=1k; sudo rm -f /tmp/output
```

#### Read speed

```text
sudo hdparm -Tt /dev/sda
```

### Overclock, memory & radios

Edit /boot/firmware/config.txt. Just paste Pi Pool additions in at the bottom.

```bash
sudo nano /boot/config.txt
```

```bash
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
#gpu_mem=16
disable-wifi
#disable-bt
```

Save and reboot.

```text
sudo reboot
```

## Configure Raspberry Pi OS

### Update The System

```text
sudo apt update && sudo apt upgrade
```

### Disable the Pi user

```text
sudo passwd -l pi
```

### Disable the root user

```text
sudo passwd -l root
```

### Secure shared memory

Mount tmpfs as read only.

Open /etc/fstab.

```text
sudo nano /etc/fstab
```

Add this line at the bottom, save & exit.

```text
tmpfs    /run/shm    tmpfs    ro,noexec,nosuid    0 0
```

### Increase open file limit

Open /etc/security/limits.conf.

```text
sudo nano /etc/security/limits.conf
```

Add the following to the bottom, save & exit.

```text
ada soft nofile 800000
ada hard nofile 1048576
```

### Optimize performance & security

Add the following to the bottom of /etc/sysctl.conf. Save and exit.

{% hint style="info" %}
[https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3)
{% endhint %}

{% hint style="warning" %}
I am disabling IPv6 and IPv4 forwarding. You may want these. I have seen claims that IPv6 is slower and gets in the way.
{% endhint %}

```text
sudo nano /etc/sysctl.conf
```

```bash
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

#### Load our changes after boot

Create a new file. Paste, save & close.

```text
sudo nano /etc/rc.local
```

```text
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

sleep 120

sysctl -p /etc/sysctl.conf

exit 0
```

### Disable IRQ balance

{% hint style="info" %}
[**http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/**](http://bookofzeus.com/harden-ubuntu/server-setup/disable-irqbalance/)
{% endhint %}

You should turn off IRQ Balance to make sure you do not get hardware interrupts in your threads. Turning off IRQ Balance will optimize the balance between power savings and performance through the distribution of hardware interrupts across multiple processors.

Open /etc/default/irqbalance and add to the bottom. Save, exit and reboot.

```text
sudo nano /etc/default/irqbalance
```

```text
ENABLED="0"
```

### Chrony

We need to get our time synchronization as accurate as possible. Open /etc/chrony/chrony.conf

```text
sudo apt install chrony
```

```bash
sudo nano /etc/chrony/chrony.conf
```

Replace the contents of the file with below, Save and exit.

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
We have found that cardano-node can safely use this compressed swap in ram essentially giving us around 20gb of ram. We already set kernel parameters for zram in /etc/sysctl.conf
{% endhint %}

Swapping to disk is slow, swapping to compressed ram space is faster and gives us some overhead before out of memory \(oom\).

{% embed url="https://haydenjames.io/raspberry-pi-performance-add-zram-kernel-parameters/" caption="" %}

{% embed url="https://lists.ubuntu.com/archives/lubuntu-users/2013-October/005831.html" %}

Disable Raspbian swapfile.

```text
sudo systemctl disable dphys-swapfile.service
```

```text
sudo apt install zram-tools
```

```bash
sudo nano /etc/default/zramswap
```

Multiply default config by 3. This will give you 12.5GB of virtual compressed swap in ram.

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
View how much zram swap cardano-node is using.

```text
CNZRAM=$(pidof cardano-node)
grep --color VmSwap /proc/$CNZRAM/status
```
{% endhint %}

