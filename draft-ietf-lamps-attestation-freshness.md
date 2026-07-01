---
v: 3
docname:  draft-ietf-lamps-attestation-freshness-latest
cat: std
ipr: trust200902
consensus: 'true'
submissiontype: IETF
lang: en
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: -o*+
  docmapping: yes
title: Requesting a Freshness Nonce for Attestation Evidence in Certificate Signing Requests
abbrev: Freshness for Evidence in CSRs
area: sec
wg: LAMPS Working Group
keyword:
- Remote Attestation
- Certificate Signing Request
- Certificate Management Protocol (CMP)
- Enrollment over Secure Transport (EST)
- Certificate Management over CMS (CMC)
date: 2026
github: "wg-lamps/lamps-attestation-freshness"
stand_alone: yes
author:
 -    ins: H. Tschofenig
      name: Hannes Tschofenig
      org: Siemens
      abbrev: Siemens
      street: Werner-von-Siemens-Strasse 1
      code: '80333'
      city: Munich
      country: Germany
      email: hannes.tschofenig@gmx.net
 -    ins: H. Brockhaus
      name: Hendrik Brockhaus
      org: Siemens
      abbrev: Siemens
      street: Werner-von-Siemens-Strasse 1
      code: '80333'
      city: Munich
      country: Germany
      email: hendrik.brockhaus@siemens.com
      uri: https://www.siemens.com
 -    ins: J. Mandel
      name: Joe Mandel
      org: AKAYLA, Inc.
      abbrev: AKAYLA
      email: joe@akayla.com
 -    ins: S. Turner
      name: Sean Turner
      org: sn3rd
      email: sean@sn3rd.com

normative:
  RFC2119:
  I-D.ietf-lamps-csr-attestation:
  I-D.ietf-lamps-rfc5272bis:
  RFC9810:
  RFC7030:
  RFC7252:
  RFC9148:
  RFC8615:
  RFC8610:
  RFC8259:
  RFC4648:
  RFC6838:
  RFC8949:
  RFC9482:
  RFC9811:
  X.680:
    target: https://www.itu.int/rec/T-REC.X.680
    title: >
      Information Technology - Abstract Syntax Notation One (ASN.1):
      Specification of basic notation
    author:
      org: ITU-T
    date: '2021-02'
    seriesinfo:
      ITU-T Recommendation X.680
  X.690:
    target: https://www.itu.int/rec/T-REC.X.690
    title: >
      Information Technology - ASN.1 encoding rules:
      Specification of Basic Encoding Rules (BER),
      Canonical Encoding Rules (CER)
      and Distinguished Encoding Rules (DER)
    author:
      org: ITU-T
    date: '2021-02'
    seriesinfo:
      ITU-T Recommendation X.690
informative:
  I-D.ietf-rats-msg-wrap:
  I-D.richardson-rats-composite-attesters:
  I-D.ietf-rats-reference-interaction-models:
  RFC2986:
  RFC4211:
  RFC9334:
  RFC9483:
  RFC9684:
  RFC9783:
  RFC9711:

--- abstract

When an end entity includes attestation statements in a Certificate Signing
Request (CSR), the freshness of the conveyed Evidence often needs to be
established. A common mechanism is a nonce that is obtained from a Relying
Party or Verifier and included by the Attester in the Evidence.

This document specifies how an end entity requests such an attestation
freshness nonce from an RA/CA when using certificate lifecycle management protocols.
It defines message formats and protocol bindings for the conveyance of nonce
request and response messages in the Certificate Management Protocol (CMP),
Enrollment over Secure Transport (EST), and Certificate Management over CMS
(CMC), including optional type-specific information needed to produce fresh
Evidence for inclusion in a CSR.

--- middle


# Introduction {#Introduction}

{{I-D.ietf-lamps-csr-attestation}} specifies how Attestation Evidence, as
defined in the RATS Architecture {{RFC9334}}, can be conveyed in a Certificate
Signing Request (CSR) using PKCS#10 {{RFC2986}} or CRMF {{RFC4211}}. The RATS
Architecture calls for establishing the freshness of Evidence, see
{{Section 10 of RFC9334}}. A common method for establishing freshness is the
use of nonces. Such nonces must be provided by the Relying Party or Verifier
and included in the Evidence by the Attester.

When the CSR is conveyed using a certificate lifecycle management protocol, such
as CMP {{RFC9810}}, EST {{RFC7030}}, or CMC {{I-D.ietf-lamps-rfc5272bis}}, the
end entity can request the required nonce from the RA/CA in a prior message
exchange and pass it to the Attester to produce fresh Evidence.

This document describes how an end entity can request a nonce from an RA/CA
using CMP, EST, and CMC.

The following topics are out of scope for this document and either covered
in other specifications or are implementation or deployment details:

- whether a CSR requires one or more nonces,
- how the end entity forwards the nonce to the Attester,
- whether the RA/CA or the Verifier generates the nonce and how those entities
communicate with each other, and
- other methods for establishing freshness.

In this context, the end entity is a device that contains one or more Attesters,
and the RA/CA acts as a Relying Party that communicates with one or more Verifiers.

# Terminology and Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 {{RFC2119}}.

This document uses RATS and PKIX terminology as described in
{{Section 3 of I-D.ietf-lamps-csr-attestation}}. It also uses terminology from
the applicable certificate lifecycle management protocols: CMP, EST, and CMC.

# Architecture and Message Formats {#Architecture}

According to {{I-D.ietf-lamps-csr-attestation}}, a CSR can contain multiple
`AttestationStatements` and multiple certificates for validating those
`AttestationStatements`. This document describes how an end entity can use CMP,
EST, or CMC to request a nonce from the RA/CA for a specific type of Evidence
to be included in the CSR.

The end entity sends a nonce request message with the following fields:

- `len`: In this OPTIONAL field, the end entity can specify the desired length of
  the requested nonce in bytes as a value between 8 and 64.
- `type`: In this OPTIONAL field, the end entity can specify the type of the
  reqInfo structure.
- `reqInfo`: If type is set, this OPTIONAL field MUST contain the type-specific
  content that the RA/CA requires to generate respInfo. If type is not set,
  `reqInfo` MUST also be omitted.

The RA/CA can return the requested nonce in a nonce response message together
with information specific to the generation of the Evidence.

The nonce response message has the following fields:

- `nonce`: This field MUST contain the nonce if the RA/CA is able and willing to
  provide it. If a specific length was requested, the RA/CA SHOULD provide a
  nonce of that size. If the RA/CA doesn't need a freshness proof, the nonce
  MUST be an empty or zero-length string.
- `expiry`: In this OPTIONAL field, the RA/CA can specify the validity period of
  the nonce in seconds as an integer value. The nonce can be used during this
  period; the response therefore needs to be conveyed promptly.
- `type`: In this OPTIONAL field, the RA/CA can specify the type of the `respInfo`
  structure. The type in the nonce response message is defined by the type in
  the nonce request message.
- `respInfo`: If type is set, this OPTIONAL field MUST contain the type-specific
  content requested by the end entity for generating the Evidence. If type is
  not set, `respInfo` MUST also be omitted.

This document does not further specify the content of the `reqInfo` and `respInfo`
structures; those structures must be defined elsewhere. Each definition must
assign an object identifier and specify the exact content of the corresponding
field. The definition of the `reqInfo` structure must also specify the type of the expected
`respInfo` structure. The message structure and the `reqInfo` or `respInfo` structures
can use different encodings. For example, an ASN.1 message can contain `reqInfo` or
`respInfo` encoded as JSON, if necessary. The `reqInfo` structure may be used to
provide the required information to the Relying Party to route the nonce request
to the appropriate Verifier when multiple verifiers are supported. The `respInfo`
structure may be used to convey Generic Information Elements
({{Section 6 of I-D.ietf-rats-reference-interaction-models}}) such as Attesting
Environment IDs and Claim Selection. {{appx-tpm-pcr-example}} provides an
informative example using a TPM PCR selection.

The generic message flow between the end entity and the RA/CA is shown in
{{fig-msgFlow}}.

~~~~ aasvg
end entity                    RA/CA
device with Attester       Relying Party                 Verifier
    |                           |                            |
    |  certificate lifecycle    |                            |
    |    management protocol    |                            |
    |<------------------------->|                            |
    |  request nonce            |                            |
    |-------------------------->|                            |
    |                           |  request nonce (optional)  |
    |                           |--------------------------->|
    |                           |  nonce (optional)          |
    |                           |<---------------------------|
    |  nonce                    |                            |
    |<--------------------------|                            |
    |  attested CSR             |                            |
    |-------------------------->|                            |
    |                           |  Evidence                  |
    |                           |--------------------------->|
    |                           |  Attestation Result        |
    |                           |<---------------------------|
    |  certificate              |                            |
    |<--------------------------|                            |
    |                           |                            |
~~~~
{: #fig-msgFlow title="Message Flow in Background Check Model"}

The nonce request and nonce response messages allow the end entity to request
only one nonce and one `respInfo` structure from the RA/CA. If the end entity
wants to include multiple Evidence statements in a CSR, it can use the
composite Attester model described in {{Section 3.3 of RFC9334}} and
{{I-D.richardson-rats-composite-attesters}}, i.e., together with a conceptual
message wrapper (CMW) {{I-D.ietf-rats-msg-wrap}} structure, as described in
{{Section 4.3 of I-D.ietf-lamps-csr-attestation}}. The lead Attester
should then pass the nonce to the sub-Attesters. Based on Figure 2 of
{{I-D.richardson-rats-composite-attesters}}, {{fig-lead-Attester}} shows an
example of how a nonce can be distributed among several Attesters in an end
entity. If multiple `respInfo` structures are required, the `reqInfo` and `respInfo`
structures can also use a conceptual message wrapper (CMW).

~~~~ aasvg
                      .---------.
                      | RA / CA |
                      '---------'
                         |   ^
                    nonce|   |CSR
                         |   |
                         |   |
                         |   |
.------------------------v---|------------------------------------.
|            .-----------------------------.                      |
|            |certificate management client|                      |
|            '-----------------------------'                      |
|                        |   ^                                    |
|                        |   |                                    |
| .----------------------|---|-------.                            |
| | .-------------.      |   | Evidence-collection CMW            |
| | | target A    |     n|   | 1: CMW(Evidence(Attester A)        |
| | | environment |     o|   | 2:     Evidence(Attester B)        |
| | '-------------'     n|   | 3:     Evidence(Attester C))       |
| |       |             c|   |       |                            |
| |       |collect      e|   |       |  nonce      .------------. |
| |       |Claims        |   |       |------------>| Attester B | |
| |       |              v   |       |<------------|            | |
| |       |          .-------------. |  Evidence B '------------' |
| |       |          | attesting   | |                            |
| |       '--------->| environment | |  nonce      .------------. |
| |                  '-------------' |------------>| Attester C | |
| |        Attester A                |<------------|            | |
| '-----------lead Attester----------'  Evidence C '------------' |
|                                                                 |
'------------------------------end entity-------------------------'
~~~~
{: #fig-lead-Attester title="Class 1 Composite Attester"}

The interaction between the end entity and the RA/CA is illustrated in {{fig-msg}}.

~~~~ aasvg
end entity                                      RA/CA
==========                                  =============

                 ------ nonce request ----->

                                           Verify request
                                           Generate or obtain nonce*
                                           Create response

                 <---- nonce response ------
                       (nonce, expiry)

Generate key pair
Generate Evidence(s)*
Generate certification
  request message

                 -- certification request -->
                +Evidence(s) including nonce

                                           Verify request
                                           Verify Evidence(s)*
                                           Check freshness/replay*
                                           Issue certificate
                                           Create response
                                           Handle response

                <-- certification response --

Store certificate

*: These steps can require interactions with the Attester (on the
   end entity side) and with the Verifier (on the RA/CA side).
~~~~
{: #fig-msg title="Exchange with Nonce and Evidence."}

The following sections define the generic data structures for nonce request and
nonce response message content. CMP and CMC use ASN.1, while EST uses JSON and CBOR,
both defined CDDL.

## ASN.1 Representation {#ASN.1}

This section defines nonce request and nonce response message content as ASN.1 types for use in
CMP, see {{CMP}}, and CMC, see {{CMC}}. Nonce values conveyed as ASN.1 OCTET STRING values
in CMP and CMC are between 8 and 64 bytes in length. A zero-length OCTET STRING indicates
that the RA/CA does not require proof of freshness for the upcoming certificate request.

~~~~
ATTESTATION-NONCE-REQUEST ::= TYPE-IDENTIFIER
AttestationNonceRequestSet ATTESTATION-NONCE-REQUEST ::= {
   ... -- None defined in this document --
}

ATTESTATION-NONCE-RESPONSE ::= TYPE-IDENTIFIER
AttestationNonceResponseSet ATTESTATION-NONCE-RESPONSE ::= {
   ... -- None defined in this document --
}

NonceRequest ::= SEQUENCE {
   len INTEGER (8..64) OPTIONAL,
   -- Indicates the required length of the requested nonce
   type ATTESTATION-NONCE-REQUEST.&id(
      {AttestationNonceRequestSet}) OPTIONAL,
   -- Identifies the nonce-request syntax for the
   --   selected Attestation statement type
   reqInfo ATTESTATION-NONCE-REQUEST.&Type(
      {AttestationNonceRequestSet}{@type}) OPTIONAL
   -- Contains type-specific nonce-request information
}

NonceResponse ::= SEQUENCE {
   nonce OCTET STRING (SIZE(0 | 8..64)),
   -- Contains the nonce of length len
   -- A zero-length OCTET STRING indicates that no freshness
   -- proof is required
   expiry INTEGER OPTIONAL,
   -- Indicates how long in seconds the nonce issuer
   --   considers the nonce valid
   type ATTESTATION-NONCE-RESPONSE.&id(
      {AttestationNonceResponseSet}) OPTIONAL,
   -- Identifies the nonce-response syntax for the
   --   selected Attestation statement type
   respInfo ATTESTATION-NONCE-RESPONSE.&Type(
      {AttestationNonceResponseSet}{@type}) OPTIONAL
   -- Contains type-specific nonce-response information
}
~~~~

## CDDL Representation {#CDDL}

This section provides a CDDL {{RFC8610}} definition for the JSON and CBOR nonce
request and nonce response message content. The nonce-request rule applies to
both JSON and CBOR. The nonce-response-json and nonce-response-cbor rules define
the encoding-specific representation of the nonce value. For JSON, the
base64url-nonce rule captures the allowed character set and encoded length; the
decoded nonce length requirements are specified in {{EST-https}}.

~~~~ cddl
nonce-request = {
  ? "len": nonce-length,
  ? "type": dotted-decimal-oid,
  ? "reqInfo": any
}

nonce-response-json = {
    "nonce": json-nonce,
  ? "expiry": uint,
  ? "type": dotted-decimal-oid,
  ? "respInfo": any
}

nonce-response-cbor = {
    "nonce": cbor-nonce,
  ? "expiry": uint,
  ? "type": dotted-decimal-oid,
  ? "respInfo": any
}

nonce-length = 8..64
cbor-nonce = h'' / bstr .size (8..64)
json-nonce = "" / base64url-nonce
base64url-nonce = tstr .regexp "[A-Za-z0-9_-]{11,86}"
dotted-decimal-oid = tstr
~~~~


# Use with CMP {#CMP}

The nonce request and nonce response message content is conveyed as ASN.1 content,
see {{ASN.1}}, in a general message (`genm`) {{Section 5.3.19 of RFC9810}}
and general response (`genp`) {{Section 5.3.20 of RFC9810}}, respectively.

~~~~
GenMsg:    {id-it TBD1}, NonceRequest
GenRep:    {id-it TBD2}, NonceResponse

id-it-nonceRequest OBJECT IDENTIFIER ::= { id-it TBD1 }
NonceRequestValue ::= NonceRequest

id-it-nonceResponse OBJECT IDENTIFIER ::= { id-it TBD2 }
NonceResponseValue ::= NonceResponse
~~~~

When CMP is transferred over HTTP, the OPTIONAL `<operation>` path segment
defined in {{Section 3.4 of RFC9811}} MAY be used for the nonce request message.

| Operation | Operation path | Details |
| --- | --- | --- |
| Get Attestation Freshness Nonce | `getnonce` | {{CMP}} |

When CMP is transferred over CoAP, the OPTIONAL `<operation>` path segment
defined in {{Section 2.1 of RFC9482}} MAY be used for the nonce request message.

| Operation | Operation path | Details |
| --- | --- | --- |
| Get Attestation Freshness Nonce | `nonce` | {{CMP}} |

In the event of a possible error or if the RA/CA is unable or unwilling to deliver the
requested nonce, CMP offers several ways to indicate this. Which variant fits depends
on the circumstances.

- Respond with an error message containing the `PKIFailureInfo` bit as defined by {{RFC9483}}.
- If HTTP or CoAP is used for transferring the general message, return a status code
  on transfer level as described in {{RFC9811}} or {{RFC9482}}.

# Use with EST {#EST}

The nonce request and nonce response message content is conveyed as JSON or CBOR
according to the CDDL definition, see {{CDDL}}, between end entity (EST client) and
RA/CA (EST server). A compliant EST server MUST provide an EST endpoint with
the path-segment /nonce for this operation.

| Operation | Operation path | Details |
| --- | --- | --- |
| Request of a nonce | `/nonce` | {{EST}} |

Depending on whether additional parameters are to be transferred, the client
uses either the `GET` or `POST` method:

- The `GET` method MUST be used if no optional content is to be transferred
- The `POST` method MUST be used if nonce request message content encoded in JSON
  or CBOR is to be transferred.

## EST over HTTPS {#EST-https}

If the nonce request and nonce response message content is transferred over HTTPS,
the specification in {{RFC7030}} applies.

The JSON nonce request object is formally described by the nonce-request CDDL
rule in {{CDDL}}. The JSON nonce response object is formally described by the
nonce-response-json CDDL rule in {{CDDL}}.

The JSON structure has the following members:

- The OPTIONAL "len" and "expiry" members, if present, MUST be unsigned
  integers.
- The "nonce" member MUST either contain a zero-length string or the nonce
  value between 8 and 64 bytes in length conveyed as a JSON string containing
  the unpadded base64url encoding, as specified in {{Section 5 of RFC4648}}.
  Such encodings are between 11 and 86 characters in length.
- The OPTIONAL "type" member, if present, MUST be a text string containing the
  object identifier as a dotted-decimal OID.
- The OPTIONAL "reqInfo" and "respInfo" members MUST only be included if the
  corresponding "type" member contains an OID. Their contents are defined by
  that OID.

If the nonce request message was successful, the EST server MUST respond with an HTTP 200
status code and the nonce response message content MUST be encoded as a JSON object.
The HTTP 200 status code MUST also be used if the nonce is an empty string.

In the event of a possible error, the EST server MUST respond with an HTTP
status code 400 (Bad Request) and MUST omit the nonce response message content. If the
nonce request message has been made correctly, but the EST server is unable or unwilling
to deliver the requested nonce, it MUST respond with an HTTP 503 (Service
Unavailable).

The EST server MAY request HTTP-based client authentication as described in
{{Section 3.2.3 of RFC7030}}.

The following media types MUST be used as the Content-Type of the `POST` request
and the response.

| Message type<br/>(per operation) | Media type(s) | Reference |
| --- | --- | --- |
| NonceRequest | N/A (for GET) or<br/>application/est-attestation-freshness+json (for POST) | {{EST-https}} |
| NonceResponse | application/est-attestation-freshness+json | {{EST-https}} |

The following example shows a nonce request and nonce response message exchange without
transmitting optional parameters in the request:


~~~
GET /.well-known/est/nonce HTTP/1.1

HTTP/1.1 200 OK
Content-Type: application/est-attestation-freshness+json

{
  "nonce": "MTIzNDU2Nzg5MDEyMzQ1Njc4OTAxMjM0NTY3ODkwMTI",
  "expiry": 600
}
~~~

The following example shows a nonce request and nonce response message exchange
transmitting optional parameters in the request:

~~~
POST /.well-known/est/nonce HTTP/1.1
Content-Type: application/est-attestation-freshness+json

{
  "len": 32,
  "type": "1.2.3.4.5",
  "reqInfo": {
    "pcr-index": [0, 1, 2, 3],
    "certificate-name": ["aik-1"]
  }
}

HTTP/1.1 200 OK
Content-Type: application/est-attestation-freshness+json

{
  "nonce": "MTIzNDU2Nzg5MDEyMzQ1Njc4OTAxMjM0NTY3ODkwMTI",
  "expiry": 600,
  "type": "1.2.3.4.6",
  "respInfo": {
    "certificate-name": "aik-1"
  }
}
~~~

## EST over Secure CoAP {#EST-coaps}

If the nonce request and nonce response message content is transferred via
secure CoAP, the specification in {{RFC9148}} applies.

The CBOR nonce request object is formally described by the nonce-request CDDL
rule in {{CDDL}}. The CBOR nonce response object is formally described by the
nonce-response-cbor CDDL rule in {{CDDL}}.

The CBOR structure has the following members:

- All map keys are text strings.
- The "nonce" member MUST either contain a zero-length octet string or the nonce
  value between 8 and 64 bytes in length conveyed as a CBOR byte string.
- The OPTIONAL dotted-decimal-oid "type" member denotes a text string containing
  an object identifier in dotted-decimal notation.
- The OPTIONAL "reqInfo" and "respInfo" members contain type-specific CBOR
  values. They MUST only be included if the corresponding "type" member contains
  an OID. Their CBOR encoding is defined by that OID.

If the nonce request was successful, the EST server MUST respond to a GET
request with a code 2.05 and to a POST request with code 2.04 and the
nonce response message content MUST be encoded as a CBOR object.
The code 2.05 or code 2.04 MUST also be used if the nonce is a zero-length
byte string.

In the event of a possible error, the EST server MUST respond with a code
4.00 (Bad Request) and MUST omit the nonce response message content. If the
nonce request has been made correctly, but the EST server is unable or
unwilling to deliver the requested nonce, it MUST respond with a code 5.03
(Service Unavailable).

The following media type MUST be used for nonce request and nonce response message
content. The corresponding CoAP Content-Format value is used in the CoAP
Content-Format and Accept options as specified in {{RFC9148}}.

| Message type<br/>(per operation) | Media type | Reference |
| --- | --- | --- |
| NonceRequest<br/>NonceResponse | application/est-attestation-freshness+cbor | {{EST-coaps}} |


# Use with CMC {#CMC}

The nonce request and nonce response message content is conveyed as ASN.1,
see {{ASN.1}}, as CMC Controls in a Full PKI Request, see
{{Section 6 of I-D.ietf-lamps-rfc5272bis}}. The received nonce can be
used for a CSR to be transferred in a Simple or Full PKI request.


To transfer the controls, the content type `id-data` SHOULD be used.


~~~~
cmc-nonceReq CMC-CONTROL ::=
    { NonceRequest IDENTIFIED BY id-cmc-nonceReq }
id-cmc-nonceReq OBJECT IDENTIFIER ::= { id-cmc TBD4 }

cmc-nonceResp CMC-CONTROL ::=
    { NonceResponse IDENTIFIED BY id-cmc-nonceResp }
id-cmc-nonceResp OBJECT IDENTIFIER ::= { id-cmc TBD5 }
~~~~

The following example shows the nonce request and nonce response message content:

~~~
ContentInfo.contentType = id-data
ContentInfo.content
   controlSequence
      {101, id-cmc-senderNonce, 10001}
      {102, id-cmc-nonceReq, <NonceRequest>}

ContentInfo.contentType = id-data
ContentInfo.content
   controlSequence
      {101, id-cmc-senderNonce, 10005}
      {102, id-cmc-recipientNonce, 10001}
      {103, id-cmc-nonceResp, <NonceResponse>}
~~~

In the event of an error, or if the server is unable to provide the requested nonce, the CMC Server MAY return status information about the request using either an Extended CMC Status Info Control or a CMC Status Info Control, as defined in {{Section 6.1 of I-D.ietf-lamps-rfc5272bis}}.

# IANA Considerations {#iana}

## CMP

### Well-Known URI Path Segment

IANA is requested to add the following entry to the "CMP Well-Known URI Path
Segments" registry defined in {{RFC8615}}:

| Path Segment | Description | Reference |
| --- | --- | --- |
| `getnonce` | Get Attestation Freshness Nonce over HTTP | This-RFC |
| `nonce` | Get Attestation Freshness Nonce over CoAP | This-RFC |

### Information Type

IANA is requested to register the following object identifier in the
"SMI Security for PKIX CMP Information Types" registry
(1.3.6.1.5.5.7.4):

| Decimal | Description | Reference |
| --- | --- | --- |
| TBD1 | id-it-nonceRequest | This-RFC |
| TBD2 | id-it-nonceResponse | This-RFC |

## EST

IANA is requested to register the following media types in the "Media Types"
registry {{RFC6838}}:

### JSON Media Type

Type name:
: application

Subtype name:
: est-attestation-freshness+json

Required parameters:
: N/A

Optional parameters:
: N/A

Encoding considerations:
: binary

Security considerations:
: See {{sec-cons}} of this RFC.

Interoperability considerations:
: The same media type is used for EST attestation freshness nonce requests and
  nonce responses. The protocol context determines whether the JSON payload is a
  request or response.

Published specification:
: This RFC.

Applications that use this media type:
: EST implementations using attestation freshness nonces over HTTP.

Fragment identifier considerations:
: The syntax and semantics of fragment identifiers are as specified for
  "application/json". At publication of this specification, no fragment
  identification syntax is defined for "application/json".

Additional information:
: Deprecated alias names for this type: N/A
: Magic number(s): N/A
: File extension(s): N/A
: Macintosh file type code(s): N/A

Person & email address to contact for further information:
: IETF LAMPS Working Group (spasm@ietf.org)

Intended usage:
: COMMON

Restrictions on usage:
: N/A

Author:
: IETF LAMPS Working Group

Change controller:
: IETF

### CBOR Media Type

Type name:
: application

Subtype name:
: est-attestation-freshness+cbor

Required parameters:
: N/A

Optional parameters:
: N/A

Encoding considerations:
: binary

Security considerations:
: See {{sec-cons}} of this RFC.

Interoperability considerations:
: The same media type is used for EST attestation freshness nonce requests and
  nonce responses. The protocol context determines whether the CBOR payload is a
  request or response.

Published specification:
: This RFC.

Applications that use this media type:
: EST implementations using attestation freshness nonces over CoAP.

Fragment identifier considerations:
: The syntax and semantics of fragment identifiers are as specified for
  "application/cbor". At publication of this specification, no fragment
  identification syntax is defined for "application/cbor".

Additional information:
: Deprecated alias names for this type: N/A
: Magic number(s): N/A
: File extension(s): N/A
: Macintosh file type code(s): N/A

Person & email address to contact for further information:
: IETF LAMPS Working Group (spasm@ietf.org)

Intended usage:
: COMMON

Restrictions on usage:
: N/A

Author:
: IETF LAMPS Working Group

Change controller:
: IETF

### CoAP Content-Format

IANA is requested to register the following Content-Format in the "CoAP
Content-Formats" subregistry within the "CoRE Parameters" registry
{{RFC7252}}. This registration uses the IETF Review or IESG Approval range
defined for that registry.

| Media Type | ID | Reference |
| --- | --- | --- |
| application/est-attestation-freshness+cbor | TBD3 | This-RFC |


## CMC

### Control

IANA is requested to register the following object identifier in the
"SMI Security for PKIX CMC Controls" registry (1.3.6.1.5.5.7.7):

| Decimal | Description | Reference |
| --- | --- | --- |
| TBD4 | `id-cmc-nonceReq` | This-RFC |
| TBD5 | `id-cmc-nonceResp` | This-RFC |

## ASN.1 Module

IANA is requested to register the following ASN.1 {{X.680}} module OID in the
"SMI Security for PKIX Module Identifier" registry (1.3.6.1.5.5.7.0). This
OID is defined in {{asn1}}.

| Decimal | Description | References |
| --- | --- | --- |
| TBDMOD | `id-mod-att-fresh-req` | This-RFC |

# Operational Considerations {#operations}

When the RA/CA is requested to provide a nonce to an end entity, it
can generate the nonce locally or obtain it through an interaction
with the Verifier. According to the IETF RATS architecture {{RFC9334}},
the Verifier is responsible for validating Evidence about an Attester
and generating Attestation Results for use by a Relying Party.

The nonce value MUST contain a random byte sequence with at least 64 bits
of entropy. The RA/CA MUST ensure that a nonce it generates is unique and
MUST NOT be reused. If the RA/CA obtains a nonce from the Verifier, it
MUST ensure that the Verifier utilizes the same policy for uniqueness and
non-reuse properties. The length of the nonce depends on the remote
attestation technology in use, as specific nonce
lengths may be required by the end entity. This specification assumes
that the RA/CA possesses knowledge, either out-of-band or through the
len field in the nonce request message, regarding the required nonce length for
the attestation technology. Nonces of incorrect length may cause the
remote attestation protocol to fail.

For instance, the PSA attestation token {{RFC9783}}
supports nonce lengths of 32, 48, and 64 bytes. Other attestation
technologies employ nonces of similar lengths.

The end entity MUST use the received nonce if the remote attestation
technology used supports the requested length. If the selected attestation
statement type defines a deterministic nonce transformation for use by a
specific Attester or sub-Attester, the end entity MUST apply only that
transformation and only for the purpose defined by that specification.

If attestation-type-specific response information is returned with the
nonce, the end entity MUST process that information according to the
selected type when invoking the attestation technology. Such information
MAY include parameters or metadata needed to apply, identify, or validate
a deterministic nonce transformation defined by the attestation statement
type.

While this specification does not address the semantics of the attestation API
or the underlying software/hardware architecture, the API returns Evidence from
the Attester in a format specific to the attestation technology used and
identified by the `AttestationStatement` type. The returned Evidence is encapsulated
in the `AttestationStatement` within the `AttestationBundle` carried in the CSR, as
defined in {{I-D.ietf-lamps-csr-attestation}}. The software generating the CSR
treats the attestation statement payload as an opaque blob and does not
interpret its format. It is crucial to note that the nonce is included in the
Evidence, either implicitly or explicitly, and MUST NOT be conveyed in CSR
structures outside of the attestation payload.

The freshness established by the nonce applies to the Evidence as
bound to the nonce used by the attestation technology. This
specification does not assign independent freshness semantics to other claims
contained in the Evidence. The interpretation of any other claim, including
whether it represents current state, historical state, configuration state, or
measurement results, is defined by the attestation technology, the appraisal
policy, and the Verifier or Relying Party processing rules.

Using nonces causes the Relying Party to create state for outstanding
freshness challenges. This state increases the attack surface for denial-of-service
attacks, for example by causing the Relying Party to allocate memory for many
nonce requests that never result in corresponding Evidence. Relying Parties
SHOULD reduce this exposure by limiting nonce lifetimes, bounding the number of
outstanding nonces, applying rate limits, associating nonce state with an
authenticated or otherwise constrained requester where possible, and promptly
discarding expired state. The use of a stateless cookie mechanism to reduce this
state-management burden is also possible but not further detailed in this
specification.

# Security Considerations {#sec-cons}

This specification details the process of obtaining a nonce via CMP, EST,
and CMC. Therefore, the security considerations of these protocols apply. Regarding
the use of attestation statements in a CSR, the security considerations
outlined in {{I-D.ietf-lamps-csr-attestation}} are pertinent to this
specification.

The nonce itself does not generally require confidentiality
protection while maintaining the security properties of the remote
attestation protocol. However, optional attestation-type-specific
request or response information conveyed together with the nonce MAY
reveal privacy-sensitive or deployment-sensitive information, such as
platform topology, certificate identifiers, or measurement selections.
{{RFC9334}} defines the IETF remote attestation architecture and
extensively discusses nonce-based freshness.

Specific attestation technology specifications such as {{RFC9711}} and
{{RFC9783}} offer guidance on replay protection using nonces. This
document defers specific recommendations to those specifications.

If an attestation technology or Composite Attester profile transforms a
received nonce before embedding it in Evidence, that transformation MUST
be deterministic, MUST preserve at least the minimum entropy required by
the attestation technology, and MUST provide the Verifier with enough
information to reconstruct or validate the transformed value when needed.
Specifications defining such transformations SHOULD use well-analyzed
cryptographic mechanisms for nonce expansion or size adjustment to ensure
that the resulting entropy and security properties remain acceptable.

If attestation-type-specific nonce exchange information is used, the
corresponding specification MUST define the syntax and semantics of that
information, including any confidentiality requirements. Implementations
SHOULD minimize the amount of such information returned by the RA/CA and
SHOULD protect it using the message or transport security mechanisms of
the enrollment protocol when confidentiality is required by deployment
policy or by the attestation technology specification.

--- back

# Example: TPM 2.0 PCR Selection for `reqInfo` and `respInfo` {#appx-tpm-pcr-example}

This appendix provides an informative example showing how `reqInfo` and
`respInfo` can be used with a TPM 2.0-based attestation procedure such as the
challenge-response exchange described in {{Section 2.1.1.3.2 of RFC9684}}.

In this example, the end entity wants to obtain a nonce for use with a TPM
attestation statement. The end entity first requests a nonce from the
Relying Party or Verifier. The `reqInfo` field can carry optional information
from the Attester side that helps the Verifier prepare the TPM 2.0 challenge,
such as which Attestation Key certificates are available.

The resulting `NonceResponse` corresponds to the TPM 2.0 challenge parameters
from {{RFC9684}}, with one exception: the `nonce-value` from the RFC 9684
challenge is conveyed in this document's top-level `nonce` field rather than
inside `respInfo`. The remaining challenge parameters, such as the selected PCR
bank, PCR indices, and the selected Attestation Key certificate, are conveyed
in `respInfo`.

The exact syntax of `reqInfo` and `respInfo` is defined by the nonce-exchange
type identified in the `type` field. The JSON objects below are only examples
used to explain the role of these fields:

~~~
NonceRequest = {
  "len": 32,
  "type": "1.2.3.4.5",
  "reqInfo": {
    "certificate-name": ["aik-1", "aik-2"],
    "supported-hash-algo": ["TPM_ALG_SHA256"]
  }
}

NonceResponse = {
  "nonce": "MTIzNDU2Nzg5MDEyMzQ1Njc4OTAxMjM0NTY3ODkwMTI",
  "expiry": 600,
  "type": "1.2.3.4.6",
  "respInfo": {
    "tpm20-pcr-selection": [
      {
        "tpm20-hash-algo": "TPM_ALG_SHA256",
        "pcr-index": [0, 1, 2, 3]
      }
    ],
    "certificate-name": "aik-1"
  }
}
~~~

In this example:

- `reqInfo.certificate-name` lets the Attester indicate more than one available
  Attestation Key certificate when multiple TPM attestation keys are present.
- `respInfo.tpm20-pcr-selection` identifies the PCR bank and PCR indices that
  the Attester is expected to include in the TPM quote or equivalent Evidence.
- `respInfo.certificate-name` identifies which Attestation Key certificate the
  Attester is to use when generating the fresh Evidence associated with the
  returned nonce.

This example is intended only to explain the purpose of `reqInfo` and
`respInfo`. It does not define a new nonce-exchange type, a new OID, or a TPM
attestation profile. The TPM quote itself, its signature, and any optional
unsigned PCR values are conveyed later as Evidence and are not part of
`respInfo`.

# ASN.1 Module {#asn1}

The following module adheres to ASN.1 specifications {{X.680}} and
{{X.690}}.

~~~ asn1
<CODE BEGINS>

att-fresh-req
  { iso(1) identified-organization(3) dod(6) internet(1)
  security(5) mechanisms(5) pkix(7) id-mod(0)
  id-mod-att-fresh-req (TBDMOD) }

DEFINITIONS IMPLICIT TAGS ::=
BEGIN
EXPORTS ALL;
IMPORTS

id-it, InfoTypeAndValue{}
  FROM PKIXCMP-2023
    { iso(1) identified-organization(3) dod(6) internet(1)
      security(5) mechanisms(5) pkix(7) id-mod(0)
      id-mod-cmp2023-02(116) }

id-cmc, CMC-CONTROL
FROM EnrollmentMessageSyntax-2025
   { iso(1) identified-organization(3) dod(6) internet(1)
   security(5) mechanisms(5) pkix(7) id-mod(0)
   id-mod-enrollMsgSyntax-2025(TBDCMC) }
-- RFC Editor: TBDCMC shall be TBD1 as defined by
--   Section 11 of draft-ietf-lamps-rfc5272bis

;

-- NonceRequest and NonceResponse types

ATTESTATION-NONCE-REQUEST ::= TYPE-IDENTIFIER

AttestationNonceRequestSet ATTESTATION-NONCE-REQUEST ::= {
   ... -- None defined in this document --
}

ATTESTATION-NONCE-RESPONSE ::= TYPE-IDENTIFIER

AttestationNonceResponseSet ATTESTATION-NONCE-RESPONSE ::= {
   ... -- None defined in this document --
}

 NonceRequest ::= SEQUENCE {
    len    INTEGER (8..64) OPTIONAL,
    -- indicates the required length of the requested nonce
    type   ATTESTATION-NONCE-REQUEST.&id(
              {AttestationNonceRequestSet}) OPTIONAL,
    -- identifies the nonce-request syntax for the selected
    --   attestation statement type
    reqInfo ATTESTATION-NONCE-REQUEST.&Type(
              {AttestationNonceRequestSet}{@type}) OPTIONAL
    -- contains type-specific nonce-request information
 }

 NonceResponse ::= SEQUENCE {
    nonce  OCTET STRING (SIZE(0 | 8..64)),
    -- contains the nonce of length len
    -- a zero-length OCTET STRING indicates that no freshness
    --   proof is required
    expiry INTEGER OPTIONAL,
    -- indicates how long in seconds the nonce issuer considers
    --   the nonce valid
    type   ATTESTATION-NONCE-RESPONSE.&id(
              {AttestationNonceResponseSet}) OPTIONAL,
    -- identifies the nonce-response syntax for the selected
    --   attestation statement type
    respInfo ATTESTATION-NONCE-RESPONSE.&Type(
               {AttestationNonceResponseSet}{@type}) OPTIONAL
    -- contains type-specific nonce-response information
 }

 id-it-nonceRequest OBJECT IDENTIFIER ::= { id-it TBD1 }
    NonceRequestValue ::= NonceRequest

 id-it-nonceResponse OBJECT IDENTIFIER ::= { id-it TBD2 }
    NonceResponseValue ::= NonceResponse

 cmc-nonceReq CMC-CONTROL ::=
     { NonceRequest IDENTIFIED BY id-cmc-nonceReq }
 id-cmc-nonceReq OBJECT IDENTIFIER ::= { id-cmc TBD4 }

 cmc-nonceResp CMC-CONTROL ::=
     { NonceResponse IDENTIFIED BY id-cmc-nonceResp }
 id-cmc-nonceResp OBJECT IDENTIFIER ::= { id-cmc TBD5 }

END
<CODE ENDS>
~~~

# Acknowledgments
{:numbered="false"}

We would like to thank Russ Housley, Thomas Fossati, Watson Ladd, Ionut Mihalcea,
Carl Wallace, and Michael StJohns for their review comments.
