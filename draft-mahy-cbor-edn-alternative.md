---
v: 3
title: "Extended Diagnostic Notation (EDN) for CBOR"
abbrev: "EDN"
docname: draft-mahy-cbor-edn-alternative-latest
# date: 2026-04-06

keyword:
  - cbor-diag
  - diagnostic notation
  - extended diagnostic notation
cat: std
stream: IETF
consensus: true
updates: 8610, 8742, 8949

pi: [toc, sortrefs, symrefs, compact, comments]
kramdown_options:
  ol_start_at_first_marker: true

author:
  -
    ins: R. Mahy
    fullname: Rohan Mahy
    role: editor
    email: "rohan.ietf@gmail.com"

contributor:
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

venue:
  mail: cbor@ietf.org
  github: cbor-wg/edn-literal
  latest: "https://cbor-wg.github.io/edn-literal/"

normative:
  RFC8949: cbor
  RFC8742: seq
  STD68: abnf
  STD63: utf8
  RFC7405: abnfcs
  RFC3339: datetime
  RFC3986: uri
  RFC3987: iri
  RFC9164: iptag
  RFC9485: iregexp
  IANA.cose: cose-reg
  IANA.cbor-tags: tags
  IANA.media-types:
  IANA.core-parameters:
  BCP26: ianacons
  STD80: ascii
  I-D.ietf-core-href: cri
  IEEE754:
    target: https://ieeexplore.ieee.org/document/8766229
    title: IEEE Standard for Floating-Point Arithmetic
    author:
    - org: IEEE
    date: false
    seriesinfo:
      IEEE Std: 754-2019
      DOI: 10.1109/IEEESTD.2019.8766229
  C:
    target: https://www.iso.org/standard/82075.html
    title: Information technology — Programming languages — C
    author:
    - org: International Organization for Standardization
    date: 2024-10
    seriesinfo:
      ISO/IEC: 9899:2024
    refcontent:
    - Edition 5
    annotation: >
       
      The standard is widely known as C23.
      Its technical content is also available via
      <https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf>.
  Cplusplus:
    target: https://www.iso.org/standard/83626.html
    title: Programming languages — C++
    author:
    - org: International Organization for Standardization
    date: 2024-10
    seriesinfo:
      ISO/IEC: 14882:2024
    refcontent:
    - Edition 7
    annotation: >
       
      The standard is widely known as C++23.
      Its technical content is also available via
      <https://open-std.org/jtc1/sc22/wg21/docs/papers/2023/n4950.pdf>.
informative:
  RFC8610: cddl
  RFC7049: old-cbor
  RFC4648: base
  RFC9290: pd
  RFC8259: json
  RFC7493: i-json
  I-D.bormann-cbor-numbers: numbers
  RFC9165: controls
  RFC9682: cddlupd
  I-D.ietf-cbor-edn-e-ref: eref
  I-D.bormann-t2trg-deref-id: deref
  RFC9512: yaml-media-type
  YAML:
    target: https://yaml.org/spec/1.2.2/
    title: YAML Ain't Markup Language (YAML™) Version 1.2
    author:
    - name: Oren Ben-Kiki
    - name: Clark Evans
    - name: Ingy | döt Net
    date: 2021-10-01
    refcontent:
    - Revision 1.2.2
  ABNFROB:
    target: https://github.com/cabo/abnftt
    title: PEG-parsing using ABNF grammars (via treetop)
  EDN-WIKI:
    target: https://github.com/cbor-wg/edn/wiki
    title: EDN Wiki

--- abstract

This document formalizes a standalone definition of the Extended
Diagnostic Notation (EDN) for the Concise Binary Object Representation
(CBOR), addressing new requirements from the implementer community.
It also explicitly defines new extension points with registries and includes a few straightforward extensions.

> This is a rewritten version of draft-ietf-cbor-edn-literals.
> It is intended to be clearer for implementers and more concise.

--- middle

# Introduction        {#intro}

CBOR {{RFC8949}} is a binary, self-delimiting data format.
In addition to its binary interchange format, the original CBOR specification described a text-based "diagnostic notation" ({{Section 6 of RFC7049}}, then {{Section 8 of RFC8949@-cbor}}), to enable simpler discussions of CBOR data items.
{{Appendix G of -cddl}} extended this into the Extended Diagnostic Notation (EDN).

Diagnostic notation syntax is based on JSON {{RFC8259}}, with additional syntax to represent other CBOR constructs such as binary data and tags, and support for convenient features such as comments.

EDN is not intended as a general-purpose interchange format, but rather a textual representation of a binary data format.
EDN enables unambiguous generation of diagnostics about CBOR data.
It is also useful for documentation about CBOR, CBOR development, document generation systems, continuous integration (CI) environments, configuration files, and user interfaces for viewing and editing for all these.
EDN therefore merits a specification that facilitates interoperability of these tools, especially reliable translation to and from CBOR data.

> It is also possible to represent CBOR data items using other text formats, such as {{YAML}} using YAML extensions.
> However, EDN natively includes support for all CBOR data types.

This document replaces previous definitions of EDN, and defines a formal grammar (see {{grammar}} and {{app-grammars}}), incorporating features requested by the community.
It obsoletes {{Section 8 of RFC8949@-cbor}}, {{Section 4.2 of RFC8742}}, and {{Appendix G of -cddl}}.

It also specifies two registry-based extension points for the diagnostic notation: one for encoding indicators, and one for application-oriented literal forms.

It includes one new encoding indicator and the following (application-oriented literal form) extensions for text representations of:

- epoch-based date/times,
- IP addresses and prefixes {{-iptag}}, and of
- Concise Resource Identifiers (CRI) {{-cri}}.

It also defines a new application-oriented literal that represents cryptographic hash values computed from byte strings.

Finally, this document registers a media type identifier and a content-format for CBOR diagnostic notation.


# Terminology and Conventions {#terminology}

{{Section 8 of RFC8949@-cbor}} includes a definition of the original CBOR "diagnostic notation".
{{Appendix G of -cddl}} added popular features such as embedded CBOR and comments; it already refers to the format as "Extended Diagnostic Notation".

Augmented Backus-Naur Form (ABNF) in this document refers to the language defined in {{-abnf}} as extended in {{-abnfcs}}, where the
"characters" of {{Section 2.3 of RFC5234@-abnf}} are Unicode scalar values.

Examples sometimes need to be quoted in the text, in particular in
cases where the typewriter font used for example text cannot be
distinguished in the plaintext rendition of this document.
ASCII quotes, however, are already taken: `true`, `"true"`, `'true'`,
and `` `true` `` are all different literals in EDN and should not be
confused.
Therefore, a different quoting convention as in »`true`« or »`"true"`«
is used for examples in the text where this is needed to remain
unambiguous.

<!-- text adapted from RFC 9581 -->
Superscript notation denotes exponentiation.
For example, 2 to the power of 64+1 is notated: 2<sup>64+1</sup>.
In the plain-text rendition of this specification, superscript
notation is not available and exponentiation is therefore rendered by
the surrogate notation seen here in the plain-text rendition.

{::boilerplate bcp14-tagged-bcp14}

# Objectives of this Document

{{Section 8 of RFC8949@-cbor}} describes the objective of defining a common human-readable diagnostic notation for CBOR.

## Converting CBOR to EDN

EDN was designed to be able to represent (almost) any legal CBOR data item or sequence.
Using EDN for debugging or diagnostic purposes allows humans to more easily and efficiently read the structure of CBOR data.
As such, converting from CBOR to EDN to CBOR should not result in a CBOR document with different data types.

## Converting from EDN to CBOR

Another important use of EDN is for test vectors, examples in specifications, and configuration.
The benefit of a text representation of this type of informations is to make it easier for humans to read, write, edit, and compare diffs.
EDN has several different ways of representing the same CBOR values.
There is absolutely no expectation that converting from EDN to CBOR to EDN will result in the same EDN document.

One important application of EDN is the notation of CBOR data for
humans: for ease of debugging, in specifications, and for entering test data.
A number of features of EDN, such as comments are useful for humans to read.
Programs also often output EDN for diagnostic purposes, such as in
error messages or to enable comparison (including generation of diffs
via tools) with test data.


# Basic Output Format {#basic}

> **TODO**: Move this section

However, there is a certain expectation that EDN generators can be
configured to some basic output format, which:

* looks like JSON where that is possible;
* inserts encoding indicators only where the binary form differs from
  Preferred Serialization ({{Section 4.1 of RFC8949@-cbor}});
* uses hexadecimal representation (`h''`) for byte strings, not
  `b64''` or embedded CBOR (`<<>>`);
* does not generate elaborate blank space (newlines, indentation) for
  pretty-printing, but does use common blank spaces such as after `,`
  and `:`.

EDN generators may provide configuration to consistently select either
the unescaped (directly readable) or an escaped (ASCII equivalent) form of
characters in string literals; the latter allows EDN to be used when the
diagnostic value of fully escaped characters may be desired or in
environments where non-ASCII characters may not enjoy full data
transparency.
Similar to JSON, EDN is designed to allow a simple tool to convert any
EDN (including EDN with application extensions unknown to the tool)
into fully escaped (printable ASCII and newlines only) form, as well
as to inversely recover unescaped characters for all escapes where
this is possible or for certain subsets of the characters (such as
Unicode categories L, M, N, P, S, plus Zs or just ASCII space).

Additional features such as ensuring
deterministic map ordering ({{Section 4.2 of RFC8949@-cbor}}) on output,
or even deviating from the basic
configuration in some systematic way, can further assist in comparing
test data.
Information obtained from a CDDL model can help in choosing
application-oriented literals or specific string representations such
as embedded CBOR or `b64''` in the appropriate places.

Overview over CBOR Extended Diagnostic Notation (EDN) {#diagnostic-notation}
=====================================================

CBOR is a binary interchange format.  To facilitate documentation and
debugging, and in particular to facilitate communication between
entities cooperating in debugging, this document defines a simple
human-readable diagnostic notation.  All actual interchange always
happens in the binary format.

Note that diagnostic notation truly was designed as a diagnostic
format; it originally was not meant to be parsed.
Therefore, no formal definition (as in ABNF) was given in the original
documents.
Recognizing that formal grammars can aid interoperation of tools and
usability of documents that employ EDN, {{grammars}} now provides ABNF
definitions.

EDN is a true superset of JSON as it is defined in {{RFC8259}} in
conjunction with {{-i-json}} (that is, any interoperable {{-i-json}} JSON
text also is an EDN text), extending it both to cover the greater
expressiveness of CBOR and to increase its usability.

EDN borrows the JSON syntax for numbers (integer and
floating-point, {{numbers}}), certain simple values ({{simple-values}}),
UTF-8 {{-utf8}} text
strings, arrays, and maps (maps are called objects in JSON; the
diagnostic notation extends JSON here by allowing any data item in the
map key position).

As EDN is used for truly diagnostic purposes, its implementations MAY
support generation and possibly ingestion of EDN for CBOR data items
that are well-formed but not valid.
It is RECOMMENDED that an implementation enables such usage only
explicitly by configuration (such as an API or CLI flag).
Validity of CBOR data items is discussed in {{Section 5.3 of RFC8949@-cbor}},
with basic validity discussed in {{Section 5.3.1 of RFC8949@-cbor}}, and
tag validity discussed in {{Section 5.3.2 of RFC8949@-cbor}}.
Tag validity is more likely a subject for individual
application-oriented extensions, while the two cases of basic validity
(for text strings and for maps) are addressed in Sections
{{<text-validity}} and {{<map-validity}} under the heading
of *validity*.

The rest of this section provides an overview over specific features
of EDN, starting with certain common syntactical features and then
going through kinds of CBOR data items roughly in the order of CBOR major
types.
Any additional detailed syntax discussion needed has been deferred to
{{grammar}}.

Additional information about implementation and use of EDN is
continuously being collected by the community in {{EDN-WIKI}}.

## Application-Oriented Extension Literals {#app-lit}

EDN provides _literals_ that represent CBOR data items textually.
Many of the forms of literals provided are predefined by this
document, but it also defines an extension point that enables defining additional
_application-oriented extension literals_, or _extension literals_ for short.

Extension literals start with a _prefix_ that identifies the
application-oriented extension, immediately followed by a sequence
literal ({{embedded}}) or a single-quoted string literal ({{strings}}).
The latter form uses its string literal as a shorthand
form for a sequence literal representing a sequence with exactly that
one text string data item.

{:aside}
> This notation is generalized from
{{Section 8 of RFC8949@-cbor}}, which provides for notating byte
strings in a number of {{-base}} base encodings, where the encoded text
is enclosed in single quotes, prefixed by a prefix (»h« for
base16, »b32« for base32, »h32« for base32hex, »b64« for base64 or
base64url).
>
> This syntax can be thought to establish a name space, with the names
"h", "b32", "h32", and "b64" taken, but other names being unallocated.
The present specification allows registering additional names for this namespace,
which it calls *application-extension identifiers*.

More precisely, an *application-extension identifier* is a registered name consisting of a
lower-case ASCII letter (`[a-z]`) and zero or more additional ASCII
characters that are either lower-case letters, digits, or hyphens (`[a-z0-9-]`).
»false«, »true«, »null«, and »undefined« cannot be used as such
identifiers and are reserved.

Application-extension identifiers are registered in the "Application-Extension Identifiers" registry
({{appext-iana}}).

An application-extension (such as `dt`) MAY also define the meaning of
one additional prefix derived from its application-extension identifier by
replacing each lower-case character by its upper-case counterpart (such
as `DT`).
As a convention, using the all-uppercase variant implies making use of
a CBOR tag appropriate for this application-oriented extension (such
as tag number 1 for `DT`, where in contrast the prefix `dt` stands for
the unwrapped tag content).

In summary, an application-extension identifier gives rise to one or two
application-extension prefixes, one that is lexically identical to the
identifier (i.e., all lowercase), and potentially another one that is an
all-uppercase variation of it.
In addition to specifying which of these two variations exhibits which
specific semantics, the application extension specifies what input the
extension takes.

When the prefix is used immediately in front of a single-quoted
string, the input takes the form of a single text string CBOR data
item.
When used immediately in front of a sequence literal, the input is a
CBOR sequence of elements of the sequence literal as input.
The application extension can provide behavior that depends on the
number of items supplied as input to it and their data types; it
cannot distinguish between its prefix being used with a single-quoted
string, or a CBOR sequence composed of a single text
string data item (as illustrated for instance in Tables {{<tab-equiv-dt}},
{{<tab-equiv-ip}}, and {{<tab-equiv-hash}}).

This specification defines a number of generally applicable
application-oriented extensions ({{app-ext}}), both to motivate
making these extensions generally available, and to illustrate the
concept.

Of these, the application-oriented extensions `h`, `b64`, `dt` and `ip` are
intended to be mandatory to implement.
(As mentioned, for simplicity we use the term "application-oriented
extensions" for the mechanism discussed in this section even if it is
used to describe a part of base EDN.)

## Comments {#comments}

For presentation to humans, EDN text may benefit from comments.
JSON famously does not provide for comments, and the original
diagnostic notation in {{Section 6 of -old-cbor}} inherited this property.

EDN now provides two comment syntaxes, which can be used where the
syntax allows blank space (outside of constructs such as numbers,
string literals, etc.):

* inline comments, delimited by slashes ("`/`") or by C-style "`/*`"
  and "`*/`":

  In a position that allows blank space, each of the following is
  considered blank space (and thus effectively a comment):

    * any text that starts with a slash followed by a character that is not a
      star or a slash, up to another slash, or
    * any text that starts with "`/*`" up to and including the next following "`*/`"

* end-of-line comments, delimited by "`#`" or "`//`" and an end of line (LINE
  FEED, U+000A):

  In a position that allows blank space, any text starting with "`#`"
  or "`//`" and ending with and including the end of the line is
  considered blank space (and thus effectively a comment).

Comments can be used to annotate a CBOR structure as in:

~~~~ cbor-diag
/grasp-message/ [/M_DISCOVERY/ 1, /session-id/ 10584416,
                 /objective/ [/objective-name/ "opsonize",
                              /D, N, S/ 7, /loop-count/ 105]]
~~~~

This reduces to `[1, 10584416, ["opsonize", 7, 105]]`.

Another example, combining
the use of inline and end-of-line comments:

~~~ cbor-diag
{
 /kty/ 1 : 4, # Symmetric
 /alg/ 3 : 5, # HMAC 256-256
  /k/ -1 : h'6684523ab17337f173500e5728c628547cb37df
             e68449c65f885d1b73b49eae1'
}
~~~

This reduces to `{1: 4, 3: 5, -1:
h'6684523AB17337F173500E5728C628547CB37DFE68449C65F885D1B73B49EAE1'}`.

{:aside}
>
Note that application-oriented extensions can define their own
internal comment syntaxes for text inside strings, which may or may
not mimic the overall comment syntax of EDN.
The h'' syntax ({{h-grammar}}), which the framework for application-oriented
extensions was designed to include as an instance, provides an
equivalent to the overall comment syntax inside its text strings.
Similarly, b64'' ({{b64-grammar}}) provides a subset of that limited to
"`#`" end-of-line comments (the slash character "`/`" is used in the
alphabet in classic base64 encoding).
None of the other application-oriented extensions supplied in this
specification provides for such a kind of internal comment syntax.

### Discussion

As a not quite backward compatible change, this specification
restricts slash-delimited comments that were allowed in {{Section G.6 of RFC8610}} in two ways:

* Inline comments now longer can be empty: The construct "`//`" that was
  an empty comment in {{Section G.6 of RFC8610}} is now used instead to introduce an
  end-of-line comment.
  (Note that "`//`" still can be used in what is visually "within" a
  slash-delimited comment; its first slash actually ends the current comment and
  the second slash starts a new one.)
* EDN now enables the use of C-style inline comments: for instance, "`/*foo/`"
  was a complete comment in {{Section G.6 of RFC8610}} and now is the beginning of a
  C-style comment that goes on up to a "`*/`".

As an example, the introduction of C-style inline comments enables a
comment explaining a COSE algorithm identifier, as in

~~~ cbor-diag
4 /* HMAC 256/64 */
~~~

instead of the conventional, but often less familiar

~~~ cbor-diag
4 / HMAC 256//64 /
~~~

## Encoding Indicators {#encoding-indicators}

Sometimes it is useful to indicate in the diagnostic notation which of
several alternative representations were actually used; for example, a
data item written »1.5« by a diagnostic decoder might have been
encoded as a half-, single-, or double-precision float.

Encoding indicators are always optional:
EDN is usually used to describe CBOR data items at the data model
level.
For some diagnostic purposes, it is useful to represent the choice of
a serialization variation by including encoding indicators.
Implementations of EDN generally do not need to provide this
functionality in full; if they do, they can be called "diagnostic
implementations".
To be able to process EDN that contains encoding indicators,
an EDN-consuming implementation MUST accept them (i.e., process or
ignore the presence or absence of each encoding indicator).
(Ignoring them could be compared to a generic CBOR decoder ignoring
the presence of the serialization variants it encounters.)
It is RECOMMENDED to by default provide a warning for each encoding
indicator value that is encountered but not further processed.

When creating EDN as input for a diagnostic CBOR encoder in order to
obtain specific encoding choices, encoding indicators may be placed
manually or by the software generating the EDN.
Where no encoding indicator is placed, a diagnostic CBOR encoder is expected to
generate Preferred Serialization ({{Section 4.1 of RFC8949@-cbor}}) with
definite length encoding only.
Similarly, when using EDN as output for a diagnostic CBOR decoder, a
basic diagnostic configuration of the tool is expected to provide
encoding indicators only in places where the CBOR input did not use
Preferred Serialization with definite length encoding (see also
{{basic}}).
Diagnostic implementations of EDN that process encoding indicators as
discussed here are expected to document their diagnostic behavior and
the processing options that can be selected.

### Syntax, Semantics, Examples

Encoding indicators start with
an underscore and comprise all immediately following characters that are alphanumeric or
underscore.
For example, `_` or `_3`.
Encoding indicators can be ignored by anyone not
interested in this information.

Encoding indicators are placed immediately to the right of the data
item or of a syntactic feature that can stand for the data item the
encoding of which the encoding indicator is controlling.
{{tab-ei}} provides examples for data items with encoding indicators used with various
kinds of data items.

| mt | examples                |
|  0 | `1_1`, `0x4711_3`       |
|  1 | `-1_1`                  |
|  2 | `'A'_1`                 |
|  3 | `"A"_1`                 |
|  4 | `[_1 "bar"]`            |
|  5 | `{_1 "bar": 1}`         |
|  6 | `1_1(4711)`             |
|  7 | `1.5_2`, `0x4711p+03_3` |
{: #tab-ei title="Examples of Encoding Indicators for Different Data
Items (mt = major type)"}

(In the following, an abbreviation of the form `ai=`nn gives nn as
the numeric value of the field _additional information_, the low-order 5
bits of the initial byte: see {{Section 3 of RFC8949@-cbor}}.
This field is used in encoding the "argument", i.e., the value, tag, or
length; `ai=0` to `ai=23` mean that the value of the `ai` field
immediately *is* the argument, `ai=24` to `ai=27` mean that the
argument is carried in 2<sup>ai-24</sup> (1, 2, 4, or 8)
additional bytes, and `ai=31` means that indefinite length
encoding is used.)

An underscore followed by a decimal digit `n` indicates that the
preceding item (or, for arrays and maps, the item starting with the
preceding bracket or brace) was or is to be encoded with an additional information
value of `ai=`24+`n`.  For example, `1.5_1` is a half-precision floating-point
number (2<sup>1</sup> = 2 additional bytes or 16 bits), while `1.5_3` is encoded as
double precision (2<sup>3</sup> = 8 additional bytes or 64 bits).
For a tool consuming EDN in a diagnostic mode, encountering an
encoding indicator that does not provide enough space to correctly
encode the unchanged data item given is an error; there is no
truncation or rounding that would change the data item encoded.

{:aside}
>
Truncation or rounding semantics imply performing changes at the data
model level, which is outside the scope of encoding indicators.
Such operations can be provided by application extensions.

The encoding indicator `_` (an underscore on its own) is used to
indicate indefinite length encoding.
Indefinite length encoding uses `ai=31`, which could have been
indicated by `_7`, which is therefore not used and marked as reserved
(as are `_4`, `_5`, and `_6`, which would stand for `ai=28` to
`ai=30`, values currently not in use in CBOR; these encoding
indicators will be available if and when CBOR is extended to make use
of them).

Note that the encoding indicator `_` is only available behind the opening
brace/bracket for `map` and `array` ({{ei-container}}): strings have a special syntax
`streamstring` for indefinite length encoding except for the special
cases `''_` and `""_` ({{ei-string}}).

The encoding indicators `_0` to `_3` can be used to indicate `ai=24`
to `ai=27`, respectively; they therefore stand for 1, 2, 4, and 8
bytes of additional information (ai) following the initial byte in the
head of the data item.

Surprisingly, {{Section 8.1 of RFC8949@-cbor}} does not address `ai=0` to
`ai=23` — the assumption seems to have been that Preferred Serialization
({{Section 4.1 of RFC8949@-cbor}}) will be used when converting CBOR
diagnostic notation to an encoded CBOR data item, so leaving out the
encoding indicator for a data item with a Preferred Serialization
will implicitly use `ai=0` to `ai=23` if that is possible.
The present specification allows making this explicit:

`_i` ("immediate") stands for encoding with `ai=0` to `ai=23`, i.e.,
it indicates that the argument is encoded directly in the initial byte
of the CBOR item.

Encoding indicators are an extension point for EDN; {{reg-ei}} defines
a registry for additional values.

Specific forms of encoding indicators are discussed in further detail
in {{ei-string}} for indefinite length strings and in {{ei-container}} for
arrays and maps.

## Numbers

<!--
## Hexadecimal, Octal, and Binary Numbers {#hexadecimal-octal-and-binary-numbers}
 -->

In addition to JSON's decimal number literals, EDN provides hexadecimal, octal,
and binary number literals in the usual C-language notation (octal with `0o`
prefix present only).

Numbers composed only of digits (of the respective base) are
interpreted as CBOR integers (major type 0/1, or where the number
cannot be represented in this way, major type 6 with tag 2/3).
A leading "`+`" sign is a no-op, and a leading "`-`" sign inverts the
sign of the number.
So `0`, `000`, `+0` all represent the same integer zero, as does `-0`.
Similarly,
`1`, `001`, `+1` and `+0001` all stand for the same integer one, and
`-1` and `-0001` both designate the same integer minus one.

Using a decimal point (`.`) and/or an exponent (`e` for decimal, `p`
for hexadecimal) turns the number into a floating point number (major
type 7) instead, irrespective of whether it is an integral number
mathematically.
Note that, in floating point numbers, `0.0` is not the same number as
`-0.0`, even if they are mathematically equal.

In {{tab-numbers}}, all the items on a row are the same number (also
shown in CBOR, hexadecimally), but they are distinct from items in a
different row.

| EDN                                            | CBOR hex            |
|------------------------------------------------|---------------------|
| `4711`, `0x1267`, `0o11147`, `0b1001001100111` | `19 1267` # uint    |
| `1.5`, `0.15e1`, `15e-1`, `0x1.8p0`, `0x18p-4` | `F9 3E00` # float16 |
| `0`, `+0`, `-0`                                | `00     ` # uint    |
| `0.0`, `+0.0`                                  | `F9 0000` # float16 |
| `-0.0`                                         | `F9 8000` # float16 |
| `Infinity`                                     | `F9 7C00` # float16 |
| `-Infinity`                                    | `F9 FC00` # float16 |
| `NaN`                                          | `F9 7E00` # float16 |
{: #tab-numbers title="Example Sets of Equivalent Notations for Some Numbers"}

The non-finite floating-point values `Infinity`, `-Infinity`, and `NaN` are
written exactly as in this sentence (this is also a way they can be
written in JavaScript, although JSON does not allow them).
`NaN` in EDN stands for the NaN value with a zero sign bit and an all-zero
significand except for a set quiet bit; this is represented as
0xF97E00 in CBOR Preferred Serialization.
{{tab-non-finite-encoding}} shows how the floating point numbers 1.1, 1.5 and
these three values are encoded in preferred serialization and when
encoding indicators are given.

<!-- $ edn-abnf -e '1.5, 1.5_1, 1.5_2, 1.5_3' -tcbor | cborseq2pretty.rb
 -->

| EDN                        | CBOR hex              |
|----------------------------|-----------------------|
| `1.1`                      | `fb 3ff199999999999a` |
| `1.1_1`, `1.1_2`           | (error)               |
| `1.1_3`                    | `fb 3ff199999999999a` |
| `1.5`, `1.5_1`             | `f9 3e00`             |
| `1.5_2`                    | `fa 3fc00000`         |
| `1.5_3`                    | `fb 3ff8000000000000` |
| `Infinity`, `Infinity_1`   | `f9 7c00`             |
| `Infinity_2`               | `fa 7f800000`         |
| `Infinity_3`               | `fb 7ff0000000000000` |
| `-Infinity`, `-Infinity_1` | `f9 fc00`             |
| `-Infinity_2`              | `fa ff800000`         |
| `-Infinity_3`              | `fb fff0000000000000` |
| `NaN`, `NaN_1`             | `f9 7e00`             |
| `NaN_2`                    | `fa 7fc00000`         |
| `NaN_3`                    | `fb 7ff8000000000000` |
{: #tab-non-finite-encoding title="Encoding indicators on floating
point values" }

See {{decnumber}} for additional details of the EDN number syntax.

(Note that literals for further number formats, e.g., for representing
rational numbers as fractions, or for other NaN values than the one called `NaN`, can
be added as application-oriented literals.
Background information beyond that in {{-cbor}} about the representation
of numbers in CBOR can be found in the informational document
{{-numbers}}.)

## Strings

CBOR distinguishes two kinds of strings: text strings (the bytes in
the string constitute UTF-8 {{-utf8}} text, major type 3), and byte strings
(CBOR does not further characterize the bytes that constitute the
string, major type 2).

EDN has two direct (unprefixed) notations for strings: double-quoted
strings for (UTF-8) text strings, and single-quoted strings for byte strings.
The latter are useful for byte strings carrying bytes that can be meaningfully
notated as UTF-8 text ({{sq-lit}}).

Many strings are best notated as extension literals, which may
provide detailed access to the bits within those bytes (see
{{encoded-byte-strings}}).
Extension literals can be constructed out of single-quoted strings and
sequence literals.

### Double-Quoted String Literals {#dq-lit}

EDN enables notating text strings in a form compatible to that of notating text
strings in JSON (i.e., as a double-quoted string literal), with a
number of usability extensions.
In JSON, no control characters are allowed to occur
directly in text string literals; if needed, they can be specified using
escapes such as `\t` or `\r`.
In EDN, string literals additionally can contain newlines (LINEFEED
U+000A), which are copied into the resulting string like other
characters in the string literal.
To deal with variability in platform presentation of newlines, any
carriage return characters (U+000D) that may be present in the EDN
string literal are not copied into the resulting string (see {{cr}}).
No other control characters can occur directly in a string literal,
and the handling of escaped characters (`\r` etc.) is as in JSON.

JSON's escape scheme for characters that are not on Unicode's basic
multilingual plane (BMP) is cumbersome (see {{Section 7 of RFC8259@-json}}).
EDN keeps it, but also adds the syntax `\u{NNN}` where NNN is the
Unicode scalar value as a hexadecimal number.
This means the following are equivalent (the first `o` is escaped as
`\u{6f}` for no particular reason):

~~~ cbor-diag
"D\u{6f}mino's \u{1F073} + \u{2318}"   # \u{}-escape 3 chars
"Domino's \uD83C\uDC73 + \u2318"       # escape JSON-like
"Domino's 🁳 + ⌘"                       # unescaped
~~~

### Single-Quoted String Literals {#sq-lit}

Analogously to text string literals delimited by double quotes, EDN
allows the use of single quotes (without a prefix) to express byte
string literals with UTF-8 text; for instance, the following are
equivalent:

~~~~
'hello world'
h'68656c6c6f20776f726c64'
~~~~

The escaping rules of JSON strings are applied equivalently for
text-based byte string literals, e.g., `\\` stands for a single
backslash and `\'` stands for a single quote.
However, to facilitate parsing, in single-quoted strings EDN excludes
certain escaping mechanisms available for double-quoted strings:

* `\/` is an escape in JSON that is available for EDN text strings as
  well to ensure all JSON texts are EDN literals.
  Since EDN's single-quoted strings do not occur in JSON, this legacy
  compatibility feature is not available for them.
* `\u`-based escapes are not available for characters in the range
  from U+0020 to U+007E (essentially, printable ASCII).

Single-quoted string literals can occur unprefixed and stand for the
byte string that encodes its text string value (the "content"), or be
prefixed by what looks like an application-extension prefix (see
{{app-lit}}).

In a prefixed string literal, the text content of the single-quoted
string literal is not used directly as a byte string, but is further
processed in a way that is defined by the meaning given to the prefix.
Depending on the prefix, the result of that processing can, but need
not be, a byte string value.

Prefixed string literals are used both for base-encoded byte string literals (see {{encoded-byte-strings}}) and for
application-oriented extension literals (see {{app-lit}}, called app-string).
(Additional kinds of base-encoded string literals can be defined as
application-oriented extension literals by registering their prefixes;
there is no fundamental difference between the two predefined
base-encoded string literal prefixes (`h`, `b64`) and any such potential
future extension literal prefixes; for simplicity of expression, both
cases are referred to as "extension literals".)


### Encoding Indicators of Strings {#ei-string}

For indefinite length encoding, strings (byte and text strings) have a
special syntax `streamstring`.
This is used (except for the special cases `''_` and `""_` below) to
notate their detailed composition into individual "chunks" ({{Section
3.2.3 of RFC8949@-cbor}}), by representing the individual chunks in
sequence within parentheses, each optionally followed by a comma, with
an encoding indicator `_` immediately after the opening parenthesis:
e.g., `(_ h'0123', h'4567')` or `(_ "foo", "bar")`.
The overall type (byte string or text string) of the string is
provided by the types of the individual chunks, which all need to be
of the same type ({{Section 3.2.3 of RFC8949@-cbor}}).

For an indefinite-length string with no chunks inside, `(_ )`
would be ambiguous as to whether a byte string (encoded 0x5fff) or a text string
(encoded 0x7fff) is meant and is therefore not used.
The basic forms `''_` and `""_` can be used instead and are reserved for
the case of no chunks only — not as short forms for the (permitted,
but not really useful) encodings with only empty chunks, which
need to be notated as `(_ '')`, `(_ "")`, etc.,
when it is desired to preserve the chunk structure.


### Base-Encoded Byte String Literals {#encoded-byte-strings}

Besides the unprefixed byte string literals that are analogous to JSON text
string literals, EDN provides extension literals that can represent
byte strings by base-encoding them, typically notated as prefixed
string literals.
The application-extension identifier selects one of the base encodings
{{-base}}, without padding.
Most often, the base encoding is
enclosed in a single-quoted string literal, prefixed by »h« for base16 or
»b64« for base64 or base64url (the actual encodings of the latter two
have the same meaning where they overlap, so the string remains unambiguous).
For example, the byte string consisting of the four bytes `12 34 56 78`
(given in hexadecimal here) could be written `h'12345678'` or
`b64'EjRWeA'`.

{:aside}
>
(Note that {{Section 8 of RFC8949@-cbor}} also mentions »b32« for
base32 and »h32« for base32hex.
This has not been implemented widely
and therefore is not directly included in this specification.
These and further byte string formats now can easily be added back as
application-oriented extension literals.)

Examples often benefit from some blank space (spaces, line breaks) in
byte strings literals.
In certain EDN prefixed byte string literals, blank space is ignored; for
instance, the following are equivalent:

~~~~ cbor-diag
   h'48656c6c6f20776f726c64'
   h'48 65 6c 6c 6f 20 77 6f 72 6c 64'
   h'4 86 56c 6c6f
     20776 f726c64'
~~~~

The internal syntax of prefixed single-quote literals such
as `h''` and `b64''` can also allow comments as blank space (see {{comments}}).

~~~~ cbor-diag
   h'68656c6c6f20776f726c64'
   h'68 65 6c /doubled l!/ 6c 6f # hello
     20 /space/
     77 6f 72 6c 64' /world/
~~~~

Slash characters are part of the base64 classic alphabet (see
Table 1 in {{Section 4 of RFC4648}}), and they therefore need to be in the
`b64''` set of characters that contribute to the byte string.
Therefore, only end-of-line comments are available inside b64 byte string
literals.

~~~~ cbor-diag
   b64'/base64 not a comment/ but one follows # comment'
   h'FDB6AC 7BAE27A2D69CA2699E9EDFDBBADA2779FA25 968C2C'
~~~~

These two byte string literals stand for the same byte string; the
deliberately confusing base64 content starts with
`b64'/bas'` which is the same as h'FDB6AC' and ends with b64'lows'
which is the same as `h'968C2C'`.


### CBOR Sequence Literals {#embedded}

In diagnostic notation, a sequence of zero or more CBOR data item literals can
be enclosed in `<<` and `>>`, optionally prefixed by an
application-extension prefix; this specification speaks of *sequence literals*.
EDN mainly deals with individual data items, not with CBOR sequences
{{-seq}}, so the CBOR sequence represented by the sequence literal needs
to be further processed to obtain the value of the literal.

Prefixed sequence literals refer to the application extension (see
{{app-lit}}) identified by the prefix and apply the extension to its
sequence content, resulting in a single data item.
This data item may be a string or may not (always) be, depending on
the definition of the application extension.

An unprefixed sequence literal applies CBOR encoding to the
data items in its content, taken as a CBOR sequence.
The value of the
literal thus is a byte string with the encoded content; this is
commonly referred to as
*embedded CBOR*.
For instance, each pair of columns in the following are equivalent:

~~~~ cbor-diag
   <<1>>              h'01'
   <<1, 2>>           h'0102'
   <<"hello", null>>  h'65 68656c6c6f f6'
   <<>>               h''
~~~~

### Validity of Text Strings {#text-validity}

To be valid CBOR, {{Section 5.3.1 of RFC8949@-cbor}} requires that text
strings are byte sequences in UTF-8 {{-utf8}} form.
EDN provides several ways to construct such byte strings (see {{concat}}
for details).
These mechanisms might operate on subsequences that do not themselves
constitute UTF-8, e.g., by building larger sequences out of
concatenating the subsequences; for validity of a text string
resulting from these mechanisms it is only of importance that the
result is UTF-8.
Double-quoted and single-quoted string literals have been defined
such that they lead to byte sequences that are UTF-8: the source
language of EDN is UTF-8, and all escaping mechanisms lead only to
adding further UTF-8 characters.
Only prefixed string literals, other application-extensions, or
in certain cases concatenation ({{concat}}) can generate non-UTF-8 byte
sequences.

As discussed at the start of {{diagnostic-notation}}, EDN
implementations MAY support generation and possibly ingestion of EDN
for CBOR data items that are well-formed but not valid; when this is
enabled, such implementations MAY relax the requirement on text
strings to be valid UTF-8.

Note that neither CBOR about its text strings nor EDN about its source
language make any requirements except for conformance to {{-utf8}}.
No additional Unicode processing or validation such as normalization
or checking whether a scalar value is actually assigned is foreseen by
EDN, particularly not any processing that is dependent on a specific
Unicode version.
Such processing, if offered, MUST NOT get in the way of processing the
data item represented in EDN (i.e., it may be appropriate to issue
warnings but not to error out or to generate output that does not match
the input at the UTF-8 level).

## Arrays and Maps

EDN borrows the JSON syntax for arrays and maps.
(Maps are called objects in JSON.)

### Mandatory Separators, Optional Terminators

For maps, EDN extends the JSON syntax by allowing any data item in the
map key position (before the colon).

JSON requires the use of a comma as a separator character between
the elements of an array as well as between the members (key/value
pairs) of a map.
(These commas also were required in the original diagnostic
notation defined in {{-cbor}} and {{-cddl}}.)
The separator commas are now optional in the places where EDN syntax
allows commas; however, where no comma is used in a separator
position, there must be blank space (composed of at least one space, newline, and/or
comment) instead.
(Stylistically, leaving out the commas is more idiomatic when they
occur at line breaks, which provide the blank space.)

In addition, EDN also allows, but does not require, a trailing comma before the closing bracket/brace,
enabling an easier to maintain "terminator" style of their use.

In summary, the following eight examples are all equivalent:

~~~ cbor-diag
[1, 2, 3]
[1, 2, 3,]
[1  2  3]
[1  2  3,]
[1  2, 3]
[1  2, 3,]
[1, 2  3]
[1, 2  3,]
~~~

as are

~~~ cbor-diag
{1: "n", "x": "a"}
{1: "n", "x": "a",}
{1: "n"  "x": "a"}
# etc.
~~~

As a comma and/or blank/comment is mandatory in a separator position,
»`[11]`« is unambiguously an array with a single element (the
integer 11), different from »`[1 1]`« or »`[1,1]`«.
As this is a general rule, »`[[] []]`« or »`[[],[]]`« are well-formed
EDN, while »`[[][]]`« is not.

{:aside}
> CDDL's comma separators in the equivalent contexts (CDDL groups) are
  entirely optional
  (and actually are terminators, which together with their optionality
  allows them to be used like separators as well, or even not at all).
  In summary, comma use is now aligned between EDN and CDDL, in a
  fully backward compatible way.
  (CDDL does allow the stylistically questionable »`a = [[][]]`«, though.)

### Encoding Indicators of Arrays and Maps {#ei-container}

A single underscore can be written after the opening brace of a map or
the opening bracket of an array to indicate that the data item was
represented in indefinite-length format.  For example, `[_ 1, 2]`
contains an indicator that an indefinite-length representation was
used to represent the data item `[1, 2]`.

At the same position, encoding indicators for specifying the size of
the array or map head for definite-length format can be used instead,
specifically `_i` or `_0` to `_3`.  For example `[_0 false, true]` can be
used to specify the encoding of the array `[false, true]` as `98 02 f4 f5`.

### Validity of Maps {#map-validity}

As discussed at the start of {{diagnostic-notation}}, EDN implementations MAY support
generation and possibly ingestion of EDN for CBOR data items that are
well-formed but not valid ({{Section 5.3 of RFC8949@-cbor}}).

For maps, this is relevant for map keys that occur more than once, as in:

~~~ cbor-diag
{1: "to", 1: "fro"}
~~~

## Tags

A tag is
written as a decimal unsigned integer for the tag number, followed by the tag content
in parentheses; for instance, a date in the format specified by RFC 3339
(ISO 8601) could be
notated as:

{: indent='5'}
0("2013-03-21T20:04:00Z")

or the equivalent epoch-based time as the following:

{: indent='5'}
1(1363896240)

The tag number can be followed by an encoding indicator giving the
encoding of the tag head.  For example:

{: indent='5'}
1_1(1363896240)

(assuming Preferred Serialization for the tag content) is encoded as

~~~ cbor-pretty
d9 0001        # tag(1)
   1a 514b67b0 # unsigned(1363896240)
~~~


## Simple values

EDN uses JSON syntax for the simple values True (»`true`«), False
(»`false`«), and Null (»`null`«).
Undefined is written »`undefined`« as in JavaScript.

These and all other simple values can be given as "simple()" with the
appropriate unsigned integer in the parentheses.  For example, »`simple(42)`«
indicates major type 7, value 42, and »`simple(20)`« indicates
»`false`«.



Application-Oriented Extension Literals {#app-ext}
=======================================

This document extends the syntax used in diagnostic notation to also
enable application-oriented extensions.
This section defines a number of application-oriented extensions.


The "dt" Extension {#dt}
------------------

The application-extension identifier "dt" is used to notate a
date/time literal that can be used as an Epoch-Based Date/Time as per
{{Section 3.4.2 of RFC8949@-cbor}}.

The content of the literal is a single Standard Date/Time String as per
{{Section 3.4.1 of RFC8949@-cbor}}, as a text or byte string.

The value of the literal is a number representing the result of a
conversion of the given Standard Date/Time String to an Epoch-Based
Date/Time.
If fractional seconds are given in the text (production
`time-secfrac` in {{abnf-grammar-dt}}), the value is a
floating-point number; the value is an integer number otherwise.
In the all-upper-case variant of the app-prefix, the value is enclosed
in a tag number 1.

Each row of {{tab-equiv-dt}} shows an example of "dt" notation and
equivalent notation not using an application-extension identifier.

| dt literal                   | plain EDN      |
|------------------------------|----------------|
| `dt'1969-07-21T02:56:16Z'`   | `-14159024`    |
| `dt'1969-07-21T02:56:16.0Z'` | `-14159024.0`  |
| `dt'1969-07-21T02:56:16.5Z'` | `-14159023.5`  |
| `DT'1969-07-21T02:56:16Z'`   | `1(-14159024)` |
{: #tab-equiv-dt title="dt and DT literals vs. plain EDN"}

See {{dt-grammar}} for an ABNF definition for the text string input of `dt` literals.


The "ip" Extension {#ip}
------------------

The application-extension identifier "ip" is used to notate an IP
address literal that can be used as an IP address as per {{Section 3 of
-iptag}}.

The input of the literal is a single text string representing an IPv4address or IPv6address as per
{{Section 3.2.2 of -uri}}.

With the lower-case app-string prefix `ip`, the value of the literal is a
byte string representing the binary IP address.
With the upper-case app-string prefix `IP`, the literal is such a byte string
tagged with tag number 54, if an IPv6address is used, or tag number
52, if an IPv4address is used.

As an additional case, the upper-case app-string prefix `IP''` can be used
with an IP address prefix such as `2001:db8::/56` or `192.0.2.0/24`, with the equivalent tag as its value.
(Note that {{-iptag}} representations of address prefixes need to
implement the truncation of the address byte string as described in
{{Section 4.2 of -iptag}}; see example below.)
For completeness, the lower-case variant `ip'2001:db8::/56'` or  `ip'192.0.2.0/24'` stands for
an unwrapped `[56,h'20010db8']` or `[24,h'c00002']`; however, in this case the information
on whether an address is IPv4 or IPv6 often needs to come from the context.

Note that this application-extension provides no direct representation
of the "Interface format"
defined in {{Section 3.1.3 of -iptag}}, an address combined with an
optional prefix length and an optional zone identifier, and therefore
no way to reference a zone identifier at all.
(If needed, this format can be put together by building their
structures explicitly, e.g., an interface format without a zone
identifier can be represented as in `52([ip'192.0.2.42',24])`, or an
interface format with zone identifier 42 as in
`54([ip'fe80::0202:02ff:ffff:fe03:0303',64,42])`.)

Each row of {{tab-equiv-ip}} shows an example of "ip" notation and
equivalent notation not using an application-extension identifier.

| ip literal          | plain EDN                                 |
|---------------------|-------------------------------------------|
| `ip'192.0.2.42'`    | `h'c000022a'`                             |
| `IP'192.0.2.42'`    | `52(h'c000022a')`                         |
| `IP'192.0.2.0/24'`  | `52([24,h'c00002'])`                      |
| `ip'2001:db8::42'`  | `h'20010db8000000000000000000000042'`     |
| `IP'2001:db8::42'`  | `54(h'20010db8000000000000000000000042')` |
| `IP'2001:db8::/64'` | `54([64,h'20010db8'])`                    |
{: #tab-equiv-ip title="ip and IP literals vs. plain EDN"}

See {{ip-grammar}} for an ABNF definition for the content of `ip` literals.


The "hash" Extension {#hash}
--------------------

The application-extension identifier "hash" is used to notate the
input to a cryptographic hash function as well as identify such a hash
function to obtain a byte string that represents the output of that
hash function.

The input of the literal is a (text or byte) string, optionally followed by either
an integer or a text string that identifies the hash function in the
COSE Algorithms registry of the CBOR Object Signing and Encryption
(COSE) registry group {{IANA.cose}}, either by the identifier (value:
integer or string), or, if no algorithm is registered with this value,
by its name used in the registry.
If the second item is not given, the default algorithm used is -16
("SHA-256").

No uppercase variant prefix is defined for the application-extension
identifier "hash".

| hash literal               | plain EDN                                                                                                                              |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `hash<<'foo'>>`            | h'2C26B46B68FFC68FF99B453C1D304134<br>13422D706483BFA0F98A5E886266E7AE'                                                                   |
| `hash<<'foo', -16>>`       | h'2C26B46B68FFC68FF99B453C1D304134<br>13422D706483BFA0F98A5E886266E7AE'                                                                   |
| `hash<<'foo', "SHA-256">>` | h'2C26B46B68FFC68FF99B453C1D304134<br>13422D706483BFA0F98A5E886266E7AE'                                                                   |
| `hash<<'foo', -44>>`       | h'F7FBBA6E0636F890E56FBBF3283E524C<br>6FA3204AE298382D624741D0DC663832<br>6E282C41BE5E4254D8820772C5518A2C<br>5A8C0C7F7EDA19594A7EB539453E1ED7' |
| `hash<<'foo', "SHA-512">>` | h'F7FBBA6E0636F890E56FBBF3283E524C<br>6FA3204AE298382D624741D0DC663832<br>6E282C41BE5E4254D8820772C5518A2C<br>5A8C0C7F7EDA19594A7EB539453E1ED7' |
{: #tab-equiv-hash title="hash literals vs. plain EDN"}


The "cri" Extension {#cri}
--------------------

The
application-extension identifier "`cri`" is used to notate
an EDN literal for a CRI reference as defined in {{-cri}}.

The input of the literal is a URI Reference as per {{-uri}} or an IRI
Reference as per {{-iri}}.

The value of the literal is a CRI reference that can be converted to
the text of the literal using the procedure of {{Section 6.1 of -cri}}.  <!-- {{cri-to-uri}}. -->
Note that there may be more than one CRI reference that can be
converted to the URI/IRI reference given; implementations are expected
to favor the simplest variant available and make non-surprising
choices otherwise.
In the all-upper-case variant of the app-prefix, the value is enclosed
in a tag number 99.

As an example, the CBOR diagnostic notation

~~~ cbor-diag
cri'https://example.com/bottarga/shaved'
CRI'https://example.com/bottarga/shaved'
~~~

is equivalent to

~~~ cbor-diag
[-4, ["example", "com"], ["bottarga", "shaved"]]
99([-4, ["example", "com"], ["bottarga", "shaved"]])
~~~

See {{cri-grammar}} for an ABNF definition for the content of `cri` literals.


# Handling information deliberately elided from an EDN document {#elision}

When using EDN for exposition in a document or on a whiteboard, it is
often useful to be able to leave out parts of an EDN document that are
not of interest at that point of the exposition.
It is also useful in a diagnostic context to view the overall structure of a document without displaying every long data item.

To facilitate this, this specification
supports the use of an _ellipsis_ (notated as three or more dots
in a row, as in `...`) to indicate parts of an EDN document that have
been elided (and therefore cannot be reconstructed).

Upon ingesting EDN as a representation of a CBOR data item for further
processing, the occurrence of an ellipsis is an error and
processing has to stop.

Elisions can stand in for entire subtrees, for example in:

~~~ cbor-diag
[1, 2, ..., 3]
{ "a": 1,
  "b": ...,
  ...: ...
}
~~~

A single ellipsis (or key/value pair of ellipses) can imply eliding
multiple elements in an array (or members in a map)

Elisions also can be used as part of a text string or byte string:

~~~ cbor-diag
{ "contract": "Herewith I buy" + ... + "gned: Alice & Bob",
  "bytes_in_IRI": 'https://a.example/' + ... + '&q=Übergrößenträger',
  "signature": h'4711...0815',
}
~~~

The example "contract" combines string concatenation via the `+`
operator ({{grammar}}) with an
ellipsis.
The example
"signature" uses special syntax that allows the use of ellipses
between the bytes notated _inside_ `h''` literals.

Note that the use of elisions is different from "commenting out" EDN
text, e.g.:

~~~ cbor-diag
{ "signature": h'4711/.../0815',
  # ...: ...
}
~~~

The consumer of this EDN will ignore the comments and therefore will
have no idea after ingestion that some information has been elided;
validation steps may then simply fail instead of being informed about
the elisions.


ABNF Definitions {#grammars}
================

This section collects grammars in ABNF form ({{-abnf}} as extended in
{{-abnfcs}}) that serve to define the syntax of EDN and some
application-oriented literals.

{:aside}
>
Implementation note: The ABNF definitions in this section are
intended to be useful in a Parsing Expression Grammar (PEG) parser
interpretation (see {{Appendix A
of -cddl}} for an introduction into PEG).

Overall ABNF Definition for Extended Diagnostic Notation {#grammar}
--------------------------------------------------------

This subsection provides an overall ABNF definition for the syntax of
CBOR extended diagnostic notation.

<aside markdown="1">

This ABNF definition treats all single-quoted string literals the same,
whether they are unprefixed and constitute byte string literals, or
prefixed and their content subject to further processing.
The text string value of the single-quoted strings that goes into that
further processing is described using separate ABNF definitions in
{{app-grammars}}; as a convention, the grammar for the content of an
app-string with prefix, say, `p`, is described by an ABNF definition
with the rule name `app-string-p`.

As an implementation note, some implementations may want to integrate
the parsing and processing of app-string content for certain
application extensions with the overall grammar.
Example grammars for such integrated parsers are provided with this
specification in {{integrated-grammars}}.
</aside>

For simplicity, the internal parsing for the built-in EDN prefixes is
specified in the same way.
ABNF definitions for `h''`/``` h`` ``` and `b64''`/```b64`` ``` are
provided in {{h-grammar}} and {{b64-grammar}}.
However, the prefixes `b32''` and `h32''` are not in wide use and an
ABNF definition in this document would therefore not have been based on
implementation experience.

~~~ abnf
{::include cbor-diag-parser.abnf}
~~~
{: #abnf-grammar title="Overall ABNF Definition of CBOR EDN"
   sourcecode-name="cbor-edn.abnf"}

While an ABNF grammar defines the set of character strings that are
considered to be valid EDN by this ABNF, the mapping of these
character strings into the generic data model of CBOR is not always
obvious.

The following additional items should help in the interpretation:

1. As mentioned in the terminology ({{terminology}}), the ABNF terminal
  values in this document define Unicode scalar values (characters)
  rather than their UTF-8 encoding.  For example, the Unicode PLACE OF
  INTEREST SIGN (U+2318) would be defined in ABNF as %x2318.

2. {: #cr} Unicode CARRIAGE RETURN characters (U+000D, often seen
  escaped as "\r" in many
  programming languages) that exist in the input (unescaped) are
  ignored as if they were not in the input wherever they appear.
  This is most important when they are found in (text or byte) string
  contexts (see the "unescaped" ABNF rule).
  On some platforms, a carriage return is always added in front of a
  LINE FEED (U+000A, also often seen escaped as "\n" in many
  programming languages), but on other platforms, carriage returns are
  not used at line breaks.
  The intent behind ignoring unescaped carriage returns is to ensure
  that input generated or processed on either of these kinds of
  platforms will generate the same bytes in the CBOR data items
  created from that input.
  (Platforms that use just a CARRIAGE RETURN by itself to signify an end of line
  are no longer relevant and the files they produce are out of scope
  for this document.)
  If a carriage return is needed in the CBOR data item, it can be
  added explicitly using the escaped form `\r`.

3. {: #decnumber}
  `decnumber` stands for an integer in the usual decimal notation, unless at
  least one of the optional parts starting with "." and "e" are
  present, in which case it stands for a floating point value in the
  usual decimal notation.  Note that the grammar now allows `3.` for
  `3.0` and `.3` for `0.3` (also for hexadecimal floating point
  below); implementers are advised that some platform numeric parsers
  accept only a subset of the floating point syntax in this document
  and may require some preprocessing to use here.

4. `hexint`, `octint`, and `binint` stand for an integer in the usual base 16/hexadecimal
  ("0x"), base 8/octal ("0o"), or base 2/binary ("0b") notation.
  `hexfloat` stands
  for a floating point number in the usual hexadecimal notation (which
  uses a mantissa in hexadecimal and an exponent in decimal notation,
  see Section 5.12.3 of {{IEEE754}}, Section 6.4.4.3 of {{C}}, or Section
  5.13.4 of {{Cplusplus}}; floating-suffix/floating-point-suffix from
  the latter two is not used here).

5. For `hexint`, `octint`, `binint`, and when `decnumber` stands for an integer, the
  corresponding CBOR data item is represented using major type 0 or 1
  if possible, or using tag 2 or 3 if not.
  In the latter case, this specification does not define any encoding
  indicators that apply.
  If fine control over encoding is desired, this can be expressed by
  being explicit about the representation as a tag:
  E.g., `987654321098765432310`, which is equivalent to `2(h'35 8a 75
  04 38 f3 80 f5 f6')` in its Preferred Serialization, might be
  written as `2_3(h'00 00 00 35 8a 75 04 38 f3 80 f5 f6'_1)` if
  leading zeros need to be added during serialization to obtain
  specific sizes for tag head, byte string head, and the overall byte
  string.

   When `decnumber` stands for a floating point value, and for
   `hexfloat` and `nonfin`, a floating point data item with major
   type 7 is used in Preferred Serialization (unless modified by an
   encoding indicator, which then needs to be `_1`, `_2`, or `_3`).
   For this, the number range needs to fit into an {{IEEE754}} binary64 (or the size
   corresponding to the encoding indicator), and the precision will be
   adjusted to binary64 before further applying Preferred Serialization
   (or to the size corresponding to the encoding indicator).
   Tag 4/5 representations are not generated in these cases.
   Future app-prefixes could be defined to allow more control for
   obtaining a tag 4/5 representation directly from a hex or decimal
   floating point literal.

6. {: #spec} `spec` stands for an encoding indicator.
  See {{encoding-indicators}} for details.

7. {: #concat}
  Extended diagnostic notation allows a (text or byte) string to be
  built up from multiple (text or byte) string literals, separated by
  a `+` operator; these are then concatenated into a single string.

    `string`, `string1e`, `string1`, and `ellipsis` realize: (1) the
    representation of strings in this form split up into multiple
    chunks, and (2) the use of ellipses to represent elisions
    ({{elision}}).

    Text strings and byte strings do not mix within such a
    concatenation, except that byte string literal notation can be used
    inside a sequence of concatenated text string notation literals, to
    encode characters that may be better represented in an encoded way.
    The following four text string values (adapted from {{Section G.4 of
    -cddl}} by updating to explicit `+` operators) are equivalent:

        "Hello world"
        "Hello " + "world"
        "Hello" + h'20' + "world"
        "" + h'48656c6c6f20776f726c64' + ""

    Similarly, the following byte string values are equivalent:

        'Hello world'
        'Hello ' + 'world'
        'Hello ' + h'776f726c64'
        'Hello' + h'20' + 'world'
        '' + h'48656c6c6f20776f726c64' + '' + b64''
        h'4 86 56c 6c6f' + h' 20776 f726c64'

    The semantic processing of these constructs is governed by the
    following rules:

    * A single `...` is a general ellipsis, which by itself can stand
      for any data item.
      Multiple adjacent concatenated ellipses are equivalent to a single
      ellipsis.
    * An ellipsis can be concatenated (on one or both sides) with string
      chunks (`string1`).
    * If there is no ellipsis in the concatenated list, the result of
      processing the list will always be a single item.
    * The bytes in the concatenated sequence of string chunks are simply
      joined together, proceeding from left to right.
      If the left hand side of a concatenation is a text string, the
      joining operation results in a text string, and that
      result needs to be valid UTF-8 except for implementations that
      support and are enabled for generation/ingestion of EDN for CBOR
      data items that are well-formed but not valid.
      If the left hand side is a byte string, the right hand side also
      needs to be a byte string.
    * Some of the strings may be app-strings.
      If the result type of the app-string is an actual (text or byte)
      string, joining of those string chunks occurs as with chunks
      directly notated as string literals; otherwise the occurrence of more than
      one app-string or an app-string together with a directly notated
      string cannot be processed.
      (This determination must be made at the time the app-string is
      interpreted)

### Discussion

Note that the syntax defined here for concatenation of components
uses an explicit `+` operator between the components to be
concatenated.

{:aside}
> This is not entirely backward compatible to {{Section G.4 of -cddl}},
> which used simple juxtaposition to indicate concatenation of strings.
> This was not widely implemented and got in the way of making the use
> of commas optional in other places via the rule `OC`.

ABNF Definitions for Application Extension Content {#app-grammars}
---------------------------------------

This subsection provides ABNF definitions for the content of
application-oriented extension literals defined in {{-cbor}} and in this
specification, where applicable.
These grammars describe the *decoded* content of the `sqstr` components that
combine with the application-extension identifiers used as prefixes to form
application-oriented extension literals.
Each of these may integrate ABNF rules defined in {{abnf-grammar}},
which are not always repeated here.

{{tab-prefixes}} summarizes the app-prefix values defined in this document.

| app-prefix | content of single-quoted string                    | result type                                       |
|------------|----------------------------------------------------|---------------------------------------------------|
| h          | hexadecimal form of binary data                    | byte string                                       |
| H          | (not used)                                         |                                                   |
| b64        | base64 forms (classic or base64url) of binary data | byte string                                       |
| B64        | (not used)                                         |                                                   |
| dt         | RFC 3339 date/time                                 | number (int or float)                             |
| DT         | "                                                  | Tag 1 on the above                                |
| ip         | IP address or prefix                               | byte string, <br/>array of length and byte string |
| IP         | "                                                  | Tag 54 (IPv6) or 52 (IPv4) on the above           |
| hash       | string (usually used with sequences)               | byte string                                       |
| cri        | RFC 3986 URI or URI reference                      | CBOR structure representing equivalent CRI        |
| CRI        | "                                                  | Tag 99 on the above                               |
{: #tab-prefixes title="App-prefix Values Defined in this Document"}

Note that implementation platforms may already provide implementations
of grammars used in application-extensions, such as of RFC 3339 for
`dt''` and of IP address syntax for `ip''`.
EDN-based tools may want to use these implementation libraries instead
of using the grammars that are provided here as a reference.

For convenience, the common definitions in {{abnf-grammar-ext-common}}
are not repeated in the below ABNF grammars.

~~~ abnf
ALPHA           = %x41-5a / %x61-7a
DIGIT           = %x30-39 ; 0-9
HEXDIG          = DIGIT / HEXDIGA
HEXDIGA         = "A" / "B" / "C" / "D" / "E" / "F"
; Note: double-quoted strings as in "A" are case-insensitive in ABNF
lblank          = %x0A / %x20  ; Not HT or CR (gone)
non-lf          = %x20-7f / NONASCII
NONASCII        = %x80-D7FF / %xE000-10FFFF
~~~
{: #abnf-grammar-ext-common sourcecode-name="cbor-edn-extcommon.abnf"
   title="Common Rules Used in app-extension ABNF grammars"
}


### h: ABNF Definition of Hexadecimal representation of a byte string {#h-grammar}


The syntax of the content of byte strings represented in hex,
such as `h''`, `h'0815'`, or `h'/head/ 63 /contents/ 66 6f 6f'`
(another representation of `<< "foo" >>`), is described by the ABNF in {{abnf-grammar-h}}.
This syntax accommodates both lower case and upper case hex digits, as
well as blank space (including comments) around each hex digit.

~~~ abnf
app-string-h    = S *(HEXDIG S HEXDIG S / ellipsis S)
                  [eol-comment *non-lf]
ellipsis        = 3*"."
non-slash       = lblank / %x21-2e / %x30-7f / NONASCII
non-slash-star  = lblank / %x21-29 / %x2b-2e / %x30-7f / NONASCII
non-star        = lblank / %x21-29 / %x2b-7f / NONASCII
ends-in-star    = *non-star 1*"*"
non-lf          = %x20-7f / NONASCII
eol-comment     = "#" / "//"
S               = *lblank *(comment *lblank)
comment         = "/" non-slash-star *non-slash "/"
                / "/*" ends-in-star
                       *(non-slash-star ends-in-star) "/"
                / eol-comment *non-lf %x0A
~~~
{: #abnf-grammar-h sourcecode-name="cbor-edn-ext-h.abnf"
title="ABNF Definition of Hexadecimal Representation of a Byte String"
}

{:aside}
>
> The comment syntax provided inside the hex string is intended to
> mimic the overall syntax for comments in EDN ({{comments}}).\\
> Implementation note: Comments and blank space are also described by
> the following search regexp, which can be used to remove them.
> For display, the regexp is split along the outer
> alternative into four lines, which need to be combined before use;
> `\z` stands for the end of the string and is notated `$` in some
> regexp dialects.
>
> ~~~
  \s|
  /\*(?:[^*]*\*+)(?:[^/*][^*]*\*+)*/|
  /[^/*][^/]*/|
  (?:#|//)[^\n]*(?:\n|\z)
> ~~~

### b64: ABNF Definition of Base64 representation of a byte string {#b64-grammar}


The syntax of the content of byte strings represented in base64 is
described by the ABNF in {{abnf-grammar-b64}}.

This syntax allows both the classic ({{Section 4 of RFC4648}}) and the
URL-safe ({{Section 5 of RFC4648}}) alphabet to be used.
It accommodates, but does not require base64 padding.
Note that inclusion of classic base64 makes it impossible to have
comments based on slash characters in b64, as "`/`" is valid base64-classic.

~~~ abnf
app-string-b64  = B *(4(b64dig B))
                  [b64dig B b64dig B ["=" B "=" / b64dig B ["="]] B]
                  ["#" *non-lf]
b64dig          = ALPHA / DIGIT / "-" / "_" / "+" / "/"
B               = *lblank *(comment *lblank)
comment         = "#" *non-lf %x0A
~~~
{: #abnf-grammar-b64 sourcecode-name="cbor-edn-ext-b64.abnf"
title="ABNF definition of Base64 Representation of a Byte String"
}

### dt: ABNF Definition of RFC 3339 Representation of a Date/Time {#dt-grammar}

The syntax of the content of `dt` literals can be described by the
ABNF for `date-time` in {{abnf-grammar-dt}}.
This is derived from {{RFC3339}} as summarized in {{Section 3 of -controls}}.

~~~ abnf
app-string-dt   = date-time

date-fullyear   = 4DIGIT
date-month      = 2DIGIT  ; 01-12
date-mday       = 2DIGIT  ; 01-28, 01-29, 01-30, 01-31 based on
                          ; month/year
time-hour       = 2DIGIT  ; 00-23
time-minute     = 2DIGIT  ; 00-59
time-second     = 2DIGIT  ; 00-58, 00-59, 00-60 based on leap sec
                          ; rules
time-secfrac    = "." 1*DIGIT
time-numoffset  = ("+" / "-") time-hour ":" time-minute
time-offset     = "Z" / time-numoffset

partial-time    = time-hour ":" time-minute ":" time-second
                  [time-secfrac]
full-date       = date-fullyear "-" date-month "-" date-mday
full-time       = partial-time time-offset

date-time       = full-date "T" full-time
~~~
{: #abnf-grammar-dt sourcecode-name="cbor-edn-ext-dt.abnf"
title="ABNF Definition of RFC3339 Representation of a Date/Time"
}


### ip: ABNF Definition of Textual Representation of an IP Address {#ip-grammar}

The syntax of the content of `ip` literals can be described by the
ABNF for `IPv4address` and `IPv6address` in {{Section 3.2.2 of -uri}},
as included in slightly updated form in {{abnf-grammar-ip}}.

~~~ abnf
app-string-ip = IPaddress ["/" uint]

IPaddress     = IPv4address
              / IPv6address

; ABNF from RFC 3986, re-arranged for PEG compatibility:

IPv6address   =                            6( h16 ":" ) ls32
              /                       "::" 5( h16 ":" ) ls32
              / [ h16               ] "::" 4( h16 ":" ) ls32
              / [ h16 *1( ":" h16 ) ] "::" 3( h16 ":" ) ls32
              / [ h16 *2( ":" h16 ) ] "::" 2( h16 ":" ) ls32
              / [ h16 *3( ":" h16 ) ] "::"    h16 ":"   ls32
              / [ h16 *4( ":" h16 ) ] "::"              ls32
              / [ h16 *5( ":" h16 ) ] "::"              h16
              / [ h16 *6( ":" h16 ) ] "::"

h16           = 1*4HEXDIG
ls32          = ( h16 ":" h16 ) / IPv4address
IPv4address   = dec-octet "." dec-octet "." dec-octet "." dec-octet
dec-octet     = "25" %x30-35         ; 250-255
              / "2" %x30-34 DIGIT    ; 200-249
              / "1" 2DIGIT           ; 100-199
              / %x31-39 DIGIT        ; 10-99
              / DIGIT                ; 0-9

DIGIT1        = %x31-39 ; 1-9
uint          = "0" / DIGIT1 *DIGIT
~~~
{: #abnf-grammar-ip sourcecode-name="cbor-edn-ext-ip.abnf"
title="ABNF Definition of Textual Representation of an IP Address"}


### cri: ABNF Definition of URI Representation of a CRI {#cri-grammar}

It can be expected that implementations of the application-extension
identifier "`cri`" will make use of platform-provided URI
implementations, which will include a URI parser.

In case such a URI parser is not available or inconvenient to
integrate,
a grammar of the content of `cri` literals is provided by the
ABNF for `URI-reference` in {{Section 4.1 of RFC3986@-uri}} with certain
re-arrangements taken from {{ip-grammar}};
these are reproduced in {{abnf-grammar-cri}}.
If the content is not ASCII only (i.e., for IRIs), first apply
{{Section 3.1 of RFC3987}} and apply this grammar to the result.

~~~ abnf
app-string-cri = URI-reference
; ABNF from RFC 3986:

URI           = scheme ":" hier-part [ "?" query ] [ "#" fragment ]

hier-part     = "//" authority path-abempty
                 / path-absolute
                 / path-rootless
                 / path-empty

URI-reference = URI / relative-ref

absolute-URI  = scheme ":" hier-part [ "?" query ]

relative-ref  = relative-part [ "?" query ] [ "#" fragment ]

relative-part = "//" authority path-abempty
                 / path-absolute
                 / path-noscheme
                 / path-empty

scheme        = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )

authority     = [ userinfo "@" ] host [ ":" port ]
userinfo      = *( unreserved / pct-encoded / sub-delims / ":" )
host          = IP-literal / IPv4address / reg-name
port          = *DIGIT

IP-literal    = "[" ( IPv6address / IPvFuture  ) "]"

IPvFuture     = "v" 1*HEXDIG "." 1*( unreserved / sub-delims / ":" )

; Use IPv6address, h16, ls32, IPv4adress, dec-octet as re-arranged
; for PEG Compatibility in Figure 6 of [RFC XXXX]:

IPv6address   =                            6( h16 ":" ) ls32
              /                       "::" 5( h16 ":" ) ls32
              / [ h16               ] "::" 4( h16 ":" ) ls32
              / [ h16 *1( ":" h16 ) ] "::" 3( h16 ":" ) ls32
              / [ h16 *2( ":" h16 ) ] "::" 2( h16 ":" ) ls32
              / [ h16 *3( ":" h16 ) ] "::"    h16 ":"   ls32
              / [ h16 *4( ":" h16 ) ] "::"              ls32
              / [ h16 *5( ":" h16 ) ] "::"              h16
              / [ h16 *6( ":" h16 ) ] "::"

h16           = 1*4HEXDIG
ls32          = ( h16 ":" h16 ) / IPv4address
IPv4address   = dec-octet "." dec-octet "." dec-octet "." dec-octet
dec-octet     = "25" %x30-35         ; 250-255
              / "2" %x30-34 DIGIT    ; 200-249
              / "1" 2DIGIT           ; 100-199
              / %x31-39 DIGIT        ; 10-99
              / DIGIT                ; 0-9

reg-name      = *( unreserved / pct-encoded / sub-delims )

path          = path-abempty    ; begins with "/" or is empty
                 / path-absolute   ; begins with "/" but not "//"
                 / path-noscheme   ; begins with a non-colon segment
                 / path-rootless   ; begins with a segment
                 / path-empty      ; zero characters

path-abempty  = *( "/" segment )
path-absolute = "/" [ segment-nz *( "/" segment ) ]
path-noscheme = segment-nz-nc *( "/" segment )
path-rootless = segment-nz *( "/" segment )
path-empty    = 0<pchar>

segment       = *pchar
segment-nz    = 1*pchar
segment-nz-nc = 1*( unreserved / pct-encoded / sub-delims / "@" )
                 ; non-zero-length segment without any colon ":"

pchar         = unreserved / pct-encoded / sub-delims / ":" / "@"

query         = *( pchar / "/" / "?" )

fragment      = *( pchar / "/" / "?" )

pct-encoded   = "%" HEXDIG HEXDIG

unreserved    = ALPHA / DIGIT / "-" / "." / "_" / "~"
reserved      = gen-delims / sub-delims
gen-delims    = ":" / "/" / "?" / "#" / "[" / "]" / "@"
sub-delims    = "!" / "$" / "&" / "'" / "(" / ")"
                 / "*" / "+" / "," / ";" / "="
~~~
{: #abnf-grammar-cri sourcecode-name="cbor-edn-ext-cri.abnf"
title="ABNF Definition of URI Representation of a CRI"
}



ABNF Definitions for Integrated Extension Parsers {#integrated-grammars}
-------------------------------------------------

For some applications of EDN, it is an optimization to integrate
parsers for the content of some prefixed single-quoted string literals into
the main parser, handling both the string literal syntax (e.g., escapes such
as `\'` and `\\`) and the syntax of the extension content in one go.

For application-extensions that only use printable ASCII characters
(from U+0020 to U+007E) minus single-quote `'` and backslash `\`, the ABNF
such as that given in {{app-grammars}} can be directly used as an
integrated parser, after adding some glue ABNF.
For instance, for app-string-dt, add an alternative to `bstr` that
points to a rule for prefixed single-quoted string literals ({{abnf-grammar-sq-glue}}).

~~~ abnf
bstr            = sq-app-string-dt /
                  app-string / sqstr / app-sequence / embedded
sq-app-string-dt = (%s"dt'"/%s"DT'") app-string-dt "'"
~~~
{: #abnf-grammar-sq-glue sourcecode-name="cbor-edn-glue.abnf"
title="Glue ABNF for Integrated DT Parser"
}

To facilitate writing integrated ABNF for more complex prefixed
string literals, the ABNF definitions in {{abnf-grammar-sq}} may
be useful and are used in the rest of this section.

~~~ abnf
i-HT =        %s"\t" / %s"\u" ("0009" / "{" *("0") "9}")
i-LF = %x0a / %s"\n" / %s"\u" ("000A" / "{" *("0") "A}")
i-CR = %x0d / %s"\r" / %s"\u" ("000D" / "{" *("0") "D}")

i-blank = i-LF / i-CR / " "
i-non-lf = i-HT / i-CR / %x20-26 / "\'" / %x28-5b
         / "\\" / %x5d-7f / i-NONASCII

i-NONASCII = NONASCII / %s"\u" ESCGE7F

; hex escaping for U+007F or greater
ESCGE7F = "D" ("8"/"9"/"A"/"B") 2HEXDIG
          %s"\u" "D" ("C"/"D"/"E"/"F") 2HEXDIG
        / FOURHEX1 / "0" HEXDIG1 2HEXDIG / "00" TWOHEX1
        / "{" *("0")
          ("10" 4HEXDIG / HEXDIG1 4HEXDIG
           / FOURHEX1 / HEXDIG1 2HEXDIG / TWOHEX1)
          "}"

; xxxx - 0xxx - Dhigh\uDloow
FOURHEX1 = (DIGIT1 / "A"/"B"/"C" / "E"/"F") 3HEXDIG
         / "D" ODIGIT 2HEXDIG
; 00xx - ASCII + 007F
TWOHEX1  = ("8"/"9" / HEXDIGA) HEXDIG / "7F"
~~~
{: #abnf-grammar-sq sourcecode-name="cbor-edn-intcommon.abnf"
title="ABNF Definitions Useful for Integrated Extension Parsers"}

Four subsections with ABNF for integrated parsers follow, a pair for
`h''` and `b64''`, and a pair for ``` h`` ``` and ``` b64`` ```.
There is no requirement for a new application-extension to supply ABNF
for an integrated parser (or any ABNF at all!), in particular if the
parsing function is likely to be fulfilled by a platform library.
If ABNF for the content of a single-quoted string is available in an
application-extension specification, ABNF for an integrated parser can
be written as a separate activity or also automatically derived.
At the time of writing, one example for a tool performing such a
derivation is available as open-source software {{ABNFROB}}.

### h'': ABNF Definition of Integrated Parser {#sq-h-grammar}

With glue ABNF similar to that in {{abnf-grammar-sq-glue}} and common
definitions in Figures {{<abnf-grammar-ext-common}} and {{<abnf-grammar-sq}}, ABNF such as
that shown in {{abnf-grammar-sq-h}} can be used as an integrated parser
for `h` prefixed single-quote strings.

~~~ abnf
sq-app-string-h = %s"h'" s-app-string-h "'"
s-app-string-h = h-S *(HEXDIG h-S HEXDIG h-S / ellipsis h-S)
    [eol-comment *i-non-lf]

h-S = *(i-blank) *(h-comment *(i-blank))
h-non-slash = i-blank / %x21-26 / "\'" / %x28-2e
            / %x30-5b / "\\" / %x5d-7f / i-NONASCII
h-non-slash-star = i-blank / %x21-26 / "\'" / %x28-29 / %x2b-2e
                 / %x30-5b / "\\" / %x5d-7f / i-NONASCII
h-non-star = i-blank / %x21-26 / "\'" / %x28-29 / %x2b-5b
           / "\\" / %x5d-7f / i-NONASCII
h-ends-in-star = *h-non-star 1*"*"
h-comment = "/" h-non-slash-star *h-non-slash "/"
          / "/*" h-ends-in-star
                 *(h-non-slash-star h-ends-in-star) "/"
          / eol-comment *i-non-lf i-LF
~~~
{: #abnf-grammar-sq-h sourcecode-name="cbor-edn-int-hsq.abnf"
title="ABNF Definition for Integrated Hex Parser"
}


### b64'': ABNF Definition of Integrated Parser {#sq-b64-grammar}

With glue ABNF similar to that in {{abnf-grammar-sq-glue}} and common
definitions in Figures {{<abnf-grammar-ext-common}} and {{<abnf-grammar-sq}}, ABNF such as
that shown in {{abnf-grammar-sq-b64}} can be used as an integrated parser
for `b64` prefixed single-quote strings.

~~~ abnf
sq-app-string-b64 = %s"b64'" s-app-string-b64 "'"
s-app-string-b64  = b64-S *(4(b64dig b64-S))
                  [b64dig b64-S b64dig b64-S
                   ["=" b64-S "=" / b64dig b64-S ["="]] b64-S]
                  ["#" *i-non-lf]
b64dig          = ALPHA / DIGIT / "-" / "_" / "+" / "/"
b64-S           = *i-blank *(b64-comment *i-blank)
b64-comment     = "#" *i-non-lf %x0A
~~~
{: #abnf-grammar-sq-b64 sourcecode-name="cbor-edn-int-b64sq.abnf"
title="ABNF Definition for Integrated Base64 Parser"
}


IANA Considerations {#sec-iana}
===================

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFC-XXXX with the RFC
    number of this RFC, \[IANA.cbor-diagnostic-notation] with a
    reference to the new registry group, and remove this note.


## CBOR Diagnostic Notation Application-extension Identifiers Registry {#appext-iana}

IANA is requested to create an "Application-Extension Identifiers"
registry in a new "CBOR Diagnostic Notation" registry group
\[IANA.cbor-diagnostic-notation], with the policy "expert review"
({{Section 4.5 of RFC8126@-ianacons}}).

The experts are instructed to be frugal in the allocation of
application-extension identifiers that are suggestive of generally applicable semantics,
keeping them in reserve for application-extensions that are likely to enjoy wide
use and can make good use of their conciseness.
The expert is also instructed to direct the registrant to provide a
specification ({{Section 4.6 of RFC8126@-ianacons}}), but can make exceptions,
for instance when a specification is not available at the time of
registration but is likely forthcoming.
If the expert becomes aware of application-extension identifiers that are deployed and
in use, they may also initiate a registration on their own if
they deem such a registration can avert potential future collisions.
{: #de-instructions}

Each entry in the registry must include:

{:vspace}
Application-Extension Identifier:
: a lower case ASCII {{-ascii}} string that starts with a letter and can
  contain letters, digits, and hyphens after that (`[a-z][a-z0-9-]*`).
  No other entry in the registry can have the same
  application-extension identifier.

Description:
: a brief description

Change Controller:
: (see {{Section 2.3 of RFC8126@-ianacons}})

Reference:
: a reference document that provides a description of the
  application-extension identifier


The initial content of the registry is shown in {{tab-iana}}; all
initial entries have the Change Controller "IETF".

| Application-extension Identifier | Description                     | Reference        |
|----------------------------------|---------------------------------|------------------|
| h                                | Reserved                        | RFC8949          |
| b32                              | Reserved                        | RFC8949          |
| h32                              | Reserved                        | RFC8949          |
| b64                              | Reserved                        | RFC8949          |
| false                            | Reserved                        | RFC-XXXX         |
| true                             | Reserved                        | RFC-XXXX         |
| null                             | Reserved                        | RFC-XXXX         |
| undefined                        | Reserved                        | RFC-XXXX         |
| dt                               | Date/Time                       | RFC-XXXX         |
| ip                               | IP Address/Prefix               | RFC-XXXX         |
| hash                             | Cryptographic Hash              | RFC-XXXX         |
| cri                              | Constrained Resource Identifier | RFC-XXXX, {{-cri}} |
{: #tab-iana title="Initial Content of Application-extension
Identifier Registry"}



## Encoding Indicators {#reg-ei}

IANA is requested to create an "Encoding Indicators"
registry in the newly created "CBOR Diagnostic Notation" registry group
\[IANA.cbor-diagnostic-notation], with the policy "specification required"
({{Section 4.6 of RFC8126@-ianacons}}).

The experts are instructed to be frugal in the allocation of
encoding indicators that are suggestive of generally applicable semantics,
keeping them in reserve for encoding indicator registrations that are likely to enjoy wide
use and can make good use of their conciseness.
If the expert becomes aware of encoding indicators that are deployed and
in use, they may also solicit a specification and initiate a registration on their own if
they deem such a registration can avert potential future collisions.
{: #de-instructions-ei}

Each entry in the registry must include:

{:vspace}
Encoding Indicator:
: an ASCII {{-ascii}} string that starts with an underscore letter and
  can contain zero or more underscores, letters and digits after that
  (`_[_A-Za-z0-9]*`). No other entry in the registry can have the same
  Encoding Indicator.

Description:
: a brief description.
  This description may employ an abbreviation of the form `ai=`nn,
  where nn is the numeric value of the field _additional information_, the
  low-order 5 bits of the initial byte (see {{Section 3 of RFC8949@-cbor}}).

Change Controller:
: (see {{Section 2.3 of RFC8126@-ianacons}})

Reference:
: a reference document that provides a description of the
  application-extension identifier


The initial content of the registry is shown in {{tab-iana-ei}}; all
initial entries have the Change Controller "IETF".

| Encoding Indicator | Description                        | Reference         |
|--------------------|------------------------------------|-------------------|
| _                  | Indefinite Length Encoding (ai=31) | RFC8949, RFC-XXXX |
| _i                 | ai=0 to ai=23                      | RFC-XXXX          |
| _0                 | ai=24                              | RFC8949, RFC-XXXX |
| _1                 | ai=25                              | RFC8949, RFC-XXXX |
| _2                 | ai=26                              | RFC8949, RFC-XXXX |
| _3                 | ai=27                              | RFC8949, RFC-XXXX |
| _4                 | Reserved (for ai=28)               | RFC-XXXX          |
| _5                 | Reserved (for ai=29)               | RFC-XXXX          |
| _6                 | Reserved (for ai=30)               | RFC-XXXX          |
| _7                 | Reserved (see _)                   | RFC8949, RFC-XXXX |
{: #tab-iana-ei title="Initial Content of Encoding Indicator Registry"}


{:aside}
>
As the "Reference" column reflects, all the encoding indicators
initially registered are already defined in {{Section 8.1 of RFC8949@-cbor}},
with the exception of `_i`, which is defined in {{grammar}} of the
present document.


## Media Type

IANA is requested to add the following Media-Type to the "Media Types"
registry {{IANA.media-types}}.

| Name            | Template                    | Reference              |
| cbor-diagnostic | application/cbor-diagnostic | RFC-XXXX, {{media-type}} |
{: #new-media-type title="New Media Type application/cbor-diagnostic"}

{:compact}
Type name:
: application

Subtype name:
: cbor-diagnostic

Required parameters:
: N/A

Optional parameters:
: N/A

Encoding considerations:
: binary (UTF-8)

Security considerations:
: {{seccons}} of RFC XXXX

Interoperability considerations:
: none

Published specification:
: {{media-type}} of RFC XXXX

Applications that use this media type:
: Tools interchanging a human-readable form of CBOR

Fragment identifier considerations:
: The syntax and semantics of fragment identifiers is as specified for
  "application/cbor".  (At publication of RFC XXXX, there is no
  fragment identification syntax defined for "application/cbor".)

Additional information:
: \\

  Deprecated alias names for this type:
  : N/A

  Magic number(s):
  : N/A

  File extension(s):
  : .diag

  Macintosh file type code(s):
  : N/A

Person & email address to contact for further information:
: CBOR WG mailing list (cbor@ietf.org),
  or IETF Applications and Real-Time Area (art@ietf.org)

Intended usage:
: LIMITED USE

Restrictions on usage:
: CBOR diagnostic notation represents CBOR data items, which are the
  format intended for actual interchange.
  The media type application/cbor-diagnostic is intended to be used
  within documents about CBOR data items, in diagnostics for human
  consumption, and in other representations of CBOR data items that
  are necessarily text-based such as in configuration files or other
  data edited by humans, often under source-code control.

Author/Change controller:
: IETF

Provisional registration:
: no

## Content-Format

IANA is requested to register a Content-Format number in the
{{content-formats ("CoAP Content-Formats")<IANA.core-parameters}}
sub-registry, within the "Constrained RESTful Environments (CoRE)
Parameters" Registry {{IANA.core-parameters}}, as follows:

| Content-Type                | Content Coding | ID   | Reference |
| application/cbor-diagnostic | -              | TBD1 | RFC-XXXX  |
{: #tab-content-format title="New Content-Format for application/cbor-diagnostic"}

TBD1 is to be assigned from the space 256..9999, according to the
procedure "IETF Review or IESG Approval", preferably a number less
than 1000.


Security considerations {#seccons}
=======================

The security considerations of {{-cbor}} and {{-cddl}} apply.

The EDN specification provides two explicit extension points,
application-extension identifiers ({{appext-iana}}) and encoding
indicators ({{reg-ei}}).
Extensions introduced this way can have their own security
considerations (see, e.g., {{Section 5 of -eref}}).
When implementing tools that support the use of EDN extensions, the
implementer needs to be careful not to inadvertently introduce a
vector for an attacker to invoke extensions not planned for by the
tool operator, who might not have considered security considerations
of specific extensions such as those posed by their use of
dereferenceable identifiers ({{Section 6 of -deref}}).
For instance, tools might require explicitly enabling the use of each
extension that is not on an allowlist.
This task can possibly be
made less onerous by combining it with a mechanism for supplying any
parameters controlling such an extension.

--- back

# Changes from draft-ietf-cbor-edn-literals-25

- Normative
  - Removed raw (backtick-quoted) literals
  - Removed stand-in representation
  - Now also obsoletes Section 4.2 of RFC8742

- Non-normative
  - Modified author / contributor list and Acknowlegements
  - Removed Structure of This Document section
  - Removed Appendix A (CDDL comparison to EDN)
  - Rewrote Abstract and Introduction
  - Simplified the Terminology / Conventions section


{::include-all lists.md}


# Acknowledgements
{: numbered="no"}

This document is a heavy-handed rewrite of {{?I-D.ietf-cbor-edn-literals}}.
Much of the text was originally written by Carsten Bormann.
He is listed as a contributor rather than an author because he likely does not agree with the editorial choices made in this document.

The concept of application-oriented extensions to diagnostic notation,
as well as the definition for the "dt" extension, were inspired by the
CoRAL work by Klaus Hartke.

(TBD)
