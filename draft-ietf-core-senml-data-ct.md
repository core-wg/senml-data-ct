---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-senml-data-ct-latest
keyword: Internet-Draft
cat: std
consensus: true
pi:
  toc: 'yes'
  tocompact: 'yes'
  tocdepth: '3'
  tocindent: 'yes'
  symrefs: 'yes'
  sortrefs: 'yes'
  comments: 'yes'
  inline: 'yes'
  strict: 'no'
  compact: 'no'
  subcompact: 'no'
title: SenML Data Value Content-Format Indication
abbrev: SenML Data Content-Format Indication
date: 2021
author:
- ins: A. Keränen
  name: Ari Keränen
  org: Ericsson
  street: ''
  city: Jorvas
  code: '02420'
  country: Finland
  email: ari.keranen@ericsson.com
-
  ins: C. Bormann
  name: Carsten Bormann
  org: Universität Bremen TZI
  street: Postfach 330440
  city: Bremen
  code: D-28359
  country: Germany
  phone: +49-421-218-63921
  email: cabo@tzi.org

normative:
  RFC2045: mime1
  RFC8428: senml
  IANA.senml:
  RFC7252: coap
  RFC5234: abnf
#  RFC7230: http0
#  RFC7231: http
  I-D.ietf-httpbis-semantics: http-new
  IANA.media-types: media-types
  IANA.core-parameters: core-parameters
  IANA.http-parameters: http-parameters
informative:
  RFC4648: base
  RFC8949: cbor
  RFC6838: mediatype-reg

--- abstract

The Sensor Measurement Lists (SenML) media type supports multiple types
of values, from numbers to text strings and arbitrary binary data values.
In order to facilitate processing of binary data values, this document
specifies a pair of new SenML fields for indicating the
content format of those binary data values, i.e., their Internet media
type including parameters as well as any content codings applied.

--- middle


# Introduction {#intro}

The Sensor Measurement Lists (SenML) media types {{-senml}} can be used
to send various kinds of data.  In the example given in
{{ex-1}}, a temperature value, an indication whether a lock is open, and
a data value (with SenML field "vd") read from an NFC reader is sent in a
single SenML pack.
The example is given in SenML JSON representation, so the "vd" (data
value) field is encoded as a base64url string (without
padding), as per {{Section 5 of -senml}}.

~~~ senml-json
[
  {"bn":"urn:dev:ow:10e2073a01080063:","n":"temp","u":"Cel","v":7.1},
  {"n":"open","vb":false},
  {"n":"nfc-reader","vd":"aGkgCg"}
]
~~~
{: #ex-1 title="SenML pack with unidentified binary data"}

The receiver is expected to know how to interpret the data in the "vd"
field based on the context, e.g., name of the data source and out-of-band
knowledge of the application. However, this context may not always be
easily available to entities processing the SenML pack, especially if
the pack is propagated over time and via multiple entities. To facilitate
automatic interpretation it is useful to be able to indicate an Internet
media type and, optionally, content codings right in the SenML Record.

The CoAP
Content-Format ({{Section 12.3 of -coap}}) provides this
information in the form of a single unsigned integer; enclosing a Content-Format number (in this case number 60 as
defined for content-type application/cbor in {{-cbor}}) in the Record is
illustrated in {{ex-2}}. All registered CoAP Content-Format numbers are listed
in the {{content-formats (COAP Content-Formats
registry)<IANA.core-parameters}} {{-core-parameters}} as specified by
{{Section 12.3 of -coap}}.
Note that, at the time of writing, the structure of this registry only
provides for zero or one content codings; nothing in the present
document needs to change if the registry is extended to allow
sequences of content codings.

~~~ json
{"n":"nfc-reader", "vd":"gmNmb28YKg", "ct":"60"}
~~~
{: #ex-2 title="SenML Record with binary data identified as CBOR"}

In this example SenML Record, the data value contains a string "foo" and a
number 42 encoded in a CBOR {{-cbor}} array. Since the example above
uses the JSON format of SenML, the data value containing the binary CBOR
value is base64-encoded ({{Section 5 of -base}}).
The data value after base64 decoding is shown
with CBOR diagnostic notation in {{ex-2-cbor}}.

~~~ cbor-pretty
82           # array(2)
   63        # text(3)
      666F6F # "foo"
   18 2A     # unsigned(42)
~~~
{: #ex-2-cbor title="Example Data Value in CBOR diagnostic notation"}

## Evolution

As with SenML in general, there is no expectation that the creator of
a SenML pack knows (or has negotiated with) each consumer of that pack,
which may be very remote in space and particularly in time.
This means that the SenML creator in general has no way to know
whether the consumer knows:

- each specific media-type-name used
- each parameter and each parameter value used
- each content coding in use
- each Content-Format number in use for a combination of these

What SenML, as well as the new fields defined here, guarantees is that
a recipient implementation *knows* when it needs to be updated to
understand these field values and the values controlled by them;
registries are used to evolve these name spaces in a controlled way.
SenML packs can be processed by a consumer while not understanding all
the information in them, and information can generally be preserved in
this processing such that it is useful for further consumers.

# Terminology

{::boilerplate bcp14-tagged}

Media Type:
: A registered label for representations (byte strings) prepared for
  interchange, identified by a Media-Type-Name {{?RFC1590}},
  {{-mediatype-reg}}.

Media-Type-Name:
: A combination of a type-name and a subtype-name registered in
  {{-media-types}} as per {{-mediatype-reg}}, conventionally
  identified by the two names separated by a slash.

Content-Type:
: A Media-Type-Name, optionally associated with parameters
  ({{Section 5 of -mime1}}, separated from
  the Media-Type-Name and from each other by a semicolon).
  In HTTP and many other protocols, used in a `Content-Type` header field.

content coding:
: A name registered in the {{content-coding (HTTP Content Coding
  registry)<IANA.http-parameters}} {{-http-parameters}} as specified by
  {{Sections 16.6.1 and 18.6 of -http-new}}, indicating an encoding
  transformation with semantics further specified in {{Section 8.4.1 of
  -http-new}}.
  Confusingly, in HTTP, content coding values are found in a header field
  called "Content-Encoding", however "content coding" is the correct
  term for the process and the registered values.

content format:
: the combination of a Content-Type and zero or more content codings, identified
  by (1) a numeric identifier defined in the {{content-formats (COAP
  Content-Formats registry)<IANA.core-parameters}} {{-core-parameters}}
  as per {{Section 12.3 of -coap}} (referred to as Content-Format
  number), or (2) a Content-Format-String.

Content-Format-String:
: the string representation of the combination of a Content-Type and
  zero or more content codings.

Content-Format-Spec:
: the string representation of a content format; either a
  Content-Format-String or the (decimal) string representation of a
  Content-Format number.

Readers should also be familiar with the terms and concepts discussed in
{{-senml}}.


# SenML Content-Format ("ct") Field

When a SenML Record contains a Data Value field ("vd"), the Record MAY
also include a Content-Format indication field, using label "ct".  The
value of this field is a Content-Format-Spec, i.e., one of:

* a CoAP Content-Format number in decimal form with no leading
  zeros (except for the value "0" itself).  This value represents an
  unsigned integer in the range of 0-65535, similar to the  "ct"
  attribute defined in {{Section 7.2.1 of -coap}} for CoRE Link
  Format {{?RFC6690}}).

* or a Content-Format-String containing a Content-Type and
  zero or more content codings (see below).

The syntax of this field is formally defined in {{abnf}}.

The CoAP Content-Format number provides a simple and efficient way
to indicate the type of the data.  Since some Internet media types and
their content coding and parameter alternatives do not have assigned
CoAP Content-Format numbers, using Content-Type and zero or more
content codings
is also allowed. Both methods use a string value in the "ct" field to
keep its data type consistent across uses.  When the "ct" field
contains only digits, it is interpreted as a CoAP Content-Format
number.

To indicate that one or more content codings are used with a Content-Type,
each of the content coding values is appended to the Content-Type value (media
type and parameters, if any), separated by a "@" sign, in the order of
the content codings were applied (the same order as in {{Section 8.4
of -http-new}}).
For example (using a content coding value of "deflate" as defined in
{{Section 8.4.1.2 of -http-new}}):

    text/plain; charset=utf-8@deflate

If no "@" sign is present after the media type and parameters,
then no content coding has been specified, and the "identity"
content coding is used — no encoding transformation is employed.

# SenML Base Content-Format ("bct") Field

The Base Content-Format Field, label "bct", provides a default value for
the Content-Format Field (label "ct") within its range.  The range of the
base field includes the Record containing it, up to (but not including)
the next Record containing a "bct" field, if any, or up to the end of the
pack otherwise.  The process of resolving ({{Section 4.6 of -senml}}) this base
field is performed by adding its value with the label "ct" to all Records
in this range that carry a "vd" field but do not already contain a
Content-Format ("ct") field.

{{ex-bct}} shows a variation of {{ex-2}} with multiple records, with the
"nfc-reader" records resolving to the base field value "60" and the
"iris-photo" record overriding this with the "image/png" media type
(actual data left out for brevity).

~~~ senml-json
[
  {"n":"nfc-reader", "vd":"gmNmb28YKg",
   "bct":"60", "bt":1627430700},
  {"n":"nfc-reader", "vd":"gmNiYXIYKw", "t":10},
  {"n":"iris-photo", "vd":".....", "ct":"image/png", "t":10},
  {"n":"nfc-reader", "vd":"gmNiYXoYLA", "t":20}
]
~~~
{: #ex-bct title="SenML pack with bct field"}




# Examples

The following examples are valid values for the "ct" and "bct" fields
(explanation/comments in parentheses):

* "60" (CoAP Content-Format number for "application/cbor")
* "0" (CoAP Content-Format number for "text/plain" with parameter
  "charset=utf-8")
* "application/json" (JSON Content-Type -- equivalent to "50" CoAP
  Content-Format number)
* "application/json@deflate" (JSON Content-Type with "deflate" as
  content coding -- equivalent to "11050" CoAP Content-Format number)
* "application/json@deflate@aes128gcm" (JSON Content-Type with
  "deflate" followed by "aes128gcm" as content codings)
* "text/csv" (Comma-Separated Values (CSV) {{?RFC4180}} Content-Type)
* "text/csv;header=present@gzip" (CSV with header row, using "gzip" as
  content coding)


# ABNF

This specification provides a formal definition of the syntax of
Content-Format-Spec strings using ABNF notation {{-abnf}}, which
contains three new rules and a number of rules collected and adapted
from various RFCs {{-http-new}} {{-mediatype-reg}} {{-abnf}} {{?RFC8866}}.

~~~~ abnf
; New in this document

Content-Format-Spec = Content-Format-Number / Content-Format-String

Content-Format-Number = "0" / (POS-DIGIT *DIGIT)
Content-Format-String   = Content-Type *("@" Content-Coding)

; Cleaned up from [RFC-httpbis-semantics],
; leaving only SP as blank space,
; removing legacy 8-bit characters, and
; leaving the parameter as mandatory with each semicolon:

Content-Type   = Media-Type-Name *( *SP ";" *SP parameter )
parameter      = token "=" ( token / quoted-string )

token          = 1*tchar
tchar          = "!" / "#" / "$" / "%" / "&" / "'" / "*"
               / "+" / "-" / "." / "^" / "_" / "`" / "|" / "~"
               / DIGIT / ALPHA
quoted-string  = %x22 *( qdtext / quoted-pair ) %x22
qdtext         = SP / %x21 / %x23-5B / %x5D-7E
quoted-pair    = "\" ( SP / VCHAR )

; Adapted from section 8.4.1 of [RFC-httpbis-semantics]

Content-Coding   = token

; Adapted from various specs

Media-Type-Name = type-name "/" subtype-name

; RFC 6838

type-name = restricted-name
subtype-name = restricted-name

restricted-name = restricted-name-first *126restricted-name-chars
restricted-name-first  = ALPHA / DIGIT
restricted-name-chars  = ALPHA / DIGIT / "!" / "#" /
                         "$" / "&" / "-" / "^" / "_"
restricted-name-chars =/ "." ; Characters before first dot always
                             ; specify a facet name
restricted-name-chars =/ "+" ; Characters after last plus always
                             ; specify a structured syntax suffix


; Boilerplate from RFC 5234 and RFC 8866

DIGIT     =  %x30-39           ; 0 – 9
POS-DIGIT =  %x31-39           ; 1 – 9
ALPHA     =  %x41-5A / %x61-7A ; A – Z / a – z
SP        =  %x20
VCHAR     =  %x21-7E           ; printable ASCII (no SP)

~~~~
{: #content-format-spec title="ABNF syntax of Content-Format-Spec"}

[^replace]

[^replace]: RFC editor: Please replace \[RFC-httpbis-semantics] by what gets
            published from {{-http-new}}.

# Security Considerations {#seccons}

The indication of a media type in the data does not exempt a consuming
application from properly checking its inputs.
Also, the ability for an attacker to supply crafted SenML data that
specify media types chosen by the attacker may expose vulnerabilities
of handlers for these media types to the attacker.
This includes "decompression bombs", compressed data that is crafted
to decompress to extremely large data items.

# IANA Considerations {#iana}

(Note to RFC Editor: Please replace all occurrences of "RFC-AAAA" with
the RFC number of this specification and remove this note.)

IANA is requested to assign new labels in the "SenML Labels"
{{subregistry<IANA.senml}}{: relative="#senml-labels"}
of the SenML registry {{IANA.senml}} (as defined in {{Section 12.2 of -senml}}) for the
Content-Format indication as per {{tbl-senml-reg}}:

| Name                | Label | JSON Type | XML Type | Reference |
| Base Content-Format | bct   | String    | string   | RFC-AAAA  |
| Content-Format      | ct    | String    | string   | RFC-AAAA  |
{: #tbl-senml-reg cols='r l l' title="IANA Registration for new SenML Labels"}

Note that as per {{Section 12.2 of -senml}}, no CBOR labels or EXI
schemaId values (EXI ID column) are supplied.

--- back

# Acknowledgements {#acks}
{: numbered="no"}

The authors would like to thank {{{Sérgio Abreu}}} for the discussions leading
to the design of this extension and {{{Isaac Rivera}}} for reviews and
feedback.
{{{Klaus Hartke}}} suggested not burdening this draft with a separate
mandatory-to-implement version of the fields.
{{{Alexey Melnikov}}}, {{{Jim Schaad}}}, and {{{Thomas Fossati}}} provided helpful
comments at Working-Group last call.
{{{Marco Tiloca}}} asked for clarifying and using the term Content-Format-Spec.
