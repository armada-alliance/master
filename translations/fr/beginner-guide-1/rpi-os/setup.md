---
description: Dans ce tutoriel, nous couvrerons la configuration de base pour le Raspberry Pi et Linux
---

# Configurer le Raspberry Pi

## Résumé <a id="h.vrhvb96nxxe9"></a>

1. Télécharger un système d'exploitation \\(OS\\). Pour ce tutoriel, nous allons utiliser le Raspberry Pi OS.
2. Installez Raspberry Pi OS en utilisant Raspberry Pi Imager
3. Flasher le système d'exploitation sur la carte SD
4. Démarrez le Pi et configurez les paramètres
5. Insérez un SSD externe et copiez la carte SD
6. Arrêter et redémarrer à partir du SSD

{% hint style="info" %}
### NE PASSEZ PAS CETTE ÉTAPE
{% endhint %}

### **Première partie:**

### Installation du système d'exploitation Raspberry Pi Debian « buster » <a id="h.lpv6ciisjqp3"></a>

Nous allons maintenant télécharger la dernière version officielle de Raspberry Pi 64 bits Debian OS. C'est la distribution officielle de Linux 64bit OS qui est conçue pour le Raspberry Pi et son processeur ARM64. Cela le rend stable et très facile de commencer avec le Raspberry Pi.

**1. Download the Debian “buster” Raspberry Pi 64bit OS image** [**here**](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-04-09/2021-03-04-raspios-buster-arm64.zip) **and save it in an accessible location for now on your computer.**

**2. Ensuite, téléchargez le logiciel Raspberry Pi Imager que nous utiliserons pour installer l’OS sur notre Raspberry Pi. Ce logiciel est situé sur le site** [**Raspberry Pi**](https://www.raspberrypi.org/software/)**. Veuillez télécharger la version correcte pour votre ordinateur.**

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

**3. Insérez la carte SD dans votre ordinateur et ouvrez le "Raspberry Pi Imager".**

* **Click on "CHOOSE OS"  then find the** _**raspios-buster-arm64.zip**_ **file you have downloaded in step \(1\) of this tutorial and select it.**
* **Ensuite, cliquez sur "CHOISIR SD" et trouvez la carte SD que vous avez insérée dans l'ordinateur**
* **Maintenant, le bouton "WRITE" apparaîtra et vous pouvez cliquer dessus pour commencer à écrire/vérifier le système d'exploitation sur la carte SD.**
* **Enfin, une fois le processus d'écriture et de vérification terminé, vous verrez une fenêtre pop-up indiquant que l'OS a été correctement écrit sur la carte SD, cliquez sur « CONTINUER » et retirez votre carte SD de l'ordinateur.**

{% hint style="info" %}
#### **If you still have issues following the written instructions, watch the video below.**
{% endhint %}

{% embed url="https://www.youtube.com/watch?v=ntaXWS8Lk34" %}



### Partie 2:

### Configuration du Raspberry Pi

The first thing that we want to do is get the Raspberry Pi booted up and configured for our use.

To do this we will need to insert the SD card we flashed earlier with the Raspberry Pi OS into the bottom of the Raspberry Pi. Then we can insert our HDMI, Keyboard, Mouse, and power supply.

Once the Raspberry Pi startup screen is finished and you have booted into the Raspberry Pi OS Desktop screen we can now begin to set up our Raspberry Pi configuration and settings.

{% hint style="info" %}
If this is your first time booting up the Raspberry Pi OS you will have to follow some initial configurations listed below
{% endhint %}

* [ ] Tout d'abord, vous devez changer le nom d'hôte et le mot de passe du Raspberry Pi cela vous assurera que vous n'utilisez pas seulement les informations de base de connexion.
* [ ] Entrer les informations de connexion au Wifi \(**vous** **pouvez** **sauter cela si vous utilisez Ethernet**\)
* Régler le fuseau horaire.
* [ ] Choisir la langue et les paramètres du clavier.
* [ ] Mettre à jour le Raspberry Pi \(sauter ceci si vous désirez mettre à jour via la ligne de commande\)

{% hint style="success" %}
#### Une fois que vous avez terminé ces étapes initiales, Il est temps de procéder pour que le Rasberry Pi démarre à partir de son port USB afin que nous puissions utiliser notre SSD externe.
{% endhint %}

### Partie 3:

### Forcer le démarrage du RPI à partir du port USB

**This is the final step in this tutorial. We are going to first insert our external SSD into one of the USB 3.0 slots marked blue.**

![](../../.gitbook/assets/pi4.jpeg)

Open the Raspberry Pi applications menu and then click on the **SD Card Copier** application.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-9.11.39-pm%20%281%29.png)

Then we want to select **COPY FROM DEVICE** - **\(mmcblk0\) SD CARD.**

Next, select **COPY TO DEVICE - \(sda\) SSD Device.**

Once the copy process is complete open a new terminal window and enter the following command.

```text
sudo raspi-config
```

This will bring you to the Raspberry Pi's system configuration settings where you can access the **Advanced Options.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.19-pm.png)

Next select **Boot Order.**

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.13.40-pm%20%281%29.png)

Choisissez ensuite **USB Boot**.

![](../../.gitbook/assets/screen-shot-2021-03-29-at-10.14.05-pm%20%281%29.png)

Vous pouvez maintenant sélectionner **&lt;Ok&gt;** puis **&lt;Finish&gt;**, fermer le menu de configuration du système Raspberry Pi et redémarrer le Pi.

Vous devriez maintenant pouvoir éteindre le Pi après le redémarrage, retirer la carte SD, vous pouvez alors allumer le Raspberry Pi et il devrait démarrer à partir de votre périphérique de stockage USB externe.

{% hint style="success" %}
#### Maintenant que nous avons terminé la majeure partie de la configuration initiale, nous pouvons continuer à préparer le Pi et passer au [tutoriel](tutorial-2-relaynode.md) suivant.
{% endhint %}

