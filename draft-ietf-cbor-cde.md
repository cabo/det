---
v: 3

title: CBOR Common Deterministic Encoding (CDE)
abbrev: CBOR CDE
docname: draft-ietf-cbor-cde-latest
category: bcp
stream: IETF

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
contributor:
- name: Laurence Lundblade
  org: Security Theory LLC
  email: lgl@securitytheory.com
  contribution: Laurence provided most of the text that became {{impcheck}}.

normative:
  STD94: cbor
#    =: RFC8949
  IEEE754:
    target: https://ieeexplore.ieee.org/document/8766229
    title: IEEE Standard for Floating-Point Arithmetic
    author:
    - org: IEEE
    date: false
    seriesinfo:
      IEEE Std: 754-2019
      DOI: 10.1109/IEEESTD.2019.8766229
  RFC8610: cddl
  IANA.cddl:


informative:
  I-D.bormann-cbor-det: det
  I-D.mcnally-deterministic-cbor: dcbor
  I-D.bormann-cbor-numbers: numbers
  UAX-15:
    title: "Unicode Normalization Forms"
    rc: Unicode Standard Annex #15
    target: https://unicode.org/reports/tr15/
    date: false

--- abstract

[^abs1-]

[^abs1-]:
    CBOR (STD 94, RFC 8949) defines "Deterministically Encoded CBOR" in
    its Section 4.2, providing some flexibility for application specific
    decisions.
    To facilitate Deterministic Encoding to be offered as a selectable
    feature of generic encoders, the present document defines a
    CBOR Common Deterministic Encoding (CDE) Profile that can be shared by a
    large set of applications with potentially diverging detailed
    requirements.
    It also defines "Basic Serialization", which stops short of the
    potentially more onerous requirements that make CDE fully
    deterministic, while employing most of its reductions of the
    variability needing to be handled by decoders.

--- middle

# Introduction

[^abs1-]

## Structure of This Document

After introductory material, {{dep}} defines the CBOR Common
Deterministic Encoding Profile (CDE).
{{cddl-support}} defines Concise Data Definition Language (CDDL) support for indicating the use of CDE.
This is followed by the conventional sections for
{{<<seccons}} ({{<seccons}}),
{{<<sec-iana}} ({{<sec-iana}}),
and {{<<sec-combined-references}} ({{<sec-combined-references}}).

The informative {{impcheck}} provides brief checklists that implementers
can use to check their CDE implementations.
{{ps}} provides a checklist for implementing Preferred Serialization.
{{bs}} introduces "Basic Serialization", a slightly more restricted form
of Preferred Serialization that may be used by encoders to hit a sweet
spot for maximizing interoperability with partial (e.g., constrained)
CBOR decoder implementations.
{{cde}} further restricts Basic Serialization to arrive at CDE.

Instead of giving rise to the definition of application-specific,
non-interoperable variants of CDE, this document identifies
Application-level Deterministic Representation (ALDR) rules as a
concept that is separate from CDE itself ({{aldr}}).
ALDR rules are layered on top of the CDE and address
requirements on deterministic representation of application data
that are specific to an application or a set of applications.
ALDR rules are often provided with a specification for a CBOR-based
protocol, or, if needed, can be provided by referencing a shared "ALDR
ruleset" that is defined in a separate document.

## Conventions and Definitions

The conventions and definitions of {{-cbor}} apply.

The term "CBOR Application" ("application" for short) is not
explicitly defined in {{-cbor}}; this document uses it in the same sense
as it is used there, specifically for applications that use CBOR as an
interchange format and use (often generic) CBOR encoders/decoders to
serialize/ingest the CBOR form of their application data to be
exchanged.
Similarly, "CBOR Protocol" is used as in {{-cbor}} for the protocol that
governs the interchange of data in CBOR format for a specific
application or set of applications.

{::boilerplate bcp14-tagged-bcp14}

# CBOR Common Deterministic Encoding Profile (CDE) {#dep}

This specification defines the *CBOR Common Deterministic Encoding
Profile* (CDE) based on the _Core Deterministic Encoding
Requirements_ defined for CBOR in
{{Section 4.2.1 of RFC8949@-cbor}}.

In many cases, CBOR provides more than one way to encode a data item,
but also provides a recommendation for a *Preferred Serialization*.
The *CoRE Deterministic Encoding Requirements* generally pick the
preferred serializations as mandatory; they also pick additional choices
such as definite-length encoding.
Finally, they define a map ordering based on lexicographic ordering of
the (deterministically) encoded map keys.

Note that this specific set of requirements is elective — in
principle, other variants of deterministic encoding can be defined
(and have been, now being phased out slowly, as detailed in {{Section 4.2.3
of RFC8949@-cbor}}).
In many applications of CBOR today, deterministic encoding is not used
at all, as its restriction of choices can create some additional
performance cost and code complexity.

{{-cbor}}'s core requirements are designed to provide well-understood and
easy-to-implement rules while maximizing coverage, i.e., the subset of
CBOR data items that are fully specified by these rules, and also
placing minimal burden on implementations.

{{Section 4.2.2 of RFC8949@-cbor}} picks up on the interaction of extensibility
(CBOR tags) and deterministic encoding.
CBOR itself uses some tags to increase the range of its basic
generic data types, e.g., tags 2/3 extend the range of basic major
types 0/1 in a seamless way.
{{Section 4.2.2 of RFC8949@-cbor}} recommends handling this transition the same
way as with the transition between different integer representation
lengths in the basic generic data model, i.e., by mandating the
preferred serialization for all integers ({{Section 3.4.3 of RFC8949@-cbor}}).

{: group="1"}
1. CDE turns this
   recommendation into a mandate: Integers that can be represented by
   basic major type 0 and 1 are encoded using the deterministic
   encoding defined for them, and integers outside this range are
   encoded using the preferred serialization ({{Section 3.4.3 of
   RFC8949@-cbor}}) of tag 2 and 3 (i.e., no leading zero bytes).

Most tags capture more specific application semantics and therefore
may be harder to define a deterministic encoding for.
While the deterministic encoding of their tag internals is often
covered by the _Core Deterministic Encoding Requirements_, the mapping
of diverging platform application data types onto the tag contents may
require additional attention to perform it in a deterministic way; see
{{Section 3.2 of -det}} for
more explanation as well as examples.
As the CDE would continually
need to address additional issues raised by the registration of new
tags, this specification recommends that new tag registrations address
deterministic encoding in the context of CDE.

A particularly difficult field to obtain deterministic encoding for is
floating point numbers, partially because they themselves are often
obtained from processes that are not entirely deterministic between platforms.
See {{Section 3.2.2 of -det}} for more details.
{{Section 4.2.2 of RFC8949@-cbor}} presents a number of choices; these need to
be made to obtain the CBOR Common Deterministic Encoding Profile (CDE).
Specifically, CDE specifies (in the order of the bullet list at the end of {{Section
4.2.2 of RFC8949@-cbor}}):

{: group="1"}
2. Besides the mandated use of preferred serialization, there is no further
   specific action for the two different zero values, e.g., an encoder
   that is asked by an application to represent a negative floating
   point zero will generate 0xf98000.
3. There is no attempt to mix integers and floating point numbers,
   i.e., all floating point values are encoded as the preferred
   floating-point representation that accurately represents the value,
   independent of whether the floating point value is, mathematically,
   an integral value (choice 2 of the second bullet).
4. Apart from finite and infinite numbers, {{IEEE754}} floating point
   values include NaN (not a number) values {{-numbers}}.
   In CDE, there is no special handling of NaN values, except that the
   preferred serialization rules also apply to NaNs (with zero or
   non-zero payloads), using the canonical encoding of NaNs as defined
   in Section 6.2.1 of {{IEEE754}}.
   Specifically, this means that shorter forms of encodings for a NaN
   are used when that can be achieved by only removing trailing zeros
   in the NaN payload (example serializations are available in
   {{Section A.1.2 of -numbers}}).
   Further clarifying a "should"-level statement in Section 6.2.1 of
   {{IEEE754}}, the CBOR encoding always uses a leading bit of 1 in the
   significand to encode a quiet NaN; the use of signaling NaNs by
   application protocols is NOT RECOMMENDED but when presented by an
   application these are encoded by using a leading bit of 0.

   Typically, most applications that employ NaNs in their storage and
   communication interfaces will only use a single NaN value, quiet NaN with payload 0,
   which therefore deterministically encodes as 0xf97e00.
5. There is no special handling of subnormal values.
6. CDE does not presume
   equivalence of basic floating point values with floating point
   values using other representations (e.g., tag 4/5).
   Such equivalences and related deterministic representation rules
   can be added at the ALDR level if desired, e.g., by stipulating
   additional equivalences and by restricting the set of data item values
   actually used by an application.

The main intent here is to preserve the basic generic data model, so
applications (in their ALDR rules or by referencing a separate ALDR
ruleset document, see
{{aldr}}) can
make their own decisions within that data model.
E.g., an application's ALDR rules can decide that it only ever allows a
single NaN value that would be encoded as 0xf97e00, so a CDE
implementation focusing on this application would not need to
provide processing for other NaN values.
Basing the definition of both CDE and ALDR rules on the
generic data model of CBOR also means that there is no effect on the
Concise Data Definition Language (CDDL)
{{-cddl}}, except where the data description is documenting specific
encoding decisions for byte strings that carry embedded CBOR.

--- back

# Application-level Deterministic Representation Rules {#aldr}

This appendix is informative.

While the CDE provides
for commonality between different applications of CBOR, for an
application or a group of applications it can be useful
to further constrain the set of data items handled (_exclusions_)
and to define further mappings
(_reductions_) that help the application(s) get by with
the exclusions.

For example, the dCBOR specification {{-dcbor}} specifies the use of CDE
together with some application-level rules, i.e., an ALDR ruleset, such as a
requirement for all text strings to be in Unicode Normalization Form C
(NFC) {{UAX-15}} — this specific requirement is an example for an _exclusion_ of
non-NFC data at the application level, and it invites implementing a _reduction_ by
routine normalization of text strings.

ALDR rules (including rules specified in a ALDR ruleset document) enable
simply using the shared CDE; they do not
"fork" CBOR in the sense of requiring distinct generic encoder/decoder
implementations.

An implementation of specific ALDR rules combined with a CDE
implementation produces well-formed,
deterministically encoded CBOR according to {{STD94}}, and existing
generic CBOR decoders will therefore be able to decode it, including
those that check for Deterministic Encoding ("CDE decoders", see also
{{impcheck}}).
Similarly, generic CBOR encoders will be able to produce valid CBOR
that can be ingested by an implementation that enforces an application's
ALDR rules if the encoder was handed data model level information
from an application that simply conformed to those ALDR rules.

Please note that the separation between standard CBOR processing and
the processing required by the ALDR rules is a conceptual one:
Instead of employing generic encoders/decoders, both ALDR rule
processing and standard CBOR processing can be combined into a specialized
encoder/decoder specifically designed for a particular set of ALDR
rules.

ALDR rules are intended to be used in conjunction with an
application, which typically will naturally use a subset of the CBOR generic
data model, which in turn
influences which subset of the ALDR rules is used by the specific application
(in particular if the application simply references a more general
ALDR ruleset document).
As a result, ALDR rules themselves place no direct
requirement on what minimum subset of CBOR is implemented.
For instance, a set of ALDR rules might include rules for the
processing of floating point values, but there is no requirement that
implementations of that set of ALDR rules support floating point
numbers (or any other kind of number, such as arbitrary precision
integers or 64-bit negative integers) when they are used with
applications that do not use them.

--- middle

# CDDL support

CDDL defines the structure of CBOR data items at the data model level;
it enables being specific about the data items allowed in a particular
place.
It does not specify encoding, but CBOR protocols can specify the use
of CDE (or simply Basic Serialization).
For instance, it allows the specification of a floating point data item
as "float16"; this means the application data model only foresees data
that can be encoded as {{IEEE754}} binary16.
Note that specifying "float32" for a floating point data item enables
all floating point values that can be represented as binary32; this
includes values that can also be represented as binary16 and that will
be so represented in Basic Serialization.

{{-cddl}} defines control operators to indicate that the contents of a
byte string carries a CBOR-encoded data item (`.cbor`) or a sequence of
CBOR-encoded data items (`.cborseq`).

CDDL specifications may want to specify that the data items should be
encoded in Common CBOR Deterministic Encoding.
The present specification adds two CDDL control operators that can be used
for this.

The control operators `.cde` and `.cdeseq` are exactly like `.cbor` and
`.cborseq` except that they also require the encoded data item(s) to be
encoded according to CDE.

For example, a byte string of embedded CBOR that is to be encoded
according to CDE can be formalized as:

~~~
leaf = #6.24(bytes .cde any)
~~~

More importantly, if the encoded data item also needs to have a
specific structure, this can be expressed by the right-hand side
(instead of using the most general CDDL type `any` here).

(Note that the `.cborseq` control operator does not enable specifying
different deterministic encoding requirements for the elements of the
sequence.  If a use case for such a feature becomes known, it could be
added.)


Obviously, specifications that document ALDR rules can define related control operators
that also embody the processing required by those ALDR rules,
and are encouraged to do so.


# Security Considerations {#seccons}

The security considerations in {{Section 10 of RFC8949@-cbor}} apply.
The use of deterministic encoding can mitigate issues arising out of
the use of non-preferred serializations specially crafted by an attacker.
However, this effect only accrues if the decoder actually checks that
deterministic encoding was applied correctly.
More generally, additional security properties of deterministic
encoding can rely on this check being performed properly.

# IANA Considerations {#sec-iana}

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFCXXXX with the RFC
    number of this RFC and remove this note.

This document requests IANA to register the contents of
{{tbl-iana-reqs}} into the registry
"{{cddl-control-operators (CDDL Control Operators)<IANA.cddl}}" of the
{{IANA.cddl}} registry group:

| Name      | Reference |
| .cde      | \[RFCXXXX] |
| .cdeseq   | \[RFCXXXX] |
{: #tbl-iana-reqs title="New control operators to be registered"}


--- back

# Implementers' Checklists {#impcheck}

This appendix is informative.
It provides brief checklists that implementers can use to check their
implementations.
It uses {{RFC2119}} language, specifically the keyword MUST, to highlight
the specific items that implementers may want to check.
It does not contain any normative mandates.
This appendix is informative.

Notes:

* This is largely a restatement of parts of {{Section 4 of
  RFC8949@-cbor}}.
  The purpose of the restatement is to aid the work of implementers,
  not to redefine anything.

  Preferred Serialization Encoders and Decoders as well as CDE
  Encoders and Decoders have certain properties that are expressed
  using {{RFC2119}} keywords in this appendix.

* Duplicate map keys are never valid in CBOR at all (see
  list item "Major type 5" in {{Section 3.1 of RFC8949@-cbor}})
  no matter what sort of serialization is used.
  Of the various strategies listed in {{Section 5.6 of RFC8949@-cbor}},
  detecting duplicates and handling them as an error instead of
  passing invalid data to the application is the most robust one;
  achieving this level of robustness is a mark of quality of
  implementation.

* Preferred serialization and CDE only affect serialization.
  They do not place any requirements, exclusions, mappings or such on
  the data model level.
  ALDR rules such as the ALDR ruleset defined by dCBOR are different as they can affect
  the data model by restricting some values and ranges.

* CBOR decoders in general (as opposed to "CDE decoders" specifically
  advertised as supporting CDE)
  are not required to check for preferred
  serialization or CDE and reject inputs that do not fulfill
  their requirements.
  However, in an environment that employs deterministic encoding,
  employing non-checking CBOR decoders negates many of its benefits.
  Decoder implementations that advertise "support" for preferred
  serialization or CDE need to check the encoding and reject
  input that is not encoded to the encoding specification in use.
  Again, ALDR rules such as those in dCBOR may pose additional
  requirements, such as requiring rejection of non-conforming inputs.

  If a generic decoder needs to be used that does not "support" CDE, a
  simple (but somewhat clumsy) way to check for proper CDE encoding is
  to re-encode the decoded data and check for bit-to-bit equality with
  the original input.

## Preferred Serialization {#ps}

In the following, the abbreviation "ai" will be used for the 5-bit
additional information field in the first byte of an encoded CBOR data
item, which follows the 3-bit field for the major type.

### Preferred Serialization Encoders {#pse}

1. Shortest-form encoding of the argument MUST be used for all major
   types.
   Major type 7 is used for floating-point and simple values; floating
   point values have its specific rules for how the shortest form is
   derived for the argument.
   The shortest form encoding for any argument that is not a floating
   point value is:

   * 0 to 23 and -1 to -24 MUST be encoded in the same byte as the
     major type.
   * 24 to 255 and -25 to -256 MUST be encoded only with an additional
     byte (ai = 0x18).
   * 256 to 65535 and -257 to -65536 MUST be encoded only with an
     additional two bytes (ai = 0x19).
   * 65536 to 4294967295 and -65537 to -4294967296 MUST be encoded
     only with an additional four bytes (ai = 0x1a).

1. If floating-point numbers are emitted, the following apply:

   * The length of the argument indicates half (binary16, ai = 0x19),
     single (binary32, ai = 0x1a) and double (binary64, ai = 0x1b)
     precision encoding.
     If multiple of these encodings preserve the precision of the
     value to be encoded, only the shortest form of these MUST be
     emitted.
     That is, encoders MUST support half-precision and
     single-precision floating point.

   * {{IEEE754}} Infinites and NaNs, and thus NaN payloads, MUST be
     supported, to the extent possible on the platform.

     As with all floating point numbers, Infinites and NaNs MUST be
     encoded in the shortest of double, single or half precision that
     preserves the value:

     * Positive and negative infinity and zero MUST be represented in
       half-precision floating point.

     * For NaNs, the value to be preserved includes the sign bit,
     the quiet bit, and the NaN payload (whether zero or non-zero).
     The shortest form is obtained by removing the rightmost N bits of the
     payload, where N is the difference in the number of bits in the
     significand (mantissa representation) between the original format
     and the shortest format.
     This trimming is performed only (preserves the value only) if all the
     rightmost bits removed are zero.
     (This will always represent a double or single quiet NaN with a zero
     NaN payload in a half-precision quiet NaN.)


1. If tags 2 and 3 are supported, the following apply:

   * Positive integers from 0 to 2^64 - 1 MUST be encoded as a type 0 integer.

   * Negative integers from -(2^64) to -1 MUST be encoded as a type 1 integer.

   * Leading zeros MUST NOT be present in the byte string content of tag 2 and 3.

   (This also applies to the use of tags 2 and 3 within other tags,
   such as 4 or 5.)

### Preferred Serialization Decoders {#psd}

There are no special requirements that CBOR decoders need to meet to
be a Preferred Serialization Decoder.
Partial decoder implementations need to pay attention to at least the
following requirements:

1. Decoders MUST accept shortest-form encoded arguments (see {{Section
   3 of RFC8949@-cbor}}).

1. If arrays or maps are supported, definite-length arrays or maps MUST be accepted.

1. If text or byte strings are supported, definite-length text or byte
   strings MUST be accepted.

1. If floating-point numbers are supported, the following apply:

   * Half-precision values MUST be accepted.
   * Double- and single-precision values SHOULD be accepted; leaving these out
     is only foreseen for decoders that need to work in exceptionally
     constrained environments.
   * If double-precision values are accepted, single-precision values
     MUST be accepted.
   * Infinites and NaNs, and thus NaN payloads, MUST be accepted and
     presented to the application (not necessarily in the platform
     number format, if that doesn't support those values).

1. If big numbers (tags 2 and 3) are supported, type 0 and type 1 integers MUST
   be accepted where a tag 2 or 3 would be accepted.  Leading zero bytes
   in the tag content of a tag 2 or 3 MUST be ignored.

## Basic Serialization  {#bs}

Basic Serialization further restricts Preferred Serialization by not
using indefinite length encoding.
A CBOR encoder can choose to employ Basic Serialization in order to
reduce the variability that needs to be handled by decoders,
potentially maximizing interoperability with partial (e.g.,
constrained) CBOR decoder implementations.

### Basic Serialization Encoders {#bse}

The Basic Serialization Encoder requirements are identical to the
Preferred Serialization Encoder requirements, with the following additions:

1. If maps or arrays are emitted, they MUST use definite-length
   encoding (never indefinite-length).

1. If text or byte strings are emitted, they MUST use definite-length
   encoding (never indefinite-length).

### Basic Serialization Decoders {#bsd}

The Basic Serialization Decoder requirements are identical to the
Preferred Serialization Decoder requirements.

## CDE

### CDE Encoders

1. CDE encoders MUST only emit CBOR fulfilling the basic
   serialization rules ({{bse}}).

1. CDE encoders MUST sort maps by the CBOR representation of the map
   key.
   The sorting is byte-wise lexicographic order of the encoded map
   key data items.

1. CDE encoders MUST generate CBOR that fulfills basic validity
   ({{Section 5.3.1 of RFC8949@-cbor}}).  Note that this includes not
   emitting duplicate keys in a major type 5 map as well as emitting
   only valid UTF-8 in major type 3 text strings.

   Note also that CDE does NOT include a requirement for Unicode
   normalization {{UAX-15}}; {{Section C of
   ?I-D.bormann-dispatch-modern-network-unicode}} contains some
   rationale that went into not requiring routine use of Unicode normalization
   processes.

### CDE Decoders

The term "CDE Decoder" is a shorthand for a CBOR decoder that
advertises _supporting_ CDE (see the start of this appendix).

1. CDE decoders MUST follow the rules for preferred (and thus basic)
   serialization decoders ({{psd}}).

1. CDE decoders MUST check for ordering map keys and for basic
   validity of the CBOR encoding (see {{Section 5.3.1 of
   RFC8949@-cbor}}, which includes a check against duplicate map keys
   and invalid UTF-8).

   To be called a CDE decoder, it MUST NOT present to the application
   a decoded data item that fails one of these checks (except maybe via
   special diagnostic channels with no potential for confusion with a
   correctly CDE-decoded data item).

# Acknowledgments
{:numbered="false"}

An earlier version of this document was based on the work of Wolf
McNally and Christopher Allen as documented in {{-dcbor}}, which
serves as an example for an ALDR ruleset document.
We would like to explicitly acknowledge that this work has
contributed greatly to shaping the concept of a CBOR Common
Deterministic Encoding and ALDR rules/rulesets on top of that.
