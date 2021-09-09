---
description: >-
  Rakenna synkronoitu node noin tunnissa (ei enää tuntia 1.29)!
---

# Pi-Node \(pikaopas\)

{% hint style="info" %}
Ketjun lataaminen kestää noin 25 minuuttia ja noin tunti että se on synkronoitu kärkeen saakka. Et voi tehdä paljoakaan ennen kuin node on synkronoitu lohkoketjun kärkeen asti.

Uudelleenkäynnistyksen jälkeen voi kestää 5-50 minuuttia synkronoida ketju uudlleen riippuen siitä, miten node suljettiin tai käynnistettiin uudelleen. Tarkista htopilla, onko prosessi käynnissä. Jos se on, käytä gLiveView.sh -skriptiä monitorointiin tai mene kävelylle. Node synkronoituu ja socket luodaan.

On parasta vain jättää se käyntiin. 🏃♀
{% endhint %}

## Pikaohje

### **1. Lataa ja asenna** [**Pi-Node.img.gz**](https://db.adamantium.online/Pi-Node.img.gz)**.**

### 2. Ota ssh-yhteys palvelimeen.

```bash
ssh ada@<pi-node private IPv4>
```

Oletustiedot = **ada:lovelace**

### 3. Mene pi-poolin kansioon.

```bash
cd /home/ada/pi-pool
```

### 4. Lataa tietokannan tilannekuva.

```bash
wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/
```

### 5. Ota käyttöön & aloita cardano-palvelu.

{% hint style="Huomaa" %}
Odota, että wget saa ketjun lataamisen loppuun ennen cardano-servicen aloittamista. Odottaessasi, voit päivittää Ubuntun avaamalla palvelimeen toisen pääteikkunan.

```bash
sudo apt update
sudo apt upgrade
```
{% endhint %}

```bash
cardano-service enable
cardano-service start
```

### 6. Ota käyttöön & aloita cardano-monitor.

```bash
cardano-monitor enable
cardano-monitor start
```

### 7. Vahvista että palvelut ovat käynnissä.

```bash
cardano-service status
cardano-monitor status
```

### 8. gLiveView.sh

```bash
cd $NODE_HOME/scripts
./gLiveView.sh
```

### 9. Grafana.

Syötä Node:n IPv4 -osoite selaimesi osoitekenttään.

Oletus käyttäjätunnus ja salasana = **admin:admin**

#### Kojelaudat löytyvät täältä.

{% embed url="https://github.com/armada-alliance/dashboards" caption="" %}

{% embed url="https://api.pooldata.live/" caption="" %}

{% hint style="info" %}
Seuraava opas rakentaa imagen, käytä sitä viitteenä ja voit vapaasti pyytää selvennystä Telegram kanavassamme. [https://t.me/armada\_alli](https://t.me/armada_alli)
{% endhint %}

