---
title: "Safe(r) Limited Domains"
abbrev: "safer-limited-domains"
docname: draft-wkumari-intarea-safe-limited-domains-latest
category: std

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

normative:
  RFC8799:
informative:
  RFC8754:

--- abstract

There is a trend towards documents describing protocols that are only intended to be used within "limited domains". Unfortunately, these drafts often do not
clearly define how the boundary of the limited domain is established and
enforced, or require that operators of these limited domains //perfectly//
implement filters to protect the rest of the Internet from these protocols.

In addition, these protocols sometimes require that networks that are outside
of (and unaffiliated with) the limited domain explicitly implement filters in
order to protect their networks if these protocols leak outside of the limited
domain.

This document discusses the concepts of "fail-open" versus "fail-closed"
protocols and limited domains, and provides a mechanism for designing limited
domain protocols that are safer to deploy.


--- middle

# Introduction

{{RFC8799}} discusses the concept of "limited domains", provides
examples of limited domains, as well as Examples of Limited Domain Solutions, including Service Function Chaining (SFC), Segment Routing, "Creative uses of IPv6 features" (including Extension headers, e.g., for segment routing {{RFC8754}})


In order to provide context, this document will quote extensively from {{RFC8799}}, but it is expected (well, hoped!) that the reader will actually read {{RFC8799}} in its entirety. It's relatively short, and it is a very good read!

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
providing a mechanism so that nodes must be explicitly configured to handle the given limited-domain protocol ("fail-closed"), rather than relying on the network operator to take active steps to protect the boundary ("fail-open").


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
