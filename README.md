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

3. Unmount the SD card form your computer and insert it to the PI and boot.

Post installation the PI should connect to your home network with SSH enabled. Check the PI's IP from your home router and connect to the PI using any SSH client (I use PuTTY). The default user name and password are 'pi' an 'raspberry'.

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

* Comment out 'dtoverlay=vc4-kms-v3d' (if present)
* Comment out all configurations related to HDMI

After reboot the display should work.

## 4. Updating and Configuring Retropie

Update the Retropie, underlying OS and Packages using the below script. It may take two or more attempts. For me the first attempt gave some errors, mostly because the 'Debian Buster' is a old archived Raspbian version, but when I tried 2-3 times it updated fine.

``` bash
sudo ~/RetroPie-Setup/retropie_setup.sh
```

Enable Samba: This enables copying the ROMs and bios from your computer to the GBA over network share.

``` bash
sudo ~/RetroPie-Setup/retropie_setup.sh # Configuration/Tools > samba > Install and enable samba share

sudo reboot
```

## 5. GPIO Pin Configuration

Raspberry Pi GPIOs needs to be used for Display, Keys and PWM audio. There are 12 keys needed, one PWM audio, all in all we need 13 pins in addition to the pins that the display TFT uses. Fom the display [documentation](https://www.waveshare.com/wiki/3.5inch_RPi_LCD_(A)) we find that the display header occupies 1-28 physical pins of the PI but the pins '3,5,7,8,10,12,13,15,16' are not used, hence they can be used. We would use the pin 2 and 6 to supply power to the PI.

We will use the below pins for the purpose mentioned.

**GBA GPIO Button Wiring Table:**

| GBA Button    | Physical Pin | BCM GPIO |
| ------------- | ------------ | -------- |
| **UP**        | 3            | GPIO 2   |
| **DOWN**      | 5            | GPIO 3   |
| **LEFT**      | 7            | GPIO 4   |
| **RIGHT**     | 8            | GPIO 14  |
| **A**         | 10           | GPIO 15  |
| **B**         | 13           | GPIO 27  |
| **X**         | 15           | GPIO 22  |
| **Y**         | 16           | GPIO 23  |
| **SELECT**    | 29           | GPIO 5   |
| **START**     | 31           | GPIO 6   |
| **L Trigger** | 35           | GPIO 19  |
| **R Trigger** | 40           | GPIO 21  |
| **Key GND**   | 30           | -        |
| **PWM Audio** | 32           | GPIO 12  |
| **VCC**       | 2            | +5V      |
| **GND**       | 6            | -        |

![Pin Diagram](/Doc/img/BOARD-Layout-CMPLETE_zero2w.jpg)

The pin 8 and 10 (GPIO 14 and GPIO 15) are default UART pins hence to use them as INPUT pins we have to disable the UART in config .txt.

We wont need a pull-down resistor for GPIO pins, I have found that the internal pull-down resistors of the PI are good enough.

Edit '/boot/config.txt' (sudo nano /boot/config.txt)

    ``` text
    # Uncomment some or all of these to enable the optional hardware interfaces
    #dtparam=i2c_arm=on
    #dtparam=i2s=on
    #enable_uart=1
    # Uncomment this to enable the lirc-rpi module
    #dtoverlay=lirc-rpi
    ```

Basically disable\comment all other optional hardware interfaces.

**Wiring Diagram (Active LOW)**

``` text
3.3V (Pin 1) ────────────────────────────────|
|                                            |
├─[Internal Pull-up]── GPIO Pin (INPUT) ── Button ── 10kΩ ── GND (Pin 30)
│
Pi GPIO Header │
(Pin 30 = GND6)┘
```

The 10kΩ Pull-Down resistor is optional.