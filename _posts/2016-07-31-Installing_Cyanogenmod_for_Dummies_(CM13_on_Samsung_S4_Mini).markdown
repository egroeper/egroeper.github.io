---
layout: post
title:  "Installing Cyanogenmod for Dummies (CM13 on Samsung S4 Mini)"
date:   2016-07-31 07:02:32 +0200
categories: [android, cyanogenmod]
---

## Introduction

I'm building, fixing and tweaking it systems for a living. So I wanted something for my mobile phone, that just works. That's the reason why I went with the stock rom for quite a long time although I don't like the idea of running an outdated Android 4.4 on a daily basis. I still don't understand how the Samsung S4 mini can still be sold as new although, to my understanding, it doesn't have a recent OS with proper security updates. But that seems to be a general problem with mobile phone manufacturers.

As Samsung [dropped its plans to deploy a newer Android version on the S4 mini][samsung-no-update], I had to look for alternatives and I'm pretty glad for the great community around Cyanogenmod (and other ROM projects), that makes recent Android versions available on older devices.

As there is already a great amount of docs out there, this manual intends to combine different docs I used and should make installing yet another bit easier or, to speak frankly, for dummies, like my future me.

As I'm a Linux guy only Linux tools are used in this guide.

## DISCLAIMER

You should be aware, that you **loose your warranty** when installing a custom rom. But this possibly is your smallest problem. Following this guide you may **get your phone bricked** (oh, a nice paperweight you have there) and **all your data destroyed**. Since the custom roms are made with love, but come with **absolutely no warranty** (as this guide!) it's perfectly possible, that you **loose your job**, because the alarm clock doesn't work or you **drive into a lake** because of a malfunctioning gps chip driver and you blindly following your navigation app.


## Let's have fun!

To have everything in one place I suggest to create a folder for this: 

```
mkdir ~/My_little_CM_adventure
cd ~/My_little_CM_adventure
```

### Overview

* Identify the Custom ROM image for your device? Are there any available?
* Making backups
* More backups
* Installing custom recovery rom (TWRP)
* Installting Cyanogenmod 13 (Android 6.0.1)

### How to find my image?

For Cyanogenmod there is a [list of supported devices][cm-devices]. As I'm having a Samsung S4 Mini, we can search for [Samsung devices with recent CM][cm-devices-samsung]. The S4 Mini has two models supported by Cyanogenmod: [LTE (GT-I9195)][serranoltexx] and [3G (GT-I9190)][serrano3gxx]. Your device model is shown in your device settings under "About phone".

### Backups, Backups, Backups!

Your are really significantly happier, if you have proper backups.

To my (little) knowledge there are two types of backups:

* full system backups (partition images, Nandroid Backups)
* backups using `adb backup` (Androids backup mechanism)

The Nandroid backups contain basically all files as complete partition images. So the data is there, but it is hard to extract it partially, if you don't know what you are doing (and you don't if you are reading this guide). They can be used to restore an old system state (**haven't tested this yet!**).

The Android backups (ab) can be used to restore individual apps (if you use [android-backup-extractor][android-backup-extractor] to split it). Their main problem is, that you can't backup all your apps and settings. Apps may simply opt-out of this mechanism by setting an attribute.
I made both types of backups and used the android backup to restore some of my apps (their settings) after installing CM13.

Additionally it could be useful to make app-specific backups of your most important apps / data. You could or should export your contacts (which I didn't...) and/or [k9-mail settings][k9-settings-export]. Just take the time and google around for your most important apps. I lost all my short messages during this update. But because afaik the short message app changes, I don't know if it is possible to migrate them.
For k9 mail I simply restored the Android backup (see below), which worked fine.


#### Android Backup

If you are already developing Android Apps (like me) and therefore have Android Studio installed, then you are fine.
Otherwise you at least need to install the [Android SDK Tools][android-sdk-dl] (example: android-sdk_r24.4.1-linux.tgz).

After installing the Android SDK Tools you can backup your phone using the adb platform tool: 

```
adb backup -apk -shared -all -f ./my_S4mini.ab
```

#### Installing Recovery Image

Now it is time to install the custom recovery image.
You *really* should follow the official guide for your device ([LTE (GT-I9195)][serranoltexx-iguide] or [3G (GT-I9190)][serrano3gxx-iguide]). But for your convenience I summed up the steps here, too (This is for Linux only!).

1. install heimdall (for Ubuntu >=14.04: `sudo apt-get install heimdall-flash`)
2. Download [recovery image][twrp-image] and unzip it `unzip openrecovery-twrp-3.0.2-1-serrano3gxx-STABLE-signed.zip` (or lte version)
3. Power off the Galaxy S4 Mini and connect the USB adapter to the computer but not to the Galaxy S4 Mini, yet.
4. Boot the Galaxy S4 Mini into *download mode*. **Vol Down & Home & Power** Accept the disclaimer on the device. Then, insert the USB cable into the device.
5. At this point, familiarize yourself with the [Flashing with heimdall notes][flashing-heimdall] (see info box in original manual) so that you are prepared for any strange behaviour if it occurs.
6. On the computer, open a terminal in the directory where the recovery image is located and type (*I prepended sudo here to avoid problems*):

   ```
   sudo heimdall flash --RECOVERY recovery.img --no-reboot
   ```
7. A blue transfer bar will appear on the device showing the recovery being transferred.
8. Unplug the USB cable from your device.

   **NOTE**: Be sure to reboot into recovery immediately after having installed the custom recovery. Otherwise the custom recovery will be overwritten and the device will reboot (appearing as though your custom recovery failed to install).
9. Manually reboot the phone into recovery mode by performing the following. **Vol Up & Home & Power**


##### udev workaround

If you are having problems flashing your phone with heimdall, it may help to [temporarely block your Samsung device in udev][udev-workaround], to prevent your os catching the device for something you currently don't want (like MTP):

1. Create udev rule

   ```
   sudo su -
   cat > /etc/udev/rules.d/90-temp-samsung.rules <<EOF
   ATTRS{idVendor}=="04e8", ENV{ID_MM_DEVICE_IGNORE}="1"
   EOF
   ```
2. Restart udev
   ```
   sudo service udev restart
   ```

Don't forget to remove this rule again after flashing with heimdall.


#### System (nandroid) backup

On first boot into TWRP you get asked, if you want to start in read-only mode. I suggest you do that to create the system backup. Here is why:


> System read only option:
>
> Devices that ship with 5.0 and higher as their initial OS are using block level OTA updates. With this style of OTA update, the update script checks to see if the system partition has ever been mounted read/write. Further, the script also usually runs an SHA sum of the entire system partition to detect if any changes have been made. If any changes have been made, the OTA update will refuse to install. Since not all OEMs and devices have factory images available, we have created a new feature in TWRP that detects if the system partition has ever been mounted read/write. If not, you will be prompted asking if you want TWRP to mount system as read/write. If you choose not to allow TWRP to mount as read/write, TWRP won’t prompt to install SuperSU and TWRP won’t try to patch the stock ROM to prevent TWRP from being replaced by stock recovery. The goal of this option is to hopefully allow the user to make a raw system image backup that they can use to get back to a state where they can take OTA updates again.

Yes, technically this shouldn't be necessary for the Samsung S4 mini as it ships with Android 4. But it doesn't do any harm and could be useful in the future (when flashing CM on the next generation of Android devices).

Now use the backup option of TWRP to create your system backup. Just to be sure I simply selected all partitions.

After finishing backup, reboot into recovery mode using **Vol Up & Home & Power**. This time we enter read-write mode.

### Finally installing CM13

1. Download the CyanogenMod build package for your device ([LTE (GT-I9195)][serranoltexx-dl] or [3G (GT-I9190)][serrano3gxx-dl]) that you'd like to install to your computer.
   Optional: Download 3rd party applications packages, like [Google Apps][gapps] which are necessary to download apps from Google Play.

   [Google Apps][gapps] comes with different [packaging options (sizes)][gapps-flavors]. I chose [nano][gapps-nano] (the default). That way I have access to Google Play and may download other Google apps from there to my liking.

3. You should be in recovery mode
2. Place the CyanogenMod .zip package, as well as any optional .zip packages, on the root of `/sdcard`:  
   Using adb: `adb push filename.zip /sdcard/`

   I did:

   ```
   adb push cm-13.0-20160418-SNAPSHOT-ZNH0EAO2NK-serrano3gxx.zip /sdcard/
   adb push open_gapps-arm-6.0-nano-20160728.zip /sdcard/
   ```

4. You should already have made backups
5. Select **Wipe** and then **Factory Reset**.
6. Select **Install**, navigate to `/sdcard` and select the CyanogenMod `.zip` package.
   
   Optional: Select any additional packages you wish (like Google Apps; CyanogenMod should be first).

   Follow the on-screen notices to install the package(s).

7. Once installation has finished, return to the main menu and select **Reboot**, then **System**. The device will now boot into CyanogenMod.


You should have a working CM13 on your Samsung S4 Mini by now. Congratulations!  

If not: *I warned you!* If you are lucky, searching for your error message will help you solve the problems, as there is a great (and big) community around Cyanogenmod. Good luck!


## Cleanup, Optimizations, data migration, accessing your (backup) data

### Undo udev workaround (if needed)

If you haven't already, this is a good moment to undo the changes from the udev workaround (if you applied them):

```
sudo rm /etc/udev/rules.d/90-temp-samsung.rules
sudo service udev restart
```

### Splitting your Android Backup to restore some apps (state)

So now you have a freshly installed CM13 on your phone, but all your apps are gone.
Just reinstall them from Google Play (or the other app stores). Unfortunately all your settings and app data vanished. I didn't export the k9 mail settings and wanted them back.

Unfortunately the Android backup we created earlier is a full backup and restoring it would mean to restore everything. IMHO this could be a problem with our fresh new OS (and we got rid auf Samsungs special apps and stuff). So how to restore only specific apps? Luckily there is already a tool for this, as we aren't the first ones with this problem: [Android Backup Extractor][abe]. 

So we download it and start splitting our backup (you should have some time and space available):

```
cd ~/My_little_CM_adventure
mv ~/Downloads/android-backup-extractor-20160710-bin.zip .
unzip android-backup-extractor-20160710-bin.zip 
cd android-backup-extractor-20160710-bin/
bash adb-split-no-extraction.sh ../my_S4mini.ab
```

After that we will get individual `.ab` files in the `split-ab` subdirectory.

If you like to peek into one, you can use this one-liner I found on [Stackexchange][ab-extract] (here for k9):

```
( printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" ; tail -c +25 com.fsck.k9.ab ) | tar tzf -
```

Restore each app backup by calling and confirming on your mobile phone afterwards:

```
adb restore app.ab
```

For k9 I called: `adb restore com.fsck.k9.ab`.

To find the `.ab` file for your app should work by simply searching in `split-ab` for a unique substring, like `find . -iname "*k9*"`.  
If this doesn't work, you could go the [Google Play][gplay] with your webbrowser, search for the app and open its play store page. You now get the package name by looking at the URL in your browser address field. For *Signal* the package name is `org.thoughtcrime.securesms`. But as a security application, it seems to have opted out on Android Backup. So you have no `.ab` file to restore.

I find this especially useful for some games I have on my phone. Who wants to loose highscores?



### Getting your contacts back

If your are as stupid as me, you are really missing your contacts after installing CM.  
I strongly suggest you find a better way to export/reimport them. But that way we can use it as an example.

Fortunately I made a full (nandroid) backup of my data before installing CM. TWRP creates a directory filled with tar files for that. To trick you, they aren't named `.tar` files. Google and `tar tf` are your friend. Looks like we are searching for `contacts2.db`. For reimport there is a nice [tool][dump-contacts2db], that converts this Sqlite3 database into a `.vcf`.

So here is what I did:

```
tar xf 2016-07-29--17-22-02_KOT49H.I9195XXUCPE1_beforeCM/data.ext4.win000 /data/data/com.android.providers.contacts/databases/contacts2.db
cd data/data/com.android.providers.contacts/databases/
sqlite3 contacts2.db # just curious, not needed
git clone https://github.com/stachre/dump-contacts2db.git ./dump-contacts2db
bash ./dump-contacts2db/dump-contacts2db.sh contacts2.db > contacts2.vcf
```

Afterwards you can import the contacts2.vcf into your contacts. Don't know if something is missing. Looks good at first sight.

## Conclusion

To me it seems, that installing CM on a Samsung S4 Mini is pretty easy thanks to the many and engaged people out there working on roms, tools and docs. Here I want to say "Thank you!".

I hope my guide is useful for someone. Have fun!

[serrano3gxx]: https://wiki.cyanogenmod.org/w/Serrano3gxx_Info
[serranoltexx]: https://wiki.cyanogenmod.org/w/Serranoltexx_Info
[serrano3gxx-iguide]: https://wiki.cyanogenmod.org/w/Install_CM_for_serrano3gxx
[serranoltexx-iguide]: https://wiki.cyanogenmod.org/w/Install_CM_for_serranoltexx
[serrano3gxx-dl]: http://download.cyanogenmod.org/?device=serrano3gxx
[serranoltexx-dl]: http://download.cyanogenmod.org/?device=serranoltexx
[cm-devices]: https://wiki.cyanogenmod.org/w/Devices
[cm-devices-samsung]: https://wiki.cyanogenmod.org/w/Devices#vendor="Samsung";type="phone";cmversions="13";
[samsung-no-update]: https://www.change.org/p/samsung-update-the-galaxy-s4-mini-international-versions-to-android-5-0-lollipop
[android-backup-extractor]: https://sourceforge.net/projects/adbextractor/
[android-sdk-dl]: https://developer.android.com/studio/index.html#downloads
[k9-settings-export]: https://github.com/k9mail/k-9/wiki/Manual-Settings-ImportExport
[twrp-image]: https://www.androidfilehost.com/?w=files&flid=33684
[flashing-heimdall]: https://wiki.cyanogenmod.org/w/Install_CM_for_serrano3gxx
[udev-workaround]: https://tribaal.io/flashing-cyanogenmod-on-a-samsung-s5-from-ubuntu.html
[gapps-nano]: https://github.com/opengapps/opengapps/wiki/Nano-Package
[gapps-flavors]: https://github.com/opengapps/opengapps/wiki/Package-Comparison
[gapps]: http://opengapps.org/
[abe]: https://sourceforge.net/projects/adbextractor/
[ab-extract]: http://android.stackexchange.com/a/78183/90744
[gplay]: https://play.google.com
[dump-contacts2db]: https://github.com/stachre/dump-contacts2db
