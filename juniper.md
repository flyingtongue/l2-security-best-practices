## Junos ELS Layer 2 Security

By default all Juniper EX switches are assigned to a default VLAN (VLAN 1) with storm control turned on. RSTP and LLDP are also enabled on every port. RSTP might not match

These are basic features that prevent loops in your network with the exception of LLDP which is a layer2 discovery protocol.

https://www.juniper.net/documentation/en_US/junos/information-products/pathway-pages/ex4300/port-security.html

### DHCP

#### DHCP Snooping

##### Overview

DHCP Snooping allows the switch to monitor DHCP messages. These messages are examined and stored in a database. This database is called the DHCP snooping database. This information allows the switch to make decisions to mitigate attacks directed to the layer2 broadcast domain for which the switch belongs to.

Snooping Process
:

- The network device sends a DHCPDISCOVER packet to request an IP address.
- The switch forwards the packet to the DHCP server.
- The server sends a DHCPOFFER packet to offer an address. If the DHCPOFFER packet is from a trusted interface, the switch forwards the packet to the network device.
- The network device sends a DHCPREQUEST packet to accept the IP address. The switch adds an IP-MAC placeholder binding to the DHCP snooping table. The entry is considered a placeholder until a DHCPACK packet is received from the server. Until then, the IP address could still be assigned to some other host.
- The server sends a DHCPACK packet to assign the IP address or a DHCPNAK packet to deny the address request.
- The switch updates the DHCP database according to the type of packet received:
- If the switch receives a DHCPACK packet, it updates lease information for the IP-MAC address bindings in its database.
- If the switch receives a DHCPNACK packet, it deletes the placeholder.

##### Mitigation:

Enable DHCP snooping
: `set vlans <VLAN> forwarding-options dhcp-security group DHCP overrides no-option82`

##### Verification Commands:

DHCP Snooping Database
: `show dhcp-security binding`

##### Possible issues:

- Switch will drop DCHP messages from a DHCP server that is not trusted.
- Will drop DHCP messages where MAC address of device and embedded client hardware MAC do not match.
- DHCP snooping will drop messages that release a lease or decline an offer
if the message is received on a switchport other than the original port where
the DHCP conversation was held.

##### Source:

https://www.juniper.net/documentation/en_US/junos/topics/concept/port-security-dhcp-snooping-els.html

#### Trusted DHCP server

##### Overview

Protects against rogue DHCP servers sending leases.

##### Enable a trusted DHCP server

By default all access ports are untrusted and trunk ports are trusted. You can override by issuing the following command:

Trusted Interface
: `set vlans <VLAN> forwarding-options dhcp-security group DHCP interface <interface>`

##### Verification Commands:

DHCP Snooping Database
: `show dhcp-security binding`

Show configuration
: `show configuration <vlan> forwarding-options dhcp-security`

##### Possible Issues:

As stated above the default behavior is to trust trunk ports so if your DHCP server is plugged directly into this switch you will need to trust that port.

##### Source

https://www.juniper.net/documentation/en_US/junos/topics/concept/port-security-trusted-dhcp-servers.html


### DHCP starvation attacks

#### Overview

In an DHCP starvation attack, the attacker floods the LAN with DHCP requests from spoofed MAC addresses. This will cause the DHCP server to run out of leasable IP addresses.

#### Enable MAC limiting to protect the switch

Limit MAC
: set vlans <VLAN> switch-options interface <INT> interface-mac-limit 1 packet-action drop

#### Verification Commands

VLAN MAC Address Learning
: `show ethernet-switching statistics vlan-name`

#### Possible Issues

You would like to plug more than one device into the the port. The above command shows "interface-mac-limit" this could be increased.

Do not use this command on a trunk port or on a port where you have more than one mac address.

#### Source

https://www.juniper.net/documentation/en_US/junos/topics/example/port-security-protect-from-dhcp-starvation-attack.html
https://www.juniper.net/documentation/en_US/junos/topics/task/configuration/port-security-mac-limiting-cli-els.html

#### DHCP Snooping Database Alteration Attacks

##### Overview

In this attack, an attacker spoofs and IP address of another client on the same switch. The result is that the DHCP table is updated and valid ARP requests are blocked.


##### Mitigation: Disable DHCP option 82

set vlans <VLAN NAME> forwarding-options dhcp-security group DHCP overrides no-option82

http://nextheader.net/2016/11/02/workaround-to-enable-dhcp-snooping-in-juniper-els-cli/

#### ARP Spoofing Attack

##### Overview

As you know ARP stands for Address Resolution Protocol. This means that a device on the same layer2 broadcast domain can send a query to the address FF:FF:FF:FF:FF:FF to discovery the link layer address with a given IP address. Example: You turn your computer on and request an IP address. What the actual packets look like (sniffing from non requesting device):

09:33:41.610973 ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 10.251.32.1 tell 10.251.32.53, length 46

This means that an attacker could associate it's own MAC address iwtht eh IP address of a device. This means the attacker would start to receive all traffic destined for the original device.

##### Mitigation: Enable IP source guard and dynamic arp inspection

`set vlans <VLAN> forwarding-options dhcp-security ip-source-guard`

`set vlans <VLAN> forwarding-options dhcp-security arp-inspection`

##### Verification Commands:

  - DHCP Snooping Database: `show dhcp-security binding`
  - Dynamic ARP Inspection: `show dhcp-security arp inspection statistics`

##### Definitions:

- IP Source Guard: Examines each packet from a host attached to an untrusted access interface on the switch. IP, MAC, VLAN, and interface associated with the host is checked against entried stroed in the DHCP snooping database.

- Dynamic ARP Inspection (DAI): Examines arp requests and responses on the LAN and validates ARP packets. ARP packets are validated by comparing with the DHCP snooping database.

##### Possible issues:

Devices with manually assigned addresses will stop wroking due to ip source guard which compares IP, MAC, VLAN, and interface with the information stored in the DHCP snooping database.


#### Rogue DHCP Server Attacks

##### Overview

A rogue DHCP server is a server that is inadvertently plugged into a switch and starts handing out IP addresses to devices that request. Most of the time this is done by an employee that plugs in a Linksys Wireless Router because wireless access was poor in their office/classroom.

Juniper mitigates this by allowing only trusted DHCP servers to hand out IP addresses. By default trunk ports are trusted and access ports are not trusted.

##### Enabling a trusted DHCP server

By default all access ports are untrusted and trunk ports are trusted. You can override by issuing the following command:

Trusted Interface
: `set vlans <VLAN> forwarding-options dhcp-security group DHCP interface <interface>`

##### Verification Commands:

DHCP Snooping Database
: `show dhcp-security binding`

Show configuration
: `show configuration <vlan> forwarding-options dhcp-security`

##### Possible Issues:

As stated above the default behavior is to trust trunk ports so if your DHCP server is plugged directly into this switch you will need to trust that port.

##### Source

https://www.juniper.net/documentation/en_US/junos/topics/task/configuration/port-security-trusted-dhcp-server-cli-els.html

### IP Source Guard

#### Overview

IP Source Guard examines each packet from a host attached to an untrusted access interface on the switch. IP, MAC, VLAN, and interface associated with the host is checked against entried stroed in the DHCP snooping database.


#### enable ip source guard

`set vlans <VLAN> forwarding-options dhcp-security ip-source-guard`

#### verification commands

- DHCP Snooping Database: `show dhcp-security binding`

#### Possible issues:

Devices with manually assigned addresses will stop wroking due to ip source guard which compares IP, MAC, VLAN, and interface with the information stored in the DHCP snooping database.

