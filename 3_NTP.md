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
  ethernets:
    ens3:
      addresses:
      - "10.10.10.123/24"
      nameservers:
        addresses:
        - 8.8.8.8
        search:
        - LAB.LOCAL
      routes:
      - to: "default"
        via: "10.10.10.1"

CTRL+O, ENTER, CTRL+X
CTRL+L
```

немесе

```shell
student@ubuntu:~$ sudo nano /etc/netplan/50-cloud-init.yaml
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: false
      addresses: [10.10.10.123/24]
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
> Daemon/Service атауы: **chronyd**  

> **chronyd** – the actual daemon to sync and serve via the Network Time Protocol  
> **chronyc** – command-line interface for the chrony daemon  

```shell
Debian/Ubuntu
$ sudo apt update 
$ sudo apt install -y chrony

RHEL/Rocky/Oracle
$ sudo dnf install -y chrony
```

```shell
$ sudo systemctl status chronyd
```

```shell
$ ss -tulpn
```

#### Step 4 - Configure Chrony as an NTP Server

```shell
# Уақыт белдеуін (Time Zone) өзгерту
$ sudo timedatectl set-timezone Asia/Almaty

$ timedatectl status
```
> Time Zones in Kazakhstan https://www.timeanddate.com/time/zone/kazakhstan  

```shell
RHEL/Rocky/Oracle
$ sudo vi /etc/chrony.conf
#pool 2.rocky.pool.ntp.org iburst        // артық DNS атауларды "#" comment-ге алып, төменгі қатарға Қазақстанға ең жақын NTP сервердің DNS атауын енгіземіз!

Debian/Ubuntu
$ sudo nano /etc/chrony/chrony.conf
#pool 2.debian.pool.ntp.org iburst       // артық DNS атауларды "#" comment-ге алып, төменгі қатарға Қазақстанға ең жақын NTP сервердің DNS атауын енгіземіз!

# Kazakhstan NTP pool
server ntp.nic.kz iburst
pool 2.kz.pool.ntp.org iburst
pool 1.kz.pool.ntp.org iburst

# Global NTP pool
pool time.google.com iburst
pool time.cloudflare.com iburst

# Listen on all interfaces
bindcmdaddress 0.0.0.0
bindcmdaddress ::

# Allow NTP client access from Local Network (жергілікті желіге рұқсат ету)
allow 172.16.111.0/24
allow 172.16.112.0/24
allow 50.1.1.1/32
allow 50.3.3.3/32
allow 50.5.5.5/32
allow 50.7.7.7/32
allow 50.8.8.8/32
allow 10.1.50.101/32

# NTP authentication
keyfile /etc/chrony/chrony.keys

# Log files location
logdir /var/log/chrony
log measurements statistics tracking

# Hardware clock synchronization
rtcsync

# Time adjustment settings (уақыт дәлдігін реттеу)
makestep 1 3
```

```shell
# Configure NTP Authentication

$ sudo nano /etc/chrony/chrony.keys
# <key_id> <algorithm> <secret_key>
1 MD5 Hello@123

CTRL+O, ENTER, CTRL+X
CTRL+L
```
> ЕСКЕРТУ: мұндағы, "MD5" **бас әріппен** жазылуы міндетті!  

#### Step 5 - Configure NTP Firewall Rule

```shell
# nftables конфигурациясы

$ sudo nft add rule inet filter input udp dport 123 ip saddr 172.16.111.0/24 accept
$ sudo nft add rule inet filter input udp dport 123 ip saddr 172.16.112.0/24 accept

$ sudo nft list ruleset | sudo tee /etc/nftables.conf
$ sudo systemctl restart nftables

$ sudo nft list ruleset
```

```shell
# iptables конфигурациясы

$ sudo iptables -A INPUT -p udp --dport 123 -s 172.16.111.0/24 -j ACCEPT
$ sudo iptables -A INPUT -p udp --dport 123 -s 172.16.112.0/24 -j ACCEPT
$ sudo iptables -A INPUT -p udp --dport 123 -s 127.0.0.1 -j ACCEPT

$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
немесе
$ sudo iptables-save > /etc/iptables/rules.v4
$ sudo systemctl restart iptables

$ sudo iptables -vnL
```

```shell
# Firewalld конфигурациясы (RHEL/Rocky)

$ sudo systemctl status firewalld

$ sudo firewall-cmd --permanent --add-port=123/udp
$ sudo firewall-cmd --permanent --add-service=ntp
немесе
$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="172.16.111.0/24" port protocol="udp" port="123" accept'
$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="172.16.112.0/24" port protocol="udp" port="123" accept'

$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" port protocol="udp" port="123" drop'

$ sudo firewall-cmd --reload

$ sudo firewall-cmd --list-rich-rules
$ sudo firewall-cmd --list-all --zone=public
```

```shell
# UFW конфигурациясы (Debian/Ubuntu)

$ sudo ufw status
$ sudo ufw enable

$ sudo ufw allow 123/udp
немесе
$ sudo ufw allow from 172.16.111.0/24 to any port 123 proto udp
$ sudo ufw allow from 172.16.112.0/24 to any port 123 proto udp

$ sudo ufw deny proto udp from any to any port 123

$ sudo ufw reload
$ sudo ufw status verbose
```

```shell
# Daemon-ды қайта жүктеу

$ sudo systemctl restart chronyd
немесе
$ sudo systemctl reload chronyd

$ sudo systemctl status chronyd
```

```shell
$ ss -tulpn
Netid  State    Local Address:Port    Peer Address:Port
udp    -        0.0.0.0:123           0.0.0.0:*
```

#### Step 6 - Check Chrony Synchronization Status

```shell
$ sudo chronyc tracking
$ sudo chronyc sources -v
$ sudo chronyc activity
$ sudo chronyc clients
```

```shell
$ sudo apt install ntpdate
$ sudo ntpdate -q 80.241.0.72
```

## Configure NTP Client using Chrony on Linux

```shell
Debian/Ubuntu
$ sudo apt install chrony

RHEL/Rocky/Oracle
$ sudo dnf install chrony

$ sudo systemctl status chronyd
```

```shell
RHEL/Rocky/Oracle
$ sudo vi /etc/chrony.conf
server 10.10.10.123 iburst

Debian/Ubuntu
$ sudo nano /etc/chrony/chrony.conf
server 10.10.10.123 iburst

$ sudo systemctl restart chronyd
```

```shell
# Verify NTP Client Synchronization
$ sudo chronyc tracking
$ sudo chronyc sources -v

$ sudo chronyc makestep
$ sudo chronyc -a makestep

Synchronizing Time (Уақытты синхрондау)
$ sudo apt install ntpdate
$ sudo ntpdate -q 10.10.10.123
$ sudo ntpdate -u 10.10.10.123                // firewall кедергі жасаған жағдайда қолдану

# System Clock
$ date
# Hardware Clock (RTC)
$ sudo hwclock -r

# Verify System Time Synchronization
$ timedatectl status
немесе
timedatectl show --property=NTPSynchronized
```

## Configure NTP Client using Chrony on Huawei VRP

```shell
system-view

clock timezone KZ add 5

ntp-service enable
ntp-service unicast-server 10.10.10.123
```

```shell
display ntp-service status
display ntp-service sessions
```

```shell
acl number 2123
 rule permit ip source 10.10.10.123 0.0.0.0
ntp-service acl 2123
```
