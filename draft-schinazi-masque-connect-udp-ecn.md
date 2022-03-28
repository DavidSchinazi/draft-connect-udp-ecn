---
title: An ECN Extension to CONNECT-UDP
abbrev: CONNECT-UDP ECN Extension
docname: draft-schinazi-masque-connect-udp-ecn-latest
submissiontype: IETF
category: std
wg: MASQUE

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

CONNECT-UDP allows proxying UDP packets over HTTP. This document describes an
extension to CONNECT-UDP that allows conveying ECN information on proxied UDP
packets.


--- middle

# Introduction {#intro}

CONNECT-UDP {{!CONNECT-UDP=I-D.ietf-masque-connect-udp}} allows proxying UDP
packets over HTTP. This document describes an extension to CONNECT-UDP that
allows conveying ECN {{!ECN=RFC3168}} information on proxied UDP packets.


## Conventions and Definitions {#defs}

{::boilerplate bcp14-tagged}


# Context Identifiers

The "Context Identifiers" section of {{CONNECT-UDP}} defines the concept of
context IDs and how they can be used to extend CONNECT-UDP. When a client wishes
to use ECN with CONNECT-UDP, it generates an unique client-allocated context ID.
In this document, we'll refer to that context ID as the "chosen context ID".
Note that, by definition of client-allocated context IDs, the chosen context ID
will always be a non-zero even number. We also add the restriction that the
chosen context ID MUST be strictly less than 10<sup>15</sup>. If the client has
run out of available context ID values that match this requirement for this
CONNECT-UDP request, it MUST NOT use the ECN extension with this CONNECT-UDP
request.


# ECN Header Definition {#header}

The "ECN" header field is an Item Structured Field, see {{Section 3.3 of
!STRUCT-FIELD=RFC8941}}; its value MUST be a Integer; any other value type MUST
be handled as if the field were not present by recipients (for example, if this
field is included multiple times, its type will become a List and the field will
therefore be ignored). This document does not define any parameters for the
"ECN" header field value, but future documents might define parameters.
Receivers MUST ignore unknown parameters.

When present, the "ECN" header indicates that the sender supports this
extension, and communicates the chosen context ID as the "ECN" field value.

For example, if the client chosen context ID is 42, it would send the following:

~~~
ECN: 42; foo=bar
~~~
{: #example-header-client title="Example Client ECN Field"}

Clients MUST NOT indicate support for this extension unless they know that the
protocol running over UDP that is being proxied supports ECN, and will react
appropriately to Congestion Experienced (CE) markings.

Proxies MUST NOT indicate support for this extension unless they know they
have the ability to read and write the IP ECN bits on its target-bound UDP
sockets.

This extension is said to have been negotiated when both client and proxy
indicated support for it in their CONNECT-UDP request and response. When
indicating support for this extension, the proxy send the client's chosen
context ID as the "ECN" field value.

For example, the proxy could reply with:

~~~
ECN: 42
~~~
{: #example-header-proxy title="Example Proxy ECN Field"}


# Encoding of ECN bits

When an HTTP Datagram {{!HTTP-DGRAM=I-D.ietf-masque-h3-datagram}} associated
with a CONNECT-UDP stream uses the chosen context ID as its context ID, its
"Payload" field contains the following format (using the notation from the
"Notational Conventions" section of {{!QUIC=RFC9000}}):

~~~
CONNECT-UDP Payload with ECN {
  Must be Zero (6),
  ECN Bits (2),
  UDP Payload (..),
}
~~~
{: #payload-format title="CONNECT-UDP Payload with ECN"}

Must be Zero:

: 6 bits that MUST be sent as zero. Receivers MUST validate that these bits are
zero and MUST silently drop the HTTP Datagram if they have any other value.
Extensions to this mechanism MAY relax this requirement.

ECN Bits:

: The ECN bits, sent in the same order as they appear in the IP header.

UDP Payload:

: The UDP Payload, as defined in the "HTTP Datagram Payload Format" section of
{{CONNECT-UDP}}.

When the proxy receives a datagram with the chosen context ID, it sets the IP
packet's ECN bits accordingly on the UDP packet it sends to the target.
Similarly, in the other direction the ECN Bits field represents which ECN bits
were seen on the UDP packets received from the target.


# A Note about Future Extensions {#note}

This CONNECT-UDP extension uses an HTTP field to register its chosen context ID.
Future extensions to CONNECT-UDP can use the same strategy to register their
chosen context ID(s) via another HTTP field. This strategy is best for
CONNECT-UDP extensions that only need to register context IDs during the HTTP
request and response.

Some extensions may need to register context IDs after the request and response
have been exchanged, for example an extension that wishes to compress QUIC
connection IDs {{QUIC}} is not aware of all connection IDs at request time. In
such cases, extensions can use new Capsule Types (see {{HTTP-DGRAM}}) to perform
context ID registration.


# Security Considerations {#security}

This document does not have additional security considerations beyond those
defined in {{CONNECT-UDP}}.


# IANA Considerations {#iana}

This document will request IANA to register the following entry in the "HTTP
Field Name" registry:

Field Name:

: ECN

Template:

: None

Status:

: provisional (permanent if this document is approved)

Reference:

: This document

Comments:

: None


--- back

# Acknowledgments {#acks}
{:numbered="false"}

This proposal was inspired directly or indirectly by prior work from many
people. The author would like to thank contributors the MASQUE working group.
