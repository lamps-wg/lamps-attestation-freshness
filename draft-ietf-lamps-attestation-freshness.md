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
title: Nonce-based Freshness for Remote Attestation in Certificate Signing Requests (CSRs) for the Certification Management Protocol (CMP) and for Enrollment over Secure Transport (EST)
abbrev: Freshness Nonces for Remote Attestation
area: sec
wg: LAMPS Working Group
keyword:
- Remote Attestation
- Certificate Signing Request
- Certificate Management Protocol (CMP)
- Enrollment over Secure Transport (EST)
date: 2024
github: "wg-lamps/lamps-attestation-freshness"
stand_alone: yes
author:
 -    ins: H. Tschofenig
      name: Hannes Tschofenig
      org: Siemens
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


normative:
  RFC2119:
  I-D.ietf-lamps-csr-attestation:
  I-D.ietf-lamps-rfc4210bis:
  RFC8295:
  RFC7030:
  RFC5280:
  RFC5785:
  RFC8615:
  RFC7159:
  RFC9482:
informative:
  RFC2986:
  RFC4211:
  RFC9334:
  I-D.tschofenig-rats-psa-token:
  I-D.ietf-rats-eat:
  TPM20:
     author:
        org: Trusted Computing Group
     title: Trusted Platform Module Library Specification, Family 2.0, Level 00, Revision 01.59
     target: https://trustedcomputinggroup.org/resource/tpm-library-specification/
     date: November 2019

--- abstract

When an end entity includes attestation Evidence in a Certificate Signing Request (CSR), it may be necessary to demonstrate the freshness of the provided Evidence. Current attestation technology commonly achieves this using nonces.

This document outlines the process through which nonces are supplied to the end entity by an RA/CA for inclusion in Evidence, leveraging the Certificate Management Protocol (CMP) and Enrollment over Secure Transport (EST)

--- middle

#  Introduction

The management of certificates, encompassing issuance, CA certificate provisioning, renewal, and revocation, has been streamlined through standardized protocols.

The Certificate Management Protocol (CMP) {{I-D.ietf-lamps-rfc4210bis}} defines messages for X.509v3 certificate creation and management. CMP facilitates interactions between end entities and PKI management entities, such as Registration Authorities (RAs) and Certification Authorities (CAs). For Certificate Signing Requests (CSRs), CMP primarily utilizes the Certificate Request Message Format (CRMF) {{RFC4211}} but also supports PKCS#10 {{RFC2986}}.

Enrollment over Secure Transport (EST) ({{RFC7030}}, {{RFC8295}}) is another certificate management protocol that provides a subset of CMP's features, primarily using PKCS#10 for CSRs.

When an end entity requests a certificate from a Certification Authority (CA), it may need to assert credible claims about the protections of the corresponding private key, such as the use of a hardware security module or the protective capabilities provided by the hardware, as well as claims about the platform itself.

To include these claims as Evidence in remote attestation, the remote attestation extension {{I-D.ietf-lamps-csr-attestation}} has been defined. It specifies how Evidence produced by an Attester is encoded for inclusion in CRMF or PKCS#10, along with any necessary certificates for its validation.

For a Verifier or Relying Party to ensure the freshness of the Evidence, knowing the exact time of its production is crucial. Current attestation technologies, like {{TPM20}} and {{I-D.tschofenig-rats-psa-token}}, often employ nonces to ensure the freshness of Evidence. Further details on ensuring Evidence freshness can be found in {{Section 10 of RFC9334}}.

{{Section 4 of I-D.ietf-lamps-csr-attestation}} provides examples where a CSR contains one or more Evidence statements. For each Evidence statement the end entity may wish to request a separate nonce.

Since an end entity requires one or more nonces from one or more Verifier via the RA/CA, an additional roundtrip is necessary. However, a CSR is a one-shot message. Therefore, CMP and EST enable the end entity to request information from the RA/CA before submitting a certification request conveniently.

Once a nonce is obtained, the end entity invokes the API on an Attester, providing the nonce as an input parameter. The Attester then returns an Evidence, which is embedded into a CSR and potentially together with further Evidence statements, submitted back to the RA/CA in a certification request message.

{{fig-arch}} illustrates this interaction:

- One or more nonces are acquired in step (1) using the extension to CMP/EST defined in this document.
- The CSR extension {{I-D.ietf-lamps-csr-attestation}} conveys one or more Evidence statements to the RA/CA in step (2).
- One ore more Verifier processes the received information and send Attestation Results to the Relying Party in step (3).

~~~ aasvg
                              .---------------.
                              |               |
                              |   Verifier    |
                              |               |
                              '---------------'
                                   |    ^  |    (3)
                                   |    |  | Attestation
                                   |    |  |  Result(s)
                    (1)            |    |  v
 .------------.  Nonce(s) in  .----|----|-----.
 |            |  CMP or EST   |    |    |     |
 |  End       |<-------------------+    |     |
 |  Entity    |               |         |     |
 |    ^       |-------------->|---------'     |
 |    |       |  Evidence     | Relying       |
 |    v       |  statement(s) | Party (RA/CA) |
 |  Attester  |  in CSR (2)   |               |
 |            |               |               |
 '------------'               '---------------'
~~~
{: #fig-arch title="Architecture with Background Check Model."}

The functionality described in this document is divided into two sections:

- {{CMP}} describes how to convey the nonce using CMP.
- {{EST}} describes the equivalent functionality for EST.

# Terminology and Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 {{RFC2119}}.

The terms Attester, Relying Party, Verifier and Evidence are defined
in {{RFC9334}}. The terms end entity, certification authority (CA),
and registration authority (RA) are defined in {{RFC5280}}.

We use the terms Certificate Signing Request (CSR) and certification
request interchangeably.

# Conveying a Nonce in CMP {#CMP}

Section 5.3.19 of {{I-D.ietf-lamps-rfc4210bis}} defines the
general request message (genm) and general response (genp).
The NonceRequest payload of the genm message, sent by the end
entity to request a nonce, optionally includes details on the
required length of the nonce from the Attester. The NonceResponse
payload of the genp message, sent by the CA/RA in response to the
request, contains the nonce itself.

~~~
 GenMsg:    {id-it TBD1}, NonceRequestValue
 GenRep:    {id-it TBD2}, NonceResponseValue | < absent >

 id-it-nonceRequest OBJECT IDENTIFIER ::= { id-it TBD1 }
 NonceRequestValue ::= SEQUENCE SIZE (1..MAX) OF NonceRequest
 NonceRequest ::= SEQUENCE {
    len INTEGER OPTIONAL,
    -- indicates the required length of the requested nonce
    type OBJECT IDENTIFIER OPTIONAL,
    -- indicates which Evidence type to request a nonce for
    hint EvidenceHint OPTIONAL
    -- indicates which Verifier to request a nonce from
 }

 id-it-nonceResponse OBJECT IDENTIFIER ::= { id-it TBD2 }
 NonceResponseValue ::= SEQUENCE SIZE (1..MAX) OF NonceResponse
 NonceResponse ::= SEQUENCE {
    nonce OCTET STRING,
    -- contains the nonce of length len
    -- provided by the Verifier indicated with hint
    expiry Time OPTIONAL,
    -- indicates how long the Verifier considers the
    -- nonce valid
    type OBJECT IDENTIFIER OPTIONAL,
    -- indicates which Evidence type to request a nonce for
    hint EvidenceHint OPTIONAL
    -- indicates which Verifier to request a nonce from
 }
~~~

The end entity may request one or more nonces for different Verifier. 
The EVIDENCE-STATEMENT type and the EvidenceHint are defined in {{I-D.ietf-lamps-csr-attestation}}.
They allow the Attester to specify to the Relying Party which Verifier should be
contacted to obtain a nonce. If a NonceRequest structure does not
contain type or hint, the RA/CA should respond with a nonce it MAY
generated by itself.

The use of the general request/response message exchange introduces an additional
roundtrip for transmitting nonce(s) from the CA/RA to the end entity (and
subsequently to the Attester within the end entity).

The end entity MUST construct a id-it-nonceRequest message to prompt
the RA/CA to send a nonce(s) in response. The message may contain one ore more NonceRequest structures, at a maximum one per Evidence statement the end entity wishes to provide in a CSR. If a NonceRequest structure does neither contain a type nor a hint, the RA/CA may generate a nonce itself and provide it in the respective NonceResponse structure.

NonceRequest, NonceResponse, and EvidenceStatement structures can contain a type field and a hint field. In terms of type and hint content, the order in which the NonceRequest structures were sent in the request message MUST match the order of the NonceResponse structures in the response message and the EvicenceStatements in the CSR later. This is important so that the RA/CA can send the Evidence statement to the Verifier who generated the nonce used by the Attester who generated it.

When reciving nonces from the RA/CA in a id-it-nonceResponse message, the end entity MUST us the 

If the end entity supports remote attestation and the policy dictates the inclusion
of Evidence in a CSR, the RA/CA responds with a NonceResponse message containing
the requested nonce.

The interaction is illustrated in {{fig-cmp-msg}}.

~~~
End Entity                                          RA/CA
==========                                      =============

              -->>-- NonceRequest -->>--
                                                Verify request
                                                Generate nonce*
                                                Create response
              --<<-- NonceResponse --<<--
                    (nonce, expiry)

Generate key pair
Generate Evidence*
Generate certification request message
              -->>-- certification request -->>--
                   +Evidence including nonce)
                                               Verify request
                                               Verify Evidence*
                                               Check for replay*
                                               Issue certificate
                                               Create response
              --<<-- certification response --<<--
Handle response
Store certificate

*: These steps require interactions with the Attester
(on the EE side) and with the Verifier (on the RA/CA side).
~~~
{: #fig-cmp-msg title="CMP Exchange with Nonce and Evidence."}

If HTTP is used to transfer the NonceRequest and NonceResponse
messages, the OPTIONAL \<operation> path segment defined in Section 3.6
of {{I-D.ietf-lamps-rfc4210bis}} MAY be used.

~~~
 +------------------------+-----------------+-------------------+
 | Operation              |Operation path   | Details           |
 +========================+=================+===================+
 | Get Attestation        | getnonce        | {{CMP}}           |
 | Freshness Nonce        |                 |                   |
 +------------------------+-----------------+-------------------+
~~~

If CoAP is used for transferring NonceRequest and NonceResponse messages,
the OPTIONAL \<operation> path segment defined in Section 2.1 of
{{RFC9482}} MAY be used.

~~~
 +------------------------+-----------------+-------------------+
 | Operation              |Operation path   | Details           |
 +========================+=================+===================+
 | Get Attestation        | nonce           | {{CMP}}           |
 | Freshness Nonce        |                 |                   |
 +------------------------+-----------------+-------------------+
~~~

# Conveying a Nonce in EST {#EST}

The EST client requests a nonce for its Attester from the EST server.
This function typically follows the request for CA certificates and
precedes other EST operations.

The EST server MUST support the path-prefix of "/.well-known/" as
defined in {{RFC5785}} and the registered name of "est".
Therefore, a valid EST server URI path begins with
"https://www.example.com/.well-known/est". Each EST operation is
indicated by a path-suffix that specifies the intended operation.

The following operation is defined by this specification:

~~~
 +------------------------+-----------------+-------------------+
 | Operation              |Operation path   | Details           |
 +========================+=================+===================+
 | Retrieval of a nonce   | /nonce          | {{EST}}           |
 +------------------------+-----------------+-------------------+
~~~
The operation path is appended to the path-prefix to form the URI
used with HTTP GET or POST to perform the desired EST operation.
An example of a valid URI absolute path for the "/nonce" operation
is "/.well-known/est/nonce".

## Request Methods

An EST client uses either a GET or a POST method, depending on
whether additional parameters need to be conveyed:

- A GET request MUST be used when the EST client does not want to
convey extra parameters.
- A POST request MUST be used when parameters, such as nonce length
or a hint about the verification service, are included in the request.

~~~
 +------------------+------------------------------+---------------+
 | Message type     | Media type(s)                | Reference     |
 | (per operation)  |                              |               |
 +==================+==============================+===============+
 | Nonce Request    | N/A (for GET) or             | This section  |
 |                  | application/json (for POST)  |               |
 +==================+==============================+===============+
 | Nonce Response   | application/json             | This section  |
 |                  |                              |               |
 +==================+==============================+===============+
~~~

## Example Requests

To retrieve a nonce using a GET request:

~~~
GET /.well-known/est/nonce HTTP/1.1
~~~

To retrieve a nonce while specifying the size and providing a hint about the
verification service using a POST request:

~~~
POST /.well-known/est/nonce HTTP/1.1
Content-Type: application/json
{
  "len": 8,
  "hint": "https://example.com"
}
~~~

The payload in a POST request MUST be of content-type "application/json" and
MUST contain a JSON object  {{RFC7159}} with the member "len" indicating the
length of the requested nonce value in bytes, and optionally the "hint" member,
which specifies an FQDN based on the definition in the EvidenceHint structure
from {{I-D.ietf-lamps-csr-attestation}}).

## Server Response

If successful, the EST server MUST respond with an HTTP 200 status code and a
content-type of "application/json", containing a JSON object {{RFC7159}} with
the "nonce" member. The "expiry" member is optional and indicates the validity
period of the nonce.



The EST server MAY request HTTP-based client authentication, as
explained in Section 3.2.3 of {{RFC7030}}.

If the request is successful, the EST server response MUST contain
a HTTP 200 response code with a content-type of "application/json"
and a JSON object  {{RFC7159}} with the member nonce. The expiry
member is optional and indicates the time the nonce is considered
valid. After the expiry time is expired, the session is likely
garbage collected.

Below is an example response:

~~~
HTTP/1.1 200 OK
Content-Type: application/json

{
    "nonce": "MTIzNDU2Nzg5MDEyMzQ1Njc4OTAxMjM0NTY3ODkwMTI=",
    "expiry": "2031-10-12T07:20:50.52Z"
}
~~~

Open Issue: Should a specific content type be registered for use
with EST over CoAP, where the nonce and expiry fields are encoded
in a CBOR structure?

# Nonce Processing Guidelines

When the RA/CA is requested to provide a nonce to an
end entity, it interacts with the Verifier.
According to the IETF RATS architecture {{RFC9334}}, the Verifier is
responsible for validating Evidence about an Attester and generating
Attestation Results for use by a Relying Party. The Verifier also acts as
the source of the nonce to prevent replay attacks.

The nonce value MUST contain a random byte sequence whereby the length
depends on the used remote attestation technology as specific nonce
length may be required by the end entity. This specification assumes
that the RA/CA possesses knowledge, either out-of-band or through the
len field in the NonceRequest, regarding the required nonce length for
the attestation technology. Nonces of incorrect length will cause the
remote attestation protocol to fail.

For instance, the PSA attestation token {{I-D.tschofenig-rats-psa-token}}
supports nonce lengths of 32, 48, and 64 bytes. Other attestation
technologies employ nonces of similar lengths.

When the end entity requests a nonce, the RA/CA SHOULD respond with a nonce
in the specified length.

If a specific length was requested, the RA/CA must provide a nonce of that size.
The end entity MUST use the received nonce if the remote attestation supports
the requested length. If necessary, the end entity MAY adjust the length of the
nonce by truncating or padding it accordingly.

While this specification does not address the semantics of the attestation API
or the underlying software/hardware architecture, the API returns Evidence from
the Attester in a format specific to the attestation technology used. The
returned Evidence is encapsulated within the CSR, as defined in
{{I-D.ietf-lamps-csr-attestation}}. The software generating the CSR treats
the Evidence as an opaque blob and does not interpret its format. It's crucial
to note that the nonce is included in the Evidence, either implicitly or
explicitly, and MUST NOT be conveyed in CSR structures outside of the Evidence
payload.

The processing of CSRs containing Evidence is detailed  in
{{I-D.ietf-lamps-csr-attestation}}. mportantly, certificates issued based
on this process do not contain the nonce, as specified in
{{I-D.ietf-lamps-csr-attestation}}.

#  IANA Considerations

This document adds new entries to the "CMP Well-Known URI Path Segments"
registry defined in {{RFC8615}}.

~~~
 +----------------+---------------------------+-----------------+
 | Path Segment   | Description               | Reference       |
 +================+===========================+=================+
 | getnonce       | Get Attestation Freshness | {{cmp}}         |
 |                | Nonce over HTTP           |                 |
 +----------------+---------------------------+-----------------+
 | nonce          | Get Attestation Freshness | {{cmp}}         |
 |                | Nonce over CoAP           |                 |
 +----------------+---------------------------+-----------------+
~~~

[Open Issue: Register path segments for EST]

#  Security Considerations

This specification details the process of obtaining a nonce via CMP and EST, assuming that the nonce does not require confidentiality protection while maintaining the security properties of the remote attestation protocol. {{RFC9334}} defines the IETF remote attestation architecture and extensively discusses nonce-based freshness.

Section 8.4 of {{I-D.ietf-rats-eat}} specifies requirements for the randomness and privacy of nonce generation when used with the Entity Attestation Token (EAT). These requirements, which are also adopted by attestation technologies like the PSA attestation token {{I-D.tschofenig-rats-psa-token}}, provide general utility:

- The nonce MUST have at least 64 bits of entropy.
- To prevent disclosure of privacy-sensitive information, it should be derived using a salt from a genuinely random number generator or another reliable source of randomness.

Each attestation technology specification offers guidance on replay protection using nonces and other techniques. Specific recommendations are deferred to these individual specifications in this document.

Regarding the use of Evidence in a CSR, the security considerations outlined in {{I-D.ietf-lamps-csr-attestation}} are pertinent to this specification.


--- back

# Acknowledgments

We would like to thank Russ Housley, Thomas Fossati, Watson Ladd, Ionut Mihalcea,
Carl Wallace, and Michael StJohns for their review comments.
