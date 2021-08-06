---
description: 'Create the ada user, add to group sudo'
---

# User Setup

{% hint style="warning" %}
If you are using a Pi-Node image you need only reference this material until we start configuring the Relay & Core.
{% endhint %}

## Create the ada user

Create a new user and add them to the sudo group.

```bash
sudo adduser ada && sudo adduser ada sudo
```

### Change password

You can change the ada users password at anytime with.

```bash
passwd
```

{% hint style="warning" %}
Careful where you use sudo. For example issuing 'sudo passwd' would change the root password. This seems to be a place where new users get confused.
{% endhint %}

### Login as ada

Let's add our ssh public key to our new $USER ada's authorized\_keys file. Then we can log in as ada and delete the default 'ubuntu' user.

```bash
# drop back into your local terminal
exit
```

Use ssh-copy-id to add your public key to ada users authorized\_keys file.

```bash
ssh-copy-id -i <ed25519-keyname.pub> ada@<server-ip>
```

and ssh into the Pi as ada.

```bash
ssh ada@<server-ip>
```

Test that ada is in the sudo group by updating your package lists and upgrading the system.

{% hint style="warning" %}
Updating Ubuntu is going to take 10 minutes or more unless they update the image used with rpi-imager.
{% endhint %}

```bash
sudo apt update && sudo apt upgrade
```

If that was successful we can delete the default ubuntu user and it's home directory.

```bash
sudo deluser --remove-home ubuntu
```

## ssh

Edit OpenSSH's configuration file and make the following changes with Nano text editor.

{% hint style="info" %}
Check out this [nano cheat sheet](https://www.nano-editor.org/dist/latest/cheatsheet.html) I just found!

All the \# commented out values in sshd\_config are the default values. Remove the pound sign and change the value to match below. They will be loaded the next time systemd restarts ssh.
{% endhint %}

```bash
sudo nano /etc/ssh/sshd_config
```

{% hint style="warning" %}
Turn off password authentication.
{% endhint %}

```bash
#    $OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 2
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile    .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem sftp  /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
```

{% hint style="info" %}
I am not a proponent of changing default ports when I don't have to. A strong key pair and fail2ban is enough for me. It is not too hard for an attacker to figure out what port ssh is listening on if they really want to.
{% endhint %}

