---
title: "Segment Routing IPv6 Security Considerations"
abbrev: "Segment Routing IPv6 Security Considerations"
category: std
pi: [toc, sortrefs, symrefs]

docname: draft-ietf-spring-srv6-security-latest
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
  latest: "https://github.com/buraglio/draft-bdmgct-spring-srv6-security"

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
    ins: F. Gont
    name: Fernando Gont
    org: SI6 Networks
    email: fgont@si6networks.com


normative:
  RFC2119:
  RFC8402:
  RFC8754:
  RFC8986:
  RFC9020:
  RFC9256:
  RFC9491:
  RFC9524:

informative:
  RFC3552:
  RFC4593:
  RFC6518:
  RFC6242:
  RFC8446:
  RFC9055:
  RFC7384:
  RFC7276:
  RFC9416:
  RFC7855:
  RFC7872:
  RFC8341:
  RFC8253:
  RFC8476:
  RFC8283:
  RFC9325:
  RFC9098:
  RFC5095:
  RFC9259:
  RFC9288:
  RFC9099:
  RFC8200:
  rfc7835:
  rfc9050:
  rfc9602:
  I-D.ietf-spring-srv6-srh-compression:
  I-D.ietf-lsr-ospf-srv6-yang:
  I-D.ietf-lsr-isis-srv6-yang:
  I-D.ietf-pce-segment-routing-policy-cp:
  I-D.ietf-idr-bgp-ls-sr-policy:
  IANAIPv6SPAR:
    target: https://www.iana.org/assignments/iana-ipv6-special-registry/iana-ipv6-special-registry.xhtml
    title: "IANA IPv6 Special-Purpose Address Registry"
  STRIDE:
    title: The STRIDE Threat Model
    target: https://msdn.microsoft.com/en-us/library/ee823878(v=cs.20).aspx
    date: 2018
  ANSI-Sec:
    title: "Operations, Administration, Maintenance, and Provisioning Security Requirements for the Public Telecommunications Network: A Baseline of Security Requirements for the Management Plane"
    target: https://www.ieee802.org/1/ecsg-linksec/meetings/July03/3m150075.pdf
    date: 2003
  CanSecWest2007:
    title: IPv6 Routing Header Security
    target: https://airbus-seclab.github.io/ipv6/IPv6_RH_security-csw07.pdf
    date: 2007

--- abstract

SRv6 is a traffic engineering, encapsulation and steering mechanism utilizing IPv6 addresses to identify segments in a pre-defined policy. This document discusses security considerations in SRv6 networks, including the potential threats and the possible mitigation methods. The document does not define any new security protocols or extensions to existing protocols.

--- middle

# Introduction

Segment Routing (SR) [RFC8402] utilizing an IPv6 data plane is a source routing model that leverages an IPv6 underlay
and an IPv6 extension header called the Segment Routing Header (SRH) [RFC8754] to signal and control the forwarding and path of packets by imposing an ordered list of
segments that are processed at each hop along the signaled path. SRv6 is fundamentally bound to the IPv6 protocol and introduces a new extension header. There are security considerations which must be noted or addressed in order to operate an SRv6 network in a reliable and secure manner.
Specifically, some primary properties of SRv6 that affect the security considerations are:

   *  SRv6 may use the SRH which is a type of Routing Extension Header defined by [RFC8754].
      Security considerations of the SRH are discussed [RFC8754] section 7, and were based in part on security considerations of the deprecated routing header 0 as discussed in [RFC5095] section 5.

   *  SRv6 uses the IPv6 data-plane, and therefore security considerations of IPv6 are applicable to SRv6 as well. Some of these considerations are discussed in Section 10 of [RFC8200] and in [RFC9099].

   *  While SRv6 uses what appear to be typical IPv6 addresses, the address space is processed differently by segment endpoints.
      A typical IPv6 unicast address is comprised of a network prefix, host identifier.
      A typical SRv6 segment identifier (SID) is comprised of a locator, a function identifier, and optionally, function arguments.
      The locator must be routable, which enables both SRv6 capable and incapable devices to participate in forwarding, either as normal IPv6 unicast or SRv6 segment endpoints.
      The capability to operate in environments that may have gaps in SRv6 support allows the bridging of islands of SRv6 devices with standard IPv6 unicast routing.

This document describes various threats to SRv6 networks and also presents existing approaches to avoid or mitigate the threats.

# Scope of this Document

The following IETF RFCs were selected for security assessment as part of this effort:

   * [RFC8402] : &quot;Segment Routing Architecture&quot;
   * [RFC8754] : &quot;IPv6 Segment Routing Header (SRH)&quot;
   * [RFC8986] : &quot;Segment Routing over IPv6 (SRv6) Network Programming&quot;
   * [RFC9020] : &quot;YANG Data Model for Segment Routing&quot;
   * [RFC9256] : &quot;Segment Routing Policy Architecture&quot;
   * [RFC9491] : &quot;Integration of the Network Service Header (NSH) and Segment Routing for Service Function Chaining (SFC)&quot;
   * [RFC9524] : &quot;Segment Routing Replication for Multipoint Service Delivery&quot;

We note that SRv6 is under active development and, as such, the above documents might not cover all protocols employed in an SRv6 deployment.

# Conventions and Definitions

## Requirements Language

{::boilerplate bcp14-tagged}

## Terminology

- HMAC TLV: Hashed Message Authentication Code Type Length Value [RFC8754]

- SID: Segment Identifier [RFC8402]

- SRH: Segment Routing Header [RFC8754]

- SRv6: Segment Routing over IPv6 [RFC8402]

# Threat Terminology {#threat}

This section introduces the threat taxonomy that is used in this document, based on terminology from the Internet threat model [RFC3552], as well as some concepts from [RFC9055], [RFC7384], [RFC7835] and [RFC9416]. Details regarding inter-domain segment routing (SR) are out of scope for this document.

Internal vs. External:

: An internal attacker in the context of SRv6 is an attacker who is located within an SR domain.  Specifically, an internal attacker either has access to a node in the SR domain, or is located within the premises of the SR domain.  External attackers, on the other hand, are not within the SR domain.

On-path vs. Off-path:

: On-path attackers are located in a position that allows interception, modification or dropping of in-flight packets, as well as insertion (generation) of packets. Off-path attackers can only attack by insertion of packets.

Data plane vs. control plane vs. Management plane:

: Attacks can be classified based on the plane they target: data, control, or management. The distinction between on-path and off-path attackers depends on the plane where the attack occurs. For instance, an attacker might be off-path from a data plane perspective but on-path from a control plane perspective.

The following figure depicts an example of an SR domain with five attacker types, labeled 1-5. For instance, attacker 2 is located along the path between the SR ingress node and SR endpoint 1, and is therefore an on-path attacker both in the data plane and in the control plane. Thus, attacker 2 can listen, insert, delete, modify or replay data plane and/or control plane packets in transit. Off-path attackers, such as attackers 4 and 5, can insert packets, and in some cases can passively listen to some traffic, such as multicast transmissions. In this example a Path Computation Element as a Central Controller (PCECC) [RFC9050] is used as part of the control plane. Thus, attacker 3 is an internal on-path attacker in the control plane, as it is located along the path between the PCECC and SR endpoint 1.

~~~~~~~~~~~
  1.on-path   2.on-path   3.mgmt.  PCE as a Central  4.off-path 5.off-path
  external    internal    plane    Controller        internal   external
  attacker    attacker    on-path  (PCECC)           attacker   attacker
       |            |           |        |            |          |
       |            |           v  _____ v ____     _ | __       |
       |     SR  __ | _  __   /        +---+   \___/  |   \      |
       | domain /   |  \/  \_/  X-----|PCECC|         v   /      v
       |        \   |           |      +---+          X   \      X
       v        /   v           |                         /
 ----->X------>O--->X---------->O------->O-------------->O---->
               ^\               ^       /^\             /^
               | \___/\_    /\_ | _/\__/ | \___/\______/ |
               |        \__/    |        |               |
               |                |        |               |
              SR               SR        SR              SR
              ingress      endpoint 1   endpoint 2       egress
              node                                       node
~~~~~~~~~~~
{: #threat-figure title="Threat Model Taxonomy"}

As defined in [RFC8402], SR operates within a "trusted domain". Therefore, in the current threat model the SR domain defines the boundary that distinguishes internal from external threats. Specifically, an attack on one domain that is invoked from within a different domain is considered an external attack in the context of the current document.

# Effect {#sec-effect}

One of the important aspects of threat analysis is assessing the potential effect or outcome of each threat. SRv6 allows for the forwarding of IPv6 packets via predetermined SR policies, which determine the paths and the processing of these packets. An attack on SRv6 may cause packets to traverse arbitrary paths and to be subject to arbitrary processing by SR endpoints within an SR domain. This may allow an attacker to perform a number of attacks on the victim networks and hosts that would be mostly unfeasible for a non-SRv6 environment.

The threat model in [ANSI-Sec] classifies threats according to their potential effect, defining six categories. For each of these categories we briefly discuss its applicability to SRv6 attacks.

- Unauthorized Access: an attack that results in unauthorized access might be achieved by having an attacker leverage SRv6 to circumvent security controls as a result of security devices being unable to enforce security policies. For example, this can occur if packets are directed through paths where packet filtering policies are not enforced, or if some security policies are not enforced in the presence of IPv6 Extension Headers.
- Masquerade: various attacks that result in spoofing or masquerading are possible in IPv6 networks. However, these attacks are not specific to SRv6, and are therefore not within the scope of this document.
- System Integrity: attacks on SRv6 can manipulate the path and the processing that the packet is subject to, thus compromising the integrity of the system. Furthermore, an attack that compromises the control plane and/or the management plane is also a means of affecting the system integrity. Specific SRv6-targeted attack may cause one or more of the following outcomes:
  - Avoiding a specific node or path: when an SRv6 policy is manipulated, specific nodes or paths may be bypassed, for example in order to bypass the billing service or avoid access controls and security filters.
  - Preferring a specific path: packets can be manipulated so that they are diverted to a specific path. This can result in allowing various unauthorized services such as traffic acceleration. Alternatively, an attacker can divert traffic to be forwarded through a specific node that the attacker has access to, thus facilitating more complex on-path attacks such as passive listening, recon and various man-in-the-middle attacks.
  - Causing header modifications: SRv6 network programming determines the SR endpoint behavior, including potential header modifications. Thus, one of the potential outcomes of an attack is unwanted header modifications.
- Communication Integrity: SRv6 attacks may cause packets to be forwarded through paths that the attacker controls, which may facilitate other attacks that compromise the integrity of user data. Integrity protection of user data, which is implemented in higher layers, avoids these aspects, and therefore communication integrity is not within the scope of this document.
- Confidentiality: as in communication integrity, packets forwarded through unintended paths may traverse nodes controlled by the attacker. Since eavesdropping of user data can be avoided by using encryption in higher layers, it is not within the scope of this document. However, eavesdropping of a network that uses SRv6 allows the attacker to collect information about SR endpoint addresses, SR policies, and network topologies, is a specific form of reconnaissance
- Denial of Service: the availability aspects of SRv6 include the ability of attackers to leverage SRv6 as a means for compromising the performance of a network or for causing Denial of Service (DoS), including:
  - Resource exhaustion: compromising the availability of the system can be achieved by sending SRv6-enabled packets to/through victim nodes in a way that results in a negative performance impact of the victim systems (e.g., [RFC9098]). For example, network programming can be used in some cases to manipulate segment endpoints to perform unnecessary functions that consume processing resources. Resource exhaustion may in severe cases cause Denial of Service (DoS).
  - Forwarding loops: an attacker might achieve attack amplification by increasing the number hops that each packet is forwarded through and thus increase the load on the network. For example, a set of SIDs can be inserted in a way that creates a forwarding loop ([RFC8402], [RFC5095], [CanSecWest2007]) and thus loads the nodes along the loop.
  - Causing packets to be discarded: an attacker may cause a packet to be forwarded to a point in the network where it can no longer be forwarded, causing the packet to be discarded.

{{attacks}} discusses specific implementations of these attacks, and possible mitigations are discussed in {{mitigations}}.

# Attacks {#attacks}

## Attack Abstractions {#abstractions}

Packet manipulation and processing attacks can be implemented by performing a set of one or more basic operations. These basic operations (abstractions) are as follows:

- Passive listening: an attacker who reads packets off the network can collect information about SR endpoint addresses, SR policies and the network topology. This information can then be used to deploy other types of attacks.
- Packet replaying: in a replay attack the attacker records one or more packets and transmits them at a later point in time.
- Packet insertion: an attacker generates and injects a packet to the network. The generated packet may be maliciously crafted to include false information, including for example false addresses and SRv6-related information.
- Packet deletion: by intercepting and removing packets from the network, an attacker prevents these packets from reaching their destination. Selective removal of packets may, in some cases, cause more severe damage than random packet loss.
- Packet modification: the attacker modifies packets during transit.

This section describes attacks that are based on packet manipulation and processing, as well as attacks performed by other means. While it is possible for packet manipulation and processing attacks against all the fields of the IPv6 header and its extension headers, this document limits itself to the IPv6 header and the SRH.

## Data Plane Attacks

### Modification Attack {#modification}

#### Overview
An on-path internal attacker can modify a packet while it is in transit in a way that directly affects the packet's segment list.

A modification attack can be performed in one or more of the following ways:

- SID list: the SRH can be manipulated by adding or removing SIDs, or by modifying existing SIDs.
- IPv6 Destination Address (DA): when an SRH is present modifying the destination address (DA) of the IPv6 header affects the active segment. However, DA modification can affect the SR policy even in the absence of an SRH. One example is modifying a DA which is used as a Binding SID [RFC8402]. Another example is modifying a DA which represents a compressed segment list {{I-D.ietf-spring-srv6-srh-compression}}. SRH compression allows encoding multiple compressed SIDs within a single 128-bit SID, and thus modifying the DA can affect one or more hops in the SR policy.
- Add/remove SRH: an attacker can insert or remove an SRH.
- SRH TLV: adding, removing or modifying TLV fields in the SRH.

It is noted that the SR modification attack is performed by an on-path attacker who has access to packets in transit, and thus can implement these attacks directly. However, SR modification is relatively easy to implement and requires low processing resources by an attacker, while it facilitates more complex on-path attacks by averting the traffic to another node that the attacker has access to and has more processing resources.

An on-path internal attacker can also modify, insert or delete other extension headers but these are outside the scope of this document.

#### Scope
An SR modification attack can be performed by on-path attackers. If filtering is deployed at the domain boundaries as described in {{filtering}}, the ability to implement SR modification attacks is limited to on-path internal attackers.

#### Effect {#mod-effect}
SR modification attacks, including adding/removing an SRH, modifying the SID list and modifying the IPv6 DA, can have one or more of the following outcomes, which are described in {{sec-effect}}.

- Unauthorized access
- Avoiding a specific node or path
- Preferring a specific path
- Causing header modifications
- Causing packets to be discarded
- Resource exhaustion
- Forwarding loops

Maliciously adding unnecessary TLV fields can cause further resource exhaustion.

### Passive Listening

#### Overview
An on-path internal attacker can passively listen to packets and specifically listen to the SRv6-related information that is conveyed in the IPv6 header and the SRH. This approach can be used for reconnaissance, i.e., for collecting segment lists.

#### Scope
A reconnaisance attack is limited to on-path internal attackers.

If filtering is deployed at the domain boundaries ({{filtering}}), it prevents any leaks of explicit SRv6 routing information through the boundaries of the administrative domain. In this case external attackers can only collect SRv6-related data in a malfunctioning network in which SRv6-related information is leaked through the boundaries of an SR domain.

#### Effect
While the information collected in a reconnaisance attack does not compromise the confidentiality of the user data, it allows an attacker to gather information about the network which in turn can be used to enable other attacks.

### Packet Insertion

#### Overview
In a packet insertion attack packets are inserted (injected) into the network with a segment list. The attack can be applied either by using synthetic packets or by replaying previously recorded packets.

#### Scope
Packet insertion can be performed by either on-path or off-path attackers. In the case of a replay attack, recording packets in-flight requires on-path access and the recorded packets can later be injected either from an on-path or an off-path location.

If filtering is deployed at the domain boundaries ({{filtering}}), insertion attacks can only be implemented by internal attackers.

#### Effect
The main effect of this attack is resource exhaustion, which compromises the availability of the network, as described in {{mod-effect}}.

### Other Attacks
Various attacks which are not specific to SRv6 can be used to compromise networks that deploy SRv6. For example, spoofing is not specific to SRv6, but can be used in a network that uses SRv6. Such attacks are outside the scope of this document.

Because SRv6 is completely reliant on IPv6 for addressing, forwarding, and fundamental networking basics, it is potentially subject to any existing or emerging IPv6 vulnerabilities [RFC9099], however, this is out of scope for this document.

## Control Plane Attacks
### Overview
The SRv6 control plane leverages existing control plane protocols, such as BGP, IS-IS, OSPF and PCE. Consequently, any security attacks that can potentially compromise these protocols are also applicable to SRv6 deployments utilizing them. Therefore, this document does not provide an exhaustive list of the potential control plane attacks. Instead, it highlights key categories of attacks, focusing on three primary areas: attacks targeting routing protocols, centralized control plane infrastructures, and Operations, Administration, and Maintenance (OAM) protocols.

### Routing Protocol Attacks
#### Overview
Generic threats applicable to routing protocols are discussed in {{RFC4593}}. Similar to data plane attacks, the abstractions outlined in {{abstractions}} are also applicable to control plane traffic. These include passive eavesdropping, message injection, replay, deletion, and modification.

Passive listening enables an attacker to intercept routing protocol messages as they traverse the network. This form of attack does not alter the content of the messages but allows the adversary to analyze routing information, infer network topology, and gather intelligence on routing behavior.

Active attacks involve the unauthorized injection or alteration of control plane messages. Such attacks can compromise routing integrity by introducing falsified information, modifying legitimate routing data, or triggering incorrect forwarding decisions. These disruptions may result in denial-of-service conditions or traffic misdirection.

For example, an attacker may advertise falsified SIDs to manipulate SR policies. Another example in the context of SRv6 is the advertisement of an incorrect Maximum SID Depth (MSD) value {{RFC8476}}. If the advertised MSD is lower than the actual capability, path computation may fail to compute a viable path. Conversely, if the value is higher than supported, an attempt to instantiate a path that can't be supported by the head-end (the node performing the SID imposition) may occur.

#### Scope
The location of an attacker in the network significantly affects the scope of potential attacks. Off-path attackers are generally limited to injecting malicious routing messages, while on-path attackers can perform a broader range of attacks, including active modification or passive listening.

#### Effect
Attacks targeting the routing protocol can have diverse impacts on network operation, including the aspects described in {{sec-effect}}. These impacts may include incorrect SR policies or the degradation of network availability, potentially resulting in service disruption or denial of service.

### OAM Attacks
#### Overview
Since SRv6 operates over an IPv6 infrastructure, existing OAM protocols designed for IPv6 networks are applicable to SRv6 as well. Consequently, the security considerations associated with conventional IPv6 OAM protocols are also relevant to SRv6 environments. As noted in {{RFC7276}}, successful attacks on OAM protocols can mislead operators by simulating non-existent failures or by concealing actual network issues. SRv6-specific OAM aspects are specified in {{RFC9259}}.

The O-flag in the SRH serves as a marking bit in user packets to trigger telemetry data collection and export at the segment endpoints. An attacker may exploit this mechanism by setting the O-flag in transit packets, thereby overloading the control plane and degrading system availability. Additionally, an on-path attacker may passively intercept OAM data exported to external analyzers, potentially gaining unauthorized insight into network topology and behavior.

#### Scope
Off-path attackers may attempt to degrade system availability by injecting fabricated OAM messages or SRv6 packets with the O-bit set, thereby triggering unnecessary telemetry processing. They may also probe SRv6 nodes to infer information about network state and performance characteristics.

On-path attackers possess enhanced capabilities due to their position within the traffic path. These include passive interception of OAM data, unauthorized modification of the O-bit in transit packets, and tampering with legitimate OAM messages to mislead network monitoring systems or conceal operational issues.

#### Effect
Attacks targeting OAM protocols may impact network availability or facilitate unauthorized information gathering. Such attacks can disrupt normal operations or expose sensitive details about network topology, performance, or state.

### Central Control Plane Attacks
#### Overview
Centralized control plane architectures, such as those based on the Path Computation Element Communication Protocol (PCECC) {{RFC8283}}, inherently introduce a single point of failure. This centralization may present a security vulnerability, particularly with respect to denial-of-service (DoS) attacks targeting the controller. Furthermore, the central controller becomes a focal point for potential interception or manipulation of control messages exchanged with individual Network Elements (NEs), thereby increasing the risk of compromise to the overall network control infrastructure.

#### Scope
As with other control plane attacks, an off-path attacker may attempt to inject forged control messages or impersonate a legitimate controller. On-path attackers, by virtue of their position within the communication path, possess additional capabilities such as passive interception of control traffic and in-transit modification of messages exchanged between the controller and Network Elements (NEs).

#### Effect
A successful attack may result in any of the adverse effects described in {{sec-effect}}, potentially impacting availability and operational correctness.

## Management Plane Attacks

### Overview
Similar to the control plane, a compromised management plane can enable a broad range of attacks, including unauthorized manipulation of SR policies and disruption of network availability. The specific threats and their potential impact are influenced by the management protocols in use. 

As with centralized control systems, a centralized management infrastructure may introduce a single point of failure, rendering it susceptible to denial-of-service (DoS) attacks or making it a target for eavesdropping and message tampering.

Unauthorized access in a network management system can enable attackers or unprivileged users to gain control over network devices and alter configurations. In SRv6-enabled environments, this can result in the manipulation of segment routing policies or cause denial-of-service (DoS) conditions by disrupting traffic or tampering with forwarding behavior.

Management functionality is often defined using YANG data models, such as those specified in {{I-D.ietf-lsr-isis-srv6-yang}} and {{I-D.ietf-lsr-ospf-srv6-yang}}. As with any YANG module, data nodes marked as writable, creatable, or deletable may be considered sensitive in certain operational environments. Unauthorized or unprotected write operations (e.g., via edit-config) targeting these nodes can adversely affect network operations. Some of the readable data nodes in a YANG module may also be considered sensitive or vulnerable in some network environments.

#### Scope
As with control plane attacks, an off-path attacker may attempt to inject forged management messages or impersonate a legitimate network management system. On-path attackers, due to their privileged position within the communication path, have additional capabilities such as passive interception of management traffic and unauthorized modification of messages in transit. An attacker with unauthorized access to a management system can cause significant damage, depending on the scope of the system and the strength of the access control mechanisms in place.

#### Effect
A successful attack may result in any of the adverse effects described in {{sec-effect}}, potentially impacting availability and operational correctness.

## Attacks - Summary
The following table summarizes the attacks that were described in the previous subsections, and the corresponding effect of each of the attacks. Details about the effect are described in {{sec-effect}}.

~~~~~~~~~~~
+=============+==================+===================================+
| Attack      | Details          | Effect                            |
+=============+==================+===================================+
|Modification |Modification of:  |* Unauthorized access              |
|             |* SID list        |* Avoiding a specific node or path |
|             |* IPv6 DA         |* Preferring a specific path       |
|             |Add/remove/modify:|* Causing header modifications     |
|             |* SRH             |* Causing packets to be discarded  |
|             |* SRH TLV         |* Resource exhaustion              |
|             |                  |* Forwarding loops                 |
+-------------+------------------+-----------------------------------+
|Passive      |Passively listen  |* Reconnaissance                   |
|listening    |to SRv6-related   |                                   |
|             |information       |                                   |
+-------------+------------------+-----------------------------------+
|Packet       |Maliciously inject|* Resource exhaustion              |
|insertion    |packets with a    |                                   |
|             |segment list      |                                   |
+-------------+------------------+-----------------------------------+
|Control plane|* Routing protocol|                                   |
|attacks      |  attacks         |                                   |
|             |* OAM attacks     |                                   |
|             |* Central control |                                   |
|             |  plane attacks   |* Unauthorized access              |
|             |                  |* Avoiding a specific node or path |
|             |                  |* Preferring a specific path       |
+-------------+------------------+* Causing header modifications     |
|Management   |* Centralized     |* Causing packets to be discarded  |
|plane attacks|  management      |* Resource exhaustion              |
|             |  attacks         |* Forwarding loops                 |
|             |* Unauthorized    |                                   |
|             |  access to the   |                                   |
|             |  management      |                                   |
|             |  system          |                                   |
+-------------+------------------+-----------------------------------+

~~~~~~~~~~~
{: #summary-table title="Attack Summary"}

# Mitigation Methods {#mitigations}

This section presents methods that can be used to mitigate the threats and issues that were presented in previous sections. This section does not introduce new security solutions or protocols.

## Trusted Domains and Filtering {#filtering}

### Overview

As specified in [RFC8402]:

~~~~~~~~~~~
   By default, SR operates within a trusted domain.  Traffic MUST be
   filtered at the domain boundaries.
   The use of best practices to reduce the risk of tampering within the
   trusted domain is important.  Such practices are discussed in
   [RFC4381] and are applicable to both SR-MPLS and SRv6.
~~~~~~~~~~~

Following the spirit of [RFC8402], the current document assumes that SRv6 is deployed within a trusted domain. Traffic MUST be filtered at the domain boundaries. Thus, most of the attacks described in this document are limited to within the domain (i.e., internal attackers).

Such an approach has been commonly referred to as the concept of "fail-open", a state of which the attributes are frequently described as containing inherently more risk than fail-closed methodologies. The reliance of perfectly crafted filters on on all edges of the trusted domain, noting that if the filters are removed or adjusted in an erroneous manner, there is a demonstrable risk of inbound or outbound leaks. It is also important to note that some filtering implementations have limits on the size, complexity, or protocol support that can be applied, which may prevent the filter adjustments or creation required to properly secure the trusted domain for a new protocol such as SRv6.

Practically speaking, this means successfully enforcing a "Trusted Domain" may be operationally difficult and error-prone in practice, and that attacks that are expected to be unfeasible from outside the trusted domain may actually become feasible when any of the involved systems fails to enforce the filtering policy that is required to define the Trusted Domain.

### SRH Filtering

Filtering can be performed based on the presence of an SRH. More generally, {{RFC9288}} provides recommendations on the filtering of IPv6 packets containing IPv6 extension headers at transit routers. However, filtering based on the presence of an SRH is not necessarily useful for two reasons:
1. The SRH is optional for SID processing as described in [RFC8754] section 3.1 and 4.1.
2. A packet containing an SRH may not be destined to the SR domain, it may be simply transiting the domain.

For these reasons SRH filtering is not necessarily a useful method of mitigation.

### Address Range Filtering

The IPv6 destination address can be filtered at the SR ingress node and at all nodes implementing SRv6 SIDs within the SR domain in order to mitigate external attacks. Section 5.1 of [RFC8754] describes this in detail and a summary is presented here:
1. At ingress nodes, any packet entering the SR domain and destined to a SID within the SR domain is dropped.
2. At every SRv6 enabled node, any packet destined to a SID instantiated at the node from a source address outside the SR domain is dropped.

In order to apply such a filtering mechanism the SR domain needs to have an infrastructure address range for SIDs, and an infrastructure address range for source addresses, that can be detected and enforced. Some examples of an infrastructure address range for SIDs are:
1. ULA addresses
2. The prefix defined in [RFC9602]
3. GUA addresses

Many operators reserve a /64 block for all loopback addresses and allocate /128 for each loopback interface. This simplifies the filtering of permitted source addresses.

Failure to implement address range filtering at ingress nodes is mitigated with filtering at SRv6 enabled nodes. Failure to implement both filtering mechanisms could result in a "fail open" scenario, where some attacks by internal attackers described in this document may be launched by external attackers.

Filtering on prefixes has been shown to be useful, specifically [RFC8754]'s description of packet filtering. There are no known limitations with filtering on infrastructure addresses, and [RFC9099] expands on the concept with control plane filtering.

## Encapsulation of Packets

Packets steered in an SR domain are often encapsulated in an IPv6 encapsulation. This mechanism allows for encapsulation of both IPv4 and IPv6 packets. Encapsulation of packets at the SR ingress node and decapsulation at the SR egress node mitigates the ability of external attackers to attack the domain.

## Hashed Message Authentication Code (HMAC) {#hmac}

The SRH can be secured by an HMAC TLV, as defined in [RFC8754]. The HMAC is an optional TLV that secures the segment list, the SRH flags, the SRH Last Entry field and the IPv6 source address. A pre-shared key is used in the generation and verification of the HMAC.

Using an HMAC in an SR domain can mitigate some of the SR Modification Attacks ({{modification}}). For example, the segment list is protected by the HMAC.

The following aspects of the HMAC should be considered:

- The HMAC TLV is OPTIONAL.
- While it is presumed that unique keys will be employed by each participating node, in scenarios where the network resorts to manual configuration of pre-shared keys, the same key might be reused by multiple systems as an (incorrect) shortcut to keeping the problem of pre-shared key configuration manageable.
- When the HMAC is used there is a distinction between an attacker who becomes internal by having physical access, for example by plugging into an active port of a network device, and an attacker who has full access to a legitimate network node, including for example encryption keys if the network is encrypted. The latter type of attacker is an internal attacker who can perform any of the attacks that were described in the previous section as relevant to internal attackers.
- An internal attacker who does not have access to the pre-shared key can capture legitimate packets, and later replay the SRH and HMAC from these recorded packets. This allows the attacker to insert the previously recorded SRH and HMAC into a newly injected packet. An on-path internal attacker can also replace the SRH of an in-transit packet with a different SRH that was previously captured.

These considerations limit the extent to which HMAC TLV can be relied upon as a security mechanism that could readily mitigate threats associated with spoofing and tampering protection for the IPv6 SRH.

## Control Plane Mitigation Methods

Mitigation strategies for control plane attacks depend heavily on the specific protocols in use. Since these protocols are not exclusive to SRv6, this section does not attempt to provide an exhaustive list of mitigation techniques. Instead, it is focused on considerations particularly relevant to SRv6 deployments.

Routing protocols can employ authentication and/or encryption to protect against modification, injection, and replay attacks, as outlined in [RFC6518]. These mechanisms are essential for maintaining the integrity and authenticity of control plane communications.

In centralized SRv6 control plane architectures, such as those described in {{I-D.ietf-pce-segment-routing-policy-cp}}, it is RECOMMENDED that communication between PCEs and PCCs be secured using authenticated and encrypted sessions. This is typically achieved using Transport Layer Security (TLS), following the guidance in [RFC8253] and best practices in [RFC9325].

When the O-flag is used for Operations, Administration, and Maintenance (OAM) functions, as defined in [RFC9259], implementations should enforce rate limiting to mitigate potential denial-of-service (DoS) attacks triggered by excessive control plane signaling.

The control plane should be confined to a trusted administrative domain. As specified in {{I-D.ietf-idr-bgp-ls-sr-policy}}, SR Policy information advertised via BGP should be restricted to authorized nodes, controllers, and applications within this domain. Similarly, the use of the O-flag is assumed to occur only within such a trusted environment, where the risk of abuse is minimized.

## Management Plane Mitigation Methods

Mitigating attacks on the management plane, much like in the control plane, depends on the specific protocols and interfaces employed. 

Management protocols such as NETCONF and RESTCONF are commonly used to configure and monitor SRv6-enabled devices. These protocols must be secured to prevent unauthorized access, configuration tampering, or information leakage.

The lowest NETCONF layer is the secure transport layer, and the mandatory-to-implement secure transport is Secure Shell (SSH) [RFC6242]. The lowest RESTCONF layer is HTTPS, and the mandatory-to-implement secure transport is TLS [RFC8446].

The Network Configuration Access Control Model (NACM) [RFC8341] provides the means to restrict access for particular NETCONF or RESTCONF users to a pre-configured subset of all available NETCONF or RESTCONF protocol operations and content.

SRv6-specific YANG modules should be designed with the same security considerations applied to all YANG-based models. Writable nodes must be protected using access control mechanisms such as NACM and secured transport protocols like SSH or TLS to prevent unauthorized configuration changes, while readable nodes that expose sensitive operational data should be access-controlled and transmitted only over encrypted channels to mitigate the risk of information leakage.

# Implications on Existing Equipment

## Middlebox Filtering Issues
When an SRv6 packet is forwarded in the SRv6 domain, its destination address changes constantly and the real destination address is hidden. Security devices on SRv6 network may not learn the real destination address and fail to perform access control on some SRv6 traffic.

The security devices on SRv6 networks need to take care of SRv6 packets. However, SRv6 packets are often encapsulated by an SR ingress device with an IPv6 encapsulation that has the loopback address of the SR ingress device as a source address. As a result, the address information of SR packets may be asymmetric, resulting in improper traffic filter problems, which affects the effectiveness of security devices.
For example, along the forwarding path in SRv6 network, the SR-aware firewall will check the association relationships of the bidirectional VPN traffic packets. And it is able to retrieve the final destination of an SRv6 packet from the last entry in the SRH. When the <source, destination> tuple of the packet from PE1 (Provider Edge 1) to PE2 is <PE1-IP-ADDR, PE2-VPN-SID>, and the other direction is <PE2-IP-ADDR, PE1-VPN-SID>, the source address and destination address of the forward and backward traffic are regarded as different flows. Thus, legitimate traffic may be blocked by the firewall.

Forwarding SRv6 traffic through devices that are not SRv6-aware might in some cases lead to unpredictable behavior. Because of the existence of the SRH, and the additional headers, security appliances, monitoring systems, and middle boxes could react in different ways if they do not incorporate support for the supporting SRv6 mechanisms, such as the IPv6 Segment Routing Header (SRH) [RFC8754]. Additionally, implementation limitations in the processing of IPv6 packets with extension headers may result in SRv6 packets being dropped [RFC7872],[RFC9098].

## Limited capability hardware

In some cases, access control list capabilities are a resource shared with other features across a given hardware platform. Filtering capabilities should be considered along with other hardware reliant functions such as VLAN scale, route table size, MAC address table size, etc. Filtering both at the control and data plane may or may not require shared resources.
For example, some platforms may require allocating resources from route table size in order to accommodate larger numbers of access lists. Hardware and software configurations should be considered when designing the filtering capabilities for an SRv6 control and data plane.

# Security Considerations

The security considerations of SRv6 are presented throughout this document.

# IANA Considerations

This document has no IANA actions.

# Topics for Further Consideration

This section lists topics that will be discussed further before deciding whether they need to be included in this document, as well as some placeholders for items that need further work.

- Add tables for attack section
- The following references may be used in the future: [RFC9256] [RFC8986]
- SRH compression
- Spoofing
- Path enumeration
- host to host scenario involving a WAN and/or a data center fabric.
- Terms that may be used in a future version: Locator Block, FRR, uSID
- L4 checksum: [RFC8200] specifies that when the Routing header is present the L4 checksum is computed by the originating node based on the IPv6 address of the last element of the Routing header.  When compressed segment lists {{I-D.ietf-spring-srv6-srh-compression}} are used, the last element of the Routing header may be different than the Destination Address as received by the final destination. Furthermore, compressed segment lists can be used in the Destination Address without the presence of a Routing header, and in this case the IPv6 Destination address can be modified along the path. As a result, some existing middleboxes which verify the L4 checksum might miscalculate the checksum. This issue is currently under discussion in the SPRING WG.
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

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to acknowledge the valuable input and contributions from Zafar Ali, Andrew Alston, Dale Carder, Bruno Decraene, Dhruv Dhody, Mike Dopheide, Darren Dukes, Joel Halpern, Boris Hassanov, Alvaro Retana, Eric Vyncke, and Russ White.
