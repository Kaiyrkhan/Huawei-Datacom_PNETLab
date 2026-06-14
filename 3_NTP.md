# Configure NTP Server on Linux

  1) Chrony
  2) LinuxPTP (Precision Time Protocol)

> Linux distribution: Debian 12/13, Ubuntu 24.04.4 LTS, Rocky 9.7, openEuler 24.03 LTS SP3, Oracle 7.9  

### Network Topology
![Topology Enterprise Network Design](images/Topology_EnterpriseNetworkDesign_Huawei_v1.png)  

## Configure NTP Server using Chrony on Ubuntu

```shell
student@ubuntu:~$ lsb_release -a
Ubuntu 24.04.4 LTS
student@ubuntu:~$ uname -rs
Linux 6.8.0-101-generic x86_64 GNU/Linux
```

#### Scenario
  1) Configure Device Hostname
  2) Configure Network interface
  3) install Chrony Package
  4) Configure Chrony as an NTP Server
  5) Configure NTP Authentication
  6) Configure NTP Firewall Rule
  7) Check Chrony Synchronization Status
  8) Verify NTP Client Synchronization

#### Step 1 - Configure Device Hostname

```shell
$ sudo hostnamectl set-hostname ntp

$ sudo nano /etc/hosts
127.0.1.1  ntp

CTRL+O, ENTER, CTRL+X
CTRL+L

$ bash
```

#### Step 2 - Configure Network interface

```shell
student@ubuntu:~$ ip address
```

```shell
student@ubuntu:~$ sudo nano /etc/netplan/50-cloud-init.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: false
      addresses:
        - 10.10.10.123/24
      gateway4: 10.10.10.1
      nameservers: 
        search: [lab.local]
        addresses: [8.8.8.8]

CTRL+O, ENTER, CTRL+X
CTRL+L
```
> **ЕСКЕРТУ:** *YAML файлында бос орындар (indentation) өте маңызды. Әр қатарда 2 бос орын қолдануды ұмытпаңыз! (Tab пернесін қолданбаған дұрыс)*  

```shell
student@ubuntu:~$ sudo netplan apply
```

```shell
student@ubuntu:~$ ip address
```

```shell
student@ubuntu:~$ networkctl status
```

#### Step 3 - install Chrony Package

> Package атауы: **chrony**  
> Daemon/Service атауы: **chrony** немесе **chronyd**  

> **chronyd** – the actual daemon to sync and serve via the Network Time Protocol  
> **chronyc** – command-line interface for the chrony daemon  

```shell
$ sudo apt update 
$ sudo apt install -y chrony
```

```shell
$ sudo systemctl status chronyd
```

Уақыт белдеуін (Time Zone) өзгерту
```shell
$ sudo timedatectl set-timezone Asia/Almaty
$ timedatectl status
```
> Time Zones in Kazakhstan https://www.timeanddate.com/time/zone/kazakhstan  
