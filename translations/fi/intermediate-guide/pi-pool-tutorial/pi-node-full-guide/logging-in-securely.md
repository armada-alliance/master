---
description: >-
  Luo vahva ssh avainpari, käynnistä Raspberry Pi, kopioi ssh pub-avain ja kirjaudu sisään
---

# Suojattu kirjautuminen

{% hint style="Huomaa" %}
Oletuksena on, että käytät Linux- tai Mac-käyttöjärjestelmää, joka lähtökohtaisesti tukee ssh:ta ja toimii paikallisena koneena. Tai jos käytät Windowsia, sinulla on työkalu, joka toimii tämän oppaan kanssa. Tai ehkä nyt onkin aika siirtyä Linuxiin eikä katsoa taaksepäin. [](https://elementary.io/)https://elementary.io.
{% endhint %}

## Luo uusi ssh avainpari

Luodaan uusi salasanasuojattu ED25519 avainpari meidän paikalliseen koneeseen. Anna sille yksilöllinen nimi ja suojaa se salasanalla.

```bash
ssh-keygen -a 64 -t ed25519
```

{% hint style="info" %}
[`-a`](https://man.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man1/ssh-keygen.1#a) rounds Yksityistä avainta tallennettaessa tämä valinta määrittää KDF \(avain johtamisfunktio, tällä hetkellä [bcrypt\_pbkdf\(3\)](https://man.openbsd.org/bcrypt_pbkdf.3)\) kierrosten määrän. Korkeammat numerot hidastavat salasanalauseiden tarkistamista ja lisäävät vastustuskykyä brute-force salasana crackäämiselle \(jos avaimet on varastettu\). Oletuksena on 16 kierrosta.

[https://flak.tedunangst.com/post/new-openssh-key-format-and-bcrypt-pbkdf](https://flak.tedunangst.com/post/new-openssh-key-format-and-bcrypt-pbkdf)
{% endhint %}

Uusi avain pari sijaitsee kansiossa ~/.ssh

```bash
cd $HOME/.ssh
ls -al
```

## Käynnistä Pi & kirjaudu sisään

Kytke verkkokaapeli, joka on kytketty reitittimeesi ja käynnistä uusi image.

### Sisäänkirjautumistiedot

| 🍓 Pi-Noden Oletustunnisteet | 🦍 Ubuntun Oletustunnukset |
|:--------------------------- |:------------------------- |
| käyttäjätunnus = ada        | käyttäjätunnus = ubuntu   |
| salasana = lovelace         | salasana = ubuntu         |

{% hint style="Huomaa" %}
Onnistuneen kirjautumisen yhteydessä sinua pyydetään vaihtamaan salasanasi & kirjautumaan uusilla tunnuksilla.
{% endhint %}

### Hae IPv4-osoite

Joko kirjaudu reitittimeesi ja paikanna dhcp-palvelimen määrittämä osoite, tai yhdistä monitori. Kirjoita Pi:n IPv4 -osoite ylös.

```bash
hostname -I | cut -f1 -d' '
```

## Kopioi ssh pub-avain uuteen palvelimeen

Lisää äskettäin luotu julkinen avain Pi:n authorized\_keys tiedostoon käyttäen ssh-copy-id.

{% hint style="info" %}
Tab-näppäimen painaminen on autotäydennys ominaisuus terminaalissa. Ota tavaksi näpäyttää jatkuvasti Tabia, niin asiat sujuvat nopeammin, saat enemmän näkemystä eri vaihtoehtoihin ja vältät suurimman osan kirjoitusvirheistä. Tässä tapauksessa ssh-copy-id antaa sinulle luettelon käytettävissä olevista julkisista avaimista, jos painat Tabia pari kertaa -i -kytkimen käytön jälkeen. Aloita kirjoittamalla avain nimi ja paina Tab autotäydentäämään oman ed25519 julkisen avaimen nimi.
{% endhint %}

Anna oletussalasana, joka on liitetty img.gz:ään.

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

ssh:n pitäisi palauttaa "1 key added" ja ehdottaa komentoa, jonka avulla voit yrittää kirjautua uudelle palvelimellesi.

> Number of key\(s\) added: 1
> 
> Now try logging into the machine, with: **&lt;run this in terminal&gt;**

## Kirjaudu palvelimellesi ssh:n avulla

Suorita ehdotus ja sinun pitäisi päästä etänä palvelimen terminaaliin. Onneksi olkoon! 🥳

