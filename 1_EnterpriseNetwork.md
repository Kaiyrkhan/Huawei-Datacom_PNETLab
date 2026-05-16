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
   LACP Link Aggregation. Eth-Trunk  
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

## Configure Access Layer Switches (A1, A2)

Configure Device Hostname
```shell
system-view
sysname A1

commit
```

Create VLANs
```shell
vlan batch 111 112 50

commit

display vlan
```

Configure Access Port
```shell
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

Configure Trunk Port and Allowed VLANs
```shell
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

## Configure Aggregation Layer Switches (D1, D2)

Configure Device Hostname
```shell
system-view
sysname D1
commit
```

Create VLANs
```shell
vlan batch 111 112 50

commit

display vlan
```

Configure Trunk Port and Allowed VLANs
```shell
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

Configure LACP Link Aggregation
```shell
interface Eth-Trunk 1                                          // Create Eth-Trunk
 port link-type trunk                                          // Trunk Port
 port trunk allow-pass vlan 111 112 50                         // Allowed VLANs         
 mode lacp-static                                              // Link Aggregation Mode
 quit

commit
```

Add a Port to the Eth-Trunk
```shell
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
display eth-trunk 1
```

Configure MSTP
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

### C1 – Core Layer Switch-ті конфигурациялау
```shell
<Huawei> undo terminal monitor
<Huawei> system-view
[Huawei] sysname C1
```

### SRV-D1 – Distribution Layer Switch-ті конфигурациялау
```shell
<Huawei> undo terminal monitor
<Huawei> system-view
[Huawei] sysname SRV-D1
```

### EdgeR1 – Edge Router-ді конфигурациялау
```shell
<Huawei> undo terminal monitor
<Huawei> system-view
[Huawei] sysname EdgeR1
```
