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


normative:
  RFC2119:

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

- SID: Segment Identifier [RFC8402]

- SRH: Segment Routing Header [RFC8754]

- SRv6: Segment Routing over IPv6 [RFC8402]

# Threat Model {#threat}

This section introduces the threat model that is used in this document. The model is based on terminology from the Internet threat model [RFC3552], as well as some concepts from [RFC9055] and [RFC7384]. Details regarding inter-domain segment routing (SR) are out of scope for this document.

## Security Components

The main components of information security are confidentiality, integrity and availability, often referred to by the acronym CIA. A short description of each of these components is presented below in the context of SRv6 security.

### Confidentiality

The purpose of confidentiality is to protect the user data from being exposed to unauthorized users, i.e., preventing attackers from eavesdropping to user data. The confidentiality of user data is outside the scope of this document. However, confidentiality aspects of SRv6-related information are within the scope; collecting information about SR endpoint addresses, SR policies, and network topologies, is a specific form of reconnaissance, which is further discussed in {{recon}}.

### Integrity

Preventing information from being modified is a key property of information security. Other aspects of integrity include authentication, which is the ability to verify the source of information, and authorization, which enforces different permission policies to different users or sources. In the context of SRv6, compromising the integrity may result in packets being routed in different paths than they were intended to be routed through, which may have various implications, as further discussed in  {{attacks}}.

### Availability

Protecting the availability of a system means keeping the system running continuously without disruptions. The availability aspects of SRv6 include the ability of attackers to leverage SRv6 as a means for compromising the performance of a network or for causing Denial of Service (DoS).

## Threat Abstractions

A security attack is implemented by performing a set of one or more basic operations. These basic operations (abstractions) are as follows:

- Passive listening: an attacker who reads packets off the network can collect information about SR endpoint addresses, SR policies and the network topology. This information can then be used to deploy other types of attacks.
- Packet replaying: in a replay attack the attacker records one or more packets and transmits them at a later point in time.
- Packet insertion: an attacker generates and injects a packet to the network. The generated packet may be maliciously crafted to include false information, including for example false addresses and SRv6-related information.
- Packet deletion: by intercepting and removing packets from the network, an attacker prevents these packets from reaching their destination. Selective removal of packets may, in some cases, cause more severe damage than random packet loss.
- Packet modification: the attacker modifies packets during transit.

## Threat Taxonomy

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

It should be noted that in some threat models the distinction between internal and external attackers depends on whether an attacker has access to a cryptographically secured (encrypted or authenticated) domain. Specifically, in some of these models there is a distinction between an attacker who becomes internal by having physical access, for example by plugging into an active port of a network device, and an attacker who has full access to a legitimate network node, including for example encryption keys if the network is encrypted. The current model does not distinguish between these two types of attackers and there is no assumption about whether the SR domain is cryptographically secured or not.

# Attacks {#attacks}

## SR Modification Attack

### Overview
An attacker can modify a packet while it is in transit in a way that directly affects the packet's SR policy. The modification can affect the destination address of the IPv6 header and/or the SRH. In this context SRH modification may refer to inserting an SRH, removing an SRH, or modifying some of the fields of an existing SRH.

Modification of an existing SRH can be further classified into several possible attacks. Specifically, the attack can include adding one or more SIDs to the segment list, removing one or more SIDs or replacing some of the SIDs with differnet SIDs. Another possible type of modification is by adding, removing or modifying TLV fields in the SRH.

When an SRH is present modifying the destination address (DA) of the IPv6 header affects the active segment. However, DA modification can affect the SR policy even in the absence of an SRH. One example is modifying a DA which is used as a Binding SID [RFC8402]. Another example is a compressed segment list that is incorporated in the DA without an SRH {{I-D.ietf-spring-srv6-srh-compression}}.

### Scope
An SR modification attack can be performed by on-path attackers. As discussed in {{threat}}, it assumed that filtering is deployed at the domain boundaries, thus limiting the ability of implementing SR modification attacks to on-path internal attackers.

### Impact {#mod-impact}
The SR modification attack allows an attacker to change the SR policy that the packet is steered through and thus to manipulate the path and the processing that the packet is subject to.

Specifically, the SR modification attack can impact the network and the forwarding behavior of packets in one or more of the following ways:

Avoiding a specific node or path:
: An attacker can manipulate the DA and/or SRH in order to avoid a specific node or path. This approach can be used, for example, for bypassing the billing service or avoiding access controls and security filters.

Preferring a specific path:
: The packet can be manipulated to avert packets to a specific path. This attack can result in allowing various unauthorized services such as traffic acceleration. Alternatively, an attacker can avert traffic to be forwarded through a specific node that the attacker has access to, thus facilitating more complex on-path attacks such as passive listening, recon and various man-in-the-middle attacks. It is noted that the SR modification attack is performed by an on-path attacker who has access to packets in transit, and thus can implement these attacks directly. However, SR modification is relatively easy to implement and requires low processing resources by an attacker, while it facilitates more complex on-path attacks by averting the traffic to another node that the attacker has access to and has more processing resources.

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

# Mitigation Methods

This section presents methods that can be used to mitigate the threats and issues that were presented in previous sections. This section does not introduce new security solutions or protocols.

## Filtering

### SRH Filtering

SRv6 packets rely on the routing header in order to steer traffic that adheres to a defined SRv6 traffic policy. Thus, SRH filtering can be enforced at the ingress and egress nodes of the SR domain, so that packets with an SRH cannot be forwarded into the SR domain or out of the SR domain.

### Address Range Filtering

The IPv6 destination address can be filtered at the SR ingress node in order to mitigate external attacks. An ingress packet with a destination address that defines an active segment with an SR endpoint in the SR domain is filtered.

In order to apply such a filtering mechanism the SR domain needs to have an allocated address range that can be detected and enforced by the SR ingress, for example by using LUA addresses.

Note that the use of GUA addressing in data plane programming could result in an fail open scenario when appropriate border filtering is not implemented or supported.

## Encapsulation of Packets

Packets steered in an SR domain are often encapsulated in an IPv6 encapsulation. This mechanism allows for encapsulation of both IPv4 and IPv6 packets. Encapsulation of packets at the SR ingress node and decapsulation at the SR egress node mitigates the ability of external attackers to impact SR steering within the domain.

# Implications on Existing Equipment

## Limitations in Filtering Capabilities

{{RFC9288}} provides recommendations on the filtering of IPv6 packets containing IPv6 extension headers at transit routers. SRv6 relies on the routing header (RH4). Because the technology is reasonably new, many platforms, routing and otherwise, do not posses the capability to filter and in some cases even provide logging for IPv6 next-header 43 Routing type 4.

## Middlebox Filtering Issues
An edge case exists that may cause a layer 4 checksum error.  Because of the methodologies used in SID compression, SRH compression does not necessarily use an SRH - in practice this means filtering based on the next header is not relevant, thereby removing an important filtering mechanism. Under certain specific conditions, a host may generate a SID list that is capable of being compressed into a single destination address (DA). Under these circumstances, the SRH may not be generated or may be removed during the process. In this case, a host may generate a layer 4 checksum that is created after SR policy and SID compression is applied and is done so using a DA that differs from the DA that will arrive at the final destination. This incorrect L4 checksum will cause any device in the path that utilizes a L4 checksum to discard or otherwise flag as erroneous the packets referenced by this checksum. Examples include deep packet inspection hardware that may exist transparently in a path or other higher layer packet inspection mechanisms that require or utilize an L4 checksum. Such behavior could result in blackholed or incorrectly dropped / filtered traffic that is otherwise legitimate.

When an SRv6 packet is forwarded in the SRv6 domain, its destination address changes constantly, the real destination address is hidden. Security devices on SRv6 network may not learn the real destination address and fail to take access control on some SRv6 traffic.

The security devices on SRv6 networks need to take care of SRv6 packets. However, the SRv6 packets usually use loopback address of the PE device a as source address. As a result, the address information of SR packets may be asymmetric, resulting in improper filter traffic problems, which affects the effectiveness of security devices.
For example, along the forwarding path in SRv6 network, the SR-aware firewall will check the association relationships of the bidirectional VPN traffic packets. And it is able to retrieve the final destination of SRv6 packet from the last entry in the . When the <source, destination> tuple of the packet from PE1 to PE2 is <PE1-IP-ADDR, PE2-VPN-SID>, and the other direction is <PE2-IP-ADDR, PE1-VPN-SID>, the source address and destination address of the forward and backward VPN traffic are regarded as different flow. Eventually, the legal traffic may be blocked by the firewall.

SRv6 is commonly used as a tunneling technology in operator networks. To provide VPN service in an SRv6 network, the ingress PE encapsulates the payload with an outer IPv6 header with the SRH carrying the SR Policy segment List along with the VPN service SID. The user traffic towards SRv6 provider backbone will be encapsulated in SRv6 tunnel. When constructing an SRv6 packet, the destination address field of the SRv6 packet changes constantly and the source address field of the SRv6 packet is usually assigned using an address on the originating device, which may be a host or a network element depending on configuration. This may affect the security equipment and middle boxes in the traffic path. Because of the existence of the SRH, and the additional headers, older security appliances, monitoring systems, and middle boxes cold react in different ways if they are unaware of the additional header and tunneling mechanisms leveraged by SRv6. This lack of awareness may be due to software limits, or in some cases as has been seen in other emerging technologies, may be due to limits in ASICs or NPUs that could silently drop or otherwise impede SRv6 packets.
[RFC6169]

## Emerging technology growing pains

# Gap Analysis

This section analyzes the security related gaps with respect to the threats and issues that were discussed in the previous sections.

# Topics for Further Consideration

## Security Considerations in Operational SRv6 Enabled Networks
[RFC9256] [RFC8986]

## Segment Routing Header

The SRv6 Segment Routing Header (SRH) is defined in [RFC8754].

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


## Spoofing

## Path enumeration

## Infrastructure and topology exposure

This seems like a non-issue from a WAN perspective. Needs more thought - could be problematic in a host to host scenario involving a WAN and/or a data center fabric.

## Terms that may be used in a future version
Locator Block
FRR
uSID

# Security Considerations

The security considerations of SRv6 are presented throughout this document.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to acknowledge the contributions from Andrew Alston, Dale Carder, Bruno Decraene, Joel Halpern, Alvaro Retana, and Eric Vyncke.
