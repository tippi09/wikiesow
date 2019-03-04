# OpenVPN over IPv6

As I only have a DS-Lite connection, I have to connect to my Raspberry Pi OpenVPN server via IPv6. The steps to accomplish this are described in this guide.

!!! note
    This guide only provides IPv6 connectivity outside the tunnel. For more information on IPv6 support in OpenVPN take a look [here](https://community.openvpn.net/openvpn/wiki/IPv6).

## Create an openvpn user

To avoid conflicts with other users on your Raspberry, we first configure a new user called «openvpn», which is used for the OpenVPN server only.
```console
# adduser «openvpn»
# passwd «openvpn»
# sudo usermod -a -G sudo «openvpn»
```

After creating the user, we can now login as «openvpn»:
```console
# su - «openvpn»
```

## Download and run PiVPN script

To install and configure the OpenVPN server, we can use the [PiVPN](http://www.pivpn.io) install script:
```console
# curl -L install.pivpn.io > pivpn.sh
# chmod +x pivpn.sh
```
If you are certain that the `pivpn.sh` script is not harmful to your system, run it:
```console
# ./pivpn.sh
```
Now follow these steps as described by [rngcntr](https://wiki.rngcntr.de/Raspberry_Pi/OpenVPN.html#run-the-pivpn-install-script) to complete the configuration process:

1. Confirm the current network settings and make sure the shown IP address (eg. 192.168.2.110) is always assigned to this device by the router.
2. Choose the user «openvpn».
3. When asked, select UDP as a protocol.
4. Confirm the default OpenVPN port 1194.
5. Select ECDSA encryption strength. (Recommended: 256-bit)
6. If you use DNS, enter the domain name.
7. Reboot after the installation has finished

## Enable IPv6 on server

Now we can add IPv6 support to the server by editing the config file of the OpenVPN server:
```console
# sudo vim /etc/openvpn/server.conf
```
Search for the line `proto udp` and change it to `proto udp6` so that it looks like this:
```
proto udp6
```

## Add clients

To add a client just run `pivpn add` and set a name and password for the client configuration. Per default, the configuration is stored in `/home/«openvpn»/ovpns`. To enable client side IPv6 connections, edit the configuration file:
```console
# sudo vim /home/«openvpn»/ovpns/«configName».ovpn
```
In this file once more search for the line `proto udp` and change it to `proto udp6`. Now copy the `.ovpn` file to your client via `scp` and add the configuration to your OpenVPN client.
