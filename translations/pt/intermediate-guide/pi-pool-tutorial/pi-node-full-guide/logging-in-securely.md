---
description: >-
  Generate a strong ssh keypair, boot your Raspberry Pi, copy ssh pub key and login
---

# Logging in Securely

{% hint style="warning" %}
It is assumed that you are using a Linux or Mac operating system with native support for ssh as your local machine. Or, if using Windows have a tool set that will work with this guide. Perhaps now is the time to switch to Linux and not look back. [https://elementary.io/](https://elementary.io/).
{% endhint %}

## Create a new ssh key pair

let's create a new password protected ED25519 key pair on our local machine. Give it a unique name and password protect it.

```bash
ssh-keygen -a 64 -t ed25519
```

{% hint style="info" %}
[`-a`](https://man.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man1/ssh-keygen.1#a) rounds When saving a private key, this option specifies the number of KDF \(key derivation function, currently [bcrypt\_pbkdf\(3\)](https://man.openbsd.org/bcrypt_pbkdf.3)\) rounds used. Higher numbers result in slower passphrase verification and increased resistance to brute-force password cracking \(should the keys be stolen\). The default is 16 rounds.

[https://flak.tedunangst.com/post/new-openssh-key-format-and-bcrypt-pbkdf](https://flak.tedunangst.com/post/new-openssh-key-format-and-bcrypt-pbkdf)
{% endhint %}

Your new key pair will be located in ~/.ssh

```bash
cd $HOME/.ssh
ls -al
```

## Boot your Pi & login

Plug in a network cable connected to your router and boot your new image.

### Login credentials

| 🍓 Default Pi-Node Credentials | 🦍 Default Ubuntu Credentials |
|:----------------------------- |:---------------------------- |
| username = ada                | username = ubuntu            |
| password = lovelace           | password = ubuntu            |

{% hint style="warning" %}
Upon successful login you will be prompted to change your password & login with new credentials.
{% endhint %}

### Obtain IPv4 address

Either log into your router and locate the address assigned by it's dhcp server or connect a monitor. Write the Pi's IPv4 address down.

```bash
hostname -I | cut -f1 -d' '
```

## Copy ssh pub key to new server

Add your newly created public key to the Pi's authorized\_keys file using ssh-copy-id.

{% hint style="info" %}
Pressing the tab key is an auto complete feature in terminal. Getting into the habit of constantly hitting tab will speed things up, give insight into options available and prevent typos. In this case ssh-copy-id will give you a list of available public keys if you hit tab a couple times after using the -i switch. Start typing the name of your key and hit tab to auto complete the name of your ed25519 public key.
{% endhint %}

Enter the default password associated with your img.gz.

{% tabs %}
{% tab title="Pi-Pool" %}
```bash
ssh-copy-id -i <ed25519-keyname.pub> ada@<server-ip>
```
{% endtab %}

{% tab title="Ubuntu" %}
```bash
ssh-copy-id -i <ed25519-keyname.pub> ubuntu@<server-ip>
```
{% endtab %}
{% endtabs %}

ssh should return 1 key added and suggest a command for you to try logging into your new server.

> Number of key\(s\) added: 1
> 
> Now try logging into the machine, with: **&lt;run this in terminal&gt;**

## Log into your server with ssh

Run the suggestion and you should be greeted with your remote shell. Congratulations! 🥳

