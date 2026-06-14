# Configure DHCP Server on Linux

  1) ISC DHCP Server
  2) Kea DHCP Server

> Linux distribution: Debian 12/13, Ubuntu 24.04.4 LTS, Rocky 9.7, openEuler 24.03 LTS SP3, Oracle 7.9  

### Network Topology
![Topology Enterprise Network Design](images/Topology_EnterpriseNetworkDesign_Huawei_v1.png)  

## Configure ISC DHCP Server on Debian

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
$ ss -tulpn | grep 67
Netid  State    Local Address:Port    Peer Address:Port
udp    -        0.0.0.0:67            0.0.0.0:*

$ ss -tulpn | grep dhcpd
```

#### Step 4 - Configure DHCP Server

```shell
$ ip address
```
```shell
# Configure DHCP Listening Interface

$ sudo nano /etc/default/isc-dhcp-server
INTERFACESv4="ens3"
INTERFACESv6=""

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ sudo nano /etc/dhcp/dhcpd.conf

default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;
log-facility local7;

subnet 10.10.10.0 netmask 255.255.255.0 {
  not authoritative;
}

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
# Check Configuration Syntax
$ sudo dhcpd -t
```

```shell
# Start DHCP Service
$ sudo systemctl start isc-dhcp-server
$ sudo systemctl status isc-dhcp-server

# Enable DHCP Service
$ sudo systemctl enable isc-dhcp-server
$ sudo systemctl is-enabled isc-dhcp-server
```

```shell
$ sudo nano /etc/dhcp/dhcpd.conf

subnet 172.16.111.0 netmask 255.255.255.0 {
  range 172.16.111.11 172.16.111.250;
  option routers 172.16.111.254;
  option subnet-mask 255.255.255.0;
  option broadcast-address 172.16.111.255;
  option domain-name "lab.local";
  option domain-name-servers 8.8.8.8;
    host h1 {
      hardware ethernet 50:91:6a:00:0d:00;
      fixed-address 172.16.111.8;
    }
}

subnet 172.16.112.0 netmask 255.255.255.0 {
  range 172.16.112.11 172.16.112.250;
  option routers 172.16.112.254;
  option subnet-mask 255.255.255.0;
  option broadcast-address 172.16.111.255;
  option domain-name "lab.local";
  option domain-name-servers 8.8.8.8;
}

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
$ cat /etc/dhcp/dhcpd.conf | sed '/^#/d;/^$/d'
```

```shell
# Check Configuration Syntax
$ sudo dhcpd -t
```

```shell
# Restart DHCP Service
$ sudo systemctl restart isc-dhcp-server
```

```shell
# Verify DHCP Lease Database
$ less /var/lib/dhcp/dhcpd.leases
```

```shell
# Monitor DHCP Service Logs
$ sudo journalctl -u isc-dhcp-server
$ sudo journalctl -f -u isc-dhcp-server
```

#### Step 5 - Configure DHCP Relay Agent

```shell
# D1 and D2 Switch

...

```

Link: [Configure DHCP Relay Agent (for PNETLab Environment)](1_EnterpriseNetwork.md#step-8--configure-dhcp-relay-agent-for-pnetlab-environment)  

#### Step 6 - Configure DHCP Logging

DHCP Logging Methods for ISC DHCP Server:  
 1) systemd-journald  
 2) rsyslog  
 3) syslog-ng  

**Step 6.1 - systemd-journald (default DHCP Logging)**

```shell
$ sudo journalctl -u isc-dhcp-server
$ sudo journalctl -f -u isc-dhcp-server
```

**Step 6.2 - Configure DHCP Logging using rsyslog**

```shell
# install rsyslog
$ sudo apt install rsyslog
```

```shell
# Configure DHCP Log Facility
$ sudo nano /etc/dhcp/dhcpd.conf
log-facility local7;

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
# Configure DHCP Logging
$ sudo nano /etc/rsyslog.conf              // $ sudo nano /etc/rsyslog.d/dhcpd.conf
local7.*  /var/log/dhcpd.log

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
# Create the DHCP Log File
$ sudo touch /var/log/dhcpd.log
$ ls -l /var/log/
```

```shell
# Restart Services
$ sudo systemctl restart rsyslog
$ sudo systemctl restart isc-dhcp-server
```

```shell
# Verify DHCP Logging
$ tail /var/log/dhcpd.log
$ tail -f /var/log/dhcpd.log
```

**Configure DHCP Log Rotation (Production ортаға арналған)**

```shell
# Configure DHCP Log Rotation

$ sudo nano /etc/logrotate.d/dhcpd

/var/log/dhcpd.log {
    weekly
    rotate 12
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root adm
}

# Verify Log Rotation Operation

$ sudo logrotate -d /etc/logrotate.d/dhcpd
$ sudo logrotate -f /etc/logrotate.d/dhcpd

# Result:
/var/log/dhcpd.log
/var/log/dhcpd.log.1.gz
```

**Step 6.3 - Configure DHCP Logging using syslog-ng**

```shell
# install syslog-ng
$ sudo apt update
$ sudo apt install syslog-ng
```

```shell
# Configure DHCP Log Facility
$ sudo nano /etc/dhcp/dhcpd.conf
log-facility local7;

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
# Create a syslog-ng Logging Rule
$ sudo nano /etc/syslog-ng/conf.d/dhcpd.conf

filter f_dhcpd {
    facility(local7);
};

destination d_dhcpd {
    file("/var/log/dhcpd.log");
};

log {
    source(s_src);
    filter(f_dhcpd);
    destination(d_dhcpd);
    flags(final);
};

CTRL+O, ENTER, CTRL+X
CTRL+L
```

```shell
# Create the DHCP Log File
$ sudo touch /var/log/dhcpd.log

$ ls -ld /var/log/dhcpd.log

$ sudo chmod 640 /var/log/dhcpd.log
$ sudo chown root:adm /var/log/dhcpd.log
```

```shell
# Check the Configuration Syntax
$ sudo syslog-ng --syntax-only
```

```shell
# Restart Services
$ sudo systemctl restart syslog-ng
$ sudo systemctl restart isc-dhcp-server
```

```shell
# Verify DHCP Logging
$ sudo tail /var/log/dhcpd.log
$ sudo tail -f /var/log/dhcpd.log
```

```shell
$ sudo systemctl status isc-dhcp-server
$ sudo systemctl status syslog-ng
```

#### Step 8 - Verify DHCP Address Assignment

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
