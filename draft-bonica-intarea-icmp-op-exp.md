---
coding: utf-8
coding: us-ascii

# this first section is yaml, not markdown
# "#" denotes comments here
# to use kramdonw, go to https://xml2rfc.tools.ietf.org/experimental.html,
# click on kramdown as input.  You can also download code to go from
# kramdown to xml and/or text.
# to debug, use the kramdown to xml convertor and pore through the xml.

# Useful links:
# https://kramdown.gettalong.org/quickref.html
# https://www.ietf.org/proceedings/92/slides/slides-92-edu-kramdown-0.pdf

title: Operational Experience With ICMP
abbrev: op-exp-icmp
docname: draft-bonica-intarea-icmp-op-exp
category: info # bcp

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
    org: Juniper Networks
    country: USA
    email: rbonica@juniper.net

#normative:
# - rfc 792
# - rfc2119
# - rfc2616
# - rfc 4443

#informative:


--- abstract

This document summarizes ICMP operational experience. The purpose of this document is to be used as a reference. A document that mentions ICMP can reference this document rather than repeating portions of its content.

--- middle

# Introduction

The Internet Control Message Protocol (ICMP) can be used to exchange control information between hosts. ICMPv4 {{!RFC792}} exchanges control information between IPv4 hosts while ICMPv6 {{!RFC4443}} exchanges control information between IPv6 hosts.

This document summarizes ICMP operational experience. The purpose of this document is to be used as a reference. A document that mentions ICMP can reference this document rather than repeating portions of its content.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# ICMP Message Types

The Internet Assigned Numbers Authority (IANA) maintains:

- a list of [ICMPv4 message types](https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml). 
- a list of [ICMPv6 message types](https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml).

{{!RFC4443}} makes a distintion between informational and error messages. According to {{!RFC4443}}, "Error messages are identified as such by a zero in the high-order bit of their message Type field values.  Thus, error messages have message types from 0 to 127; informational messages have message types from 128 to 255."

Error messages indicate that a downstream node could not forward a packet. The error message includes as much of the original packet as possible, without violating the ICMP message length limitations (see below.)

Informational message provide information regarding downstream nodes. They do not include an original packet field.

{{!RFC792}} does not make a distinction between informational and error messages. However, some ICMPv4 messages indicate that that a downstream node could not forward a packet. Thes messages include as much of the original packet as possible, without violating the ICMP message length limitations (see below.) Other ICMPv4 message provide information regarding downstream nodes.

So, one could make a distinction between informational and error messages in ICMPv4.

# Acceptable Uses

{{!RFC7279}} defines accpetable uses for ICMP. The following are acceptable uses:

- to inform a datagram's originator that a forwarding plane anomaly has been encountered downstream.  The datagram originator must be able to determine whether or not the datagram was discarded by examining the ICMP message.

- to discover and convey dynamic information about a node (other than information usually carried in routing protocols), to discover and convey network-specific parameters, and to discover on-link routers and hosts.

Normally, ICMP SHOULD NOT be used to implement a general-purpose routing or network management protocol.  However, ICMP does have a role to play in conveying dynamic information about a network, which would belong in the second catagory,  above.

# Management Applications

The following management applications rely on ICMP:

- PING {{?RFC2151}}
- Traceroute {{?RFC2151}}
- PROBE {{?RFC8335}}

# Extensibility:

Some ICMP messages can include extensions {{!RFC4443}}. However, the extension header MUST not cause the ICMP message to violate the length restrictions mentioned below. IANA maintains a registry of [ICMP Extension Object Classes and Class Sub-types](https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml#icmp-parameters-ext-classes). 

# General Issues

## Transport

ICMP messages can be lost in transport. Therefore, the originator of an ICMP message cannot rely on the message arriving at its intended destination.

## Rate Limits

As per {{!RFC1812}} and {{!RFC4443}}, nodes must rate limit ICMP messages that they originate. Neither RFC specifies how strict that rate limit must be. However, the rate limit MUST be strict enough to prevent a node from congesting one of its own interfaces with outbound ICMP messages.

## Forgery

ICMP messages are easily forged. {{!RFC5927}} describes how forged ICMP messages can be used to attack TCP {{!RFC793}}. It also decribes mitigations against these attacks.

## Message Length

As per {{!RFC1812}}, an ICMPv4 message must not exceed 576 bytes. {{!RFC4443}} specifies no limit for ICMPv6 messages. However, to avoid fragmentation, ICMPv6 messages SHOULD not exceed the IPv6 minimum required MTU {{!RFC8200}} (i.e., 1280 bytes).

## Source Address Selection

{{!RFC1812}} offers the following guidance regarding ICMPv4 source address selection:

> Except where this document specifies otherwise, the IP source address in an ICMP message originated by the router MUST be one of the IP addresses associated with the physical interface over which the ICMP message is transmitted.  If the interface has no IP addresses associated with it, the router's router-id is used instead.

However {{!RFC4443}} offer different guidance regarding ICMPv6 source address selection:

> If the message is a response to a message sent to one of the node's unicast addresses, the Source Address of the reply MUST be that same address.

> If the message is a response to a message sent to any other address, the Source Address of the ICMPv6 packet MUST be a unicast address belonging to the node.  The address SHOULD be chosen according to the rules that would be used to select the source address for any other packet originated by the node, given the destination address of the packet.  However, it MAY be selected in an alternative way if this would lead to a more informative choice of address reachable from the destination of the ICMPv6 packet.

## Message Processing

### Known Message Types

When an ICMP message of known type is recieved, it must be passed to the appropriate upper-layer application. If the ICMP message contains an Original Packet field, the packet must be passed to the application that originated that packet, if it can be identified. If the ICMP message does not contain an Original Packet field, it should be passed to the appropriate management application.

### Unknown Message Types

{{!RFC1812}} offers the following guidance regarding unknown ICMPv4 message types:

> If an ICMP message of unknown type is received, it MUST be passed to the ICMP user interface (if the router has one) or silently discarded (if the router does not have one).

However {{!RFC4443}} offer different guidance regarding unknown ICMPv6 messages:

> If an ICMPv6 error message of unknown type is received at its destination, it MUST be passed to the upper-layer process that originated the packet that caused the error, where this can be identified

> If an ICMPv6 informational message of unknown type is received, it MUST be silently discarded.

As the ICMPv6 guidance is more specific, it SHOULD also be applied to ICMPv4.



