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

This specification defines a mechanism by which a transmitter of a Security Event Token (SET) [RFC8417] can deliver multiple SETs to an intended SET Recipient via HTTP POST [RFC7231] over TLS in a single POST call. [RFC8935] focuses on the delivery of the single SET to the receiver. This specification builds onto [RFC8935] to transmit multiple SETs to the receiver in a single POST call.

Multi-push SET delivery is intended to help in following scenarios:
   - The transmitter of the SET has multiple outstanding SETs to be communicated to the receiver
   - The transmitter wants to reduce the number of outbound calls to the same receiver to optimize performance, avoid being ratelimited when number of SETs to be communicated is high
   - The receiver wants to optimize processing multiple SETs

Multi-push specification will handle all the usecases and scenarios for the [RFC8935] and make it more extensible to support multiple SETs per one outbound POST call.

Similar to [RFC8935] this specification makes mechanism for exchanging configuration metadata such as endpoint URLs, cryptographic keys, and possible implementation constraints such as buffer size limitations between the transmitter and recipient is out of scope.

# Multi-Push Endpoint 

Each Receiver that supports this specification MUST support a "multi-push" endpoint. This endpoint MUST be capable of serving HTTP [RFC9110] requests. This endpoint MUST be TLS [RFC8446] enabled and MUST reject any communication not using TLS.

The existing PUSH endpoint could also be extended to support accepting multiple SETs


## Transmitting SETs

A Transmitter may initiate communication with the receiver in order to:
   - Send SETs to the Receiver
   - Recive acknowledgement of the SETs in response

The body of this request is of the content type "application/json". It MAY contains the following fields:

sets
OPTIONAL. A JSON object containing key-value pairs in which the key of a field is a string that contains the jti value of the SET that is specified in the value of the field. This field MAY be omitted to indicate that no SETs are being delivered by the initiator in this communication.

moreAvailable
A JSON boolean value that indicates if more unacknowledged SETs are available to be returned. This member MAY be omitted, with the meaning being the same as including it with the boolean value false.


The following is a non-normative example of a Communication Object

```
{
  "sets": {
    "4d3559ec67504aaba65d40b0363faad8":
    "eyJhbGciOiJub25lIn0.
    eyJqdGkiOiI0ZDM1NTllYzY3NTA0YWFiYTY1ZDQwYjAzNjNmYWFkOCIsImlhdC
    I6MTQ1ODQ5NjQwNCwiaXNzIjoiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tIiwi
    YXVkIjpbImh0dHBzOi8vc2NpbS5leGFtcGxlLmNvbS9GZWVkcy85OGQ1MjQ2MW
    ZhNWJiYzg3OTU5M2I3NzU0IiwiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tL0Zl
    ZWRzLzVkNzYwNDUxNmIxZDA4NjQxZDc2NzZlZTciXSwiZXZlbnRzIjp7InVybj
    ppZXRmOnBhcmFtczpzY2ltOmV2ZW50OmNyZWF0ZSI6eyJyZWYiOiJodHRwczov
    L3NjaW0uZXhhbXBsZS5jb20vVXNlcnMvNDRmNjE0MmRmOTZiZDZhYjYxZTc1Mj
    FkOSIsImF0dHJpYnV0ZXMiOlsiaWQiLCJuYW1lIiwidXNlck5hbWUiLCJwYXNz
    d29yZCIsImVtYWlscyJdfX19.",
    "3d0c3cf797584bd193bd0fb1bd4e7d30":
    "eyJhbGciOiJub25lIn0.
    eyJqdGkiOiIzZDBjM2NmNzk3NTg0YmQxOTNiZDBmYjFiZDRlN2QzMCIsImlhdC
    I6MTQ1ODQ5NjAyNSwiaXNzIjoiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tIiwi
    YXVkIjpbImh0dHBzOi8vamh1Yi5leGFtcGxlLmNvbS9GZWVkcy85OGQ1MjQ2MW
    ZhNWJiYzg3OTU5M2I3NzU0IiwiaHR0cHM6Ly9qaHViLmV4YW1wbGUuY29tL0Zl
    ZWRzLzVkNzYwNDUxNmIxZDA4NjQxZDc2NzZlZTciXSwic3ViIjoiaHR0cHM6Ly
    9zY2ltLmV4YW1wbGUuY29tL1VzZXJzLzQ0ZjYxNDJkZjk2YmQ2YWI2MWU3NTIx
    ZDkiLCJldmVudHMiOnsidXJuOmlldGY6cGFyYW1zOnNjaW06ZXZlbnQ6cGFzc3
    dvcmRSZXNldCI6eyJpZCI6IjQ0ZjYxNDJkZjk2YmQ2YWI2MWU3NTIxZDkifSwi
    aHR0cHM6Ly9leGFtcGxlLmNvbS9zY2ltL2V2ZW50L3Bhc3N3b3JkUmVzZXRFeH
    QiOnsicmVzZXRBdHRlbXB0cyI6NX19fQ."
  }
}
```
Figure 1: Example of SET Transmission

In the above example, the Transmitter is sending 2 SETs to the Receiver.

```
{
  "sets": {
    "4d3559ec67504aaba65d40b0363faad8":
    "eyJhbGciOiJub25lIn0.
    eyJqdGkiOiI0ZDM1NTllYzY3NTA0YWFiYTY1ZDQwYjAzNjNmYWFkOCIsImlhdC
    I6MTQ1ODQ5NjQwNCwiaXNzIjoiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tIiwi
    YXVkIjpbImh0dHBzOi8vc2NpbS5leGFtcGxlLmNvbS9GZWVkcy85OGQ1MjQ2MW
    ZhNWJiYzg3OTU5M2I3NzU0IiwiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tL0Zl
    ZWRzLzVkNzYwNDUxNmIxZDA4NjQxZDc2NzZlZTciXSwiZXZlbnRzIjp7InVybj
    ppZXRmOnBhcmFtczpzY2ltOmV2ZW50OmNyZWF0ZSI6eyJyZWYiOiJodHRwczov
    L3NjaW0uZXhhbXBsZS5jb20vVXNlcnMvNDRmNjE0MmRmOTZiZDZhYjYxZTc1Mj
    FkOSIsImF0dHJpYnV0ZXMiOlsiaWQiLCJuYW1lIiwidXNlck5hbWUiLCJwYXNz
    d29yZCIsImVtYWlscyJdfX19.",
    "3d0c3cf797584bd193bd0fb1bd4e7d30":
    "eyJhbGciOiJub25lIn0.
    eyJqdGkiOiIzZDBjM2NmNzk3NTg0YmQxOTNiZDBmYjFiZDRlN2QzMCIsImlhdC
    I6MTQ1ODQ5NjAyNSwiaXNzIjoiaHR0cHM6Ly9zY2ltLmV4YW1wbGUuY29tIiwi
    YXVkIjpbImh0dHBzOi8vamh1Yi5leGFtcGxlLmNvbS9GZWVkcy85OGQ1MjQ2MW
    ZhNWJiYzg3OTU5M2I3NzU0IiwiaHR0cHM6Ly9qaHViLmV4YW1wbGUuY29tL0Zl
    ZWRzLzVkNzYwNDUxNmIxZDA4NjQxZDc2NzZlZTciXSwic3ViIjoiaHR0cHM6Ly
    9zY2ltLmV4YW1wbGUuY29tL1VzZXJzLzQ0ZjYxNDJkZjk2YmQ2YWI2MWU3NTIx
    ZDkiLCJldmVudHMiOnsidXJuOmlldGY6cGFyYW1zOnNjaW06ZXZlbnQ6cGFzc3
    dvcmRSZXNldCI6eyJpZCI6IjQ0ZjYxNDJkZjk2YmQ2YWI2MWU3NTIxZDkifSwi
    aHR0cHM6Ly9leGFtcGxlLmNvbS9zY2ltL2V2ZW50L3Bhc3N3b3JkUmVzZXRFeH
    QiOnsicmVzZXRBdHRlbXB0cyI6NX19fQ."
  },
  "moreAvailable": 10

}
```
Figure 2: Example of SET Transmission with "moreAvailable"

In the above example, the Transmitter is sending 2 SETs to the Receiver. The Tranmitter is also communicating to the reciver the outstanding SETs to be transmitted.


```
{
  "sets": {},
}
```
Figure 2: Example of empty SET transmission

In the above example, the Transmitter is sending zero SETs to the Receiver. This placeholder/empty request provides the Receiver to respond back with ack/err for previously transmitted SETs

## Response Communication

A Receiver MUST repond to the communication by sending an HTTP response. The body of this response is of the content type "application/json". It contains MAY contain following fields:

ack
OPTIONAL. An array of strings, in which each string is the jti value of a previously received SET that is acknowledged in this object. This array MAY be empty or this field MAY be omitted to indicate that no previously received SETs are being acknowledged in this communication.

setErrs
OPTIONAL. A JSON object containing key-value pairs in which the key of a field is a string that contains the jti value of a previously received SET that the sender of the communication object was unable to process. The value of the field is a JSON object that has the following fields:

err
OPTIONAL. The short reason why the specified SET failed to be processed.

description
OPTIONAL. An explanation of why the SET failed to be processed

### Success Response

If the Receiver is successful in processing the request, it MUST return the HTTP status code 200 (OK). The response MUST have the content-type "application/json" and the response MUST include a Communication Object Section X.

```
HTTP/1.1 200 OK
Content-type: application/json

{
  "ack": [
    "3d0c3cf797584bd193bd0fb1bd4e7d30"
  ]
}
```
Figure 3: Example of SET Transmission response with ack

In the above example, the Receiver acknowledges one of the SETs it previously received. There are no errors reported by the Receiver.

```
 {
  "ack": [
    "f52901c4-3996-11ef-9454-0242ac120002",
    "0636e274-3997-11ef-9454-0242ac120002",
    "d563c724-79a0-4ff0-ba41-657fa5e2cb11"
  ],
  "setErrs": {
    "5c436b19-0958-4367-b408-2dd542606d3b" : {
      "err": "invalid subject",
      "description": "subject format not supported"
    }
  }
 }
```
Figure 3: Example of SET Transmission response, with ack and errors

In the above example, the Receiver acknowledges three of the SETs it previously received. There are errors reported by the Receiver for acklowledging one SET.

### Error Response

The Responder MUST respond with an error response if it is unable to process the request. The error response MUST include the appropriate error code as described in Section 2.4 of DeliveryPush [RFC8935].

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
