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
using named elements on the "Datagram-Flow-Id" header. These names are
"ecn-ect0", "ecn-ect1", and "ecn-ce". For example:

~~~
  Datagram-Flow-Id = 42, 44; ecn-ect0, 46; ecn-ect1, 48; ecn-ce
~~~

If the proxy wishes to support datagram encoding of this extension, it echoes
those named elements in its CONNECT-UDP response. The unnamed element now
represents Not-ECT, whereas the one in "ecn-ect0" represents ECT(0), "ecn-ect1"
represents ECT(1) and "ecn-ce" represents CE; see Section 5 of {{ECN}} for the
definition of these IP header fields.

When the proxy receives a datagram from the given flow identifier, it sets
the IP packet's ECN bits accordingly on the UDP packet it sends to the target.
Similarly, in the other direction the flow identifier represents which ECN bits
were seen on the UDP packets received from the target.


# Stream Encoding of Proxied UDP Packets {#stream-encoding}

If HTTP/3 datagrams are not supported, the stream is used to convey UDP
payloads, and the CONNECT-UDP Stream Chunk Type is used to indicate the values
of the ECN bits, as defined below:

~~~
  +-------+-----------------+-----------+
  | Value |      Type       | ECN Field |
  +-------+-----------------+-----------+
  | 0x00  | UDP_PACKET      |  Not-ECT  |
  +-------+-----------------+-----------+
  | 0x31  | UDP_PACKET_ECT0 |  ECT(0)   |
  +-------+-----------------+-----------+
  | 0x32  | UDP_PACKET_ECT1 |  ECT(1)   |
  +-------+-----------------+-----------+
  | 0x33  | UDP_PACKET_CE   |  CE       |
  +-------+-----------------+-----------+
~~~

The proxy then uses the the CONNECT-UDP Stream Chunk Type on received UDP
payloads to set the ECN bits on the IP packets it sends to the target, and
in the reverse direction to indicate which ECN bits received from the target.


# HTTP Intermediaries {#intermediaries}

HTTP/3 DATAGRAM flow identifiers are specific to a given HTTP/3 connection.
However, in some cases, an HTTP request may travel across multiple HTTP
connections if there are HTTP intermediaries involved; see Section 2.3 of
{{!RFC7230}}.

Intermediaries that support this extension and HTTP/3 datagrams MUST negotiate
all four flow identifiers separately on the client-facing and server-facing
connections. This is accomplished by having the intermediary parse the unnamed
element and the "ecn-ect0", "ecn-ect1", and "ecn-ce" named elements in the
"Datagram-Flow-Id" header on all CONNECT-UDP requests it receives, and sending
the same four flow identifiers in the "Datagram-Flow-Id" header on the response.

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


## Flow Identifier Parameters {#iana-params}

This document will request IANA to register the "ecn-ect0", "ecn-ect1", and
"ecn-ce" flow identifier parameters in the "HTTP Datagram Flow Identifier
Parameters" registry (see {{H3DGRAM}}):

~~~
  +----------+-------------------------+---------+---------------+
  |   Key    |       Description       | Is Name |   Reference   |
  +----------+-------------------------+---------+---------------+
  | ecn-ect0 | UDP payload with ECT(0) |   Yes   | This document |
  +----------+-------------------------+---------+---------------+
  | ecn-ect1 | UDP payload with ECT(1) |   Yes   | This document |
  +----------+-------------------------+---------+---------------+
  | ecn-ce   | UDP payload with CE     |   Yes   | This document |
  +----------+-------------------------+---------+---------------+
~~~


## Stream Chunk Type Registration {#iana-chunk-type}

This document will request IANA to register the following entry in the
"CONNECT-UDP Stream Chunk Type" registry {{CONNECT-UDP}}:

~~~
  +-------+-----------------+-------------------------+---------------+
  | Value |      Type       |       Description       |   Reference   |
  +-------+-----------------+-------------------------+---------------+
  | 0x31  | UDP_PACKET_ECT0 | UDP payload with ECT(0) | This document |
  +-------+-----------------+-------------------------+---------------+
  | 0x32  | UDP_PACKET_ECT1 | UDP payload with ECT(1) | This document |
  +-------+-----------------+-------------------------+---------------+
  | 0x33  | UDP_PACKET_CE   | UDP payload with CE     | This document |
  +-------+-----------------+-------------------------+---------------+
~~~


--- back

# Acknowledgments {#acks}
{:numbered="false"}

This proposal was inspired directly or indirectly by prior work from many
people. The author would like to thank contributors the MASQUE working group.
