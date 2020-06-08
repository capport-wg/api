---
title: Captive Portal API
abbrev: Captive Portal API
docname: draft-ietf-capport-api-latest
date:
category: std

ipr: trust200902
keyword: Internet-Draft
workgroup: Captive Portal Interaction
stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: T. Pauly
    name: Tommy Pauly
    role: editor
    org: Apple Inc.
    street: One Apple Park Way
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

--- abstract

This document describes an HTTP API that allows clients to interact with a Captive Portal system. With this API, clients can discover how to get out of captivity and fetch state about their Captive Portal sessions.

--- middle

# Introduction

This document describes a HyperText Transfer Protocol (HTTP) Application Program Interface (API) that allows clients to interact with a Captive Portal system. The API defined in this document has been designed to meet the requirements in the Captive Portal Architecture {{?I-D.ietf-capport-architecture}}. Specifically, the API provides:

- The state of captivity (whether or not the client has access to the Internet)
- A URI that a client browser can present to a user to get out of captivity
- An encrypted connection (using TLS for connections to both the API and user-facing web portal)

# Terminology {#terms}

This document leverages the terminology and components described in {{?I-D.ietf-capport-architecture}} and additionally defines the following terms:

  - Captive Portal Client: The client that interacts with the Captive Portal API is typically some application running on the User Equipment that is connected to the Captive Network. This is also referred to as the "client" in this document.
  - Captive Portal API Server: The server exposing the APIs defined in this document to the client. This is also referred to as the "API server" in this document.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{!RFC2119}} {{!RFC8174}} when, and only when,
they appear in all capitals, as shown here.

# Workflow

The Captive Portal Architecture defines several categories of interaction between clients and Captive Portal systems:

1. Provisioning, in which a client discovers that a network has a captive portal, and learns the URI of the API server.
2. API Server interaction, in which a client queries the state of the captive portal and retrieves the necessary information to get out of captivity.
3. Enforcement, in which the enforcement device in the network blocks disallowed traffic.

This document defines the mechanisms used in the second category. It is assumed that the location of the Captive Portal API server has been discovered by the client as part of Provisioning. A set of mechanisms for discovering the API Server endpoint is defined in {{?I-D.ietf-capport-rfc7710bis}}.

# API Connection Details {#api-details}

The API server endpoint MUST be accessed using HTTP over TLS (HTTPS) and SHOULD be served on port 443 {{!RFC2818}}.
The client SHOULD NOT assume that the URI for a given network attachment will stay the same, and SHOULD rely on the discovery or provisioning process each time it joins the network.

For example, if the Captive Portal API server is hosted at "example.org", the URI of the API could be "https://example.org/captive-portal/api"

As described in Section 3 of {{?I-D.ietf-capport-architecture}}, the identity of the client needs to be visible to the Captive Portal API server in order for the server to correctly reply with the client's portal state. If the identifier used by the Captive Portal system is the client's set of IP addresses, the system needs to ensure that the same IP addresses are visible to both the API server and the enforcement device.

If the API server needs information about the client identity that is not otherwise visible to it, the URI provided to the client during provisioning can be distinct per client. Thus, depending on how the Captive Portal system is configured, the URI might be unique for each client host and between sessions for the same client host.

For example, a Captive Portal system that uses per-client session URIs could use "https://example.org/captive-portal/api/X54PD" as its API URI.

## Server Authentication {#server-auth}

The purpose of accessing the Captive Portal API over an HTTPS connection is twofold: first, the encrypted connection protects the integrity and confidentiality of the API exchange from other parties on the local network; and second, it provides the client of the API an opportunity to authenticate the server that is hosting the API. This authentication is aimed at allowing a user to be reasonably confident that the entity providing the Captive Portal API has a valid certificate for the hostname in the URI ("example.org", in the example above). The hostname of the API SHOULD be displayed to the user in order to indicate the entity which is providing the API service.

Clients performing revocation checking will need some means of accessing revocation information for certificates presented by the API server. Online Certificate Status Protocol {{!RFC6960}} (OCSP) stapling, using the TLS Certificate Status Request extension {{!RFC6066}} SHOULD be used. OCSP stapling allows a client to perform revocation checks without initiating new connections. To allow for other forms of revocation checking, a captive network could permit connections to OCSP responders or Certificate Revocation Lists (CRLs) that are referenced by certificates provided by the API server. In addition to connections to OCSP responders and CRLs, a captive network SHOULD also permit connections to Network Time Protocol (NTP) {{!RFC5905}} servers or other time-sync mechnisms to allow clients to accurately validate certificates.

Certificates with missing intermediate certificates that rely on clients validating the certificate chain using the URI specified in the Authority Information Access (AIA) extension {{!RFC5280}} SHOULD NOT be used by the Captive Portal API server. If the certificates do require the use of AIA, the captive network MUST allow client access to the host specified in the URI.

If the client is unable to validate the certificate presented by the API server, it MUST NOT proceed with any of the behavior for API interaction described in this document. The client will proceed to interact with the captive network as if the API capabilities were not present. It may still be possible for the user to access the network if the network redirects a cleartext webpage to a web portal.

# API State Structure {#json-keys}

The Captive Portal API data structures are specified in JavaScript Object Notation (JSON) {{!RFC8259}}. Requests and responses for the Captive Portal API use the "application/captive+json" media type. Clients SHOULD include this media type as an Accept header in their GET requests, and servers MUST mark this media type as their Content-Type header in responses.

The following key MUST be included in the top-level of the JSON structure returned by the API server:

- "captive" (boolean): indicates whether the client is in a state of captivity, i.e it has not satisfied the conditions to access the external network. If the client is captive (i.e. captive=true), it will still be allowed enough access for it to perform server authentication ({{server-auth}}).

The following keys can be optionally included in the top-level of the JSON structure returned by the API server:

- "user-portal-url" (string): provides the URL of a web portal with which a user can interact.
- "venue-info-url" (string): provides the URL of a webpage or site on which the operator of the network has information that it wishes to share with the user (e.g., store info, maps, flight status, or entertainment).
- "can-extend-session" (boolean): indicates that the URL specified as "user-portal-url" allows the user to extend a session once the client is no longer in a state of captivity. This provides a hint that a client system can suggest accessing the portal URL to the user when the session is near its limit in terms of time or bytes.
- "seconds-remaining" (integer): indicates the number of seconds remaining, after which the client will be placed into a captive state. The API server SHOULD include this value if the client is not captive (i.e. captive=false) and the client session is time-limited, and SHOULD omit this value for captive clients (i.e. captive=true) or when the session is not time-limited.
- "bytes-remaining" (integer): indicates the number of bytes remaining, after which the client will be in placed into a captive state. The byte count represents the sum of the total number of IP packet (layer 3) bytes sent and received by the client, including IP headers. Captive portal systems might not count traffic to whitelisted servers, such as the API server, but clients cannot rely on such behavior. The API server SHOULD include this value if the client is not captive (i.e. captive=false) and the client session is byte-limited, and SHOULD omit this value for captive clients (i.e. captive=true) or when the session is not byte-limited.

The valid JSON keys can be extended by adding entries to the Captive Portal API Keys Registry ({{iana-section}}). If a client receives a key that it does not recognize, it MUST ignore the key and any associated values. All keys other than the ones defined in this document as "required" will be considered optional.

# Example Interaction {#example}

A client connected to a captive network upon discovering the URI of the API server will query the API server to retrieve information about its captive state and conditions to escape captivity. In this example, the client discovered the URI "https://example.org/captive-portal/api/X54PD" using one of the mechanisms defined in {{?I-D.ietf-capport-rfc7710bis}}.

To request the Captive Portal JSON content, a client sends an HTTP GET request:

~~~~~~~~~~
GET /captive-portal/api/X54PD
Host: example.org
Accept: application/captive+json
~~~~~~~~~~

The server then responds with the JSON content for that client:

~~~~~~~~~~
HTTP/1.1 200 OK
Cache-Control: private
Date: Mon, 02 Mar 2020 05:07:35 GMT
Content-Type: application/captive+json

{
   "captive": true,
   "user-portal-url": "https://example.org/portal.html"
}
~~~~~~~~~~

Upon receiving this information the client will use this information to direct the user to the the web portal (as specified by the user-portal-url value) to enable access to the external network. Once the user satisfies the requirements for external network access, the client SHOULD query the API server again to verify that it is no longer captive.

When the client requests the Captive Portal JSON content after gaining external network access, the server responds with updated JSON content:

~~~~~~~~~~
HTTP/1.1 200 OK
Cache-Control: private
Date: Mon, 02 Mar 2020 05:08:13 GMT
Content-Type: application/captive+json

{
   "captive": false,
   "user-portal-url": "https://example.org/portal.html",
   "venue-info-url": "https://flight.example.com/entertainment",
   "seconds-remaining": 326,
   "can-extend-session": true
}
~~~~~~~~~~

Captive Portal JSON content can contain per-client data that is not appropriate to store in an intermediary cache. Captive Portal API servers SHOULD set the Cache-Control header in any responses to "private", or a more restrictive value {{!RFC7234}}.

# Security Considerations {#security}

One of the goals of this protocol is to improve the security of the communication between client hosts and Captive Portal systems. Client traffic is protected from passive listeners on the local network by requiring TLS-encrypted connections between the client and the Captive Portal API server, as described in {{api-details}}. All communication between the clients and the API server MUST be encrypted.

In addition to encrypting communications between clients and Captive Portal systems, this protocol requires a basic level of authentication from the API server, as described in {{server-auth}}. Specifically, the API server MUST present a valid certificate on which the client can perform revocation checks. This allows the client to ensure that the API server has authority for a hostname that can be presented to a user.

It is important to note that while the server authentication checks can validate a specific hostname, it is certainly possible for the API server to present a valid certificate for a hostname that uses non-standard characters or is otherwise designed to trick the user into believing that its hostname is some other, more trustworthy, name. This is a danger of any scenario in which a hostname is not typed in by a user.

## Privacy Considerations {#privacy}

Information passed between a client and a Captive Portal system may include a user's personal information, such as a full name and credit card details. Therefore, it is important that Captive Portal API Servers do not allow access to the Captive Portal API over unencrypted sessions.

# IANA Considerations {#iana-section}

IANA is requested to create a registration for an "application/captive+json" media type ({{iana-json}}) and a registry for fields in that format ({{iana-field-reg}}).

## Captive Portal API JSON Media Type Registration {#iana-json}

This document registers the media type for Captive Portal API JSON text, "application/captive+json".

Type name: application

Subtype name: captive+json

Required parameters: N/A

Optional parameters: N/A

Encoding considerations: Encoding considerations are identical to those specified for the "application/json" media type.

Security considerations: See {{security}}

Interoperability considerations: This document specifies format of conforming messages and the interpretation thereof.

Published specification: This document

Applications that use this media type: This media type is intended to be used by servers presenting the Captive Portal API, and clients connecting to such captive networks.

Fragment identifier considerations: N/A

Additional information: N/A

Person and email address to contact for further information: See Authors' Addresses section

Intended usage: COMMON

Restrictions on usage: N/A

Author: CAPPORT IETF WG

Change controller: IETF

## Captive Portal API Keys Registry {#iana-field-reg}

IANA is asked to create and maintain a new registry called "Captive Portal API Keys", which will reserve JSON keys for use in Captive Portal API data structures. The initial contents of this registry are provided in {{json-keys}}.

Each entry in the registry contains the following fields:

Key:
: The JSON key being registered, in string format.

Type: 
: The type of the JSON value to be stored, as one of the value types defined in {{!RFC8259}}.

Description:
: A brief description explaining the meaning of the value, how it might be used, and/or how it should be interpreted by clients.

Specification:
: A reference to a specification that defines the key and explains its usage.

New assignments for Captive Portal API Keys Registry will be administered by IANA using the Specification Required policy {{!RFC8126}}.
The Designated Expert is expected to validate the existence of documentation describing new keys in a permanent
publicly available specification. The expert is expected to validate that new keys have a clear meaning and do not
create unnecessary confusion or overlap with existing keys. Keys that are specific to non-generic use cases, particularly
ones that are not specified as part of an IETF document, are encouraged to use a domain-specific prefix.

# Acknowledgments {#thanksall}

This work in this document was started by Mark Donnelly and Margaret Cullen. Thanks to everyone in the CAPPORT Working Group who has given input.
