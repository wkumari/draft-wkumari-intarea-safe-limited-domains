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

informative:


--- abstract

There is a noticeable trend towards network behaviors and semantics that are
specific to a particular set of requirements applied within a limited region
of the Internet (termed a "limited domain"). This trend is reflected in
drafts which describe protocols that are only intended to be used within
these "limited domains". Unfortunately, these drafts often do not
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

TODO Introduction


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
