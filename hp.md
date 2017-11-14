## HP ProCurve layer 2 security best practices
By default all HP ProCurve switcehs are assigned to a default VLAN (VLAN 1) with no STP, storm control, or other mechanism for loops. LLDP is enabled globally. For now we will leave LLDP enabled, it could pose a security risk but I currently believe the benefits outweight the consequences (show lldp neighbors).

### DHCP

#### DHCP Snooping

#### Overview

DHCP Snooping allows the switch to monitor DHCP messages. These messages are examined and stored in a database. This database is called the DHCP snooping database. This information allows the switch to make decisions to mitigate attacks directed to the layer2 broadcast domain for which the switch belongs to.

Snooping Process :

- The network device sends a DHCPDISCOVER packet to request an IP address.
- The switch forwards the packet to the DHCP server.
- The server sends a DHCPOFFER packet to offer an address. If the DHCPOFFER packet is from a trusted interface, the switch forwards the packet to the network device.
- The network device sends a DHCPREQUEST packet to accept the IP address. The switch adds an IP-MAC placeholder binding to the DHCP snooping table. The entry is considered a placeholder until a DHCPACK packet is received from the server. Until then, the IP address could still be assigned to some other host.
- The server sends a DHCPACK packet to assign the IP address or a DHCPNAK packet to deny the address request.
- The switch updates the DHCP database according to the type of packet received:
- If the switch receives a DHCPACK packet, it updates lease information for the IP-MAC address bindings in its - database.
- If the switch receives a DHCPNACK packet, it deletes the placeholder.

#### Enable DHCP Snooping

Enable DHCP snooping globally
: `dhcp-snooping`

Set authorized server
: `dhcp-snooping authorized-server <ip address>`

Set vlans for which you would like to snoop
: `dhcp-snooping vlan <your vlans separated by comma or dash>`

Set trusted port
: `dhcp-snooping

Set option 82
: `dhcp-snooping option 82`

#### Verification

Verify
: `show dhcp-snooping`



#### Definitions

 - Option 82: when enabled option 82 will allow the switch to add information about the clients network location into the packet header of that request.

#### Source

https://support.hpe.com/hpsc/doc/public/display?docId=emr_na-c02595798


### Port Security

#### Overview 

Port Security is a mitigation technique for the following attacks:

- DHCP starvation attack
- ARP spoofing attack

#### Enable

This will limit the port to one mac address, do not use on ports with more than one source mac (AP, Switch, etc). 

Port Security
: `port-security <ports 1-48> learn-mode limited-continuous acti
on send-disable address-limit 1`


#### Verification

Verify port-security
: `show port-security`

Verify port-security per port
: `show port-security <port>`

#### Source

ftp://ftp.hp.com/pub/networking/.../59906024e01-security-ch08-Port_Security.pdf


### Dynamic ARP Inspection 

#### Overview

Dynamic ARP Inspection protects switches against arp spoofing. ARP packets are validated against the DHCP snooping database to verify authenticity.

#### Configuration

Enable DAI
: `arp-protect vlan <vlan id>`


Trust Ports
: `arp-protect trust <uplink ports>`


#### Verification

Verify arp protect is enabled
: `show arp-protect`

Verify arp protect statistics per vlan
: `show arp-protect statistics <vlan id>`

### IP Source Guard

#### Overview

IP Source Guard checks the source ip address of packets that are received on any given port and verifies they are not spoofed by correlating the IP address and mac address with the dhcp snooping database. 

Unforutantely at the time of this writing I am unable to find a current configuration method for guarding against IP address spoofing. It has been alleged that enabling arp-protect, port security, and dhcp snooping should be enough to get you by.

#### Verification

Verify port-security
: `show port-security`

Verify port-security per port
: `show port-security <port>`

Verify arp protect is enabled
: `show arp-protect`

Verify arp protect statistics per vlan
: `show arp-protect statistics <vlan id>`

Enable DHCP snooping globally
: `dhcp-snooping`

Set authorized server
: `dhcp-snooping authorized-server <ip address>`

Set vlans for which you would like to snoop
: `dhcp-snooping vlan <your vlans separated by comma or dash>`

Set trusted port
: `dhcp-snooping trust <uplink ports or dhcp server>`

Set option 82
: `dhcp-snooping option 82`


#### Source

https://j-netstuff.blogspot.com/2015/04/dynamic-arp-protection-in-hp-procurve.html

