# Inter-VLAN Routing Lab

This lab focuses on configuring inter-VLAN routing using Cisco packet tracer.

## Objectives
- Configure VLANs
- Implement inter-VLAN routing
- Troubleshoot inter-VLAN routing issues

## Requirements
- Cisco Packet Tracer
- Switches and Routers

## VLAN Configuration

1. **Create VLANs**:
   ```plaintext
   Switch(config)# vlan 10
   Switch(config-vlan)# name Accounting
   
   Switch(config)# vlan 20
   Switch(config-vlan)# name Sales
   ```



## Inter-VLAN Routing Configuration

To enable inter-VLAN routing, configure the router as follows:

### Router Configuration
```plaintext
Router(config)# interface g0/1
Router(config-if)# no shutdown
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# exit
Router(config)# interface g0/2
Router(config-if)# ip address 192.168.20.1 255.255.255.0
Router(config-if)# exit
```

### Example of CLI Commands
```bash
# Show VLANs
show vlan brief

# Show IP interface brief
show ip interface brief
```

## ASCII Diagrams

### Topology Diagram
```plaintext
        +----------------+        +----------------+
        |   Switch 0     |        |   Switch 1     |
        |                |        |                |
        |   VLAN 10      |        |   VLAN 20      |
        +----------------+        +----------------+
              |                           |
              |                           |
         +----------------+       +----------------+
         |     Router     |       |   End Device    |
         +----------------+       +----------------+
```

## Troubleshooting Common Issues

### Troubleshooting Steps
```plaintext
1. Verify VLANs are created and configured properly.
2. Check interfaces on the router are up and configured with correct IP addresses.
3. Ensure devices in the same VLAN can ping each other.
4. Test inter-VLAN connectivity using ping.
```