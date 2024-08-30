---
title: "Multi-Push-Based Security Event Token (SET) Delivery Using HTTP"
abbrev: "Multi-Push SET"
category: std

docname: draft-deshpande-secevent-http-multi-push-latest
submissiontype: IETF
number:
date:
v: 3
area: "Security"
workgroup: "Security Events"
keyword:
 - security event
 - secevent
venue:
  group: "Security Events"
  type: "Working Group"
  mail: "id-event@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/id-event/"
  github: "aaronpk/draft-deshpande-secevent-http-multi-push"
  latest: "https://aaronpk.github.io/draft-deshpande-secevent-http-multi-push/draft-deshpande-secevent-http-multi-push.html"

author:
 -
    fullname: Apoorva Deshpande
    organization: Okta
    email: apoorva.deshpande@okta.com
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com

normative:

informative:


--- abstract

This specification defines how multiple Security Event Tokens (SETs) can be
delivered to an intended recipient using HTTP POST over TLS.  The SETs
are transmitted in the body of an HTTP POST request to an endpoint
operated by the recipient, and the recipient indicates successful or
failed transmission via the HTTP response.

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
