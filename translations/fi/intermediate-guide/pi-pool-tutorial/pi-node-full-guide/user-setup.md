---
description: 'Luo ada käyttäjä, lisää ryhmään sudo'
---

# Käyttäjän Asetukset

{% hint style="Huomaa" %}
Jos käytät Pi-Node -imagea, tarvitset tätä aineistoa vain antamaan osviittaa, kunnes aloitamme Relay & Coren asetusten määrittämisen.
{% endhint %}

## Luo ada käyttäjä

Luo uusi käyttäjä ja lisää se sudo-ryhmään.

```bash
sudo adduser ada && sudo adduser ada sudo
```

### Vaihda salasana

Voit vaihtaa ada käyttäjien salasanaa milloin tahansa.

```bash
passwd
```

{% hint style="Huomaa" %}
Ole huolellinen milloin käytät sudo-komentoa. Esimerkiksi 'sudo passwd' muuttaisi root -käyttäjän salasanan. Tämä tuntuu olevan kohta, jossa uudet käyttäjät hämmentyvät helposti.
{% endhint %}

### Kirjaudu sisään ada käyttäjänä

Lisätään meidän ssh julkinen avaimemme uuteen $USER adan authorized\_keys tiedostoon. Sitten voimme kirjautua sisään ada:na ja poistaa oletuksena olevan 'ubuntu' käyttäjätilin.

```bash
# siirry takaisin paikalliseen päätelaitteeseen
exit
```

Käytä ssh-copy-id:tä lisätäksesi julkisen avaimesi ada-käyttäjän authorized\_keys tiedostoon.

```bash
ssh-copy-id -i <ed25519-keyname.pub> ada@<server-ip>
```

ja kirjaudu sisään ssh:n kautta käyttäjänä ada.

```bash
ssh ada@<server-ip>
```

Testaa, että ada on sudo-ryhmässä päivittämällä pakettiluettelot ja päivittämällä järjestelmä.

{% hint style="Huomaa" %}
Ubuntun päivittäminen kestää 10 minuuttia tai enemmän, ellei rpi-imagerilla käytettyä kuvaa ole päivitetty.
{% endhint %}

```bash
sudo apt update && sudo apt upgrade
```

Jos päivitys onnistui, voimme poistaa ubuntu käyttäjän ja sen kotihakemiston.

```bash
sudo deluser --remove-home ubuntu
```

## ssh

Muokkaa OpenSSH:n asetustiedostoa ja tee seuraavat muutokset Nanon tekstieditorilla.

{% hint style="info" %}
Tutustu tähän [nano lunttilistaan](https://www.nano-editor.org/dist/latest/cheatsheet.html)!

Kaikki \# kommentoidut arvot sshd\_config tiedostossa ovat oletusarvoja. Poista punta merkki ja vaihda arvo vastaamaan alla olevaa. Ne ladataan seuraavan kerran järjestelmän uudelleenkäynnistäessä ssh-palvelun.
{% endhint %}

```bash
sudo nano /etc/ssh/sshd_config
```

{% hint style="Huomaa" %}
Poista salasanan todennus käytöstä.
{% endhint %}

```bash
#    $OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

# Tämä on sshd-palvelimen koko järjestelmän asetustiedosto.  Katso lisätietoja kohdasta
# sshd_config5.

# Tämä sshd on rakennettu PATH=/usr/bin:/bin:/bin:/usr/sbin:/sbin

# Strategia, jota käytetään oletuksena sshd_config:ssä
# OpenSSH:n kanssa on määrittää vaihtoehtoja oletusarvolla, jos
# mahdollista, mutta jättä ne kommentoiduiksi.  Kommentoimattomat asetukset ohittavat
# oletusarvon.

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
En ole oletusporttien vaihtamisen puolestapuhuja, kun minun ei tarvitse sitä tehdä. Vahva avainpari ja fail2ban riittää minulle. Hyökkääjän ei ole liian vaikeaa selvittää, mitä porttia ssh kuuntelee, jos he todella haluavat.
{% endhint %}

