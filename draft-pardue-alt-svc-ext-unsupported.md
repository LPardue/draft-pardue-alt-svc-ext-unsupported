---
title: HTTP Alternative Services That Do Not Support Desired Extensions
abbrev: "Alt-Svc Extension Unsupported"
docname: draft-pardue-alt-svc-ext-unsupported
category: std

ipr: trust200902
area: Applications and Real-Time
workgroup: HTTP
keyword: Internet-Draft

stand_alone: yes
pi: [toc, docindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: L. Pardue
    name: Lucas Pardue
    organization: Cloudflare
    email: lucaspardue.24.7@gmail.com

normative:

informative:

--- abstract

This document describes an error case when an HTTP Alternative Service does not
support the extension points of the original connection. It defines an error
code that can be used by endpoints to signal the encounter.

--- middle

# Introduction

HTTP/2 ({{!HTTP2=RFC7540}}) and HTTP/3 ({{!HTTP3=I-D.ietf-quic-http}}) provide
several extension points, only some of which require negotiation before use. For
any HTTP connection that is actively using extensions, it is possible that the
server might advertise an Alternative Service ({{?ALT-SVC=RFC7838}}) that
contains no indication of whether the current extension points are supported by
indicated server. A client has to proactively engage with the Alternative in
order to determine what extension points it supports, which will often require
the client to establish a new transport-layer connection and HTTP handshake. The
client might determine that the Alternative does not support the same extensions
as the current connection and decide to close the connection. For an extension
that is negotiated via SETTINGS frames, the client might be able to detect the
lack of extensions early in the lifecycle. For other forms of extension, the
problem might not be detectable until later on, for instance when extension
frame exchanges fail.

The guidance in {{ALT-SVC}} has lead to clients that are robust to different
types of failures. Following a "make-before-break" approach avoids active
transfers being interupted by an opportunistic change in connection. As such,
testing an Alternative and finding that its extension points are incompatible is
unlikely to be a hard failure cases. However, Alt-Svc is the primary method of
switching between HTTP/2 and HTTP/3 and there is a possibility that extensions
offered in one version of a connection are not provided in another. It is also
feasible that support for extension points are not unilaterally deployed across
a fleet of servers, whether in the same organization domain or not; a client
might encounter problems with the Alternative could fail due to synchronization
or coordination issues between the origin and the Alternative.

Advertising Alternative Services has quite a low barrier and does require up
front coordination, meaning it is quite easy for an origin administrator to
configure things that are logically invalid. Alternatives can detect problems
and take action; {{ALT-SVC}} defines status 421 Misdirected Request that allows
the Alternative to signal such a problem.  However, there is are cases where
clients will be misdirected to Alternatives that cannot satisfy the extension
expectations of the client. A server that detects this issue can either close
the connection or degrade into a mode that does not use the extension.

This document defines an error code for HTTP/2 and HTTP/3 that can be sent by an
endpoint when closing streams or connections due to a failure with extenstion
expectations. This clearly disambiguates the failure case from other types, i.e.
general protocol error, which can allow the peer to take more specific meaures,
such as identifying configuration errors.


# Conventions and Definitions

{::boilerplate bcp14}


# HTTP/2 Error Code

EXTENSION_UNSUPPORTED (0xf10): The endpoint detected that its peer wants to
use an extension that is not locally supported, or the peer does not support an
extension that is desired.

# HTTP/3 Error Code

H3_EXTENSION_UNSUPPORTED (0x1f10): The endpoint detected that its peer wants to
use an extension that is not locally supported, or the peer does not support an
extension that is desired.


# Security Considerations

None, probably.


# IANA Considerations

This specification registers the following entry in the HTTP/2 Error Code registry
established by {{HTTP2}}:

Name:
: EXTENSION_UNSUPPORTED

Code:
: 0xf10

Description:
: Extension unsupported

Specification:
: This document

This specification registers the following entry in the HTTP/3 Error Code registry
established by {{HTTP3}}:

Name:
: EXTENSION_UNSUPPORTED

Code:
: 0x1f10

Description:
: Extension unsupported

Specification:
: This document


--- back

# Acknowledgments
{:numbered="false"}

Patrick McManus, Mike Bishop and Ryan Hamilton shared some opinions on the error
cases of Alt-Svc, in various channels, that have been incorporated into this
document.
