# Huawei-Based Enterprise Network Design and Implementation / Huawei құрылғылары негізінде корпоративті желіні жобалау және конфигурациялау

### Network Topology
![Topology Enterprise Network Design](images/Topology_PNETLab_EnterpriseNetworkDesign_HQ1_v1_Huawei.png)
[Download Link for PNETLab Topology File](Topology/Topology_EnterpriseNetworkDesign_HQ1_v1_Huawei.zip)

| Device Name    | Device Type                            |
| ---------------| ---------------------------------------|
| ISP            | ISP (Internet Service Provider) Router |
| EdgeR1         | Edge Router                            |
| С1             | Core Layer Switche                     |
| D1, D2, SRV-D1 | Aggregation Layer Switche              |
| A1, A2         | Access Layer Switche                   |
| H1, H2, H3, H4 | End Device                             |

## Scenario
1) Configure VLAN (Create VLANs and Access Port, Trunk Port)  
   Link Aggregation. Eth-Trunk  
   MSTP (Multiple Spanning Tree Protocol)  
2) VRRP (Virtual Router Redundancy Protocol)
3) Single-Area OSPF
4) DHCP
5) NAT (Easy IP)
6) Remote Access (SSH, Telnet)
7) NTP
8) DNS
9) HTTP
10) FTP
11) TFTP
12) ...

## Step 1 – Configure Device Hostname

```shell
# Access Layer Switches (A1, A2)

system-view
sysname A1

commit
```

```shell
# Aggregation Layer Switches (D1, D2, SRV-D1)

system-view
sysname D1

commit
```

```shell
# Core Layer Switches (C1, C2)

system-view
sysname C1

commit
```

```shell
# Edge Router (EdgeR1)

Username: super
Password: super
Warning: The password is already expired.
The password needs to be changed. Change now? [Y/N]: Y
Please enter old password: super
Please enter new password: Huawei@123
Please confirm new password: Huawei@123
The password has been changed successfully.
<Huawei>

<Huawei> system-view
[Huawei] sysname EdgeR1
[EdgeR1]
```

## Step 2 – Configure VLAN (Create VLANs and Access/Trunk Ports)

**A1 and A2 Switch**

```shell
# Create VLANs
vlan batch 111 112 50

commit

display vlan
```

```shell
# Configure Access Port

interface g1/0/3
 port link-type access
 port default vlan 111
 quit

interface g1/0/4
 port link-type access
 port default vlan 112
 quit

commit

display port vlan
```

```shell
# Configure Trunk Port and Allowed VLANs

interface g1/0/1
 port link-type trunk
 port trunk allow-pass vlan 111 112 50
 quit

interface g1/0/2
 port link-type trunk
 port trunk allow-pass vlan 111 112 50
 quit

commit

display port vlan
```

**D1 and D2 Switch**

```shell
# Create VLANs
vlan batch 111 112 50

commit

display vlan
```

```shell
# Configure Trunk Port and Allowed VLANs

interface g1/0/2
 port link-type trunk
 port trunk allow-pass vlan 111 112 50
 quit

interface g1/0/3
 port link-type trunk
 port trunk allow-pass vlan 111 112 50
 quit

commit

display port vlan
```

**SRV-D1 Switch**

```shell
# Create VLANs
vlan 10

commit

display vlan
```

```shell
# Configure Access Port

interface g1/0/2
 port link-type access
 port default vlan 10
 quit

interface g1/0/3
 port link-type access
 port default vlan 10
 quit

interface g1/0/4
 port link-type access
 port default vlan 10
 quit

...

commit

display port vlan
```

## Step 3 – Configure Link Aggregation. Eth-Trunk

**D1 and D2 Switch**

```shell
interface Eth-Trunk 1                                          // Create Eth-Trunk interface
 port link-type trunk                                          // Trunk Port
 port trunk allow-pass vlan 111 112 50                         // Allowed VLANs         
 mode lacp-static                                              // Link Aggregation Mode
 quit

commit
```

```shell
# Add a Port to the Eth-Trunk

interface g1/0/11
 eth-trunk 1
 quit
interface g1/0/12
 eth-trunk 1
 quit

commit

display int brief
```

```shell
# Verify Configuration

display eth-trunk 1
```

## Step 4 – Configure MSTP (Multiple Spanning Tree Protocol)

**D1 and D2 Switch**

```shell
display stp

stp enable
stp mode mstp

commit

display stp
```

```shell
stp region-configuration
 region-name HQ1
 revision-level 1
 instance 1 vlan 111 50
 instance 2 vlan 112
 check region-configuration
 quit

commit

display cu | begin stp
```

```shell
# D1 Switch
stp instance 1 root primary
stp instance 2 root secondary
```

```shell
# D2 Switch
stp instance 1 root secondary
stp instance 2 root primary
```

**A1 and A2 Switch**

```shell
display stp

stp enable
stp mode mstp

commit

display stp
```

```shell
stp region-configuration
 region-name HQ1
 revision-level 1
 instance 1 vlan 111 50
 instance 2 vlan 112
 check region-configuration
 quit

commit

display cu | begin stp
```

```shell
# Verify Configuration

display stp vlan 111
display stp vlan 112
display stp vlan 50

немесе

display stp instance 1 brief
display stp instance 2 brief
```

## Step 5 – Configure VRRP (Virtual Router Redundancy Protocol)

**D1 Switch**

```shell
interface vlanif 111
 ip address 172.16.111.1 24
 vrrp vrid 1 virtual-ip 172.16.111.254
 vrrp vrid 1 priority 105
 quit

interface vlanif 112
 ip address 172.16.112.1 24
 vrrp vrid 2 virtual-ip 172.16.112.254
 quit

interface vlanif 50
 ip address 10.1.50.1 24
 vrrp vrid 50 virtual-ip 10.1.50.254
 vrrp vrid 50 priority 105
 quit

commit

display ip int brief
display vrrp
```

**D2 Switch**

```shell
interface vlanif 111
 ip address 172.16.111.2 24
 vrrp vrid 1 virtual-ip 172.16.111.254
 quit

interface vlanif 112
 ip address 172.16.112.2 24
 vrrp vrid 2 virtual-ip 172.16.112.254
 vrrp vrid 2 priority 105
 quit

interface vlanif 50
 ip address 10.1.50.2 24
 vrrp vrid 50 virtual-ip 10.1.50.254
 quit

commit

display ip int brief
display vrrp
```

## Step 6 – Configure Single-Area OSPF

**D1 Switch**

```shell
interface g1/0/1
 ip address 10.1.1.106 30
 quit
```

```shell
interface Loopback 50            // Create Loopback interface
 ip address 50.7.7.7 32
 quit
```

```shell
display ip int brief
```

```shell
ospf 1 router-id 50.7.7.7
 area 0
 network 10.1.1.104 0.0.0.3
 network 172.16.111.0 0.0.0.255
 network 172.16.112.0 0.0.0.255
 network 10.1.50.0 0.0.0.255
 network 50.7.7.7 0.0.0.0
 quit
 quit

display cu | begin ospf
```

**D2 Switch**

```shell
interface g1/0/1
 ip address 10.1.1.110 30
 quit
```

```shell
interface Loopback 50            // Create Loopback interface
 ip address 50.8.8.8 32
 quit
```

```shell
display ip int brief
```

```shell
ospf 1 router-id 50.8.8.8
 area 0
 network 10.1.1.108 0.0.0.3
 network 172.16.111.0 0.0.0.255
 network 172.16.112.0 0.0.0.255
 network 10.1.50.0 0.0.0.255
 network 50.8.8.8 0.0.0.0
 quit
 quit

display ospf peer brief
```

**C1 Switch**

```shell
interface g1/0/1
 ip address 10.1.1.102 30
 quit
interface g1/0/2
 ip address 10.1.1.105 30
 quit
interface g1/0/3
 ip address 10.1.1.109 30
 quit
interface g1/0/4
 ip address 10.1.1.113 30
 quit
interface Loopback 50
 ip address 50.3.3.3 32
 quit

display ip int brief
```

```shell
display ip int brief

ospf 1 router-id 50.3.3.3
 area 0
 network 10.1.1.100 0.0.0.3
 network 10.1.1.104 0.0.0.3
 network 10.1.1.108 0.0.0.3
 network 10.1.1.112 0.0.0.3
 network 50.3.3.3 0.0.0.0
 quit
 quit

display ospf peer brief
```

**SRV-D1**

```shell
interface g1/0/1
 ip address 10.1.1.114 30
 quit

# Create Loopback interface
interface Loopback 50
 ip address 50.5.5.5 32
 quit

# Create VLANIF interface
interface VLANIF 50
 ip address 10.10.10.1 24
 quit

display ip int brief
```

```shell
display ip int brief

ospf 1 router-id 50.5.5.5
 area 0
 network 10.1.1.112 0.0.0.3
 network 10.10.10.0 0.0.0.255
 network 50.5.5.5 0.0.0.0
 quit
 quit

display ospf peer brief
```

**EdgeR1 Router**

```shell
Username: super
Password: Huawei@123
```

```shell
interface g0/0/2
 ip address dhcp-alloc
 quit
interface g0/0/3
 ip address 10.1.1.101 30
 quit
interface Loopback 50
 ip address 50.1.1.1 32
 quit

display ip int brief
```

```shell
ping 10.0.137.1
 Reply from 10.0.137.1: bytes=56 Sequence=2 ttl=64 time=1 ms
```

```shell
display ip int brief

ospf 1 router-id 50.1.1.1
 area 0
 network 10.1.1.100 0.0.0.3
 network 50.1.1.1 0.0.0.0
 quit
 quit

display ospf peer brief
```

## Step 7 – Configure DHCP Server on Linux

```shell
dhcp enable

ip pool VLAN111
 network 172.16.111.0 mask 24
 gateway-list 172.16.111.254
 dns-list 8.8.8.8
 excluded-ip-address 172.16.111.1 172.16.111.10
 excluded-ip-address 172.16.111.251 172.16.111.253
 lease day 5
 quit

ip pool VLAN112
 network 172.16.112.0 mask 24
 gateway-list 172.16.112.254
 dns-list 172.16.128.53
 excluded-ip-address 172.16.112.1 172.16.112.10
 excluded-ip-address 172.16.112.251 172.16.112.253
 lease day 5
 quit

interface g0/0/0
 dhcp select global
 quit
```

Verify Configuration
```shell
display ip pool
display ip pool name VLAN111
display dhcp server statistics
```

## Step 8 – Configure DHCP Relay Agent

**D1 and D2 Switch**

```shell
dhcp enable

interface vlanif 111
 dhcp select relay
 dhcp relay server-ip 172.16.128.67
 quit

interface vlanif 112
 dhcp select relay
 dhcp relay server-ip 172.16.128.67
 quit
```

```shell
PC1> ipconfig
PC2> ipconfig
PC3> ipconfig
PC4> ipconfig /renew
```

## Step 9 – Configure NAT (Easy IP)

**EdgeR1**

```shell
ping 192.168.137.1
 Reply from 192.168.137.1: bytes=56 Sequence=2 ttl=128 time=10 ms

ping 8.8.8.8
 Request time out
```

Default Static Route
```shell
ip route-static 0.0.0.0 0.0.0.0 192.168.137.1

display cu | include static
```

```shell
ping 8.8.8.8
 Reply from 8.8.8.8: bytes=56 Sequence=1 ttl=107 time=90 ms
```

Advertise the Default Route
```shell
ospf 1
 default-route-advertise
 quit
```

```shell
<EdgeR1> display ip routing-table

Destination/Mask   Proto   Pre   Cost   Flags   NextHop         Interface
       0.0.0.0/0   Static  60    0      RD      192.168.137.1   GigabitEthernet 0/0/1
```

```shell
<C1> display ip routing-table
Destination/Mask   Proto   Pre   Cost   Flags   NextHop         Interface
       0.0.0.0/0   O_ASE   150   1      D       10.1.1.101      GigabitEthernet 0/0/0

<C1> display ospf routing
 Routing for ASEs
 Destination        Cost      Type       Tag         NextHop         AdvRouter
 0.0.0.0/0          1         Type2      1           10.1.1.101      50.1.1.1
```

```shell
acl 2000
 rule permit source 172.16.111.0 0.0.0.255
 rule permit source 172.16.112.0 0.0.0.255
 quit

int g0/0/1
 nat outbound 2000
 quit
```

Verify Configuration
```shell
display cu section acl
display nat outbound
```

```shell
PC1> ping 8.8.8.8
PC2> ping 8.8.8.8
PC3> ping 8.8.8.8
PC4> ping 8.8.8.8
 From 8.8.8.8: bytes=32 seq=3 ttl=105 time=156 ms
```
```shell
PC1> ping google.com
PC3> ping google.com
 From 142.250.181.238: bytes=32 seq=1 ttl=106 time=156 ms
```

NAT Table
```shell
[EdgeR1] display nat session all verbose
```
