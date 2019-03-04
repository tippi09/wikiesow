# Multi-purpose Raspbian setup
In this guide we will setup Raspbian on a Raspberry Pi so that it can be used for many purposes while keeping energy consumption as low as possible. Therefor we mainly run Raspbian headless and enable graphical output only if necessary (e.g. for media uses with Kodi).

## Download Raspbian image
First of all download the latest **Raspbian Desktop** image from [raspberrypi.org](https://www.raspberrypi.org/downloads/raspbian/) *(Note: Do not use the Raspbian Lite image if you want to get graphical output as well)*.

Extract the zip file and remove it afterwards. Make sure to adjust the date if necessary:

```console
# unzip 2018-11-13-raspbian-stretch.zip
# rm 2018-11-13-raspbian-stretch.zip
```

## Write image to the SD Card

To write the image we have to find our SD card first. On macOS we run `diskutil list` with the SD card still **unplugged**. Then you can insert the card and run `diskutil list` once more. You should see another device listed as a new entry just like this one:
```
/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *32.0 GB   disk2
   1:                 DOS_FAT_32 LABEL                   32.0 GB   disk2s1
```

!!! danger
    Make sure to **replace the disk number** (disk2 to disk#) corresponding to your `diskutil list` output before calling the following statements.

With the identifier of the SD card we can now unmount it before writing the image:
```console
# diskutil unmountDisk /dev/disk2
```

Now use `dd` to write the image:

```console
# sudo dd bs=1m if=2018-11-13-raspbian-stretch.img of=/dev/rdisk2 conv=sync
```

!!! note
    If this command fails you can try using `disk2` instead of `rdisk2`:
    ```console
    # sudo dd bs=1m if=2018-11-13-raspbian-stretch.img of=/dev/disk2 conv=sync
    ```

After the `dd` command has finished writing the data you can eject the card:
```console
# sudo diskutil eject /dev/rdisk2
```
Now insert the SD card into your Raspberry and boot Raspbian for the first time.

## Default user and root password settings
After completing the initial configuration process you should enable SSH (if not already done) and login to your Raspberry via SSH. The default credentials are:
```
    User: pi
Password: raspberry
```
```
# ssh pi@raspberrypi
```
First change the default root password by calling `passwd root` and create a new user for SSH access:
```console
# adduser «user»
# usermod -a -G sudo «user»
```
After testing sudo access with the created user the default user pi can be deleted:
```console
# deluser -remove-home pi
```

## General Raspbian settings
A nice tool to easily adjust settings for Raspbian is the `raspi-config`. After using SSH to login to your Raspberry it can be started by calling:
```console
# sudo raspi-config
```
After running the aforementioned statement, navigate to **Network Options -> Hostname**. Here you can change the default hostname `raspberrypi` to a new name you prefer.

Afterwards go to **Boot Options -> Desktop / CLI** in `raspi-config`. Choose the console option to avoid booting the more expensive desktop mode after a restart.

## Configure boot configuration
Now we can configure some services and device trees to be loaded on boot. All the options in this section are set in the `/boot/config.txt` file:
```console
# sudo vim /boot/config.txt
```
### Disable HDMI output
Do disable complete HDMI output on boot and avoid that the Raspberry turns on a connected TV with a HDMI-CEC event, add the following lines in your `config.txt`:
```
# uncomment to enable hdmi output on boot
hdmi_blanking=2
hdmi_ignore_cec_init=1
```
### Disable WiFi and Bluetooth

!!! danger
    Do not disable WiFi if you need it for SSH. Otherwise you could lock yourself out of your own system.
As I do not need the Bluetooth of my Raspberry and with it being connected to the internet via LAN, I turn off the WiFi and Bluetooth modules to save some energy by adding the following lines to my `config.txt`:
```
# uncomment to enable wifi and bluetooth
dtoverlay=pi3-disable-wifi
dtoverlay=pi3-disable-bt
```
### Load HiFiBerry device tree
As my HiFi system is connected to my Raspberry, I am using a HiFiBerry board for an improved sound experience. To load the correct device tree and disable the onboard, I added the following lines to my `config.txt`:
```
# Disabled onboard audio, loaded HiFiBerry device tree
# dtparam=audio=on
dtoverlay=hifiberry-dac
```

!!! note
    The device tree that you have to load for your HiFiBerry depends on the concrete board you use. I am using a DAC+ DSP. For information of device trees for different models, take a look [here](https://www.hifiberry.com/build/documentation/configuring-linux-3-18-x/).

To check out which sound card is loaded, reboot and afterwards call:
```console
# aplay -l
```
You should see an output that looks something like this:
```
**** Liste der Hardware-Geräte (PLAYBACK) ****
Karte 0: sndrpihifiberry [snd_rpi_hifiberry_dac], Gerät 0: HifiBerry DAC HiFi pcm5102a-hifi-0 []
  Sub-Geräte: 1/1
  Sub-Gerät #0: subdevice #0
```
## Configure Kodi
Because of the fact that HDMI output per default is disabled in this configuration, we have to find a way to bring it on when starting Kodi. For that reason, I wrote a small script called `kodi.sh`, that enables me to start Kodi via SSH:
```bash
if [ "$1" = "start" ] # Start Kodi, turn TV on and change TV output to Pi input
then
    vcgencmd display_power 1
    POWERSTATUS="$(echo "pow 0" | cec-client -s -d 1)"
    case "$POWERSTATUS" in
        *standby* ) echo "on 0" | cec-client -s
                echo "as" | cec-client RPI -s -d 1;;
        *off* ) echo "on 0" | cec-client -s
                echo "as" | cec-client RPI -s -d 1;;
        *on* ) echo echo "as" | cec-client RPI -s -d 1;;
        * ) echo "Error: Unknown TV power status"
    esac
    nohup kodi &
elif [ "$1" = "stopkodi" ] # Stop Kodi and disable Pi HDMI output again
then
    vcgencmd display_power 0
    pkill kodi
elif [ "$1" = "stopkodiandtv" ] # Stop Kodi, disable PI HDMI output and set TV to standby
then
    echo "standby 0" | cec-client -s
    vcgencmd display_power 0
    pkill kodi
fi
```
After creating the script you have to make it executable by running:
```console
# chmod +x /path/to/kodi.sh
```
As the script has to be run with `sudo`, we have to enable your user to do so by editing the `/etc/sudoers.tmp` file:
```console
#  sudo visudo
```
Add the following line:
```
«user» ALL= NOPASSWD: /path/to/kodi.sh
```
Now it should be possible to run the script by running:
```console
#  ssh -t «user»@«hostname» "sudo /path/to/kodi.sh «option»"
```
With the following options available:

* **start** Starts Kodi, turns the TV on if necessary by checking the current power status and changes the TV output to the Pis input.
* **stopkodi** Stops Kodi, disables the Pis HDMI output once again and keeps the TV turned on (in case you want to use it differently).
* **stopkodiandtv** Stops Kodi, disables the Pis HDMI output once again and turns the TV into standby mode.

As a result, just run the following command to start Kodi:
```console
#  ssh -t «user»@«hostname» "sudo /path/to/kodi.sh start"
```
To make starting and stopping Kodi from my Mac easier, I have also written a script for [Bitbar](https://getbitbar.com). My BitBar setup is documented [here](../macOS/bitbar.md) as well. If you have problems with the Kodi output, update Kodi and check your overscan options in `/boot/config.txt`.

## Install Raspotify
If you would like to use your Raspberry Pi as a Spotify connect client, it is recommended to use the [Raspotify](https://dtcooper.github.io/raspotify/) project. To install Raspotify we use the provided install script:
```console
# curl -L https://dtcooper.github.io/raspotify/install.sh > raspotify.sh
# chmod +x raspotify.sh
```
If you are certain that the script is harmless, run it:
```console
# ./raspotify.sh
```
To configure your Spotify client, just edit the `/etc/default/raspotify` file. For example, I changed the clients device name and the bitrate to higher quality by adding the lines:
```
DEVICE_NAME="HiFiPi" # default is raspotify
BITRATE="320"        # default is 160
```
