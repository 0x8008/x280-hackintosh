
# Lenovo ThinkPad T480 - OpenCore Configuation

<img align="right" src="https://i.imgur.com/zbcphj3.jpg" alt="macOS Ventura running on the T480" width="350">

[![macOS](https://img.shields.io/badge/macOS-Monterey-brightgreen.svg)](https://developer.apple.com/documentation/macos-release-notes)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.8.5-blue)](https://github.com/acidanthera/OpenCorePkg)
[![License](https://img.shields.io/badge/license-MIT-purple)](/LICENSE)

<p align="center">
   <strong>Status: Maintained (and sorta works)</strong>
   <br />
   <strong>Version: </strong>1.0
   <br />
</p>
</br>

## ⚠️ Disclaimer
**This guide is heavily based on [valnoxy's guide for the T480.](https://github.com/valnoxy/t480-oc) This is possible due to hardware similarities. Keep in mind that this still isn't perfect and you WILL encounter issues.**

This guide is only for the Lenovo ThinkPad X280. I am NOT responsible for any harm you cause to your device. This guide is provided "as-is" and all steps taken are done at your own risk.

> The ACPI patches and the style of this README are from [EETagent](https://github.com/EETagent/T480-OpenCore-Hackintosh).

## Introduction

<details>
<summary><strong>💻 My Hardware</strong></summary>
<br>
This is the configuration of my X280, however this configuation <strong>should still work</strong> with all X280 variants.

> **Note** Check the model of your WiFi & Bluetooth card. Intel cards should be compatible with itlwm (or AirportItlwm). If your card is from another manufacturer, please check if your card supports macOS.

| Category  | Component                            |
| --------- | ------------------------------------ |
| CPU       | Intel Core i5-8250U                  |
| GPU       | Intel UHD Graphics 620               |
| SSD       | Intel SSDPEKKF256G8L M.2 NVMe SSD    |
| Memory    | 8GB DDR4 2400Mhz                     |
| WiFi      | Intel Dual Band Wireless-AC 8265     |

</details>  

&nbsp;

## Installation

<details>  
<summary><strong>📝 Requirements</strong></summary>
</br>

You must have the following items:
- a working X280 (obviously);
- access to a working Windows machine with Python 3 installed;
- a pendrive with more than 4 GB (remember that during the preparation we will format the flash drive to create the installation media, which will **wipe all the data currently on it**);
- an Internet connection (recommended via Ethernet via dongle, but Wi-Fi should work fine);
- a few hours to troubleshoot everything. **By rushing the install you are bound to fuck something up!**

</details>

<details>  
<summary><strong>⚙️ Preperation</strong></summary>
</br>

### Create the install media

First of all, you will need an installer of macOS. We'll use [macrecovery](https://github.com/acidanthera/OpenCorePkg) to download and create the USB drive.

With macrecovery, the process is, as follows:
- Download [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) as a ZIP.
- Extract the OpenCorePkg-master.zip file.
- Open ```cmd.exe``` with Administrator privileges and change the directory to OpenCorePkg-master\Utilities\macrecovery.
- Enter the following command to download macOS 12:
```
python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download
```
- After the download succeeded, type ```diskpart``` and wait until you see ```DISKPART>```

- Plug-in your pendrive and type ```list disk``` to see your disk id.

- Select your pendrive by typing ```select disk <diskid>```. **Make sure you selected the correct device! Use ```detail disk``` to check.**

- Now, clean the pendrive and convert it to GPT; first, type ```clean``` and then ```convert gpt```.

>  **Note**: If an error occurred, try to convert again by typing ```convert gpt```.

- **If your drive is over 32GB, you will encounter issue with the next step** - DiskPart can only handle FAT32 formatting for drives up to 32GB, hence to create a FAT32 partition you will need an external tool. I personally recommend [minitool's partition wizard](https://cdn2.minitool.com/?p=pw&e=pwfree-64bit-portable). A tutorial for this will be added later. If you follow this path, skip the next step (and **ONLY** the next step).

- After the pendrive is cleaned and converted, create a new partition for the installer and EFI; type ```create partition primary```, then select the new partition with ```select partition 1``` and format it ```format fs=fat32 quick```.

- Finally, mount your pendrive by typing ```assign```

- Now, close the Command Prompt and copy the folder ```com.apple.recovery.boot``` (with its contents) from ```OpenCorePkg-master\Utilities\macrecovery``` onto the root the pendrive.

After the install media was created, you need to make the USB drive bootable.

### Configure and install OpenCore
Download the EFI folder from this repo, download the repo as a .zip and drag the ```EFI``` folder onto the root of your pendrive.

#### GenSMBIOS
To configure your SMBIOS, you need to use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS). The tool will automatically generate a fake serial number, UUID and MLB for you. **This step is essential to have working iMessage, so do not skip it!**

- Download GenSMBIOS as a ZIP, then extract it.
- Start GenSMBIOS.bat and use option ```1``` to download MacSerial.
- Choose option ```2```, to select the path of the config.plist file. It will be located in ```EFI -> OC``` folder on your pendrive.
- Choose option ```3```, and enter ```MacBookPro15,2``` as the machine type.
- Press ```Q``` to quit. Your config now should contain the requied serials.

#### Enter the proper ROM value
After adding serials to your config.plist, you have to add the computer's MAC address to the config.plist file. **This step is also essential to have a working iMessage, so do not skip it.** We need a ```.plist``` editor to write the MAC address into the config.plist file. I recommend[ProperTree](https://github.com/corpnewt/ProperTree) as it works on Windows. You have to change the MAC address value in the config.plist at

```PlatformInfo -> Generic -> ROM```

Delete the generic ```123456789012``` value, and enter your MAC address into the field, without any colons. You can get this address on [Windows](https://kb.netgear.com/1005/How-do-I-find-my-device-s-MAC-address) and [Linux](https://cets.seas.upenn.edu/answers/find-mac-address.html) by following the linked tutorials. Save the ```.plist``` file by pressing ```Ctrl+S```.

#### Default keyboard layout and language
The default keyboard layout and language is ```Polish```. To change the language, edit the value of ```NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> prev-lang:kbd``` to the value of your language. This part is a bit complicated; if your value contains an underscore "```_```", replace it with a hyphen "```-```". The value for English would be ```en-US:0```. You can find a list of all language values [here](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt). For example, if you were to pick Portugese, the value in this file is ```[10] pt_PT - Portuguese``` - hence, the value that you need to put in your ```.plist``` would be ```pt_PT:10```.

### Install OpenCore
If you followed all the steps correctly and copied the ```EFI``` folder before, OpenCore should already be installed and ready to boot.

</details>

<details>  
<summary><strong>🚚 Installation</strong></summary>
</br>

### Prepare BIOS
The bios **must** be properly configured prior to installing macOS.
In Security menu, set the following settings:

-  `Security > Security Chip`: must be **Disabled**
-  `Memory Protection > Execution Prevention`: must be **Enabled**
-  `Virtualization > Intel Virtualization Technology`: must be **Enabled**
-  `Virtualization > Intel VT-d Feature`: must be **Enabled**
-  `Anti-Theft > Computrace -> Current Setting`: must be **Disabled**
-  `Secure Boot > Secure Boot`: must be **Disabled**
-  `Intel SGX -> Intel SGX Control`: must be **Disabled**
-  `Device Guard`: must be **Disabled**

In Startup menu, set the following options:

-  `UEFI/Legacy Boot`: **UEFI Only**
-  `CSM Support`: **No**

In Thunderbolt menu, set the following options:

-  `Thunderbolt BIOS Assist Mode`: **Disabled**. Do **NOT** set this to anything else, as it is known to cause potential bricks on the X280, which you **won't be able to fix at home!**
-  `Wake by Thunderbolt(TM) 3`: **No**
-  `Security Level`: **No**
-  `Support in Pre Boot Environment > Thunderbolt(TM) device`: **No**

Now you can go through the install.

### Install macOS
1. Boot from USB, press ```SPACE``` and select the USB drive inside of OpenCore ```"NO NAME (DMG)" or similar```.
>  **Note:** The first boot may take up to 20 minutes.
2. Wait for the macOS Utilities screen.
3. Connect to your network if using Wi-Fi.
4. Select Disk Utility, select your disk and click erase. Give a name and choose **APFS** with **GUID Partition Map**.
5. After erasing, go back and select **Reinstall macOS** and follow the steps on your screen. The installation make take up to **2 hours**, in my case with a 300Mbps download connection it took around an hour.
>  **Note:** Your PC will restart multiple times. If this happens (and it will), just boot from USB again and select your disk inside of OpenCore. It will be named either macOS Installer or the name you picked when formatting the drive.
6. Once you see the `Region selection` screen, you are good to proceed.
7. Create your user accound and everything else. You should be able to log in with iCloud if you generated your SMBIOS correctly and changed the MAC address.

</details>

<details>  
<summary><strong>♻️ Upgrade macOS / Switch EFI</strong></summary>
</br>

If you want to upgrade macOS, download the desired macOS version in the Settings app and simply perform the upgrade like on a real Mac; however, if you plan to upgrade your EFI, you'll need a different OpenCore configuation.

> Note: Download the desired macOS version in the Settings before following these steps, if you are connected via WiFi.

1. Download the newest release & [ProperTree](https://github.com/corpnewt/ProperTree) and extract it.
2. Start ProperTree and load the ```Config.plist``` on your EFI partition. (File -> Open)
> Note: You can mount your EFI partition by pressing ```ALT + SPACE```, typing Terminal and enter the following command: ```sudo diskutil mountDisk disk0s1```.
3. Now also load the new configuration file from the repo for the desired macOS installation (or HeliPort config). 
4. You should now have 2 ProperTree-windows open on your screen.
5. Go in both windows to ```Root -> PlatformInfo -> Generic```. Transfer ```MLB, ROM, SystemProductName, SystemSerialNumber and SystemUUID``` to the new config. 
6. Save the new config (File -> Save) and close both windows.
7. Now delete your existing EFI folder from the EFI partition and copy the new one to it. (Make sure that the Directorys ```Boot and OC``` are in ```EFI```).

</details>

&nbsp;

## Post-install (optional)

<details>  
<summary><strong>💾 Install OpenCore to Hard Drive (recommended)</strong></summary>
</br>

1. Press `ALT + SPACE` and open terminal. Type `sudo diskutil mountDisk disk0s1` (where disk0s1 corresponds to the EFI partition of the main disk - disks can be listed using the command `sudo diskutil list`, but most likely it will be `disk0s1`)
2. Open Finder and copy the EFI folder of your USB device to the main disk's EFI partition.
3. Unplug the USB device and reboot your laptop. Now you can boot macOS without your USB device.

</details>

<details>  
<summary><strong>✏️ Create a offline install media (Optional)</strong></summary>
</br>

In case of reinstalling macOS, having a full system installer on your USB drive. You also don't need internet connection for the installation. To create a offline install media, you need the following stuff: 

- macOS Installer from the App Store.
- A 16 GB+ pendrive (Keep in mind, during the preperation we will format the disk to create the install media).

Download the installer from App Store. Press `ALT + SPACE` and open Disk utility. Select your USB device and click erase. Name it `MyUSB` and choose **Mac OS Extended** with **GUID Partition Map**. After erasing the USB device, close Disk utility.

Now press `ALT + SPACE` and open terminal. Type the following command:

Big Sur:
```sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

Monterey:
```sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB --downloadassets```

After creating the install media, copy your EFI folder to the EFI partition of your USB device.


</details>

&nbsp;

## Status

<details>  
<summary><strong>✅ What's working</strong></summary>
</br>
 
- [X] Intel WiFi & Bluetooth (thanks to [itlwn](https://github.com/OpenIntelWireless/itlwm))
- [X] Brightness / Volume Control
- [X] Battery Information
- [X] Audio (Audio Jack & Speaker)
- [X] USB Ports
- [X] Graphics Acceleration
- [X] Trackpoint / Touchpad
- [X] Power management
- [X] Sleep (can be wonky but mostly works)
- [X] FaceTime / iMessage (iServices)
- [X] HDMI
- [X] Automatic OS updates
- [X] Handoff / Universal Clipboard
- [X] Sidecar (Cable) / AirPlay to Mac
- [X] SIP / FireVault 2 (would not recommend encryption due to potential future troubleshooting, though)
- [X] USB-C

</details>

<details>  
<summary><strong>⚠️ What's not working</strong></summary>
</br>

- [ ] Safari DRM ```Use Chromium powered Browser or Firefox to watch Amazon Prime Video, Netflix, Disney+ and others```
- [ ] AirDrop & Continuity
- [ ] Fingerprint Reader (Disabled with NoTouchID kext + can't test because my model doesn't have one)
- [ ] Thunderbolt 3
- [ ] Sidecar Wireless
- [ ] Apple Watch Unlock

</details>

<details>  
<summary><strong>🔄 Not tested</strong></summary>
</br>

- [ ] WWAN
- [ ] Dualbooting Windows / Linux (with OpenCore)
- [ ] Built-in camera

</details>

&nbsp;

## ⭐️ Feedback
If you find any bugs or just have some questions, feel free to provide your feedback using the Discussions tab.

&nbsp;

## 📜 License

This repo is licensed under the [MIT License](https://github.com/valnoxy/t480-oc/blob/main/LICENSE).

OpenCore is licensed under the [BSD 3-Clause License](https://github.com/acidanthera/OpenCorePkg/blob/master/LICENSE.txt).

<hr>
<h6 align="center">2023 - 20xx 0x8008. 
<br>
<h6 align="center">Mostly based on work done by <a href="https://github.com/valnoxy/t480-oc">valnoxy</a>.  Cheers mate, I owe you one if our paths cross at some point.
<br>
