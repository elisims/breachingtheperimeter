# Ubuntu Server Pentetration Test

## Initial Setup
- The VM image was imported into VMWare, and the proper Operating System (Ubuntu version16)was selected. The ubuntu server that was imported needed to be run in “host-only” mode- therefore, using VMWare’s virtual network manager, a host-only connection was createdunder the name “VMnet1”. The network was configured to use the IP address 192.168.141.0.

![1](https://github.com/elisims/breachingtheperimeter/raw/main/images/1.jpg)

- Then under the DHCP settings, I set the range of IP’s to be between 192.168.141.128 and192.168.141.132 in order to make finding these addresses much easier during the fingerprintingprocess.

![2](https://github.com/elisims/breachingtheperimeter/raw/main/images/2.jpg)
