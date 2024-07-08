---
title: "Safe(r) Limited Domains"
abbrev: "safer-limited-domains"
docname: draft-wkumari-intarea-safe-limited-domains-latest
category: std
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
    organization: Liquid Intelligent Technologies
    email: andrew-ietf@liquid.tech
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

normative:
  RFC8799:
informative:
  RFC8994:
  RFC8995:
  RFC8754:
  RFC3031:
  RFC9378:
  RFC9542:
  I-D.ietf-intarea-rfc7042bis:
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

There is a trend towards documents describing protocols that are only intended
to be used within "limited domains". These documents often do not
clearly define how the boundary of the limited domain is implemented and
enforced, or require that operators of these limited domains //perfectly//
implement filters to protect the rest of the global Internet from these protocols
and vice-versa.

This document discusses the concepts of "fail-open" versus "fail-closed"
protocols and limited domains, and specifies a layer-2 mechanism that
can be used for designing limited
domain protocols that are safer to deploy.

--- middle

# Introduction

{{RFC8799}} discusses the concept of "limited domains", provides examples of
limited domains, as well as Examples of Limited Domain Solutions, including
Service Function Chaining (SFC), Segment Routing, "Creative uses of IPv6
features" (including Extension headers, e.g., for in situ
Operations, Administration, and maintenance {{RFC9378}}).

In order to provide context, this document will quote extensively from
{{RFC8799}}, but it is assumed that the reader will actually
read {{RFC8799}} in its entirety.

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

This document addresses the problem of "leakage" of limited domain protocols by
providing a mechanism so that nodes must be explicitly configured to handle the
given limited-domain protocol ("fail-closed"), rather than relying on the
network operator to take active steps to protect the boundary ("fail-open").

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Fail-open versus Fail-closed

Protocols can be broadly classified as either "fail-open" or "fail-closed".
Fail-closed protocols are those that require explicit interface
or device-wide configuration to enable
them to be accepted or processed when received on an interface.
A classic example of a fail-closed protocol is
MPLS ({{RFC3031}}): In order to allow MPLS to transit an interface, the
operator must enable the MPLS protocol on that interface and on the
device itself. This ensures
that
outside MPLS traffic does not leak in.

Fail-open protocols are those that require explicit configuration in order
to ensure that they do not leak out of a domain, for example, through the
application of filters. An example of a fail-open protocol is SRv6 - in order
to ensure that SRv6 traffic does not leak out of a network, the operator must
explicitly filter this traffic, and, in order to ensure that SRv6 traffic does
not leak in, the operator must explicitly filter SRv6 traffic.

Fail-open protocols are inherently more risky than fail-closed protocols, as
they rely on perfect configuration of
filters on all interfaces at the boundary of a domain, and, if the filters are
removed for any reason (for example, during troubleshooting), there
is a risk of inbound or oubound leaks.
In addition, some devices or interfaces may have limitations in the size
and complexity of filters that
can be applied, and so adding new filter entries to limit leaks of a
new protocol may not be possible.

Fail-closed protocols, on the other hand, do not require any explicit
filtering. In order for the protocol to be accepted and processed
when received on an interface, the operator must
explicitly enable the protocol on that interface and on the device itself.
In addition, there is less risk of operational mistakes,
as it does not rely on filters that may be limited in
number and complexity. Finally, fail-closed protocols
do not require that operators of networks outside of the
limited domain implement filters to protect their networks from the limited
domain traffic.

# Making a layer-3 type limited-domain protocol fail-closed

One way to make a limited-domain protocol fail-closed is to assign it a unique
EtherType (this is the mechanism used by MPLS). In modern router and hosts, if
the protocol (and so its associated EtherType) is not enabled on an interface,
then the Ethernet chipset will ignore the frame, and the node OS will not
process it. This is a very simple and effective mechanism to ensure that the
protocol does not leak out of the limited domain if and when an operator makes
a mistake in configuring filters.

Note that this only works for transport-type limited domain protocols (i.e.,
protocols running at layer 3). Higher layer protocols cannot necessarily be
protected in this way, and so cryptographically enforced mechanisms may need to
be used instead (e.g as  done used by ANIMA in {{RFC8994}} and {{RFC8995}}).

The EtherType is a 16-bit field in an Ethernet frame, and so it is a somewhat
limited resource.

Note that "Since EtherTypes are a fairly scarce resource, the IEEE RAC has let
   us know that they will not assign a new EtherType to a new IETF protocol
   specification until the IESG has approved the protocol specification for
   publication as an RFC.  In exceptional cases, the IEEE RA is willing to
   consider "early allocation" of an EtherType for an IETF protocol that is
   still under development as long as the request comes from and has been
   vetted by the IESG." ({{I-D.ietf-intarea-rfc7042bis}} Appendix B.1, citing
   {{IESG_EtherType}})

During development and testing, the protocol can use a "Local Experimental
Ethertype" (0x88b5 and 0x88b6 - {{IANA_EtherType}}). Once the protocol is
approved for publication, the IESG can request an EtherType from the IEEE.

For discussion: or simply defining one single EtherType for this testing?
I.e., IPv4 and IPv6 can be identified by their first 4 bits.

[ Editor note: EtherTypes are a scarce resource, and so we need to be
careful about how we use them. It is likely that there will only
be a very limited (sorry!) number of protocols that need to be protected in
this way (on the order of 3 or 4).

However, it is worth considering if there are other ways to achieve the same
goal. An option would be to set aside a single EtherType, and then have a
registry of "limited sub-EtherTypes" that are assigned by IANA. This would
allow us to protect a large number of protocols, while only using a single
EtherType, but would require something that looks more like a new L2 header. ]


Another option to make a limited-domain protocol fail-closed is to use an
identifier under the IANA OUI (00-00-5E) as explained in {{RFC9542}}.

[Editor note: This is a good idea, but it is not clear if it is practical. This
above would need a bunch more text / discussion. It ends up being a bit like
the "limited sub-EtherTypes" idea above, but with the additional complexity of
not having the MAC address be the "normal" MAC address of the device that you
are sending the traffic to. This will require more discussion with the WG, and
the IEEE liaisons. ]


# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

Much thanks to Brian Carpenter, for his review and comments.

Also much thanks to Deborah Brungard, for her review and comments.

Also much thanks to everyone else with whom we have discussed this topic; I've
had numerous discussions with many many people on this, and I'm sure that I've
forgotten some of them. Apologies if you were one of them.

# Changelog
{:numbered="false"}
* 00-01:
  * Deborah pointed out that "this only works for transport-type limited domain
    protocols (e.g., SRv6)" could be read as SRv6 fails-closed.
