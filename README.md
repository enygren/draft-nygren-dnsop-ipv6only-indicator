<!-- regenerate: off (set to off if you edit this file) -->

# Indicating IPv6-only SVCB Endpoints and IPv4 Deprecation in the DNS

This is the working area for the individual Internet-Draft, "Indicating IPv6-only SVCB Endpoints and IPv4 Deprecation in the DNS".

As the DNS is the primary mechanism for translating from hostnames to IP addresses, it is a logical place to signal that endpoints are IPv6-only. It is thus also a logical place to signal that legacy endpoints supporting IPv4 are being deprecated.  This specification introduces two SvcParams for SVCB-compatible RR types that signal IPv6-only endpoints (`ipv6only`) as well as deprecated endpoints (`deprecated`).

* [Editor's Copy](https://enygren.github.io/draft-nygren-dnsop-ipv6only-indicator/#go.draft-nygren-dnsop-ipv6only-indicator.html)
* [Datatracker Page](https://datatracker.ietf.org/doc/draft-nygren-dnsop-ipv6only-indicator)
* [Individual Draft](https://datatracker.ietf.org/doc/html/draft-nygren-dnsop-ipv6only-indicator)
* [Compare Editor's Copy to Individual Draft](https://enygren.github.io/draft-nygren-dnsop-ipv6only-indicator/#go.draft-nygren-dnsop-ipv6only-indicator.diff)


## Contributing

See the
[guidelines for contributions](https://github.com/enygren/draft-nygren-dnsop-ipv6only-indicator/blob/main/CONTRIBUTING.md).

The contributing file also has tips on how to make contributions, if you
don't already know how to do that.

## Command Line Usage

Formatted text and HTML versions of the draft can be built using `make`.

```sh
$ make
```

Command line usage requires that you have the necessary software installed.  See
[the instructions](https://github.com/martinthomson/i-d-template/blob/main/doc/SETUP.md).

