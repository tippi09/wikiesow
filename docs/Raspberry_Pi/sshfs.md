# SSHFS hard drive sharing

In this guide I will explain how I have implemented easy hard drive sharing on my Raspberry Pi by using `sshfs`.

## Raspberry Pi configuration

First of all we have to install some packages. Depending on the file system you are using, it must not be necessary to install the exFAT packages.

```console
# sudo apt-get install exfat-fuse exfat-utils
# sudo apt-get install hdparm
```

The `hdparm` package will be used later to set the hard drive to standby and reduce energy consumption if it is not mounted. After installing the required packages we first have to find the hard drive we want to mount:

```console
# sudo lsblk -o name,mountpoint,label,size,uuid
```

This command should produce information about all connected drives:
```
NAME        MOUNTPOINT LABEL   SIZE UUID
sda                            3,7T
├─sda1                 EFI     200M 67E3-17ED
└─sda2                 HiFiPi  3,7T 5B8D-1D20
mmcblk0                       29,7G
├─mmcblk0p1 /boot      boot   43,9M AF14-47C0
└─mmcblk0p2 /          rootfs 29,7G b57a7299-0677-4dc6-b4b0-c10f107f0f6d
```

In this case `mmcblk0` is the SD card of the Pi and `sda` the hard drive we want to mount. With `hdparm` we can find out the current power status by running `sudo hdparm -C /dev/sda`. It is also possible to use the disks label with `sudo hdparm -C /dev/disk/by-label/HiFiPi`. If you get the output `drive state is:  unknown`, your disk controller does not support `hdparm` and you should try to use `hd-idle` instead.

To mount the hard drive, I have written a small script which also sets the drive to standby if it is unmounted. This script will also be used to start the drive again from accessing computer:

```bash
if [ "$1" = "mount" ]
then
    mkdir /home/«piuser»/«pimountdirectory»
    sudo mount -L HiFiPi /home/«piuser»/«pimountdirectory»
elif [ "$1" = "unmount" ]
then
    sudo umount /home/«piuser»/«pimountdirectory»
    sudo rm -rf /home/«piuser»/«pimountdirectory»
    sudo hdparm -y /dev/disk/by-label/HiFiPi
fi
```

In the following this script is named «pimount.sh» and it is located in the home directory. To make it executable first run `chmod +x «pimount.sh»`. Then run `sudo visudo` and add the following line:
```
«username» ALL= NOPASSWD: /home/«username»/pimount.sh
```

Now it should be possible to mount the hard drive with `./«pimount.sh» mount` and unmount by running `./«pimount.sh» unmount`.

## Computer configuration

First we have to install `sshfs`. This can easily done with Homebrew by running:
```console
# brew install sshfs
```
Now I have added another «pimount.sh» script in my home directory:
```bash
#!/bin/bash
if [ "$1" = "mount" ]; then
    mkdir /Users/«user»/«mountdirectory»
    ssh -t «piuser»@«pihostname» "sudo ./pimount.sh $1" # Mount drive on Raspberry
    sshfs «piuser»@«pihostname»:/home/«piuser»/«pimountdirectory» /Users/«user»/«mountdirectory»
elif [ "$1" = "unmount" ]; then
    ssh -t «piuser»@«pihostname» "sudo ./pimount.sh $1" # Run unmount script on Raspberry
    umount -f /Users/«user»/«mountdirectory»
    rm -rf /Users/«user»/«mountdirectory»
fi
```
Once again make it executable with `chmod +x «pimount.sh»`. Now I have added two aliases in my `.zshrc`, to mount the hard drive with a single command in the terminal:
```
alias pimount='/Users/«user»/pimount.sh mount'
alias piunmount='/Users/«user»/pimount.sh unmount'
```
