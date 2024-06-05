---
title: "SRv6 Security Considerations"
abbrev: "SRv6 Security Considerations"
category: std
pi: [toc, sortrefs, symrefs]

docname: draft-bdmgct-spring-srv6-security-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Source Packet Routing in Networking"
keyword:
 - Internet-Draft
venue:
  group: "Source Packet Routing in Networking"
  type: "Working Group"
  mail: "spring@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spring/"
  github: "buraglio/draft-bdmgct-spring-srv6-security"
  latest: "https://buraglio.github.io/draft-bdmgct-spring-srv6-security/draft-bdmgct-spring-srv6-security.html"

author:
 -
    ins: N. Buraglio
    name: Nick Buraglio
    org: Energy Sciences Network
    email: buraglio@forwardingplane.net
 -
    ins: T. Mizrahi
    name: Tal Mizrahi
    org: Huawei
    email: tal.mizrahi.phd@gmail.com
 -
    ins: T. Tong
    name: Tian Tong
    org: China Unicom
    email: tongt5@chinaunicom.cn
 -
    ins: L.M. Contreras
    name: Luis M. Contreras
    org: Telefonica
    email: luismiguel.contrerasmurillo@telefonica.com

 -
    ins: L.M. Contreras
    name: Luis M. Contreras
    org: Telefonica
    email: luismiguel.contrerasmurillo@telefonica.com


normative:
  RFC2119:
  RFC9416:

informative:
  RFC8754:
  RFC3552:
  RFC8799:
  RFC9256:
  RFC9055:
  RFC7384:
  RFC8986:
  RFC7855:
  RFC5095:
  RFC8402:
  RFC4301:
  RFC4302:
  RFC4303:
  RFC4942:
  RFC9288:
  RFC9099:
  RFC6169:
  I-D.ietf-spring-srv6-srh-compression:
  IANAIPv6SPAR:
    target: https://www.iana.org/assignments/iana-ipv6-special-registry/iana-ipv6-special-registry.xhtml
    title: "IANA IPv6 Special-Purpose Address Registry"
  STRIDE:
    title: The STRIDE Threat Model
    target: https://msdn.microsoft.com/en-us/library/ee823878(v=cs.20).aspx
    date: 2018
  CanSecWest2007:
    title: IPv6 Routing Header Security
    target: https://airbus-seclab.github.io/ipv6/IPv6_RH_security-csw07.pdf
    date: 2007
  RFC9098:

--- abstract

SRv6 is a traffic engineering, encapsulation and steering mechanism utilizing IPv6 addresses to identify segments in a pre-defined policy. This document discusses security considerations in SRv6 networks, including the potential threats and the possible mitigation methods. The document does not define any new security protocols or extensions to existing protocols.

--- middle

# Introduction

Segment Routing (SR) [RFC8402] utilizing an IPv6 data plane is a source routing model that leverages an IPv6 underlay
and an IPv6 extension header called the Segment Routing Header (SRH) to signal and control the forwarding and path of packets by imposing an ordered list of
path details that are processed at each hop along the signaled path. Because SRv6 is fundamentally bound to the IPv6 protocol, and because of the reliance of a
new header there are security considerations which must be noted or addressed in order to operate an SRv6 network in a reliable and secure manner.
Specifically, some of the main properties of SRv6 that affect the security considerations are:

   *  SRv6 makes use of the SRH which is a new type of Routing Extension
      Header.  Some of the security considerations of the SRH are discussed in [RFC5095] and
      [RFC8754].

   *  SRv6 consists of using the SRH on the IPv6 dataplane, and therefore
      known security considerations of IPv6 [RFC9099] are applicable to SRv6 as well.

   *  While SRv6 uses what appear to be typical IPv6 addresses, the address space is processed differently by segment endpoints.
      A typical IPv6 unicast address is comprised of a network prefix, host identifier, and a subnet mask.
      A typical SRv6 segment identifier (SID) is broken into a locator, a function identifier, and optionally, function arguments.
      The locator must be routable, which enables both SRv6 capable and incapable devices to participate in forwarding, either as normal IPv6 unicast or SRv6.
      The capability to operate in environments that may have gaps in SRv6 support allows the bridging of islands of SRv6 devices with standard IPv6 unicast routing.

This document describes various threats to SRv6 networks and also presents existing approaches to avoid or mitigate the threats.

# Conventions and Definitions

## Requirements Language

{::boilerplate bcp14-tagged}

## Terminology

- HMAC TLV: Hashed Message Authentication Code Type Length Value [RFC8754]

- SID: Segment Identifier [RFC8402]

- SRH: Segment Routing Header [RFC8754]

- SRv6: Segment Routing over IPv6 [RFC8402]


# Threat Model {#threat}

This section introduces the threat model that is used in this document. The model follows the guidelines from BCP 72 ([RFC3552] [RFC9416]), and also leverages the STRIDE model [STRIDE].

Note: An assessment of inter-domain segment routing (SR) is left for future revisions of this document.

The threat terminology used in this document is based on [RFC3552]. Threats are classified according to two main criteria: internal vs. external attackers, and on-path vs. off-path attackers, as discussed in [RFC9055].

Internal vs. External:
: An internal attacker in the context of SRv6 is an attacker who is located within an SR domain. Specifically, an internal attacker either has access to a node in the SR domain, or is located on an internal path between two nodes in the SR domain. In this context, the latter means that the attacker can be reached from a node in the SR domain without traversing an SR egress node, and can reach a node in the SR domain without traversing an SR ingress node. External attackers, on the other hand, are not within the SR domain.

On-path vs. Off-path:

: On-path attackers are located in a position that allows interception, modification or dropping of in-flight packets, as well as insertion (generation) of packets. Off-path attackers can only attack by insertion of packets.

The following figure depicts the attacker types according to the taxonomy above. As illustrated in the figure, on-path attackers are located along the path of the traffic that is under attack, and therefore can listen, insert, delete, modify or replay packets in transit. Off-path attackers can insert packets, and in some cases can passively listen to some of the traffic, such as multicast transmissions.

~~~~~~~~~~~
     on-path         on-path        off-path      off-path
     external        internal       internal      external
     attacker        attacker       attacker      attacker
       |                   |        |__            |
       |     SR      __    | __   _/|  \           |
       |     domain /  \_/ |   \_/  v   \__        v
       |            \      |        X      \       X
       v            /      v                \
 ----->X---------->O------>X------->O------->O---->
                   ^\               ^       /^
                   | \___/\_    /\_ | _/\__/ |
                   |        \__/    |        |
                   |                |        |
                  SR               SR        SR
                  ingress        endpoint    egress
                  node                       node
~~~~~~~~~~~
{: #threat-figure title="Threat Model Taxonomy"}

In the current threat model the SR domain defines the boundary that distinguishes internal from external threats. As specified in [RFC8402]:

~~~~~~~~~~~
   By default, SR operates within a trusted domain.  Traffic MUST be
   filtered at the domain boundaries.
   The use of best practices to reduce the risk of tampering within the
   trusted domain is important.  Such practices are discussed in
   [RFC4381] and are applicable to both SR-MPLS and SRv6.
~~~~~~~~~~~

In the context of the current document it is assumed that SRv6 is deployed within a limited domain [RFC8799] with filtering at the domain boundaries, forming a trusted domain with respect to SRv6. Thus, external attackers are outside of the trusted domain. Specifically, an attack on one domain that is invoked from within a different domain is considered an external attack in the context of the current document.

Following the spirit of [RFC8402], the current document  mandates a filtering mechanism that eliminates the threats from external attackers. This approach limits the scope of the attacks described in this document to within the domain (i.e., internal attackers).

It should be noted that in some threat models the distinction between internal and external attackers depends on whether an attacker has access to a cryptographically secured (encrypted or authenticated) domain. Specifically, in some of these models there is a distinction between an attacker who becomes internal by having physical access, for example by plugging into an active port of a network device, and an attacker who has full access to a legitimate network node, including for example encryption keys if the network is encrypted. The current model does not distinguish between these two types of attackers and there is no assumption about whether the SR domain is cryptographically secured or not. Thus, some of the attacks that are described in the next section can be mitigated by cryptographic means, as further discussed in {{hmac}}.


# Impact of Spoofing and Tampering on SRv6 Networks

SRv6 allows for the sending of IPv6 packets via arbitrary paths. An attacker that is able to spoof or tamper with SRv6 packets may cause packets to traverse arbitrary paths within an SR domain.
This may allow an attacker to perform a number of attacks on the victim networks and hosts that would be mostly unfeasible for a non-SRv6 environment.

Spoofing and tampering protection for the IPv6 SRH is provided via the HMAC TLV (see Section "2.1.2. HMAC TLV" of [RFC8754]). However, at least two considerations should be made:

   * The HMAC TLV is OPTIONAL.

   * While it is presumed that unique keys will be employed by each participating node, in scenarios where the
     network resorts to manual configuration of pre-shared keys, the same key might be reused by multiple systems
     as an (incorrect) short-cut to keeeping the problem of pre-shared key configuration manageable.

The impact of spooding and tampering attacks on SRv6 may result in:

   * Denial of Service (DoS)

   * Privilege Escalation

   * Information Leakage

Denial of Service (DoS) may be achieved by sending multiple SRv6-enabled packets to/through victim nodes, where the SRv6-enabled packets result in a negative performance impact of the victim systems (see [RFC9098] for further details).
Alternatively, an attacker might achieve attack amplification by causing packets to "bounce" multiple times between a set of victim nodes, with the goal of exausing processing resources and/or bandwidth (please see [CanSecWest2007] for a discussion of this type of attack).
Privilege Escalation might be achieved by having an attacker leverage the SRH to circumvent security controls as a result of security devices being unable to enforce security policies in the presence of IPv6 Extension Headers (see [RFC9098]), or by directing packets through paths where packet-filtering policies are not enforced.
Information Leakages might happen in multiple ways. Firstly an attacker could cause packets to traverse paths that the attacker controls, such that packets can be eavesdropped. Secondly, SRv6 could be leveraged to perform network reconnaissance on arbirtrary paths. Thirdly,the SRv6 information encoded as IPv6 addresses may itself be leveraged for network reconnaissance purposes.

{{attacks}} discusses specific implementations of these attacks, along with possible mitigations.



# Attacks based on Packet Manipulation and Processing {#attacks}

The following sub-sections describe attacks based on basic operations on SRv6 packets. These basic operations (abstractions) are as follows:

- Passive listening: an attacker who reads packets off the network can collect information about SR endpoint addresses, SR policies and the network topology. This information can then be used to deploy other types of attacks.
- Packet replaying: in a replay attack the attacker records one or more packets and transmits them at a later point in time.
- Packet insertion: an attacker generates and injects a packet to the network. The generated packet may be maliciously crafted to include false information, including for example false addresses and SRv6-related information.
- Packet deletion: by intercepting and removing packets from the network, an attacker prevents these packets from reaching their destination. Selective removal of packets may, in some cases, cause more severe damage than random packet loss.
- Packet modification: the attacker modifies packets during transit.

## SR Modification Attack {#modification}

### Overview
An attacker can modify a packet while it is in transit in a way that directly affects the packet's SR policy. The modification can affect the destination address of the IPv6 header and/or the SRH. In this context SRH modification may refer to inserting an SRH, removing an SRH, or modifying some of the fields of an existing SRH.

Modification of an existing SRH can be further classified into several possible attacks. Specifically, the attack can include adding one or more SIDs to the segment list, removing one or more SIDs or replacing some of the SIDs with differnet SIDs. Another possible type of modification is by adding, removing or modifying TLV fields in the SRH.

When an SRH is present modifying the destination address (DA) of the IPv6 header affects the active segment. However, DA modification can affect the SR policy even in the absence of an SRH. One example is modifying a DA which is used as a Binding SID [RFC8402]. Another example is modifying a DA which represents a compressed segment list {{I-D.ietf-spring-srv6-srh-compression}}. SRH compression allows to encode multiple compressed SIDs within a single 128-bit SID, and thus modifying the DA can affect one or more hops in the SR policy.

### Scope
An SR modification attack can be performed by on-path attackers. As discussed in {{threat}}, it assumed that filtering is deployed at the domain boundaries, thus limiting the ability of implementing SR modification attacks to on-path internal attackers.

### Impact {#mod-impact}
The SR modification attack allows an attacker to change the SR policy that the packet is steered through and thus to manipulate the path and the processing that the packet is subject to.

Specifically, the SR modification attack can impact the network and the forwarding behavior of packets in one or more of the following ways:

Avoiding a specific node or path:
: An attacker can manipulate the DA and/or SRH in order to avoid a specific node or path. This approach can be used, for example, for bypassing the billing service or avoiding access controls and security filters.

Preferring a specific path:
: The packet can be manipulated to avert packets to a specific path. This attack can result in allowing various unauthorized services such as traffic acceleration. Alternatively, an attacker can avert traffic to be forwarded through a specific node that the attacker has access to, thus facilitating more complex on-path attacks such as passive listening, recon and various man-in-the-middle attacks. It is noted that the SR modification attack is performed by an on-path attacker who has access to packets in transit, and thus can implement these attacks directly. However, SR modification is relatively easy to implement and requires low processing resources by an attacker, while it facilitates more complex on-path attacks by averting the traffic to another node that the attacker has access to and has more processing resources.

Forwarding through a path that causes the packet to be discarded:
: SR modification may casue a packet to be forwarded to a point in the network where it can no longer be forwarded, causing the packet to be discarded.

Manipulating the SRv6 network programming:
: An attacker can trigger a specific endpoint behavior by modifying the destination address and/or SIDs in the segment list. This attack can be invoked in order to manipulate the path or in order to exhaust the resources of the SR endpoint.

Availability:
: An attacker can add SIDs to the segment list in order to increase the number hops that each packet is forwarded through and thus increase the load on the network. For example, a set of SIDs can be inserted in a way that creates a forwarding loop ([RFC8402], [RFC5095]) and thus loads the nodes along the loop. Network programming can be used in some cases to manipulate segment endpoints to perform unnecessary functions that consume processing resources. Path inflation, malicious looping and unnecessary instructions have a common outcome, resource exhaustion, which may in severe cases cause Denial of Service (DoS).


##  Attack on compressed headers

###  Overview

Header compression in SRv6 allows to encode multiple compressed SIDs within a single 128-bit SID. That multiple compressed SIDs are concatenated in the DA reflecting each of the nodes that the flow will traverse up to reaching its destination, thus conforming the traffic engineering path to follow. As long as the flow traverses the SRv6 nodes, the compressed SIDs corresponding to each of those nodes is removed with the DA being shifted to the left. The process is repeated up to reaching the last SRv6 node.

With this processing of the SIDs, each node only determines the next compressed SID applicable only after processing its own corresponding compressed SID. It is only at that moment where issues can be revealed in terms of the validity of the next compressed SID.

###  Scope

A modification or alteration of the compressed header can motivate different undesirable situations. An attack of these characteristics can be performed by on-path attackers.

A first case is that the same kind of issues as the ones identified for SR modification also apply in this case, despite the encoding of the traffic engineered path is different.

A second case can be the insertion of a wrong compressed SID, or the modification of an existing one, in some point of the DA implies the processing of the packet flows up to a point in the network where no longer can be forwarded.

Finally, it is also possible the creation of loops by sequencing the compressed SIDs in a manner that the flows are rerouted towards preceding nodes.

###  Impact

Similar effects as the ones described for the case of SR modification also apply to this case, in terms of affection to the flow steering.

Furthermore, the attacks focused on introducing wrong compressed SIDs or the creation of loops can generate DoS situations either by keeping the nodes processing multiple flows with wrong compressed SIDs, or by generating artificial traffic between nodes in the network which can lead to link congestion.


## Reconnaissance {#recon}

### Overview
An on-path attacker can passively listen to packets and specifically to the SRv6-related information that is conveyed in the IPv6 header and the SRH. This approach can be used for collecting information about SIDs and policies, and thus to facilitate mapping the structure of the network and its potential vulnerabilities.

### Scope
A recon attack is limited to on-path internal attackers.

It is assumed that the SRv6 domain is filtered in a way that prevents any leaks of explicit SRv6 routing information through the boundaries of the administrative domain. External attackers can only collect SRv6-related data in a malfunctioning network in which SRv6-related information is leaked through the boundaries of an SR domain.

### Impact
While the information collected in a recon attack does not compromise the confidentiality of the user data, it allows an attacker to gather information about the network which in turn can be used to enable other attacks.

## Packet Insertion

### Overview
In this attack packets are inserted (injected) into the network with a segment list that defines a specific SR policy. The attack can be applied either by using synthetic packets or by replaying previously recorded packets.

### Scope
Packet insertion can be performed by internal attackers, either on-path or off-path. In the case of a replay attack, recording packets in-flight requires on-path access and the recorded packets can later be injected either from an on-path or an off-path location.

SRv6 domains are assumed to be filtered in a way that mitigates insertion attacks from external attackers.

### Impact
The main impact of this attack is resource exhaustion which compromises the availability of the network, as described in {{mod-impact}}.

## Control and Management Plane Attacks

### Overview
Depending on the control plane protocols used in a network, it is possible to use the control plane as a way of compromising the network. For example, an attacker can advertise SIDs in order to manipulate the SR policies used in the network. A wide range of attacks can be implemented, including injecting control plane messages, selectively removing legitimate messages, replaying them or passively listening to them.

A compromised management plane can also facilitate a wide range of attacks, including manipulating the SR policies or compromising the network availability.

### Scope
Control plane attacks can be performed by internal attackers. Injection can be performed by off-path attackers, while removal, replaying and listening require on-path access. The scope of management attacks depends on the specific management protocol and architecture.

It is assumed that SRv6 domain boundary filtering is used for mitigating potential control plane and management plane attacks from external attackers. Segment routing does not define any specific security mechanisms in existing control plane or management plane protocols. However, existing control plane and management plane protocols use authentication and security mechanisms to validate the authenticity of information.

### Impact
A compromised control plane or management plane can impact the network in various possible ways. SR policies can be manipulated by the attacker to avoid specific paths or to prefer specific paths, as described in {{mod-impact}}. Alternatively, the attacker can compromise the availability, either by defining SR policies that load the network resources, as described in {{mod-impact}}, or by blackholing some or all of the SR policies. A passive attacker can use the control plane or management plane messages as a means for recon, in a similar manner to {{mod-impact}}.

## Other Attacks
Various attacks which are not specific to SRv6 can be used to compromise networks that deploy SRv6. For example, spoofing is not specific to SRv6, but can be used in a network that uses SRv6. Such attacks are outside the scope of this document.

Because SRv6 is completely reliant on IPv6 for addressing, forwarding, and fundamental networking basics, it is potentially subject to any existing or emerging IPv6 vulnerabilities [RFC9099], however, this is out of scope for this document.

# Mitigation Methods {#mitigations}

This section presents methods that can be used to mitigate the threats and issues that were presented in previous sections. This section does not introduce new security solutions or protocols.

## Filtering

### SRH Filtering

SRv6 packets rely on the routing header in order to steer traffic that adheres to a defined SRv6 traffic policy. Thus, SRH filtering can be enforced at the ingress and egress nodes of the SR domain, so that packets with an SRH cannot be forwarded into the SR domain or out of the SR domain. Specifically, such filtering is performed by detecting Next Header 43 (Routing Header) with Routing Type 4 (SRH).

Because of the methodologies used in SID compression {{I-D.ietf-spring-srv6-srh-compression}}, SRH compression does not necessarily use an SRH. In practice this means that when compressed segment lists are used without an SRH, filtering based on the Next Header is not relevant, and thus filtering can only be applied baed on the address range, as described below.

### Address Range Filtering

The IPv6 destination address can be filtered at the SR ingress node in order to mitigate external attacks. An ingress packet with a destination address that defines an active segment with an SR endpoint in the SR domain is filtered.

In order to apply such a filtering mechanism the SR domain needs to have an allocated address range that can be detected and enforced by the SR ingress, for example by using LUA addresses.

Note that the use of GUA addressing in data plane programming could result in an fail open scenario when appropriate border filtering is not implemented or supported.

## Encapsulation of Packets

Packets steered in an SR domain are often encapsulated in an IPv6 encapsulation. This mechanism allows for encapsulation of both IPv4 and IPv6 packets. Encapsulation of packets at the SR ingress node and decapsulation at the SR egress node mitigates the ability of external attackers to impact SR steering within the domain.

## Hashed Message Authentication Code (HMAC) {#hmac}

The SRH can be secured by an HMAC TLV, as defined in [RFC8754]. The HMAC is an optional TLV that secures the segment list, the SRH flags, the SRH Last Entry field and the IPv6 source address. A pre-shared key is used in the generation and verification of the HMAC.

Using an HMAC in an SR domain can mitigate some of the SR Modification Attacks ({{modification}}). For example, the segment list is protected by the HMAC. However, an internal attacker who does not have access to the pre-shared key can capture legitimate packets, and later replay the SRH and HMAC from these recorded packets. This allows the attacker to insert the previously recorded SRH and HMAC into a newly injected packet. An on-path internal attacker can also replace the SRH of an in-transit packet with a different SRH that was previously captured.

# Implications on Existing Equipment

## Limitations in Filtering Capabilities

{{RFC9288}} provides recommendations on the filtering of IPv6 packets containing IPv6 extension headers at transit routers. SRv6 relies on the routing header (RH4). Because the technology is reasonably new, many platforms, routing and otherwise, do not posses the capability to filter and in some cases even provide logging for IPv6 next-header 43 Routing type 4.

## Middlebox Filtering Issues
When an SRv6 packet is forwarded in the SRv6 domain, its destination address changes constantly, the real destination address is hidden. Security devices on SRv6 network may not learn the real destination address and fail to take access control on some SRv6 traffic.

The security devices on SRv6 networks need to take care of SRv6 packets. However, the SRv6 packets usually use loopback address of the PE device a as source address. As a result, the address information of SR packets may be asymmetric, resulting in improper filter traffic problems, which affects the effectiveness of security devices.
For example, along the forwarding path in SRv6 network, the SR-aware firewall will check the association relationships of the bidirectional VPN traffic packets. And it is able to retrieve the final destination of SRv6 packet from the last entry in the . When the <source, destination> tuple of the packet from PE1 to PE2 is <PE1-IP-ADDR, PE2-VPN-SID>, and the other direction is <PE2-IP-ADDR, PE1-VPN-SID>, the source address and destination address of the forward and backward VPN traffic are regarded as different flow. Eventually, the legal traffic may be blocked by the firewall.

SRv6 is commonly used as a tunneling technology in operator networks. To provide VPN service in an SRv6 network, the ingress PE encapsulates the payload with an outer IPv6 header with the SRH carrying the SR Policy segment List along with the VPN service SID. The user traffic towards SRv6 provider backbone will be encapsulated in SRv6 tunnel. When constructing an SRv6 packet, the destination address field of the SRv6 packet changes constantly and the source address field of the SRv6 packet is usually assigned using an address on the originating device, which may be a host or a network element depending on configuration. This may affect the security equipment and middle boxes in the traffic path. Because of the existence of the SRH, and the additional headers, older security appliances, monitoring systems, and middle boxes cold react in different ways if they are unaware of the additional header and tunneling mechanisms leveraged by SRv6. This lack of awareness may be due to software limits, or in some cases as has been seen in other emerging technologies, may be due to limits in ASICs or NPUs that could silently drop or otherwise impede SRv6 packets.
[RFC6169]


## Emerging technology growing pains

# Gap Analysis

This section analyzes the security related gaps with respect to the threats and issues that were discussed in the previous sections.

# Security Considerations

The security considerations of SRv6 are presented throughout this document.

# IANA Considerations

This document has no IANA actions.

# Topics for Further Consideration

This section lists topics that will be discussed further before deciding whether they need to be included in this document, as well as some placeholders for items that need further work.

- The following references may be used in the future: [RFC9256] [RFC8986]
- SRH compression
- Spoofing
- Path enumeration
- Infrastructure and topology exposure: this seems like a non-issue from a WAN perspective. Needs more thought - could be problematic in a host to host scenario involving a WAN and/or a data center fabric.
- Terms that may be used in a future version: Locator Block, FRR, uSID
- L4 checksum: [RFC8200] specifies that when the Routing header is present the L4 checksum is computed by the originating node based on the IPv6 address of the last element of the Routing header.  When compressed segment lists {{I-D.ietf-spring-srv6-srh-compression}} are used, the last element of the Routing header may be different than the Destination Address as received by the final destination. Furthermore, compressed segment lists can be used in the Destination Address without the presence of a Routing header, and in this case the IPv6 Destination address can be modified along the path. As a result, some existing middleboxes which verify the L4 checksum might miscalculate the checksum. This issue is currently under discusison in the SPRING WG.
- Segment Routing Header figure: the SRv6 Segment Routing Header (SRH) is defined in [RFC8754].

~~~~~~~~~~~
     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Next Header   |  Hdr Ext Len  | Routing Type  | Segments Left |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Last Entry   |     Flags     |              Tag              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    |            Segment List[0] (128 bits IPv6 address)            |
    |                                                               |
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    |                                                               |
                                  ...
    |                                                               |
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    |            Segment List[n] (128 bits IPv6 address)            |
    |                                                               |
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~


# Security Considerations

The security considerations of SRv6 are presented throughout this document.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to acknowledge the contributions from Andrew Alston, Dale Carder, Bruno Decraene, Joel Halpern, Alvaro Retana, and Eric Vyncke.
