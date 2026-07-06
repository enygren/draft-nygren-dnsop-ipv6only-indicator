---
title: Indicating IPv6-only SVCB Service Endpoints and IPv4 Deprecation in the DNS
abbrev: ipv6only and deprecated SvcParams
docname: draft-nygren-dnsop-ipv6only-indicator-latest
date: {DATE}
category: std
updates: 9460

ipr: trust200902
area: General
workgroup: DNSOP Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: E. Nygren
    name: Erik Nygren
    org: Akamai Technologies
    email: erik+ietf@nygren.org

normative:

    RFC9460:

informative:

  IANA-SVCB:
    title: "Service Binding (SVCB) Parameter Registry"
    author:
      -
        org: IANA
    date: false
    target: https://www.iana.org/assignments/dns-svcb/dns-svcb.xhtml

  KONECIPV4:
    title: "Czech Republic sets IPv4 end date"
    author:
      -
        org: CZ.NIC
    date: 2024-01
    target: https://endofipv4.cz/en/

--- abstract

As the DNS is the primary mechanism for translating from hostnames to IP addresses, it is a logical place to signal that endpoints are IPv6-only. It is thus also a logical place to signal that legacy endpoints supporting IPv4 are being deprecated.  This specification introduces two SvcParams for SVCB-compatible RR types that signal IPv6-only endpoints (`ipv6only`) as well as deprecated endpoints (`deprecated`).


TO BE REMOVED: This document is being collaborated on in Github at:
[https://github.com/enygren/draft-nygren-dnsop-ipv6only-indicator](https://github.com/enygren/draft-nygren-dnsop-ipv6only-indicator).
The most recent working version of the document, open issues, etc. should all be
available there.  The authors (gratefully) accept pull requests.

--- middle

# Introduction

This specification introduces the `ipv6only` and `deprecated` SvcParamKeys to allow service providers to indicate that preferred Service Endpoints are IPv6-only and that legacy IPv4-supporting Service Endpoints are deprecated and will be tetired.

As part of the multi-decade transition from IPv4 to IPv6 ({{?RFC8200}}) there is a desire to phase out support for providing services over IPv4. This involves switching from the long-standing assumption that all services are available over IPv4 or dual-stack IPv4+IPv6 to instead being _only_ available over IPv6. As clients lacking IPv6 connectivity will not be able to access IPv6-only resources, there will be a transitional phase where service providers will want to indicate that legacy endpoints supporting IPv4 are deprecated and will go away at some point.

For example, the government of Czechia has set an end-date after which government services may only be provided over IPv6 {{KONECIPV4}}. Leading up to this, there will be a need to signal that the IPv6-only services are preferred and that legacy IPv4-supporting services are deprecated.

In {{!SVCB=RFC9460}}, the SVCB ("Service Binding") and HTTPS DNS RR types are specified to provide clients with complete instructions for accessing a service. Individual service bindings (SVCB RRs) describe Service Endpoints and their properties and relative priorities.

By adding the `ipv6only` and `deprecated` SvcParamKeys, individual Service Endpoints can be annotated to indicate their IPv6-only and Deprecated nature to clients. It is expected that the `deprecated` SvcParamKey may be also be used for other future purposes.

As an example, the following provides two Service Endpoints for `www.example.com`. The preferred endpoint of `modern.example.com` (the one with the lowest SvcPriority) is listed as `ipv6only`. An additional Service Endpoint of `legacy.example.com` is also available but is listed as `deprecated`. The optional use of `mandatory=ipv6only` will cause clients not understanding the new `ipv6only` SvcParamKey to ignore the modern IPv6-only endpoint and skip directly to the legacy endpoint.

~~~ dns
    www.example.com. 300 IN HTTPS 1 modern.example.com. (
        ipv6hint=2001:db8::6 ipv6only mandatory=ipv6only )
                            HTTPS 2 legacy.example.net. (
        ipv6hint=2001:db8::64 ipv4hint=203.0.113.4 deprecated )
~~~


# Conventions and Definitions

{::boilerplate bcp14}

Terminology used in this specification includes:

* `Service Endpoint`: a ServiceMode SVCB (or SVCB-compatible) DNS RR ({{!SVCB=RFC9460}}) that specifies how to access a service.

Additional DNS terminology intends to be consistent with {{?DNSTerm=RFC9499}}.


# The "ipv6only" SvcParamKey {#ipv6only}

The `ipv6only` SvcParamKey indicates that the associated Service Endpoint is only available over IPv6. It takes no value.

With `ipv6only`, the presentation and wire format values MUST be
empty.

To be "self-consistent" (see {{SVCB}} Section 2.4.3), the `ipv6only` and `ipv4hint` SvcParamKeys SHOULD NOT be included in the same SVCB RR. When `ipv6only` is present, the `ipv4hint` SvcParam MUST be ignored by clients.

## Client behavior for "ipv6only" {#ipv6only-client-behavior}

Clients implementing the `ipv6only` SvcParamKey do the following when encountering a SVCB RR with this SvcParam, as a modification to the algorithm specified in {{SVCB}} Section 3:

1. Clients SHOULD NOT perform a `A` DNS lookup for an IPv4 address for the TargetName. (It is possible that clients already performed the `A` lookup or have one in their DNS cache, in which case it MUST be ignored.)
2. Clients MUST only attempt to connect to the Service Endpoint over IPv6 and MUST NOT attempt to connect to the Service Endpoint over IPv4.
3. Clients who are certain that they have no IPv6 connectivity SHOULD treat this Service Endpoint as "not compatible" ({{SVCB}} Section 3) and not include it in their candidate list.

If the only Service Endpoints available have `ipv6only`, and if the service has no DNS `A` record, the Client MAY present the user with a notice that the service is only available over IPv6 and that the user lacks IPv6 connectivity.

## Clients using a Proxy

Clients using a domain-oriented transport proxy like HTTP CONNECT
({{!RFC9110}}) or SOCKS5 ({{!RFC1928}}) with named destinations may not know if their proxy supports IPv6 or only IPv4.

Clients lacking information about whether a Proxy supports IPv6 SHOULD opportunistically use Service Endpoints with `ipv6only`, but MUST retry with subsequent Service Endpoints if this fails. Clients MAY use the {{?RFC9209}} `Proxy-Status` response header field to get an indication that a proxy is unable to reach a given target over IPv6 (such as looking for regular occurrances of `error=destination_ip_unroutable`) but MUST NOT extrapolate this to general IPv6 unreachability from the Proxy absent some other explicit signal.


# The "deprecated" SvcParamKey {#deprecated}

The `deprecated` SvcParamKey indicates that the associated Service Endpoint is deprecated. It takes an optional value with a freeform textual deprecation reason.

This text is NOT intended for automated processing, and clients MUST NOT alter their connection behavior on the basis of its content beyond treating the mere presence of the "deprecated" key as advisory.

The `deprecated` SvcParam SHOULD always be on the lowest priority Service Endpoints.

## Presentation Format for "deprecated"

The presentation value of "deprecated" is OPTIONAL. When present, it follows the generic SvcParamValue presentation format defined in {{Section 2.1 of RFC9460}}.

If no value is given, the key appears with no reason text, e.g.:

~~~ dns
   www.example.com. 3600 IN HTTPS 2 legacy.example.com. deprecated
~~~

If a reason is given:

~~~ dns
   www.example.com. 3600 IN HTTPS 2 legacy.example.com. (
      deprecated="IPv4 support to be removed prior to 2032-07-06" )
~~~

## Wire Format for "deprecated"

In wire format, the SvcParamValue for "deprecated" consists of the UTF-8 {{!RFC3629}} encoding of the freeform reason text, with no internal length, count, or type octets. If no reason text is present, the SvcParamValue is zero-length: the SvcParamKey appears with a SvcParamValue length of 0, per the general wire format described in {{Section 2.2 of RFC9460}}.

Implementations MUST NOT assume any particular character encoding beyond UTF-8, MUST NOT parse the value for embedded structure, and SHOULD treat malformed UTF-8 by ignoring the value while still honoring the mere presence of the key as an indication of deprecation.

## Client behavior for "deprecated" {#deprecated-client-behavior}

The `deprecated` indicator is intended for operational uses and is not intended to substantively impact general end-user behavior.

Clients SHOULD NOT provide special treatment to Service Endpoints due to the presence of `deprecated`.

Clients MAY provide an indicator and SHOULD log a warning when using a Service Endpoint with `deprecated`.


# Operational Usage

A typical operational workflow for using this will involve moving through a deprecation process of:

1) Introducing both the ipv6only Service Endpoint and marking the legacy Service Endpoint as deprecated.
2) Removing the deprecated endpoint, leaving only the `ipv6only` Service Endpoint.

## Step One: Indicating Deprecation

In the first step, the service is configured with at least two ServiceMode Service Endpoints:

* The higher priority Service Endpoint (lesser numeric SvcPriority) MUST have `ipv6only` and `mandatory=ipv6only` SvcParams. Its TargetName MUST only have DNS `AAAA` records and no DNS `A` records. By marking the highest priority Service Endpoint with `ipv6only` as `mandatory=ipv6only` we cause clients who do not understand the `ipv6only` SvcParamKey to ignore that ServiceMode record and move on to the next one.
* The the lower priority Service Endpoint (greater numeric SvcPriority) SHOULD have a `deprecated` SvcParams with an optional description. Its TargetName MUST be dual-stacked with both DNS `AAAA` records and `A` records.
* The fallback hostname MUST be be dual-stacked with both DNS `AAAA` records and `A` records.

For example:

~~~ dns
    www.example.com. 300 IN HTTPS 1 modern.example.com. (
        ipv6hint=2001:db8::6 ipv6only mandatory=ipv6only )
                            HTTPS 2 legacy.example.net. (
        ipv6hint=2001:db8::64 ipv4hint=203.0.113.4
        deprecated="IPv4 support to be removed prior to 2032-07-06" )
    www.example.com.    300 IN AAAA 2001:db8::64
    www.example.com.    300 IN A    203.0.113.4
    modern.example.com. 300 IN AAAA 2001:db8::6
    legacy.example.com. 300 IN CNAME www.example.com.
~~~

The `www.example.com` endpoint remains as dual-stacked with A and AAAA records for clients not understanding the `ipv6only` SvcParamKey.


## Step Two: Removing Deprecated Service Endpoint

In the second step we remove the dual-stack service endpoint (and also make the fallback name dual-stack):

* The remaining Service Endpoint MUST have a `ipv6only` SvcParams. It MUST NOT have the `mandatory=ipv6only` SvcParam, as this would cause clients not implementing the `ipv6only` SvcParam to have a Service Endpoint to use, even if they support IPv6. Its TargetName MUST only have DNS `AAAA` records and no DNS `A` records.
* The fallback hostname MUST be be IPv6-only with only DNS `AAAA` records and no `A` records.

For example, to complete deprecation we remove the deprecated Service Endpoint:

~~~ dns
    www.example.com. 300 IN HTTPS 1 . ( ipv6hint=2001:db8::6 ipv6only )
    www.example.com.    300 IN AAAA 2001:db8::6
~~~

At this point the service no longer has any IPv4 support.


# Security Considerations

Communications in the DNS are subject to inspection and modification unless DNSSEC and secure communication from the client to DNS resolver is used.

How this impacts this specification depends on the threat model for the environment, but for the `ipv6only` SvcParam an attacker who can exploit these could also likely block individual IPv4 or IPv6 flows and force fallback without this specification.

The free-form text provided with `deprecated` should be assumed to be coming from a hostile source. Clients MUST NOT present it to users in a way that it could lead to confusion or be used for phishing or other attacks. Any logging of this provided text MUST assume that it contains potential cross-site-scripting or prompt injection or other malicious content and must escape and annotate it accordingly.

# Privacy Considerations

TBD


# IANA Considerations

## New registry for Service Parameters {#svcparamregistry}


### Updates to IANA SvcParam Registry {#iana-keys}

The "Service Binding (SVCB) Parameter Registry" ({{IANA-SVCB}}) shall have the following SvcParam Keys added:

| Number      | Name            | Meaning                                         | Format Reference               |
| ----------- | ------          | ----------------------                          | ------------------------------ |
| TBA         | ipv6only        | The endpoint is only available over IPv6        | (This document) {{ipv6only}}   |
| TBA         | deprecated      | The endpoint is deprecated and may go away soon | (This document) {{deprecated}} |


_(TO BE REMOVED but potential considerations for assignment:
* A cute assignment for ipv6only might be "864", with the meaning of "getting rid of IPv4 due to running out".
* Similarly, "86" might be a memorable assignment for "deprecated"?)_

--- back

# Acknowledgments

Thank you to ... for their feedback and suggestions on this draft.

Some of the initial thoughts around this go back to the IETF sunset4 working group, so additional thanks go to people who participated in that work, as well as to others working towards the eventual universal deprecation of IPv4.

# Change history

(This section to be removed by the RFC editor.)

