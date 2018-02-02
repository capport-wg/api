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
    RFC3339:

informative:
    I-D.ietf-capport-architecture:

--- abstract

This document describes an HTTP API that allows hosts to interact with a Captive Portal system.

--- middle

# Introduction

This document describes a HyperText Transfer Protocol (HTTP) Application Program Interface (API) that allows hosts to interact with a Captive Portal system. The API defined in this document has been designed to meet the requirements in the Captive Portal Architecture [I-D.ietf-capport-architecture]. Specifically, the API provides:

- The state of captivity (whether or not the host has access to the Internet)
- A URI that a host's browser can present to a user to get out of captivity
- An encrypted connection (TLS for both the API and portal URI)

# Workflow

The Captive Portal Architecture defines three steps of interaction between hosts and a Captive Portal service:

1. Provisioning, in which a host discovers that a network has a captive portal, and learns the URI of the API server
2. API Server interaction, in which a host queries the state of the captive portal and retrieves the necessary information to get out of captivity
3. Enforcement, in which the enforcement device in the network blocks disallowed traffic, and sends ICMP messages to let hosts know they are blocked by the captive portal

This document is focused on the second step. It is assumed that the location of the Captive Portal API server has been discovered by the host as part of the first step. The mechanism for discovering the API Server endpoint is not covered by this document.

# API Details

## URI of Captive Portal API endpoint

The URI of the API endpoint MUST be accessed using HTTP over TLS (HTTPS) and SHOULD be served on port 443 [RFC2818].
The host SHOULD not assume that the URI will be the same each time. Depending on how the Captive Portal system is configured, the URI may be unique for each host and between session for the same host.

For example, if the Captive Portal API server is hosted at example.org, the URI's of the API could be:
  
  - "https://example.org/captive-portal/api"
  - "https://example.org/captive-portal/api/X54PD"

## JSON Keys

The Captive Portal API data structures are specified in JavaScript Object Notation (JSON) [RFC7159].

The following keys are defined at the top-level of the JSON structure returned by the API server:

- "permitted" (required, boolean): indicates whether or not the Captive Portal is open to the requesting host
- "hmac-key" (required, string): provides a per-host key that can be used to authenticate messages from the Captive Portal enforcement server
- "user-portal-url" (required, string): provides the URL of a web portal that can be presented to a user to interact with
- "expire-date" (optional, string formatted as [RFC3339] datetime): indicates the date and time after which the host will be in a captive state
- "bytes-remaining" (optional, integer): indicates the number of bytes left, after which the host will be in a captive state

Note that the use of the hmac-key is not defined in this document, but is intended for use in the enforcement step of the Captive Portal Architecture.

## Example Exchange

To request the Captive Portal JSON content, a host sends an HTTP GET request:

~~~~~~~~~~
GET /captive-portal/api/X54PD
Host: example.org
Accept: application/json

~~~~~~~~~~

The server then responds with the JSON content for that client:

~~~~~~~~~~
HTTP/1.1 200 OK
Cache-Control: private
Date: Mon, 04 Dec 2013 05:07:35 GMT
Content-Type: application/json

{
   "permitted": false,
   "hmac-key": "7cec81acce3176b262a46363666a01881b0e3bf60d97a98b5409b71cc60a1ac0"
   "user-portal-url": "https://example.org/portal.html"
   "expire-date": "2014-01-01T23:28:56.782Z"
}
~~~~~~~~~~

# Security Considerations

TBD: Provide complete security requirements and analysis.

## Privacy Considerations

Information passed in this protocol may include a user's personal information, such as a full name and credit card details. Therefore, it is important that Captive Portal API Servers do not allow access to the Captive Portal API over unecrypted sessions.

# IANA Considerations

TBD: None

# Acknowledgments

This work in this document was started by Mark Donnelly and Margaret Cullen. Thanks to everyone in the CAPPORT Working Group who has given input.
