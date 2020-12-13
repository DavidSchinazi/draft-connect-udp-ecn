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


# Security Considerations {#security}

This document does not have additional security considerations beyond those
defined in {{CONNECT-UDP}}.


# IANA Considerations {#iana}

This document does not contain any requests to IANA.


--- back

# Acknowledgments {#acks}
{:numbered="false"}

This proposal was inspired directly or indirectly by prior work from many
people. The author would like to thank contributors the MASQUE working group.
