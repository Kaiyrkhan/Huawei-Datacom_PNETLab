# Configure DHCP Server on Linux

  1) Configure DHCP Server on Debian 12/13;
  2) Configure DHCP Server on Ubuntu 24.04.4 LTS;
  3) Configure DHCP Server on Rocky 9.7;
  4) Configure DHCP Server on openEuler 24.03 LTS SP3;
  5) Configure DHCP Server on Oracle 7.9.

### Network Topology
![Topology Enterprise Network Design](images/Topology_EnterpriseNetworkDesign_Huawei_v1.png)  

## Configure DHCP Server on Debian

#### Scenario
  1) Configure Device Hostname;
  2) Configure Network interface;
  3) install DHCP Package;
  4) Configure DHCP Server;
  5) Configure DHCP Relay Agent;
  6) Verify IP Address Assignment.

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
$ sudo apt update
$ sudo apt install -y isc-dhcp-server
```
```shell
$ sudo dpkg -l isc-dhcp-server
$ sudo dpkg -s isc-dhcp-server
```

#### Step 4 - Configure DHCP Server

```shell
$ ip address
```
```shell
$ sudo nano /etc/default/isc-dhcp-server
INTERFACESv4="ens3"
INTERFACESv6=""

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ sudo nano /etc/dhcp/dhcpd.conf

option domain-name "lab.local";
option domain-name-servers 8.8.8.8;
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;
log-facility local7;

subnet 10.10.10.0 netmask 255.255.255.0 {
}

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ sudo systemctl start isc-dhcp-server
$ sudo systemctl status isc-dhcp-server

$ sudo systemctl enable isc-dhcp-server
$ sudo systemctl is-enabled isc-dhcp-server
```

```shell
$ sudo nano /etc/dhcp/dhcpd.conf

subnet 172.16.111.0 netmask 255.255.255.0 {
  range 172.16.111.11 172.16.111.250;
  option routers 172.16.111.254;
    host h1 {
      hardware ethernet 50:91:6a:00:0d:00;
      fixed-address 172.16.111.8;
    }
}

subnet 172.16.112.0 netmask 255.255.255.0 {
  range 172.16.112.11 172.16.112.250;
  option routers 172.16.112.254;
}

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ cat /etc/dhcp/dhcpd.conf | sed '/^#/d;/^$/d'
```

```shell
$ sudo systemctl restart isc-dhcp-server
```

```shell
$ sudo dhcpd -t
```

```shell
# Lease Database
$ less /var/lib/dhcp/dhcpd.leases
```

```shell
# Systemd-journald

$ sudo journalctl -u isc-dhcp-server
$ sudo journalctl -f -u isc-dhcp-server
```

#### Step 5 - Configure DHCP Relay Agent

```shell
# D1 and D2 Switch

...

```

Link: [Configure DHCP Relay Agent (for PNETLab Environment)](1_EnterpriseNetwork.md#step-8--configure-dhcp-relay-agent-for-pnetlab-environment)  

#### Step 6 - Configure DHCP Logging with rsyslog

> **DHCP Logging for ISC DHCP Server:**  
> systemd-journald  
> rsyslog  
> syslog-ng  

```shell
$ sudo apt install rsyslog
```

```shell
$ sudo nano /etc/rsyslog.conf
local7.*  /var/log/dhcpd.log

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ sudo touch /var/log/dhcpd.log
$ ls -l /var/log/
```

```shell
$ sudo systemctl restart rsyslog
$ sudo systemctl restart isc-dhcp-server
```

```shell
$ tail /var/log/dhcpd.log
$ tail -f /var/log/dhcpd.log
```

#### Step 7 - Verify IP Address Assignment

```shell
# H1 (Debain)
student@h1:~$ ip address
student@h1:~$ sudo dhclient -v ens3

student@h1:~$ ip address
student@h1:~$ ip route
student@h1:~$ cat /etc/resolv.conf

student@h1:~$ sudo systemctl restart networking
```

```shell
# H2 (Ubuntu)
student@h2:~$ ip address
student@h2:~$ ip route
student@h2:~$ resolvectl status

student@h2:~$ sudo netplan apply
```

```shell
# H3 (Rocky)
student@h3:~$ ip address
student@h3:~$ ip route
student@h3:~$ cat /etc/resolv.conf

student@h3:~$ sudo systemctl restart NetworkManager
```

```shell
# H4 (openEuler)
student@h4:~$ ip address
student@h4:~$ ip route
student@h4:~$ cat /etc/resolv.conf

student@h4:~$ sudo systemctl restart NetworkManager
```

```shell
# DHCP Server
student@dhcp:~$ cat /var/lib/dhcp/dhcpd.leases
```

## Configure DHCP Server on Ubuntu 24.04.4 LTS

#### Step 1 - Configure Device Hostname

```shell
$ sudo hostnamectl set-hostname dhcp

немесе

$ sudo nano /etc/hostname
dhcp
CTRL+O, ENTER, CTRL+X
CTRL+L

$ bash
```
