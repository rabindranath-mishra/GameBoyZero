# Raspberry Pi Zero 2W GBA Handheld Build Guide

Ultimate **Game Boy Advanced style Handheld** using Pi Zero 2W + Retropie + Waveshare 3.5" Display + GPIO buttons + PWM audio. Perfect Pokémon/Game Boy Advance and upto PS1 emulation. Can copy ROMS on to the GBA over SMB (Shared Folder) and play.

![GBA Handheld](Doc/img/GBA1.jpeg)

## BOM (Bill of Materials) - ₹4575 = ~₹5000 Total

| Component | Model | Qty | Price (₹) | Source |
| --------- | ----- | --- | --------- | ------ |
| **Pi** | Raspberry Pi Zero 2 W with Header | 1 | 2054 | Robu.in |
| **Display** | Waveshare 3.5" TFT (A) | 1 | 790 | QuartzComponents.com |
| **Speaker** | SmartElex Digital Speaker Module 8Ω 0.5W | 1 | 267 | Robu.in |
| **microSD** | 16GB Class 10 | 1 | 300 | Amazon.in |
| **Buttons** | Tactile 12x12mm Switch | 12 | 60 | Robu.in |
| **Battery** | WLY103443 3.7V 1500mAh 1S LiPo Battery | 1 | 416 | Robu.in |
| **Batter Charger** | Seeed Studio Lipo Rider Plus (Charger/Booster) - 5V/2.4A USB Type C | 1 | 658 | Robu.in |
| **Slide Switch** | SS8-7-Small Slide Switch 2 way | 1 | 30 | Robu.in |
| **Case** | 3D printed GBA shell | 1 | - | Print @ home |
| **MSC** | Wires, Soldering, etc. | - | - | Robu.in |

## 1. 3D Printed Case

I have provided the CAD and STL files (/3D Model) , one can 3d print at home or order a printed case from any 3d print vendor.

## 2. Installing Retropie on the Raspberry Pi Zero 2 W

Download the latest retropie image from retropie.or.uk and install on the PI. Follow  the [instructions](https://retropie.org.uk/docs/First-Installation/).

The PI Zero 2 W has a mini HDMI port and needs a converter to connect to any monitor or TV. I did not have one so followed the below steps to install in a headless manner.

1. After the image is written on the the SD card connect he SD car to a computer.
2. Create the below two files in the root of the SD card.

     * Create a empty text file named 'ssh' without file extension. This will enable the SSH after the first boot.
     * Create a text file named 'wpa_supplicant.conf' and update your home WIFI details. This way PI will connect to the home network automatically.

     ``` text
     country=in
     update_config=1
     ctrl_interface=/var/run/wpa_supplicant
     
     network={
         scan_ssid=1
         ssid="your ssid"
         psk="your key"
         key_mgmt=WPA-PSK
    }
    ```

3. Unmount the SD car connect it to the PI and boot.

Post installation the PI should connect to your home network with SSH enabled. Check the PI's IP from your home router and connect the the PI using any SSH client (I use PuTTY). The default user name and password are 'pi' an 'raspberry'.

Rest of the document use SSH for installation and configuration.

## 3. Display (Waveshare 3.5" TFT)

The 3.5" TFT display directly fits on the the raspberry pi GPIO header no soldering needed. Installing the driver is bit cumbersome as the [instructions](https://www.waveshare.com/wiki/3.5inch_RPi_LCD_(A)) are not well written. Pay attention to the raspberry pi model used and the version of operating system installed. Mine was a 'pi zero 2 w' and the OS was 'debian buster' hence I followed the below steps.

```  bash
git clone https://github.com/waveshare/LCD-show.git
cd LCD-show/
sudo ./LCD35-show

sudo reboot
```

* Configure PI (Debian Buster) to use SPI instaed of HDMI as display with strps below.

    Edit '/boot/config.txt' (sudo nano /boot/config.txt)

    ``` text
    dtparam=spi=on
    dtoverlay=waveshare35a
    display_rotate=0  # Adjust 90/180/270 as needed

    #dtoverlay=vc4-kms-v3d
    #hdmi_force_hotplug=1
    #max_usb_current=1
    #hdmi_group=2
    #hdmi_mode=87
    #hdmi_cvt=480 320 60 6 0 0 0
    #hdmi_drive=2
    ```

* Comment out dtoverlay=vc4-kms-v3d (if present)
* Comment out all configurations related to HDMI

After reboot the display should work.


