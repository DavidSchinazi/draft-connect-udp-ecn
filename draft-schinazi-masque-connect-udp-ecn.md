---
title: An ECN Extension to CONNECT-UDP
abbrev: CONNECT-UDP ECN Extension
docname: draft-schinazi-masque-connect-udp-ecn-latest
category: std

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: "D. Schinazi"
    name: "David Schinazi"
    organization: "Google LLC"
    street: "1600 Amphitheatre Parkway"
    city: "Mountain View, California 94043"
    country: "United States of America"
    email: dschinazi.ietf@gmail.com


--- abstract

The CONNECT-UDP method allows proxying UDP packets over HTTP. This document
describes an extension to CONNECT-UDP that allows conveying ECN information on
proxied UDP packets.

Discussion of this work is encouraged to happen on the MASQUE IETF mailing list
([masque@ietf.org](mailto:masque@ietf.org)) or on the GitHub repository which
contains the draft: [](https://github.com/DavidSchinazi/draft-connect-udp-ecn).


--- middle

# Introduction {#intro}

The CONNECT-UDP {{!CONNECT-UDP=I-D.ietf-masque-connect-udp}} method allows
proxying UDP packets over HTTP. This document describes an extension to
CONNECT-UDP that allows conveying ECN {{!ECN=RFC3168}} information on proxied
UDP packets.

Discussion of this work is encouraged to happen on the MASQUE IETF mailing list
([masque@ietf.org](mailto:masque@ietf.org)) or on the GitHub repository which
contains the draft: [](https://github.com/DavidSchinazi/draft-connect-udp-ecn).


## Conventions and Definitions {#defs}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# ECN Header Definition {#header}

"ECN" is a Item Structured Header
{{!STRUCT-HDR=I-D.ietf-httpbis-header-structure}}. Its value MUST be a Boolean.
Its ABNF is:

~~~
  ECN = sf-boolean
~~~

The "ECN" header indicates whether the sender supports this extension. A value
of 1 indicates support whereas a value of 0 (or the absence of the header)
indicates lack of support.

Clients MUST NOT indicate support for this extension unless they know that the
protocol running over UDP that is being proxied supports ECN, and will react
appropriately to Congestion Experienced (CE) markings.

Proxies MUST NOT indicate support for this extension unless they know they
have the ability to read and write the IP ECN bits on its target-bound UDP
sockets.

This extension is said to have been negotiated when both client and proxy
indicated support for it in their CONNECT-UDP request and response.

# Datagram Encoding of Proxied UDP Packets {#datagram-encoding}

If a client supports this extension and HTTP/3 datagrams
{{!H3DGRAM=I-D.schinazi-quic-h3-datagram}}, it can attempt to use datagrams for
ECN information. This is done by allocating four datagram flow identifiers (as
opposed to one in traditional CONNECT-UDP) and communicating them to the proxy
using REGISTER_DATAGRAM_FLOW_ID HTTP/3 frames. These frames carry the "ecn" key
in their Extension String with values that allow differentiating between ECN
codepoints. The values are "ect0", "ect1", and "ce". For example:

~~~
  ecn=ect0
~~~

When the proxy receives a datagram from the given flow identifier, it sets
the IP packet's ECN bits accordingly on the UDP packet it sends to the target.
Similarly, in the other direction the flow identifier represents which ECN bits
were seen on the UDP packets received from the target.


# Stream Encoding of Proxied UDP Packets {#stream-encoding}

The RELIABLE_DATAGRAM HTTP/3 frame can be used to convey UDP payloads.


# HTTP Intermediaries {#intermediaries}

HTTP/3 DATAGRAM flow identifiers are specific to a given HTTP/3 connection.
However, in some cases, an HTTP request may travel across multiple HTTP
connections if there are HTTP intermediaries involved; see Section 2.3 of
{{!RFC7230}}.

Intermediaries that support this extension and HTTP/3 datagrams MUST negotiate
all four flow identifiers separately on the client-facing and server-facing
connections. This is accomplished by having the intermediary parse the
REGISTER_DATAGRAM_FLOW_ID frames it receives on all CONNECT-UDP streams, and sending
the same four flow identifiers in the REGISTER_DATAGRAM_FLOW_ID in the reverse direction.

Intermediaries MUST NOT send the "ECN" header with a value of 1 to the client
on its response unless it has received that same value in the response it
received from the server.


# Security Considerations {#security}

This document does not have additional security considerations beyond those
defined in {{CONNECT-UDP}}.


# IANA Considerations {#iana}

## HTTP Header {#iana-header}

This document will request IANA to register the "ECN" header in the
"Permanent Message Header Field Names" registry maintained at
<[](https://www.iana.org/assignments/message-headers)>.

~~~
  +-------------------+----------+--------+---------------+
  | Header Field Name | Protocol | Status |   Reference   |
  +-------------------+----------+--------+---------------+
  |        ECN        |   http   |  std   | This document |
  +-------------------+----------+--------+---------------+
~~~


--- back

# Acknowledgments {#acks}
{:numbered="false"}

This proposal was inspired directly or indirectly by prior work from many
people. The author would like to thank contributors the MASQUE working group.
