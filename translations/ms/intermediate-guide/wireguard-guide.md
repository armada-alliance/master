# Wireguard Guide

From the [WireGuard](https://www.wireguard.com) project homepage:

WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPsec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it is now cross-platform (Windows, macOS, BSD, iOS, Android) and widely deployable.

{% hint style="success" %}
This guide will create a VPN from a core node behind a firewall to a relay node with Wireguard listening on the default port 51820. We will then control traffic between the connected interfaces with UFW.

Feel free to use a different port!
{% endhint %}

{% embed url="https://github.com/pirate/wireguard-docs" %}

### Install Wireguard

Do this on both machines.

```bash
sudo apt install wireguard
```

Become root.

```bash
sudo su
```

Enter the Wireguard folder and set permissions for any new files created to root only.

```bash
cd /etc/wireguard
umask 077
```

#### Configure

Generate key pairs on each machine.

{% tabs %}
{% tab title="C1" %}
```bash
wg genkey | tee C1-privkey | wg pubkey > C1-pubkey
```
{% endtab %}

{% tab title="R1" %}
```bash
wg genkey | tee R1-privkey | wg pubkey > R1-pubkey
```
{% endtab %}
{% endtabs %}

Create a Wireguard configuration file on both machines.

```bash
nano /etc/wireguard/wg0.conf
```

Use cat to print out the key values. Public keys are then used in the other machines conf file.

{% tabs %}
{% tab title="C1" %}
```bash
cat C1-privkey
cat C1-pubkey
```
{% endtab %}

{% tab title="R1" %}
```bash
cat R1-privkey
cat R1-pubkey
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="C1" %}
```bash
[Interface]
Address = 10.220.0.1/22
SaveConfig = true
ListenPort = 51820
PostUp = wg set %i private-key <path to private key>
##PostUp = resolvectl domain %i "~."; resolvectl dns %i 10.220.0.2; resolvectl dnssec %i yes

[Peer]
PublicKey = <result of cat R1-pubkey>
AllowedIPs = 10.220.0.2/22
Endpoint = <R1 nodes public ip or hostname>:51820
PersistentKeepalive = 21
```
{% endtab %}

{% tab title="R1" %}
```bash
[Interface]
Address = 10.220.0.2/22
SaveConfig = true
ListenPort = 51820
PostUp = wg set %i private-key <path to private key>

[Peer]
PublicKey = <result of cat C1-pubkey>
AllowedIPs = 10.220.0.1/22
#Endpoint = endpoint is not needed on the listening side
PersistentKeepalive = 21
```
{% endtab %}

{% tab title="Example" %}
```bash
[Interface]
Address = 10.220.0.1/22
SaveConfig = true
ListenPort = 51820
PostUp = wg set %i private-key /etc/wireguard/C1-privkey

[Peer]
PublicKey = FnXP9t17JXTCf3kyuTBh/z83NeJsE8Ar2HtOCy2VPyw=
AllowedIPs = 10.220.0.2/22
Endpoint = r1.armada-alliance.com:51820
PersistentKeepalive = 21
```
{% endtab %}
{% endtabs %}

#### [wg-quick](https://manpages.debian.org/unstable/wireguard-tools/wg-quick.8.en.html)

Use wg-quick to create the interface & manage Wireguard as a Systemd service on both machines

```bash
wg-quick up wg0
```

Useful commands.

```bash
sudo wg show # metrics on the interface
ip a # should see a wg0 interface
```

Once both interfaces are up you can try and ping each other.

{% tabs %}
{% tab title="C1" %}
```bash
ping 10.220.0.2
```
{% endtab %}

{% tab title="R1" %}
```bash
ping 10.220.0.1
```
{% endtab %}
{% endtabs %}

If they are connected bring them down and back up with Systemd

```bash
wg-quick down wg0
sudo systemctl start wg-quick@wg0
```

Enable the Wireguard service on both machines to automatically start on boot.

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl status wg-quick@wg0
```

{% hint style="danger" %}
SaveConfig saves the loaded wg0.conf file in runtime and overwrites the file when it stops. Therefore you must stop the wg-quick@wg0 service before editing the configuration file or your changes will be overwritten when you try to restart the service or reboot the server.

Like so

```bash
# become root
sudo su
# stop the service
systemctl stop wg-quick@wg0
# edit the configuration file
nano /etc/wireguard/wg0.conf
# start the service
systemctl start wg-quick@wg0
```
{% endhint %}

#### Topology

You can now update your C1 & R1 topology files so they point 10.220.0.2 & 10.220.0.1 respectively through the Wireguard VPN.

#### Prometheus

Likewise update IPv4 address' in /etc/prometheus/prometheus.yml to use the VPN.

### UFW

Control traffic through the VPN. The following allows for Prometheus/Grafana on C1 to scrape metrics from node-exporter on R1.

{% tabs %}
{% tab title="C1" %}
```c
# allow ssh access on lan behind router
sudo ufw allow 22
# deny ssh access from R1 to C1
sudo ufw deny in on wg0 to any port 22 proto tcp
# cardano-node port
sudo ufw allow 3000
```
{% endtab %}

{% tab title="R1" %}
```c
# allow ssh access
sudo ufw allow 22
# wireguard service
sudo ufw allow 51820/udp
# cardano-node port
sudo ufw allow 3001
# allow prometheus on C1 to scrape exporter metrics on R1
sudo ufw allow in on wg0 to any port 12798 proto tcp
sudo ufw allow in on wg0 to any port 9090 proto tcp
```
{% endtab %}
{% endtabs %}

**Bring up ufw**

When you're sure you are not going to lock yourself out and that all the ports for your pool that need to be open are you can bring up the firewall. Don't forget 80 & 443 if you have nginx proxying Grafana.

```c
sudo ufw enable
# view rules
sudo ufw status numbered
```

Notes & links/To Do

```c
PostUp = resolvectl domain %i "~."; resolvectl dns %i 192.0.2.1; resolvectl dnssec %i yes
```

```c
PostUp = wg set %i private-key /etc/wireguard/wg0.key <(cat /some/path/%i/privkey)
```

Reload conf without taking VPN down.

```
alias wgstrip='wg syncconf wg0 <(wg-quick strip wg0)'
```
