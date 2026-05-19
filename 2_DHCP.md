# Configure DHCP Server on Linux

### Network Topology
![Topology Enterprise Network Design](images/Topology_PNETLab_EnterpriseNetworkDesign_HQ1_v1_Huawei.png)  

## Configure DHCP Server on Debian

### Scenario
1) DHCP пакетін (package) орнату;
2) DHCP серверді конфигурациялау;
3) Нәтижені тексеру.

### Құрылғының атауын (Device Hostname) өзгерту
```shell
$ sudo hostnamectl set-hostname dhcp

$ sudo nano /etc/hosts
127.0.1.1  dhcp
CTRL+O, ENTER, CTRL+X
CTRL+L

$ bash
```

### Желілік интерфейсті конфигурациялау
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

$ sudo systemctl restart networking

$ ip address
$ ip route
$ cat /etc/resolv.conf
```

### DHCP пакетін (package) орнату
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

### DHCP серверді конфигурациялау
```shell
$ ip address
```
```shell
$ sudo nano /etc/default/isc-dhcp-server
INTERFACESv4="ens3"
INTERFACESv6=""
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
```

```shell
$ sudo systemctl status isc-dhcp-server
$ sudo systemctl start isc-dhcp-server

$ sudo systemctl is-enabled isc-dhcp-server
$ sudo systemctl enable isc-dhcp-server
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
```

```shell
$ cat /etc/dhcp/dhcpd.conf | sed '/^#/d;/^$/d'
```

```shell
$ sudo systemctl restart isc-dhcp-server
$ sudo dhcpd -t
```

```shell
$ cat /var/lib/dhcp/dhcpd.leases
```

### Нәтижені тексеру
```shell
Debain
student@h1:~$ ip address
student@h1:~$ ip route
student@h1:~$ cat /etc/resolv.conf

Ubuntu
student@h2:~$ ip address
student@h2:~$ ip route
student@h2:~$ resolvectl status

Rocky
student@h3:~$ ip address
student@h3:~$ ip route
student@h3:~$ cat /etc/resolv.conf

openEuler
student@h4:~$ ip address
student@h4:~$ ip route
student@h4:~$ cat /etc/resolv.conf
```
