# Configure Kea DHCP Server on Linux

**Linux Distribution:**
  1) Debian 13.x
  2) Ubuntu 24.04.4 LTS
  3) Rocky 9.x
  4) openEuler 24.03 LTS SP3
  5) Oracle 7.9

### Network Topology
![Topology Enterprise Network Design](images/Topology_EnterpriseNetworkDesign_Huawei_v1.png)  

## Configure Kea DHCP Server on Debian

```shell
# System Information

student@debian:~$ lsb_release -a
student@debian:~$ cat /etc/debian_version
student@debian:~$ uname -rs
```

#### Scenario
  1) Configure Device Hostname
  2) Configure Network interface
  3) install DHCP Package
  4) Configure DHCP Server
  5) Configure DHCP Relay Agent
  6) Configure DHCP Logging
  7) Verify IP Address Assignment

#### Step 1 - Configure Device Hostname

```shell
$ sudo hostnamectl set-hostname dhcp

$ sudo nano /etc/hosts
127.0.1.1  dhcp

CTRL+O, ENTER, CTRL+X
CTRL+L

$ bash
```

#### Step 2 - Configure Network interface

```shell
$ ip address
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet static
    address 10.10.10.67
    netmask 255.255.255.0
    gateway 10.10.10.1
    dns-nameservers 8.8.8.8

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ sudo systemctl restart networking
```

```shell
$ ip address
$ ip route
$ cat /etc/resolv.conf
```

#### Step 3 - install DHCP Package

```shell
$ ping 8.8.8.8
$ ping google.com
```

```shell
# install DHCP Package
$ sudo apt update
$ sudo apt install -y isc-dhcp-server
```
```shell
# Verify DHCP Package Installation
$ sudo dpkg -l isc-dhcp-server

# View DHCP Package Details
$ sudo dpkg -s isc-dhcp-server
```

```shell
# Verify DHCP Listening Port

$ ss -tulpn
Netid  State    Local Address:Port    Peer Address:Port
udp    -        0.0.0.0:67            0.0.0.0:*
```
