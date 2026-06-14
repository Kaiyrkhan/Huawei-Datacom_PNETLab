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
Distributor ID: Debian
Description:    Debian GNU/Linux 13 (trixie)
Release:        13
Codename:       trixie

student@debian:~$ cat /etc/debian_version
13.5

student@debian:~$ uname -rs
Linux 6.12.73+deb13-amd64 x86_64 GNU/Linux
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
$ apt search kea | grep dhcp4
kea-dhcp4-server/stable,now 2.6.3-1 amd64

$ systemctl list-units | grep kea
kea-dhcp4-server.service
```

> Package атауы: kea-dhcp4-server  
> Daemon/Service атауы: kea-dhcp4-server  

```shell
# install DHCP Package

$ sudo apt update
$ sudo apt install -y kea-dhcp4-server
```
```shell
# Verify DHCP Package Installation
$ sudo dpkg -l kea-dhcp4-server

# View DHCP Package Details
$ sudo dpkg -s kea-dhcp4-server
```

```shell
# Status DHCP Service
$ sudo systemctl status kea-dhcp4-server
active (running)

$ sudo systemctl is-enabled kea-dhcp4-server
enabled
```

```shell
# Verify DHCP Listening Port

$ ss -tulpn
Netid  State    Local Address:Port    Peer Address:Port
udp    -        0.0.0.0:67            0.0.0.0:*
```

#### Step 4 - Configure Kea DHCP Server

```shell
$ sudo grep -vE '^\s*(//|#|$)' /etc/kea/kea-dhcp4.conf
немесе
$ sudo cat /etc/kea/kea-dhcp4.conf | sed '/^\s*\/\//d;/^\s*$/d'
$ sudo cat /etc/kea/kea-dhcp4.conf | sed '/^\s*#/d;/^\s*\/\//d;/^\s*$/d'
немесе
$ sudo sed '/^\s*\/\//d;/^\s*$/d' /etc/kea/kea-dhcp4.conf
$ sudo sed '/^\s*#/d;/^\s*\/\//d;/^\s*$/d' /etc/kea/kea-dhcp4.conf
```

```shell
$ sudo cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.backup
$ sudo truncate -s 0 /etc/kea/kea-dhcp4.conf

$ sudo nano /etc/kea/kea-dhcp4.conf
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [ "ens3" ]
    },

    "valid-lifetime": 600,
    "renew-timer": 300,
    "rebind-timer": 500,

    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },

    "option-data": [
      {
        "name": "domain-name",
        "data": "lab.local"
      },
      {
        "name": "domain-name-servers",
        "data": "8.8.8.8"
      }
    ],

    "subnet4": [
      {
        "id": 1,
        "subnet": "10.10.10.0/24"
      },
      {
        "id": 2,
        "subnet": "172.16.111.0/24",
        "pools": [
          {
            "pool": "172.16.111.11 - 172.16.111.250"
          }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "172.16.111.254"
          }
        ],
        "reservations": [
          {
            "hostname": "h1",
            "hw-address": "50:91:6a:00:0d:00",
            "ip-address": "172.16.111.8"
          }
        ]
      }
    ]
  }
}

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
# Check Configuration Syntax
$ sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
```

```shell
$ sudo systemctl restart kea-dhcp4-server
$ sudo systemctl status kea-dhcp4-server
```

```shell
$ sudo journalctl -u kea-dhcp4-server
$ sudo journalctl -f -u kea-dhcp4-server
```

```shell
```

```shell
```
