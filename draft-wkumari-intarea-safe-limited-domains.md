---
title: "Fail-closed Limited Domains"
abbrev: "fail-closed-limited-domains"
docname: draft-wkumari-intarea-safe-limited-domains-latest
category: info
submissionType: IETF

ipr: trust200902
area: "Internet"
workgroup: "Internet Area Working Group"
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: W. Kumari
    name: Warren Kumari
    organization: Google, LLC
    email: warren@kumari.net
  -
    ins: A. Alston
    name: Andrew Alston
    organization: Alston Networks
    email: alston.networks@gmail.com
  -
    ins: É. Vyncke
    name: Éric Vyncke
    organization: Cisco
    email: evyncke@cisco.com
  -
    ins: S. Krishnan
    name: Suresh Krishnan
    organization: Cisco
    email: suresh.krishnan@gmail.com
  -
    ins: D. Eastlake
    name: Donald Eastlake
    organization: Independent
    email: d3e3e3@gmail.com

normative:
  RFC8799:
informative:
  RFC2784:
  RFC3031:
  RFC3682:
  RFC3927:
  RFC4291:
  RFC5771:
  RFC7665:
  RFC8085:
  RFC8754:
  RFC8926:
  RFC8994:
  RFC8995:
  RFC9378:
  RFC9542:
  RFC9674:
  IESG_EtherType:
    title: IESG Statement on EtherTypes
    author:
    - org:
    date: 2023-05-01
    seriesinfo:
      Web: <https://www.ietf.org/about/groups/iesg/statements/ethertypes>

  IANA_EtherType:
    title: IANA EtherType Registry
    author:
    - org:
    date: false
    seriesinfo:
      Web: <https://www.iana.org/assignments/ieee-802-numbers/ieee-802-numbers.xhtml#ieee-802-numbers-1>

--- abstract

Documents describing protocols intended solely for use within "limited domains"
often rely on edge filtering at every boundary node to prevent domain-internal
traffic from leaking to the global Internet (and vice versa). While edge
filtering is a critical and standard operational practice, relying solely on
administrative policies can create "fail-open" designs if configuration errors,
ACL bypass, or hardware table limitations occur.

This document describes design principles and concrete mechanisms that allow
limited-domain protocols to "fail-closed" by default. By leveraging Layer-2
encapsulation identifiers (such as dedicated or extended EtherTypes), local
address scoping, and / or Hop-Limit boundaries, protocol designers can provide
robust defense-in-depth and significantly reduce the operational and security
risks associated with limited-domain protocol leakage.

These mechanisms are not applicable to all protocols intended for use in a
limited domain, but if implemented on certain classes of protocols, can
significantly reduce the risks.


--- middle

# Introduction

{{RFC8799}} discusses the concept of "limited domains", provides examples of
limited domains, as well as Examples of Limited Domain Solutions, including
Service Function Chaining (SFC {{RFC7665}}), Segment Routing, "Creative uses
of IPv6 features" (including Extension headers, e.g., for in situ Operations,
Administration, and maintenance {{RFC9378}}).

In order to provide context, this document will quote extensively from
{{RFC8799}}, but it is assumed that the reader will actually read {{RFC8799}}
in its entirety.

{{RFC8799}} Section 3, notes:

> A common argument is that if a protocol is intended for limited use, the
> chances are very high that it will in fact be used (or misused) in other
> scenarios including the so-called open Internet. This is undoubtedly true and
> means that limited use is not an excuse for bad design or poor security. In
> fact, a limited use requirement potentially adds complexity to both the
> protocol and its security design, as discussed later.

Notably, in {{RFC8799}} Section 2, states:

> Domain boundaries that are defined administratively (e.g., by address
> filtering rules in routers) are prone to leakage caused by human error,
> especially if the limited domain traffic appears otherwise normal to the
> boundary routers. In this case, the network operator needs to take active
> steps to protect the boundary. This form of leakage is much less likely if
> nodes must be explicitly configured to handle a given limited-domain
> protocol, for example, by installing a specific protocol handler.

In addition, {{RFC8799}} Section 6, notes:

> Today, where limited domains exist, they are essentially created by careful
> configuration of boundary routers and firewalls. If a domain is characterized
> by one or more address prefixes, address assignment to hosts must also be
> carefully managed. This is an error-prone method, and a combination of
> configuration errors and default routing can lead to unwanted traffic
> escaping the domain. Our basic assumption is therefore that it should be
> possible for domains to be created and managed automatically, with minimal
> human configuration. We now discuss requirements for automating domain
> creation and management.

This document discusses some of the mechanisms which protocol designers can use
to limit the scope of their protocols to a single link. If the protocol is
intended to be used across multiple links, but should not be forwarded beyond a
single administrative domain, then the protocol designer should consider making
the protocol "fail-closed" rather than "fail-open", as described below.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Types of limited domain protocols

{{RFC8799}} Section 3 discusses some examples of Limited Domains, based mainly
on the network type (e.g. Home, Sensor Networks, Data Centers, etc).

For the functional purposes of this document, limited-domain protocols can be
categorized high-level into two broad groups (though some protocols may span or
hybridize these categories):

* Layer-2 type limited domain protocols: These are protocols that are intended
to be used within a single local-area network (LAN) segment or broadcast
domain.

* Transport and infrastructure-type services: These protocols provide multi-hop
transport or routing functionality, intended to remain strictly within a single
administrative domain (such as an Enterprise or a Service Provider network).

# Fail-open versus Fail-closed

Protocols can be classified based on how they behave when encountering an
unconfigured interface or boundary.

"Fail-closed" protocols require explicit device or interface-wide configuration
to enable them to be accepted, processed, or forwarded when received on an
interface. If the protocol is not explicitly enabled on an interface, incoming
packets of that protocol type are dropped or ignored by default. This design
ensures that traffic does not transit across boundary nodes into networks where
it is not supported or authorized.

"Fail-open" protocols are those that rely on explicit configuration (such as
filtering or Access Control Lists) at domain boundaries to *prevent* them from
leaking. Without active filtering, standard forwarding hardware will parse and
forward these packets normally across boundaries if the routing table allows
it.

While edge filtering is an essential tool, fail-open designs are inherently
more susceptible to accidental configuration omissions, ACL resource
exhaustion, or transient states during troubleshooting. Fail-closed protocols
provide defense-in-depth by eliminating the dependency on boundary filters.

Note that a protocol may exhibit different containment properties at different
points in the network. For example, a protocol may be designed to fail-closed
at the boundary/edge of an administrative domain via interface-specific
encapsulation configuration, even if it is forwarded natively in the core.

# IP Hop-Limit Limiting

Some limited-domain protocols are intended to operate strictly within a single
IP subnet or link. In these cases, protocol specifications SHOULD require that
the IP Hop-Limit (or IPv4 TTL) to be set to 1 upon transmission.

As standard routers decrement the Hop-Limit upon forwarding and discard packets
when the count reaches zero, setting the Hop-Limit to 1 ensures that the
traffic cannot be forwarded off-link. This differs from the Generalized TTL
Security Mechanism (GTSM, {{RFC3682}}), which sets the Hop-Limit to 255 to
verify that the sender is on-link and protect against off-link spoofing.

Which option to choose (if either) depends on the specific requirements of the
protocol.

# IPv4 Local Multicast Addressing

Some protocols (e.g. OSPF) use addresses from the IP Local Network Control
Block {{RFC5771}} (224.0.0.0/24). In addition to providing a discovery
mechanism, standard routers do not forward this traffic off-link, providing a
simple and effective way to limit the scope of the protocol.

In some cases, IPv4 "Link Local" addresses {{RFC3927}} may be an appropriate
mechanism to limit the scope of the protocol, but this is a niche case that is
not discussed further here.

# IPv6 Link Local and Multicast Addresses

Link-Local IPv6 Unicast Addresses ({{RFC4291}} Section 2.5.6) are used for
communication between nodes on a single link. They are not routable and are not
forwarded by routers. In cases where a limited-domain protocol is intended to
be used only within a single link, the use of IPv6 Link-Local addresses is an
effective containment mechanism.

Similarly, IPv6 Link-Local Multicast Addresses in the `ff02::/16` range
({{RFC4291}} Section 2.7) are restricted to the local link scope and are not
forwarded by routers, providing a reliable on-link multicast discovery and
communication channel.

# Making a layer-3 type limited-domain protocol fail-closed

One way to make a limited-domain protocol fail-closed is to assign it a unique
layer-2 protocol identifier, usually an EtherType. In modern router and host
interfaces, if a specific Layer-2 protocol identifier is not administratively
enabled on that port, the network interface card or chipset will ignore the
frame.

Thus, the protocol must be explicitly enabled on all relevant interfaces inside
the domain, automatically blocking it at the domain boundary where the protocol
has not been enabled. This is a robust and simple mechanism that ensures the
protocol does not leak out of the limited domain, even if an operator makes a
mistake in configuring filters based on identifiers appearing deeper in the
packet (such as IP addresses, IP protocols, or header options).

A Layer-2 protocol identifier is a powerful tool, but it is not the only option
for fail-closed designs, and protocol designers should evaluate it alongside
other mechanisms such as local address scoping, hop-limit boundaries, and
cryptographic mechanisms depending on the target environment.

# Ethernet Protocol Identification

Figure 1 shows the general format of Ethernet frames. The relevant protocol
identification field occurs after the destination and source MAC addresses and
any tags (such as VLAN tags). The alternatives for protocol identification are
discussed in Section 3 of {{RFC9542}}.

    +-----------+-----------+- - - - +----------+--------- - - -+-------+
    |Destination|  Source   |Optional| Protocol |  Body of      |Trailer|
    |MAC Address|MAC Address|  Tags  |Identifier|   Frame        |       |
    +-----------+-----------+ - - - -+----------+--------- - - -+-------|
{: #fig1 title="Ethernet Frame Format"}

This document considers EtherType protocol identification. An EtherType is an
unsigned 16-bit field in an Ethernet frame with a value in the range of 0x0600
to 0xFFFF, and so it is a somewhat limited resource; however, there exists a
special Extended EtherType (0x88B7) that can be suffixed by an Organizationally
Unique Identifier (OUI) followed by a further 16-bits identifying the protocol
relative to that OUI as discussed in Section 3 of {{RFC9542}}. These
alternatives of a direct EtherType or use of the Extended EtherType for the
case of the IANA OUI are illustrated in Figure 2. The following subsections
discuss the factors which may influence the choice between these alternatives
when use of such layer 2 protocol identification, to make the isolation of a
limited domain more robust, is warranted.

      01234567 01234567
    +--------+--------+
    |    EtherType    |
    +--------+--------+
    Specific EtherType

      01234567 01234567 01234567 01234567 01234567 01234567 01234567
    +--------+--------+--------+--------+--------+--------+--------+
    |  0x88  |  0xB7  |  0x00  |  0x00  |  0x5E  | Protocol Number |
    +--------+--------+--------+--------+--------+--------+--------+
    Extended EtherType|         IANA OUI         |IANA Protocol Number
{: #fig2 title="EtherType Based Protocol Identification"}

Because specific EtherTypes are a limited resource, an Extended EtherType
SHOULD be used unless there is a strong reason why it will not work
satisfactorily and a specific EtherType is required.

## Extended EtherType Protocol Identification

The main advantage of using an Extended EtherType with an IANA Protocol Number,
as shown in Figure 2, is that such a number can be allocated by IANA with
Expert Review based on an Internet Draft and is thus relatively easy to obtain.
The main disadvantage is that the protocol identification is 5 bytes longer
than a specific dedicated EtherType.

### Hardware Forwarding Considerations

While Extended EtherTypes ({{RFC9542}}) solve namespace exhaustion, protocol
designers should remain mindful of hardware parsing pipelines. High-throughput
switching ASICs are heavily optimized to parse standard 16-bit Layer-2
EtherTypes at line rate within a fixed-depth initial parsing window.

Variable-length or extended EtherType encapsulations (such as 8-octet Extended
EtherTypes) can introduce forwarding trade-offs on certain switching silicon:
* Parser Depth: Some hardware engines cannot parse beyond a fixed offset in a
  single pass, potentially triggering packet recirculation or forcing packets
  to a slower exception path.
* Flow Matching: Access Control Lists (ACLs) and flow-match engines often have
  native 16-bit EtherType primitives. Matching on an OUI plus extended ID may
  consume extra TCAM resources.

## Specific EtherType Protocol Identification

The primary disadvantage of using a specific EtherType, as opposed to an
Extended EtherType, is that assignment of such an EtherType is significantly
more difficult. As discussed in {{RFC9542}}, a specific EtherType can only be
assigned by the IEEE Registration Authority under the following policy: "Since
EtherTypes are a fairly scarce resource, the IEEE RAC has let us know that they
will not assign a new EtherType to a new IETF protocol specification until the
IESG has approved the protocol specification for publication as an RFC."
({{RFC9542}} Appendix B.1, citing {{IESG_EtherType}})

During development and testing, a protocol can use a "Local Experimental
Ethertype" (0x88b5 and 0x88b6 - {{IANA_EtherType}}). Once approved, the IESG
can request an EtherType from the IEEE.

The primary advantage of using a specific EtherType is the saving of 5 bytes
relative to the use of the Extended EtherType.


# Non-Ethernet Encapsulations and Layer-3 Overlays

Many modern data centers, enterprise networks, and WANs deploy limited-domain
features across routed multi-hop IP underlays using Layer-3 encapsulations
(such as Geneve {{RFC8926}}, VXLAN-GPE {{RFC9674}}, GRE {{RFC2784}}, or generic
UDP encapsulation {{RFC8085}}).

When designing limited-domain protocols for Layer-3 overlay environments, the
same fundamental tension between fail-open and fail-closed designs applies:

## The Fragility of IP-Level Filtering in Overlays

Tunneling limited-domain traffic inside standard unicast IP/UDP packets without
domain-specific transport identifiers exposes the traffic to the same leakage
risks as native IP:

1. **Routability by Default**: If an inner packet is encapsulated in a standard
   UDP/IP header destined for an IP address that leaks into the global routing
   table, intermediate underlay routers will forward the encapsulated frame
   across domain boundaries unless explicitly filtered.
2. **Transit Inspection Invisibility**: Intermediate nodes that inspect only
   the outer IP/UDP headers cannot distinguish between legitimate transit
   traffic and sensitive limited-domain control payloads.


# Alternative Isolation Mechanisms

This layer-2 protocol identifier technique discussed in Section 8 is primarily
effective for transport-type limited-domain protocols running directly at Layer
3. Higher-layer protocols or those running on top of complex routing substrates
may not be easily protectable via Layer-2 EtherTypes.

In such scenarios, alternative isolation methods must be considered. These
include cryptographically enforced containment and automated control-plane
isolation, such as the mechanisms standardized by ANIMA in {{RFC8994}} and
{{RFC8995}}, where domain boundaries are enforced through secure cryptographic
handshakes and mutual trust verification.


# Security Considerations

Protocols are designated as "limited domain" because unexpected security issues
might arise if they leak outside of a unified management domain. For example,
VLAN, VPN, or overlay identifiers may be misinterpreted, resulting in the
delivery of data to (or acceptance of data from) unauthorized network nodes.

Implementing a "fail-closed" boundary via Layer-2 protocol identifiers, strict
link-local address scoping, or Hop-Limit boundaries significantly improves
security by ensuring that unconfigured or external nodes drop the traffic by
default, preventing unintended leakage and misinterpretation.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

Much thanks to Deborah Brungard and Brian Carpenter for their review and
comments.

Also much thanks to everyone else with whom we have discussed this topic; I've
had numerous discussions with many many people on this, and I'm sure that I've
forgotten some of them. Apologies if you were one of them.

# Changelog
{:numbered="false"}
* - 02-08:
  * Title changed to "Fail-closed Limited Domains".
  * Abstract updated to reframe edge filtering as a positive, standard
    practice.
  * Section 3 updated to qualify categorization.
  * Removed Appendix A comparing MPLS/SRv6 to avoid direct protocol-specific
    comparisons.
  * Added Section 7 detailing IPv6 Link-Local Multicast scoping (`ff02::/16`).
  * Moved cryptographic/higher-layer discussion out of Section 8 into a new
    Section 11.
  * Streamlined hardware considerations in Section 9.1.1.
  * Added clarifying language regarding fail-closed design options and boundary
    behaviors.

* - 02-07:
  * Add Security Considerations text about the risks of fail-open protocols and
    the benefits of fail-closed protocols.
  * Added Appendix contrasting MPLS and SRv6.
  * Added hardware processing notes.
    * Used Gemini to help me reformulate mess into something readable.
  * Rewrite Abstract.

* -02-03:
  * Add Security Considerations text about the risks of fail-open protocols and
    the benefits of fail-closed protocols.

* 01-02:
  * Add Donald Eastlake as an author.
  * Substantial re-write and expansion of material concerning specific and
    Extended EtherType protocol identification.
  * Add initial Security Considerations text.

* 00-01:
  * Deborah pointed out that "this only works for transport-type limited domain
    protocols (e.g., SRv6)" could be read as SRv6 fails-closed.

