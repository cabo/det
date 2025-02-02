---
v: 3

title: "CBOR: On Deterministic Encoding and Representation"
abbrev: "On Deterministic Encoding/Representation"
docname: draft-bormann-cbor-det-latest

category: info
submissiontype: IETF
date:
consensus: true
area: "Applications and Real-Time"
workgroup: CBOR
keyword:

venue:
  group: "Concise Binary Object Representation Maintenance and Extensions (CBOR)"
  mail: "cbor@ietf.org"
  github: cbor-wg/draft-ietf-cbor-cde

author:
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
  STD94:
    -: cbor
#    =: RFC8949
  I-D.ietf-cbor-cde: cde
  IANA.cbor-tags: tags
#  IANA.cbor-simple-values: simple
  RFC8610: cddl

informative:
  RFC9679: thumb
  STD63:
    -: utf8
#    =: RFC3629
  STD90:
    -: json
#    =: RFC8259
  STD96:
    -: cose
#    =: RFC9052
  RFC7493: ijson
  RFC3339:
  RFC9557: sedate
  IEEE754:
    target: https://ieeexplore.ieee.org/document/8766229
    title: IEEE Standard for Floating-Point Arithmetic
    author:
    - org: IEEE
    date: false
    seriesinfo:
      IEEE Std: 754-2019
      DOI: 10.1109/IEEESTD.2019.8766229
  ECMA262:
    target: https://www.ecma-international.org/publications/standards/Ecma-262.htm
    title: ECMAScript 2020 Language Specification
    author:
    - org: Ecma International
    date: 2020-06
    refcontent:
    - Standard ECMA-262, 11th Edition
  RFC8742: seq
  RFC8746: array
  I-D.ietf-cbor-edn-literals: edn
  I-D.mcnally-deterministic-cbor: dcbor-orig
  CTAP2:
    target: https://fidoalliance.org/specs/fido-v2.0-id-20180227/fido-client-to-authenticator-protocol-v2.0-id-20180227.html#ctap2-canonical-cbor-encoding-form
    title: Client to Authenticator Protocol (CTAP)
    author:
      org: FIDO Alliance
    date: 2018-02-27
    refcontent: CTAP2 canonical CBOR encoding form (in Section 6)
  RFC6838: mediatypes
  I-D.ietf-mediaman-suffixes: multiple

--- abstract

[^abs1-]

[^abs1-]:
    CBOR (STD 94, RFC 8949) defines "Deterministically Encoded CBOR" in
    its Section 4.2.  The present document provides additional information
    about use cases, deployment considerations, and implementation choices
    for Deterministic Encoding and Representation.

--- middle

# Introduction


The Concise Binary Object Representation (CBOR, {{STD94}} as documented
in RFC 8949) is a data format whose design goals include the
possibility of extremely small code size, fairly small message size,
and extensibility without the need for version negotiation.

In many cases, CBOR allows some information to be encoded in several
variants, which provide different amounts of space and thus lengths in
Bytes.
The encoder is generally free to choose the length that
is most practical for it (with the constraint, of course, that the
data need to fit).
For most encoders, it is natural to always choose the shortest form
available (essentially avoiding leading zeros).
{{Section 4.1 (Preferred Serialization) of RFC8949@STD94}} names this practice
and provides additional guidance for CBOR implementations; another
term in use is "Preferred Encoding".

{{Section 4.2 (Deterministically Encoded CBOR) of RFC8949@STD94}} goes beyond
the Preferred Serialization practice by providing rules for
*Deterministic Encoding*.
The objective of Deterministic Encoding is to, deterministically,
always produce the same encoding for data items that are equivalent at
the data model level.
To achieve this, Preferred Serialization is mandated, an encoding choice
intended for incremental encoding
(indefinite length encoding) is disabled, and additional effort is
expended for encoding key/value pairs in maps (the order of which
does not matter semantically) in a deterministic order.

Given that additional effort needs to be expended and/or implementation
choices are taken away, neither Preferred Serialization nor
Deterministic Encoding are mandatory in CBOR.
(Contrast this with UTF-8 ({{Section 3 of RFC3629@STD63}}), which is always treating as
"invalid" any encoding variants that are longer than necessary.)

Deterministic Encoding is defined in {{Section 4.2 of RFC8949@STD94}} (note
that {{Section 4.2.3 of RFC8949@STD94}} defines a variant that was needed at
the time for backward compatibility and will not be discussed further
in this document).
The present document elaborates on this normative definition by
providing additional information about use cases, deployment
considerations, and implementation choices for Deterministic Encoding;
it is an informational document that however may still be cited where a
single reference for the background of Deterministic Encoding is convenient.
This document is intended to be used in conjunction with CBOR Common
Deterministic Encoding (CDE, {{-cde}}), a normative specification that
clarifies {{Section 4.2 of RFC8949@STD94}} in order to allow
fully generic CBOR implementations that can provide common support for
a variety of applications of deterministic encoding.

## Conventions and Definitions

The definitions of {{STD94}} apply.
Readers are expected to be familiar with CBOR, and particularly so with
{{Sections 4.1 and 4.2 of RFC8949@STD94}}.

The following terms introduced in the text of {{STD94}} receive their own
separate definitions here:

Preferred Serialization:
: a set of choices made during Serialization (Encoding) that generally
  leads to shortest-form encodings where a choice of encoding lengths
  is available, without expending additional effort on converting
  between different kinds of data item.
  See {{Section 4.1 of RFC8949@STD94}} and the terms defined in that section.
  The Preferred Encoding rules for data items in the Basic Generic
  Data Model may be augmented by rules for specific Tags, see for
  instance {{Section 3.4.3 of RFC8949@STD94}}.

Preferred Encoding:
: Preferred Serialization

Deterministic Encoding:
: An encoding process that employs Preferred Serialization and makes
  additional decisions to always (deterministically) lead to the
  exact same encoding for equivalent inputs at the data model level.
  Similar to Preferred Serialization, the equivalence model as defined
  for the Basic Generic Data Model may be augmented by equivalence
  rules defined for specific Tags (see also {{Section 2.1 of
  RFC8949@STD94}}).

Deterministic Encoding assumes that the application already has formed
data items at the CBOR generic data model level, presumably in a way
that satisfies its determinism requirements
We add two terms to encompass the entire process from application data in
platform and application specific forms towards an encoded CBOR data
item:

Deterministic Representation:
: A representation process that employs application-layer processing
  of data into deterministically formed CBOR data items and then
  applies Deterministic Encoding to achieve a deterministic
  representation of the application-layer data.

Application-Layer Deterministic Representation (ALDR):
: The part of Deterministic Representation that is not covered by
  Deterministic Encoding.

ALDR rules:
: rules (or a ruleset) that achieves ALDR.
  A CBOR protocol that is described in terms of data in some platform
  or application specific form may set up ALDR rules to describe how
  an ALDR of these data is achieved, as input to Deterministic Encoding.

In this document, CBOR data items at the data model level are
represented in the CBOR diagnostic notation ({{Section 8 of
RFC8949@STD94}} as extended by {{Appendix G of -cddl}},
further elaborated in {{-edn}}), abbreviated with "EDN" (extended
diagnostic notation).

While this document is informative, it does use certain keywords to
indicate practical requirements for interoperability.
{::boilerplate bcp14-tagged-bcp14}


# Use Cases for Deterministic Encoding

Before discussing further details of Deterministic Encoding, we would
like to point out three areas of use cases, which differ enough in the
resulting objectives that it is worth to have terminology for them.

## Diagnostics

In many cases, diagnostic procedures benefit from having available a
single, easily comparable representation of some data:

* Comparing outputs of a test or validation suite during development

  * CI (Continuous Integration) may capture Deterministically Encoded copies of process output,
    of data in flight or data at rest, of specific test output etc.
    Being able to compare them over time or between systems without
    differences occurring as false positives can help indicate the presence or absence of
    certain problems.

  * Test vectors and other kinds of tests often represent some input
    and desired output of a transformation.
    By making sure the output is deterministically
    encoded, a simple bytewise comparison can find out whether the
    transformation was performed successfully.

* Improving the presentation of diagnostic information to humans

  By minimizing inconsequential differences between representations of
  similar data, humans may be faster in finding information they are
  interested in.  In particular inconsistent map ordering can easily
  hide information that would have been useful for diagnostic purposes.
  Transformation to human-readable forms may be easier and more useful
  if there is
  only one form of representation for the interchanged data.

## Caching

Many systems cache (memoize) results of a request so they can reply
with the cached result when the same request comes in again and the
context of the reply has not changed.

If two requests that are semantically the same also have the same
representation, the representation (or its hash) can serve as an
efficient cache key.  If the request is already encoded
deterministically, this is by definition the case; alternatively, the
recipient can re-encode a request with Deterministic Encoding.

Were the Deterministic Encoding to fail, this could lead to cache failures, which
could be benign, but also could be specifically evoked by an active
attacker to degrade a system.

As usual for deterministically encoded data, not all forms of
application equivalence imply equivalence at the data model level, so
some equivalence processing (*deterministic representation*) may be
required at the application level as well,
to achieve equivalent representations and thus a good cache hit rate.

## Security: Signing Inputs

Security Frameworks such as COSE and JOSE sign or MAC (authenticate with a
Message Authentication Code, MAC) information in
the form in which it
has actually been interchanged, making representation variants
less relevant.

(Note that {{Section 9 of RFC9052@STD96}} defines deterministic encoding
rules for its own derivation of signing inputs from interchange data
and additional cryptographic parameters; these are a compatible subset
of the Core Deterministic Encoding Requirements specified in {{Section
4.2.1 of RFC8949@STD94}} and thus of CDE.)

However, in some cases, the signing input for a signature or a MAC may
need to be derived from data at rest and/or specific transformations of
the data that was interchanged.  Such a transformation is fraught with
perils at the application level that may be exploited by attackers;
the present document focuses on Deterministic Encoding and just
mentions ALDR rules as a way to manage required application processing.
Deterministic Encoding may remove one potential source of variability
that might make signatures or MACs useless between systems.


# Support by Generic Encoders and Decoders

CBOR implementations can be specific to a particular application, or
they can be _Generic_.  There is a strong incentive to be able to use
a Generic encoder/decoder across the spectrum of CBOR applications;
CBOR applications that require specific support from an
encoder/decoder can considerably reduce the wide implementation
support CBOR enjoys from existing generic implementations.  So, as a
general best practice, we want to minimize the number of ways an
application may need to influence a generic coder/decoder by options,
flags, switches, etc.

## Basic Support

There is some expectation that, barring any particular constraints
that would make this more difficult than normally, a CBOR encoder will
use Preferred Encoding, in particular generic encoders.
Deterministic Encoding, however, will need to be switched on explicitly in
most implementations.
Note that Preferred Encoding, while using the shortest form available
for the specific data item to be encoded, doesn't have that shortness
as the overriding objective: Conversions of a data item into a
different one to achieve shorted encoding are not part of the
processing labeled "Preferred Encoding".
(This is particularly relevant for CBOR's different numeric systems;
see {{numeric}} below.)

Some applications will also want to check that an encoded input actually
satisfies the requirements for Deterministic Encoding.
By the definition of Deterministic Encoding, this can be done after
decoding a data item by
deterministically encoding the just decoded data item and comparing
the result with the decoding input.
However, specific support for checking immediately in the decoding
process can be more efficient.

As a result, support for Deterministic Encoding in generic encoder
implementations is RECOMMENDED to be provided by a flag to switch on
(or separate function that enables) Deterministic Encoding.
Similarly, generic decoders are RECOMMENDED to have a flag to switch
on/separate function to enable checking for Deterministic Encoding,
whether that is efficiently
implemented during decoding or less efficiently by comparing
a re-encoding.

## Application Requirements and Tags {#tags}

The definition of Deterministic Encoding can become more complicated
with the addition of Tags ({{Section 3.4 of RFC8949@STD94}}{{-tags}}).
Not all tags come with a strong equivalence model.
Worse, the equivalence model may be more
application specific than for basic Deterministic Encoding.

### Example with Tags 0 and 1 (Date/Time)

For instance, are the following Tag 0 timestamps (expressed in CBOR
diagnostic notation) equivalent?

<!-- 23 Oct 2013 14:52:23 -0700 -->

~~~
0("2013-10-23T21:52:23Z")
0("2013-10-23T21:52:23+00:00")
0("2013-10-23T14:52:23-07:00")
~~~

They all denote the same instance in time, so if that is the relevant application
semantics, they should all be represented as
`0("2013-10-23T21:52:23Z")` in Deterministic Encoding as that is the
shortest form.
However, they carry additional semantics that may be incidental or
intentional (the e-mail message from which this date/time example was taken originated
from California, which then was at a time zone the time offset of
which is expressed by the `-07:00`).
Whether the first two are exactly equivalent or not is the subject of
{{Section 2 of -sedate}}.

If the additional semantics conveyed by the `time-offset` ({{Section
5.6 of RFC3339}}) is not relevant to the application, an
application-specific (ALDR) rule may be needed to convert text-based
timestamps into the "Z" form before encoding.
Some applications may also process this timestamp as `1(1382565143)`,
losing the additional semantics as well, and using a quite different form.
Is that maybe an even better Deterministic Encoding?
(Note that `0("2016-12-31T23:59:60Z")` does not have an equivalent
form with Tag 1, so the application can either decide to never use
such a date/time, or to exceptionally encode the rare leap second with Tag 0.)

### Example with Major Types 0, 1, and 7, and Tags 2 and 3 {#numeric}

{:aside}
>
In some of the following examples, we will use the number
65 536 000 000 (or its floating-point form, 65536000000.0, in
diagnostic notation), as it has
both binary and non-binary (decimal) factors: it is equal to
`2`<sup>16</sup>⋅`10`<sup>6</sup> (and thus to
`2`<sup>22</sup>⋅`5`<sup>6</sup>).

CBOR has four different sets of numeric representations:

* Major types 0 and 1.

  These provide for a variable-length representation of 64-bit
  unsigned integer numbers (major type 0) or negative numbers (major
  type 1) and, by combining these, of 65-bit signed integer numbers.
  The various lengths are intended to be semantically without meaning; the
  Preferred Encoding always chooses the shortest one.

* Tags 2 and 3 ("bignums")

  These provide for a variable-length representation of arbitrarily
  large unsigned (Tag 2) or negative (Tag 3) integer numbers.
  According to {{Section 3.4.3 of RFC8949@STD94}},
  the Preferred Encoding of an integer that fits into major type 0 or
  1 is just that, i.e., the boundary between regular integers and
  bignums is intentionally thin.  This means that, in Preferred
  Encoding, the value space of integral numbers is cleanly split into
  basic integers (64-bit unsigned integers or 64-bit negative
  integers) and bignums (Tag 2/3 integers that fit into neither of the
  two 64-bit forms).

  As a result, an application may want to place any distinctions it
  needs in the area of integer numbers not on the representation as a
  regular integer or a bignum, but on the value: e.g., an application
  could provide a 64-bit signed integer range separate from a bignum-based
  arbitrary size integer range that is outside 64-bit signed space, and
  would map half of the 65-bit space into the arbitrary size range.


  Note that, accordingly, Preferred Encoding as defined in {{Section
  3.4.3 of RFC8949@STD94}} selects the shortest encoding in major type 0/1
  space if that is available and the shortest encoding (no leading
  zero bytes) in Tag 2/3 space only if the former is not available.
  This means that the integer number 65 536 000 000 in preferred
  representation is encoded as (9 bytes)

  ~~~ cbor-pretty
  1b 00 00 00 0f 42 40 00 00
  ~~~

  and not as (7 bytes)

  ~~~ cbor-pretty
  c2 45 0f 42 40 00 00
  ~~~

  (`2(h'0f 42 40 00 00')` in diagnostic notation), even though
  the latter is shorter by two bytes.


* Major type 7

  CBOR directly provides the {{IEEE754}} types binary16, binary32, and
  binary64, colloquially known as half-precision, single-precision,
  and double-precision floating point.  Note that other {{IEEE754}}
  binary floating types are indirectly supported via Tag 4, as well as
  decimal fractions via Tag 5.

  The set of values that binary32 and binary64 can represent are
  proper supersets of the value sets of the binary16 and binary32,
  respectively.  These sets have CDDL names of float16, float32, and
  float64 ({{Section 3.3 of -cddl}}).
  Again, preferred encoding chooses the smallest of the encodings;
  e.g., an application float64 such as 1.5 will be represented in a
  binary16 (0xf93e00) because that representation is the shortest floating
  point that provides the range and precision needed for this value.
  (Bulk encoding of floating point values, where the need for detection of this
  situation might cause a performance limitation, is handled by
  tagged arrays {{-array}}.)

  While the three major type 7 floating point representations are
  semantically equivalent among each other in the same way as the major type 0/1
  integer representations are to each other, implementers have
  indicated that between these
  two groups, numbers need to be kept separated into integers and
  floating point numbers at the generic data model level.

  This means that the integer number 65 536 000 000 in preferred
  representation is encoded as (9 bytes)

  ~~~
  1b 00 00 00 0f 42 40 00 00
  ~~~

  and not as (5 bytes)

  ~~~
  fa 51 74 24 00
  ~~~

  which would be considered to be the semantically separate floating point value
  65536000000.0 (CBOR diagnostic notation).


* Tag 4 and 5 (decimal fractions, "bigfloats")

  Instead of adopting further formats such as decimal64 or binary128 from
  {{IEEE754}}, CBOR defines two generalized tags that can be used for extended
  precision representation: Tag 5 for general binary floating point
  numbers ("bigfloats") and Tag 4 for general decimal floating point
  (decimal fractions).
 {{Section 3.4.4 of
  RFC8949@STD94}} also states that "Bigfloats may also be used by constrained
  applications that need some basic binary floating-point capability
  without the need for supporting IEEE 754", while decimal fractions
  "are most useful if an application needs the exact representation of
  a decimal fraction such as 1.1 because there is no exact
  representation for many decimal fractions in binary floating-point
  representations", as might occur when representing literal JSON
  {{STD90}} instead of I-JSON-interpreted JSON {{-ijson}}.

  Neither bigfloats nor decimal fractions provide rules for preferred
  encoding, except implicitly by providing a choice between basic
  integer and bignum representation for the mantissa value that will
  in turn
  be governed by the preferred encoding rules for integers.
  Beyond that, the assumption is that these Tags create separate
  number spaces, and that any deterministic representation of numbers
  via these tags is shaped by application rules for the use of Tag 4
  and 5.

# Specification Considerations

In many specifications, asserting that interchange is based on
deterministically encoded data items (and specifying what has to
happen if that is not the case) is all that is needed.

## Media Type Considerations

Some specifications define a media type for their interchange formats.
This definition is a good place to reiterate that a deterministically
encoded data item is required for instances of that media type.

A question arises whether a Structured Syntax Suffix (SSS, {{RFC6838}})
should be defined for CBOR data items in Deterministic Encoding
(and similarly for CBOR sequences {{-seq}} of such).

There is precedent for this approach, as ASN.1 DER (Distinguished
Encoding Rules) has an SSS, `+der`.
However, this appears misguided as the purpose of an SSS is to enable
processing of the underlying data representation format, and any ASN.1
BER (Basic Encoding Rules) processor (`+ber`) can also process a
`+der` instance, which is not apparent from the `+der` suffix.
(This was maybe mitigated by introducing both SSS at the same time.)
Similarly, any CBOR decoder today can process deterministically
encoded data items as plain CBOR data items (without any mitigation of
having introduced a related suffix at the same time), so the SSS
should be the usual `+cbor`/`+cbor-seq`.
(The additional processing that would be enabled by identifying data
items as deterministically encoded appears rather limited.)

Alternatively, instead of replacing `+cbor`, an indication of
Deterministic Encoding could be provided by adding multiple suffixes
to the SSS concept.
There is an ongoing effort to define a more complex structure of media
type suffixes, as documented in {{-multiple}}.
In general, the combination of multiple SSS in one media type name
raises similar questions to the multiple inheritance problem in
object-oriented programming languages, so it may not be easy to use
such a mechanism in practice.

## The Need for Maps

As an extension to JSON objects in JSON {{STD90}},
maps are an important data structure in the CBOR generic data model to
obtain extensibility of "struct"-like data (see {{Section 2 of
RFC8610}}).
Where this is not needed or can be provided in another way, expressing
the entire data item without the use of maps can be an efficient
option, avoiding any additional processing for Deterministic Encoding
beyond that needed for Preferred Encoding.
(This requires ensuring that a similar kind of uncertainty then does not occur
at the application level, though.)

# Implementation Considerations for Deterministic Encoding

## API Considerations

Support for Deterministic Encoding can be added to an API for a
generic CBOR encoder and decoder by adding one flag each:

* a flag for the encoder to produce Deterministic Encoding
* a flag for the decoder to check for Deterministic Encoding (optional)

Additional elements could be added to a decoder API to give diagnostic
information about inputs that were not deterministically encoded, e.g.,
by flagging elements with error codes.  It is often useful to give the
application full information about well-formed CBOR that is not
deterministically encoded even when it should be.
However, if a flag for checking is provided and switched on, there
SHOULD be no chance that any other decoded data item is mistaken for
one that was encoded deterministically.

As reordering maps for Deterministic Encoding is relatively expensive,
a generic encoder can also offer additional APIs for providing map
content in a pre-ordered form.  If an encoder complies with Preferred
Encoding and maps can be supplied in ordered form, an explicit
Deterministic Encoding flag may not be required.  If it is, it is
RECOMMENDED that the encoder not simply rely on the assumption that
inputs were properly ordered by the application.

## Map Key Ordering

Generating deterministically encoded data items requires arranging
key/value pairs in maps into an order defined in {{Section 4.2.1 of
RFC8949@STD94}}.

This map is ordered by the byte-wise lexicographic ordering of the
deterministically encoded map keys.
{{Section 4.2.1 of RFC8949@STD94}} notes:

{:quote}
>
Implementation note: the self-delimiting nature of the CBOR
encoding means that there are no two well-formed CBOR encoded
data items where one is a prefix of the other.
The bytewise lexicographic comparison of deterministic encodings of
different map keys therefore always ends in a position where the byte
differs between the keys, before the end of a key is reached.

Also, an implementation may be able to make use of the property that
map keys in Deterministic Encodings are ordered by the following
information, in order of precedence:

* the key's major type
* the numeric value of the argument of the key
* any content of the key data item, such as
  * the string value in a byte or text string key
  * the elements of an array key, in order
  * the key/value pairs of a map-shaped key, deterministically ordered
  * the tag content of a tagged key

I may be expeditious to use this property, e.g. by processing integers
first, starting with unsigned integers in ascending order and then
negative integers in descending order, and then strings (byte strings
first), ordered by their length in bytes (encoded in the argument) and
then the string content, arrays ordered by length and then content,
and maps ordered by length and then content.
Often, and particularly with integer and string keys, it may not be
necessary to actually build a deterministically encoded data item for
a map key to perform the overall map content ordering.

# Application Rules for Deterministic Representation (ALDR Rules)

To enable the use of generic encoders,
applications are encouraged to define (ALDR) rules for representing
application information in the CBOR generic data model that enable
the use of Preferred Encoding on that level as well.

## The need for CBOR Common Deterministic Encoding (CDE)

Applications can also define their own deterministic encoding rules,
as for instance FIDO CTAP2 (Client to Authenticator Protocol {{CTAP2}})
does with the *CTAP2 canonical CBOR encoding form* (Section 6 of {{CTAP2}}).
Its description appears
to be derived from an equivalent of {{Section 4.2.3 of RFC8949@STD94}}.
(The actual
structure of CTAP2 limits its use to cases where that is compatible
with standard Deterministic Encoding and thus CDE; there is text in the
specification that calls for revisiting the definition when this would no
longer be the case.)

Application-specific deterministic encoding rules can make it
difficult to use existing generic encoders and may therefore diminish
the value of using a standard representation format.

Instead, applications can define (ALDR) transformations of their data
into a more limited data model that reduces the cases the
Deterministic Encoding rules have to implement.
This allows both the following implementation choices:

* the use of generic encoders with standard Deterministic Encoding
rule implementations after some application processing, or
* the use of specialized encoders which combine encoding with the
implementation of the application transformations.

The next subsection describes some of the considerations that led to
one such application-layer specification for deterministic
representation at the data model level, building on CDE for
deterministic encoding of these data.

## Numeric Reduction in dCBOR {#reduction}

The dCBOR specification {{-dcbor-orig}} describes the pervasive use of Deterministic
Encoding throughout an application.  It also defines a simplified
application data model of numbers, where there no longer is a distinction
between integers and floating point numbers at the application data
model level — all numbers are of a
single numeric type, and the choice of integer or floating point
representations is made based on value:

* integral numbers that fit into Major Type 0 and 1 are represented in
  this way even if they were originally represented as floating point values;
* all other numbers are represented as floating point
  values (and all NaN values are mapped to a single quiet NAN).

The underlying CBOR Deterministic Encoding rules ensure that, in both
cases, the shortest form for the case will then be used for encoding.

Reducing the separate integer and floating point spaces to a single
numeric space is particularly attractive in implementation languages that
also only have a single such space, such as JavaScript {{ECMA262}}.
(While JavaScript recently has acquired a separate integer type, it is much
less well integrated into the language and existing libraries than the
more well-established general numeric type.)

Within the CBOR working group of the IETF, the dCBOR specification
prompted a discussion about "profiles" for deterministic encoding, which
led to the CBOR Common Deterministic Encoding (CDE) specification
{{-cde}} and the concept of rulesets for deterministic representation
at the application data model level (ALDR, see {{Appendix A of -cde}}).
Without help of the CDE specification at the time, an early version of
the dCBOR specification restated much of {{Section 4.2 of
RFC8949@STD94}} and added a rule that gets in the way of compatibility
with Deterministic Encoding (disallowing the interchange of basic
negative integers in the range `-2`<sup>64</sup> to
-`2`<sup>63</sup>`-1`).

{:aside}
>
Early dCBOR specifications also had a mention as future work of
subnormal values {{IEEE754}}, which work fine for interchange (even with
Deterministic Encoding) in {{STD94}}.
Note that specific values may not be available to applications that
employ floating point hardware implementing the FTZ ("flush to zero")
and/or DAZ ("denormals are zero") strategies.
These may then require special handling in the application data model.
It is generally difficult to
rely on exact equality of floating point values, which however is what
Deterministic Encoding requires.

## ALDR in {{-thumb}}

{{-thumb}} describes a process that derives a deterministic thumbprint
from a COSE Key {{-cose}}.
{{Section 4 of -thumb}} defines the application-layer subsetting that is
applied to the COSE Key to derive the input for the deterministic
encoding that finally supplies the input to a hashing process; this
section essentially serves as the ALDR ruleset for this specification.
The paragraph at the end of {{Section 4.2 of -thumb}} is an example for
an application-specific *reduction*: the ALDR ruleset only allows
uncompressed point representations, and it therefore needs a process
to derive these: "if an implementation uses the compressed point
representation, it MUST first convert it to the uncompressed form for
the purpose of thumbprint calculation".

# Using Deterministically Encoded CBOR as a Deterministic Encoding of JSON

Certain applications could make use of a Deterministic Encoding for
JSON {{STD90}} data.
Deterministically Encoded CBOR provides an attractive solution to that
as it is already well-defined.

While the data model of JSON is not well-defined, I-JSON provides one
interpretation that is generally accepted {{-ijson}}.
{{Section 6.2 (Converting from JSON to CBOR) of RFC8949@STD94}} provides a way
to transform JSON data that conform to this data model to CBOR.
When used with its default parameters, the combination of (1) I-JSON,
(2) the
JSON-to-CBOR transformation, and (3) the rules for CBOR Deterministic
Encoding provide a well-defined Deterministic Encoding for JSON data.

Transforming decoded CBOR data after interchange back to data-model
level JSON data can be done with the inverse of {{Section 6.2 of RFC8949@STD94}}
(the full generality of {{Section 6.1 (Converting from CBOR to JSON) of
RFC8949@STD94}} is obviously not required as only the JSON subset of the CBOR
generic data model is used).

Comparing the handling of numeric data in the JSON-to-CBOR
transformation to that reported in {{reduction}}, the main difference is
that the former only maps integral values between
`-2`<sup>53</sup>`+1` and `2`<sup>53</sup>`-1` to basic CBOR integers
and leaves the others in floating point form.
(The rationale is that only this range is injective ("unambiguous" or
"exact") in the mapping of integers to binary64 floating point
values, which may be a desirable property beyond the use in JSON
encoding.)

# Security Considerations

One of the major use cases of Deterministic Encoding is in security,
namely in the derivation of signing inputs from some CBOR data only
available at the model level.
Any transformation error from the application data to the CBOR model
level and then to deterministic encoding can lead to a potential exploit
by an attacker.

Pertinent Security Considerations are further discussed {{Section 8 of
-cde}}.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document was motivated by the work of Wolf McNally and
Christopher Allen as documented in {{-dcbor-orig}} and discussed in 2023
in the CBOR working group.
It collects information that is present in the apps-discuss and CBOR
WG mailing list discussions since 2013, but not necessarily easy to
find.
The author is grateful to the many contributors to these discussions.


<!--  LocalWords:  deterministically
 -->
