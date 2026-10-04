# Lab-3---DHCP-and-DHCP-Relay
Cisco Packet Tracer lab demonstrating VLAN segmentation, centralized DHCP, DHCP relay, and troubleshooting across multiple VLANs

## Objective

The objective of this lab was to configure centralized DHCP services across multiple VLANs and use DHCP relay to allow clients on remote VLANs to obtain IP configuration from a DHCP server located on a different network.

The lab also demonstrates VLAN segmentation, 802.1Q trunking, router-on-a-stick, DHCP troubleshooting, APIPA addressing, and the DHCP DORA process.

---

## Network Topology

The network consists of three VLANs connected through a Cisco switch and router. A centralized DHCP server is located in VLAN 20.

| VLAN | Department | Network | Default Gateway |
|---|---|---|---|
| 10 | HR | 192.168.10.0/24 | 192.168.10.1 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 |
| 30 | OPS | 192.168.30.0/24 | 192.168.30.1 |

**DHCP Server:** 192.168.20.10
![Network Topology](images/Lab%203%20DHCP%20Topology.png)
---

## DHCP Configuration

The DHCP server was configured with separate address pools for each VLAN.

| Pool | Starting Address | Gateway | Subnet Mask |
|---|---|---|---|
| HR | 192.168.10.100 | 192.168.10.1 | 255.255.255.0 |
| IT | 192.168.20.100 | 192.168.20.1 | 255.255.255.0 |
| OPS | 192.168.30.100 | 192.168.30.1 | 255.255.255.0 |

Because the DHCP server resides in VLAN 20, DHCP relay was configured on the router interfaces serving VLANs 10 and 30.
![DHCP Pool](images/Lab%203%20DHCP%20Pools.png)
```text
interface g0/0.10
 ip helper-address 192.168.20.10

interface g0/0.30
 ip helper-address 192.168.20.10
```

---

## Testing and Troubleshooting

Before DHCP relay was configured, a client in VLAN 10 was unable to reach the DHCP server and received a `169.254.x.x` APIPA address.

After configuring the correct DHCP pool and `ip helper-address`, the client successfully received:

```text
IP Address:      192.168.10.100
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
DNS Server:      8.8.8.8
```
![DHCP PC0 / VLAN 10 DHCP Success](images/Lab%203%20DHCP%20PC0%20Success.png)
The OPS client in VLAN 30 also successfully received:

```text
IP Address:      192.168.30.100
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.30.1
DNS Server:      8.8.8.8
```
![DHCP PC2 / VLAN 30 Success](images/Lab%203%20DHCP%20PC2%20Success.png)
Packet Tracer Simulation Mode was used to observe the DHCP DORA process:

**Discover → Offer → Request → Acknowledge**

---

## Key Concepts Demonstrated

- VLAN segmentation
- 802.1Q trunking
- Router-on-a-stick
- DHCP scopes/pools
- DHCP relay using `ip helper-address`
- DHCP DORA process
- APIPA troubleshooting
- Default gateway configuration
- Inter-VLAN communication
- Cisco IOS verification and troubleshooting

---

## Key Commands

```text
show vlan brief
show interfaces trunk
show ip interface brief
show running-config

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.20.10

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip helper-address 192.168.20.10
```

---

## Lessons Learned

This lab demonstrated that DHCP broadcasts do not normally cross router boundaries. When the DHCP server is located on a different subnet, a DHCP relay must be configured on the client-facing Layer 3 interface and pointed toward the DHCP server.

The lab also reinforced a practical troubleshooting approach:

**Client → VLAN → DHCP Pool → Gateway → DHCP Relay → DHCP Server**

A `169.254.x.x` address is an important troubleshooting indicator that a client was unable to obtain an IPv4 address from DHCP.
