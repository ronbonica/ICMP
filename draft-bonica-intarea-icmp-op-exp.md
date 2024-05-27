---
coding: utf-8
coding: us-ascii

title: 'Internet Control Message Protocol (ICMP): Standards and Operational Experience'
abbrev: op-exp-icmp
docname: draft-bonica-intarea-icmp-op-exp
category: info 

stand_alone: yes
pi: [toc, tocompact, tocindent, sortrefs, symrefs, compact]

ipr: trust200902
area: Internet
wg: INTAREA Group
kw: ICMP operations

author:
  -
    ins: R. Bonica
    name: Ron Bonica
    city: Herndon
    region: Virginia
    org: Juniper Networks
    country: USA
    email: rbonica@juniper.net

normative:
  RFC0768:
  RFC0791:
  RFC0792:
  RFC0793:
  RFC1122:
  RFC1812:
  RFC4443:
  RFC5927:
  RFC7279:
  RFC8200:

informative:
  RFC2151:
  RFC4950:
  RFC5837:
  RFC8335:

  V4MSG:
    title: Internet Control Message Protocol (ICMP) Parameters
    author:
      - org: Internet Assigned Numbers Authority (IANA)
    date: false
    seriesinfo:
      Web: https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml

  V6MSG:
    title: Internet Control Message Protocol version 6 (ICMPv6) Parameters
    author:
      - org: Internet Assigned Numbers Authority (IANA)
    date: false
    seriesinfo:
      Web: https://www.iana.org/assignments/icmpv6-parameters/icmpv6-parameters.xhtml


--- abstract

The Internet Control Message Protocol (ICMP) can be used to exchange control information between hosts. This document summarizes ICMP standards and operational experience. The purpose of this document is to be used as a reference. A document that mentions ICMP can reference this document rather than repeating portions of its content.

--- middle

# Introduction

The Internet Control Message Protocol (ICMP) can be used to exchange control information between hosts. ICMPv4 {{RFC0792}} exchanges control information between IPv4 {{RFC0791}} hosts while ICMPv6 {{RFC4443}} exchanges control information between IPv6 {{RFC8200}} hosts.

This document summarizes ICMP standards and operational experience. The purpose of this document is to be used as a reference. A document that mentions ICMP can reference this document rather than repeating portions of its content.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# ICMP Message Types

{{RFC1122}} and {{RFC4443}} make a distintion between ICMP error messages and ICMP information messages. An ICMP error messages indicates that a downstream node recieved a packet that it could not process. {{RFC0792}} refers to the unprocessed packet as the "original data datagram", while {{RFC4443}} refers to the unprocessed packet as the "invoking packet". In this document, we will adopt {{RFC4443}} terminology and call the unprocessed packet the "invoking packet". 

Having received an invoking packet, the downstream node can send an ICMP error message to its originator. The ICMP error message indicates why the packet could not be processed. It also includes as much of the invoking packet as possible, without violating the ICMP message length limitations mentioned in ({{length}}). 

In ICMPv6, error messages are identified as such by a zero in the high-order bit of their message Type field values.  Thus, ICMPv6 error messages have message types from 0 to 127. ICMPv6 informational messages have message types from 128 to 255.

In ICMPv4, error messages cannot be identified as such by a zero in the high-order bit of their message Type field values. 

The Internet Assigned Numbers Authority (IANA) maintains one registy of ICMPv4 message types {{V4MSG}} and another registry of ICMPv6 message types {{V6MSG}}. Many ICMPv4 message types have been deprecated.


## Message Processing

### Known ICMP Message Types

According to {{RFC1122}} and {{RFC4443}}, when a node receives an ICMP error message, it must examine the message to determine: 

* whether the invoking packet originated on the local node
* which local application or transport-layer protocol originated the invoking packet

If the invoking packet originated on the local node, the ICMP error message must be delivered to the application or transport-layer prototcol that originated it (if possible). If the originating transport-layer protocol was the User Datagram Protocol (UDP) {{RFC0768}}, UDP must, in turn, deliver the ICMP error message to the application that caused the invoking packet to be sent.

When a node receives an ICMP information message, the  message must be delivered to the application that caused it to be sent (if possible).

### Unknown ICMP Message Types

According to {{RFC1122}}, if an ICMPv4 message of unknown type is received, it must be silently discarded.

According to {{RFC4443}}, if an ICMPv6 error message of unknown type is received, it must be passed to the upper-layer process that originated the invoking packet (if possible). However, if an ICMP informatonal message of unknown type is received, it must be silently discarded.

# ICMP Restrictions

According to {{RFC1122}}, an ICMPv4 error message must not be sent as the result of receiving:

* an ICMP error message
* a datagram destined to an IP broadcast or IP multicast address
* a datagram sent as a link-layer broadcast
* a non-initial fragment
* a datagram whose source address does not define a single host (e.g., a zero address, a broadcast address, a multicast address)

According to {{RFC4443}}, an ICMPv4 error message must not be sent as the result of receiving:

* An ICMPv6 error message
* An ICMPv6 redirect message
* A packet destined to an IPv6 multicast address.  
* A packet sent as a link-layer multicast 
* A packet sent as a link-layer broadcast 
* A packet whose source address does not uniquely identify a single node e.g., the IPv6 Unspecified Address, an IPv6 multicast address, or an address known by the ICMP message originator to be an IPv6 anycast address)

The following are exceptions to the IPv6 multicast, link-layer multicast, and link_layer broadcast rules above:

* The Packet Too Big Message can be sent to allow Path MTU discovery to work for IPv6 multicast
* The Parameter Problem Message, reporting an unrecognized IPv6 option that has the Option Type highest-order two bits set to 10 can be sent


# Extensibility:

Some ICMP messages are extensible {{RFC4443}}. However, the extension header MUST not cause the ICMP message to violate the length restrictions mentioned below. IANA maintains a registry of [ICMP Extension Object Classes and Class Sub-types](https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml#icmp-parameters-ext-classes). 

# Acceptable Uses

{{RFC7279}} defines accpetable uses for ICMP. The following are acceptable uses:

- to inform a datagram's originator that a forwarding plane anomaly has been encountered downstream.  The datagram originator must be able to determine whether or not the datagram was discarded by examining the ICMP message.

- to discover and convey dynamic information about a node (other than information usually carried in routing protocols), to discover and convey network-specific parameters, and to discover on-link routers and hosts.

Normally, ICMP SHOULD NOT be used to implement a general-purpose routing or network management protocol.  However, ICMP does have a role to play in conveying dynamic information about a network, which would belong in the second catagory,  above.

# Management Applications

The following management applications rely on ICMP:

- PING {{RFC2151}}
- Traceroute {{RFC2151}}
- Traceroute with extensions for Multiprotocol Label Switching (MPLS) {{RFC4950}}
- Traceroute with extensions for Interface and Next-Hop Identification {{RFC5837}}
- PROBE {{RFC8335}}


# General Issues

## Transport

ICMP messages can be lost in transport. Therefore, the originator of an ICMP message cannot rely on the message arriving at its intended destination.

## Rate Limits

As per {{RFC1812}} and {{RFC4443}}, nodes must rate limit ICMP messages that they originate. Neither RFC specifies how strict that rate limit must be. However, the rate limit MUST be strict enough to prevent a node from congesting one of its own interfaces with outbound ICMP messages.

## Forgery

ICMP messages are easily forged. {{RFC5927}} describes how forged ICMP messages can be used to attack TCP {{RFC0793}}. It also decribes mitigations against these attacks.

## Message Length {#length}

As per {{RFC1812}}, an ICMPv4 message must not exceed 576 bytes. {{RFC4443}} specifies no limit for ICMPv6 messages. However, to avoid fragmentation, ICMPv6 messages SHOULD not exceed the IPv6 minimum required MTU {{RFC8200}} (i.e., 1280 bytes).

## Source Address Selection

{{RFC1812}} offers the following guidance regarding ICMPv4 source address selection:

> Except where this document specifies otherwise, the IP source address in an ICMP message originated by the router MUST be one of the IP addresses associated with the physical interface over which the ICMP message is transmitted.  If the interface has no IP addresses associated with it, the router's router-id is used instead.

However {{RFC4443}} offer different guidance regarding ICMPv6 source address selection:

> If the message is a response to a message sent to one of the node's unicast addresses, the Source Address of the reply MUST be that same address.

> If the message is a response to a message sent to any other address, the Source Address of the ICMPv6 packet MUST be a unicast address belonging to the node.  The address SHOULD be chosen according to the rules that would be used to select the source address for any other packet originated by the node, given the destination address of the packet.  However, it MAY be selected in an alternative way if this would lead to a more informative choice of address reachable from the destination of the ICMPv6 packet.

# Translation Considerations

Dan Wing: Please add text here

# IANA Considerations

This document does not make any requests of IANA

# Security Considerations

As this document does not introduce any new protocols or operational procedures, it does not introduce any new security considerations

# Acknowledgements

The authors wish to acknowledge Joe Touch for his review and hekpful comments.



