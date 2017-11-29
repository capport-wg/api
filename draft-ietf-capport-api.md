---
title: Captive Portal API
abbrev: Captive Portal API
docname: draft-ietf-capport-api-latest
date:
category: std

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: T. Pauly
    name: Tommy Pauly
    role: editor
    org: Apple Inc.
    street: 1 Infinite Loop
    city: Cupertino, California 95014
    country: United States of America
    email: tpauly@apple.com
  -
    ins: D. Thakore
    name: Darshak Thakore
    role: editor
    org: CableLabs
    street: 858 Coal Creek Circle
    city: Louisville, CO 80027
    country: United States of America
    email: d.thakore@cablelabs.com

normative:
    RFC2818:
    RFC5785:
    RFC7159:

informative:
    I-D.larose-capport-architecture:

--- abstract

This document describes an HTTP API that allows hosts to interact with a Captive Portal system on a local network.

--- middle

# Introduction

This document describes a HyperText Transfer Protocol (HTTP) Application Program Interface (API) that allows hosts to interact with a Captive Portal system on a local network. The API defined in this document has been designed to meet the requirements in the Captive Portal Architecture [I-D.larose-capport-architecture]. Specifically, the API provides:

- The state of captivity (whether or not the host has access to the Internet)
- A URI that a host's browser can present to a user to get out of captivity
- An encrypted connection (TLS for both the API and portal URI)

# Workflow

The Captive Portal Architecture defines three steps of interaction between hosts and a Captive Portal service:

1. Provisioning, in which a host discovers that a network has a captive portal, and learns the URI of the API server
2. API Server interaction, in which a host queries the state of the captive portal and retrieves the necessary information to get out of captivity
3. Enforcement, in which the enforcement device in the network blocks disallowed traffic, and sends ICMP messages to let hosts know they are blocked by the captive portal

This document is focused on the second step. It is assumed that the location of the Captive Portal API server has been provisioned as part of the first step. The mechanism for discovering this value not covered by this document.

# API Details

## URI of Captive Portal API

In order to communicate with the captive portal API server, the URI provisioned to the host SHOULD use the form of a well-known URI [RFC5785] ending with the string ".well-known/captive-portal.json". If this suffix is not included in the URI already provisioned, a host SHOULD append the suffix when communicating to the server.

The Captive Portal API MUST be accessed using HTTP over TLS on TCP port 443 (HTTPS) [RFC2818].

For example, if the Captive Portal API server is hosted at example.org, the URI of the API would be: "https://example.org/.well-known/captive-portal.json".

## JSON Keys

The Captive Portal API data structures are specified in JavaScript Object Notation (JSON) [RFC7159].

The following keys are defined at the top-level of the JSON structure returned by the API server:

- "permitted" (required, boolean): indicates whether or not the Captive Portal is open to the requesting host
- "hmac-key" (required, string): provides a per-host key that can be used to authenticate messages from the Captive Portal enforcement server
- "user-portal-url" (required, string): provides the URL of a web portal that can be presented a user to authenticate
- "expire-date" (optional, datetime): indicates the time at which the Captive Portal is expected to close
- "bytes-remaining" (optional, integer): indicates the number of bytes left, after which the Captive Portal is expected to close

Note that the use of the hmac-key is not defined in this document, but is intended for use in the enforcement step of the Captive Portal Architecture.

## Example Exchange

To request the Captive Portal JSON content, a host sends an HTTP GET request:

~~~~~~~~~~
GET /.well-known/captive-portal.json
Host: example.org
~~~~~~~~~~

The server then responds with the JSON content for that client:

~~~~~~~~~~
HTTP/1.1 200 OK
{
   "permitted": false,
   "hmac-key": "7cec81acce3176b262a46363666a01881b0e3bf60d97a98b5409b71cc60a1ac0"
   "user-portal-url": "https:example.org/portal.html"
   "expire-date": "2014-01-01T23:28:56.782Z":
}
~~~~~~~~~~

# Security Considerations

TBD: Provide complete security requirements and analysis.

## Privacy Considerations

Information passed in this protocol may include a user's personal information, such as a full name and credit card details. Therefore, it is important that Captive Portal API Servers do not allow access to the Captive Portal API over unecrypted sessions.

# IANA Considerations

This document defines one ".well-known" URIs using the registration procedure and template from Section 5.1 of [RFC5785].

##  captive-portal.json Well-Known URI Registration

URI suffix:  captive-portal.json

Change controller:  IETF

Specification document(s):  This RFC

Related information:  None

# Acknowledgments

This work in this document was started by Mark Donnelly and Margaret Cullen. Thanks to everyone in the CAPPORT Working Group who has given input.
