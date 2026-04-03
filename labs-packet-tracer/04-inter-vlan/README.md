# Inter-VLAN Routing Lab

This lab focuses on configuring inter-VLAN routing using Cisco Packet Tracer.

## Objectives

- Configure VLANs
- Implement inter-VLAN routing
- Troubleshoot inter-VLAN routing issues

## Requirements

- Cisco Packet Tracer
- Switches and Routers

## VLAN Configuration

### Step 1: Create VLANs on Switch

```console
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name Accounting
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name Sales
Switch(config-vlan)# exit

Switch(config)# interface range FastEthernet 0/1-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit

Switch(config)# interface range FastEthernet 0/11-20
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit

Switch(config)# end
```

## Inter-VLAN Routing Configuration

### Step 2: Configure Router for Inter-VLAN Routing

```console
Router# configure terminal
Router(config)# interface GigabitEthernet 0/1
Router(config-if)# no shutdown
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# exit

Router(config)# interface GigabitEthernet 0/2
Router(config-if)# no shutdown
Router(config-if)# ip address 192.168.20.1 255.255.255.0
Router(config-if)# exit

Router(config)# end
```

### Step 3: Verify Configuration

```console
Router# show ip interface brief
Interface       IP-Address      OK? Method Status                Protocol
Gi0/1          192.168.10.1    YES manual up                    up
Gi0/2          192.168.20.1    YES manual up                    up

Router# show running-config | include interface
interface GigabitEthernet 0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet 0/2
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

## Network Topology Diagram

```
                    +------------------+
                    |    Router        |
                    | Gi0/1  | Gi0/2   |
                    +--------+---------+
                       /            \
                      /              \
           192.168.10.0/24    192.168.20.0/24
                  /                      \
            +------+                   +------+
            |      |                   |      |
        +---+---+  |               +---+---+  |
        |       |  |               |       |  |
      Switch0 ------ VLAN 10     Switch1 ------ VLAN 20
        |       |  |               |       |  |
        | PC-1  |  |               | PC-3  |  |
        | PC-2  |  |               | PC-4  |  |
        +-------+  |               +-------+  |
```

## Troubleshooting Common Issues

### Step 4: Verify Inter-VLAN Connectivity

```console
PC-1> ping 192.168.20.3
Reply from 192.168.20.3: bytes=32 time=10ms TTL=63
Reply from 192.168.20.3: bytes=32 time=10ms TTL=63
```

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Devices in different VLANs cannot communicate | Router interfaces not configured | Configure router interfaces with correct IPs |
| VLAN not assigned to interfaces | Missing switchport commands | Use `switchport access vlan X` on each port |
| Router interfaces are down | Interfaces not enabled | Use `no shutdown` on router interfaces |
| Incorrect subnet masks | Configuration error | Verify IP addresses and subnet masks match |

### Verification Commands

```console
Switch# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- ------
1    default                          active    Fa0/24
10   Accounting                       active    Fa0/1-10
20   Sales                            active    Fa0/11-20

Switch# show interfaces switchport | include Name|Access Mode
Name: Fa0/1
Switchport Mode: static access

Router# show ip route
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP

Gateway of last resort is not set

C    192.168.10.0/24 is directly connected, GigabitEthernet 0/1
C    192.168.20.0/24 is directly connected, GigabitEthernet 0/2
```