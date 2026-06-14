# Configure NTP Server on Linux

  1) Chrony
  2) LinuxPTP (Precision Time Protocol)

> Linux distribution: Debian 12/13, Ubuntu 24.04.4 LTS, Rocky 9.7, openEuler 24.03 LTS SP3, Oracle 7.9  

### Network Topology
![Topology Enterprise Network Design](images/Topology_EnterpriseNetworkDesign_Huawei_v1.png)  

## Configure NTP Server using Chrony on Ubuntu

#### Scenario
  1) install Chrony Package
  2) Configure Chrony as an NTP Server
  3) Configure NTP Authentication
  4) Configure NTP Firewall Rule
  5) Verify NTP Server Operation

#### Step 1 - install Chrony Package

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
