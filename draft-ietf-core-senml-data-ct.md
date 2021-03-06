---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-senml-data-ct-latest
keyword: Internet-Draft
cat: std
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
abbrev: SenML Data Value Content-Format Indication
date: 2020
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
  IANA.senml:
informative:
  I-D.bormann-core-media-content-type-format: mtct

--- abstract

The Sensor Measurement Lists (SenML) media type supports multiple types
of values, from numbers to text strings and arbitrary binary data values.
In order to simplify processing of the data values this document proposes
to specify a new SenML field for indicating the Content-Format of the
data.

--- middle


# Introduction {#intro}

The Sensor Measurement Lists (SenML) media type {{!RFC8428}} can be used
to send various kinds of data.  In the example given in
{{ex-1}}, a temperature value, an indication whether a lock is open, and
a data value (with SenML field "vd") read from an NFC reader is sent in a
single SenML pack.

~~~
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
easily available to entities processing the SenML pack. To facilitate
automatic interpretation it is useful to be able to indicate an Internet
media type and content-coding right in the SenML Record. The CoAP
Content-Format (Section 12.3 in {{!RFC7252}}) provides just this
information; enclosing a Content-Format number (in this case number 60 as
defined for content-type application/cbor in {{RFC7049}}) in the Record is
illustrated in {{ex-2}}. All registered CoAP Content-Formats are listed
in the Content-Formats subregistry of the CoRE Parameters registry
{{?IANA.core-parameters}}.

~~~
{"n":"nfc-reader", "vd":"gmNmb28YKg", "ct":"60"}
~~~
{: #ex-2 title="SenML Record with binary data identified as CBOR"}

In this example SenML Record the data value contains a string "foo" and a
number 42 encoded in a CBOR {{?RFC7049}} array. Since the example above
uses the JSON format of SenML, the data value containing the binary CBOR
value is base64-encoded. The data value after base64 decoding is shown
with CBOR diagnostic notation in {{ex-2-cbor}}.

~~~
82           # array(2)
   63        # text(3)
      666F6F # "foo"
   18 2A     # unsigned(42)
~~~
{: #ex-2-cbor title="Example Data Value in CBOR diagnostic notation"}

# Terminology

{::boilerplate bcp14}

Readers should also be familiar with the terms and concepts discussed in
{{RFC8428}}. Awareness of terminology issues discussed in
{{-mtct}} can also be very helpful.

# SenML Content-Format ("ct") Field

When a SenML Record contains a Data Value field ("vd"), the Record MAY
also include a Content-Format indication field, using label "ct".  The
value of this field is a string value, one of:

* a CoAP Content-Format identifier in decimal form with no leading
  zeros (except for the value "0" itself).  This value represents an
  unsigned integer in the range of 0-65535, similar to the CoRE Link
  Format {{?RFC6690}} "ct" attribute).

* or a Content-Format-String {{-mtct}} containing a Content-Type and
  optionally a Content-Coding (see below).

The CoAP Content-Format identifier provides a simple and efficient way
to indicate the type of the data.  Since some Internet media types and
their content coding and parameter alternatives do not have assigned
CoAP Content-Format identifiers, using Content-Type and Content-Coding
is also allowed. Both methods use a string value in the "ct" field to
keep its data type consistent across uses.  When the "ct" field
contains only digits, it is interpreted as a CoAP Content-Format
identifier.

To indicate that a Content-Coding is used with a Content-Type, the
Content-Coding value (e.g., "deflate" {{?RFC7230}}) is appended to the
Content-Type value (media type and parameters, if any), separated by a "@"
sign.  For example: "text/plain; charset=utf-8@deflate".  If no "@" sign is
present outside the media type parameters, the Content-Coding is not
specified and the "identity" Content-Coding is used -- no
encoding transformation is employed.

# SenML Base Content-Format ("bct") Field

The Base Content-Format Field, label "bct", provides a default value for
the Content-Format Field (label "ct") within its range.  The range of the
base field includes the Record containing it, up to (but not including)
the next Record containing a "bct" field, if any, or up to the end of the
pack otherwise.  Resolution (Section 4.6 of {{RFC8428}}) of this base
field is performed by adding its value with the label "ct" to all Records
in this range that carry a "vd" field but do not already contain a
Content-Format ("ct") field.

# Examples

The following examples are valid values for the "ct" and "bct" fields
(explanation/comments in parenthesis):

* "60" (CoAP Content-Format for "application/cbor")
* "0" (CoAP Content-Format for "text/plain" with parameter
  "charset=utf-8")
* "application/json" (JSON Content-Type -- equivalent to "50" CoAP
  Content-Format identifier)
* "application/json@deflate" (JSON Content-Type with "deflate" as
  Content-Coding - equivalent to "11050" CoAP Content-Format identifier)
* "text/csv" (Comma-Separated Values (CSV) {{?RFC4180}} Content-Type)
* "text/csv@gzip" (CSV with "gzip" as Content-Coding)

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

IANA is requested to assign new labels in the "SenML Labels" subregistry
of the SenML registry {{IANA.senml}} (as defined in {{RFC8428}}) for the
Content-Format indication as per {{tbl-senml-reg}}:

| Name                | Label | JSON Type | XML Type | Reference |
| Base Content-Format | bct   | String    | string   | RFC-AAAA  |
| Content-Format      | ct    | String    | string   | RFC-AAAA  |
{: #tbl-senml-reg cols='r l l' title="IANA Registration for new SenML Labels"}

--- back

# Acknowledgements {#acks}
{: numbered="no"}

The authors would like to thank Sergio Abreu for the discussions leading
to the design of this extension and Isaac Rivera for reviews and
feedback.
Klaus Hartke suggested not burdening this draft with a separate
mandatory-to-implement version of the fields.
Alexey Melnikov, Jim Schaad, and Thomas Fossati provided helpful
comments at Working-Group last call.
