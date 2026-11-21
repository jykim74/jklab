---
title: "ASN.1 BER vs DER: Encoding Rules and Examples"
description: "Compare ASN.1 BER and DER encoding rules with hexadecimal examples, including lengths, Boolean values, strings, certificates, and signatures."
---

# ASN.1 BER vs DER: Differences and Encoding Examples

**BER (Basic Encoding Rules)** and **DER (Distinguished Encoding Rules)** specify how to encode ASN.1 values as bytes. BER allows multiple valid encodings for some values. DER adds restrictions to BER so that a given value, under a given ASN.1 type and schema, has a canonical encoding. Consequently, every valid DER encoding is also valid BER, but not every valid BER encoding is valid DER.

This guide compares the rules with hexadecimal examples and explains why DER matters for X.509 certificates and digital signatures. Both sets of rules are specified in [ITU-T X.690](https://www.itu.int/rec/T-REC-X.690/en).

## ASN.1, BER, and DER at a glance

**ASN.1 (Abstract Syntax Notation One)** describes data types and structures independently of their encoded form. Encoding rules specify how an ASN.1 value is represented as bytes for storage or transfer.

| Feature | BER | DER |
| --- | --- | --- |
| Purpose | Flexible encoding of ASN.1 values | Canonical encoding with additional restrictions |
| Encodings for a value | Some values have multiple valid encodings | One canonical encoding for a given value and type |
| Length | Definite, or indefinite for constructed encodings | Definite, using the shortest permitted form |
| Boolean `TRUE` | Any nonzero contents octet | Contents octet must be `FF` |
| String encoding | Some string types allow primitive or constructed forms | String types covered below use primitive form |
| `SET` and `SET OF` | Component order can vary within BER rules | Canonical ordering rules apply |
| Typical use | Protocols and applications that allow BER | X.509 certificates and other formats requiring canonical bytes |

The applicable protocol or file format determines which rules to use. DER is useful when a standard requires reproducible bytes; BER is useful when its flexibility is allowed.

## How ASN.1 tag-length-value encoding works

BER and DER encodings generally follow a **tag-length-value (TLV)** structure:

```text
Tag       Length    Contents
01        01        FF
```

In this example, `01` is the universal tag for `BOOLEAN`, the next `01` says the contents are one octet long, and `FF` represents `TRUE`. A tag or length can occupy more than one octet. For constructed values, the contents contain encodings of component values.

## Boolean values: flexible BER versus canonical DER

A BOOLEAN has exactly one contents octet. Under BER, `00` means `FALSE` and any nonzero octet means `TRUE`. DER requires `FF` for `TRUE`:

```text
01 01 00    FALSE: valid BER and DER
01 01 01    TRUE:  valid BER, but not DER
01 01 FF    TRUE:  valid BER and DER
```

The two `TRUE` encodings represent the same ASN.1 value but differ in bytes. DER removes that choice.

## Length encoding: definite and indefinite forms

### Definite length

A definite length specifies the number of contents octets before the contents. For lengths from 0 through 127, the short form uses one length octet:

```text
04 03 41 42 43
│  │  └─────── Contents: ASCII bytes for ABC
│  └────────── Length: 3 octets
└───────────── OCTET STRING tag
```

For larger lengths, the long form uses an initial octet indicating how many subsequent octets encode the length. For example, `81 80` specifies a contents length of 128 octets.

BER permits long-form lengths even where the short form would suffice. DER requires the shortest permitted length form:

```text
04 03    41 42 43    OCTET STRING "ABC": valid BER and DER
04 81 03 41 42 43    OCTET STRING "ABC": valid BER, but not DER
```

### Indefinite length

BER also permits an **indefinite length** for constructed encodings. The length octet is `80`, and an end-of-contents marker, `00 00`, terminates the contents:

```text
30 80             SEQUENCE, indefinite length
   02 01 05       INTEGER 5
00 00             End-of-contents marker
```

This is valid BER but not DER. A DER encoding of the same sequence uses a definite length:

```text
30 03             SEQUENCE, 3 contents octets
   02 01 05       INTEGER 5
```

Indefinite length can be useful when encoding constructed data incrementally. It is not permitted for primitive values, and DER does not permit it for any value.

## Primitive and constructed string encodings

BER allows certain string types, including `OCTET STRING`, to use either primitive encoding or constructed encoding made of segments. Consider the octets for `ABC`:

```text
04 03 41 42 43                  Primitive OCTET STRING: BER and DER

24 80                           Constructed OCTET STRING: BER only
   04 02 41 42                  Segment: AB
   04 01 43                     Segment: C
00 00                           End-of-contents marker
```

The constructed example is valid BER, but not DER. DER requires the primitive form for `OCTET STRING` and `BIT STRING`, as well as the other restricted character string types covered by X.690. Constructed encoding itself is not forbidden in DER: types such as `SEQUENCE` and `SET` are constructed.

## SET ordering and default values

DER adds rules beyond Boolean values and lengths:

- **`SET`:** Components are encoded in the canonical order specified by their tags.
- **`SET OF`:** Encoded elements are sorted according to the ordering rules for their complete encodings. This is distinct from the `SET` rule.
- **`DEFAULT` components:** A component whose value equals its ASN.1 default must be omitted in DER. BER permits it to be encoded explicitly.

For example, with `Example ::= SEQUENCE { flag BOOLEAN DEFAULT FALSE }`, both examples below are valid BER encodings of the default-valued sequence, but only the first is DER:

```text
30 00                Empty SEQUENCE: default applies; BER and DER
30 03 01 01 00       Explicit FALSE: BER, not DER
```

The schema is essential to interpreting a default value; the bytes alone do not identify the ASN.1 definition.

## INTEGER encoding: a rule BER already enforces

Not every constraint is unique to DER. BER already requires an `INTEGER` to use the minimum number of contents octets needed for its signed two's-complement value. For example:

```text
02 01 7F       INTEGER 127: valid BER and DER
02 02 00 80    INTEGER 128: valid BER and DER; 00 keeps it positive
02 02 00 7F    INTEGER 127: invalid BER and DER; 00 is redundant
```

An extra leading octet is therefore not a valid way to illustrate a BER-versus-DER difference for an INTEGER.

## Why DER matters for X.509 certificates and digital signatures

A digital signature is calculated over bytes. If two encodings represent the same logical value but differ byte for byte, their hashes can differ too. Canonical DER encoding provides a consistent representation when a specification requires one.

For example, [RFC 5280](https://www.rfc-editor.org/rfc/rfc5280.html) specifies DER encoding of `tbsCertificate` for X.509 certificate signature calculation. A verifier must follow the relevant protocol and verify the exact signed data according to its rules; simply decoding and re-encoding arbitrary input does not make it valid DER or guarantee the correct verification result.

## BER, DER, and PEM: what is the difference?

BER and DER are ASN.1 encoding rules. **PEM** is a text format commonly used to carry Base64-encoded binary data between labeled boundary lines. A PEM certificate typically contains DER-encoded X.509 data:

```text
-----BEGIN CERTIFICATE-----
...Base64-encoded certificate bytes...
-----END CERTIFICATE-----
```

A file extension alone does not reliably identify the encoding. Check the file contents and the format expected by the application.

## How to inspect BER and DER data

[Download BerEditor](../bereditor/download.md) to inspect and edit ASN.1 BER/DER data. When comparing an encoding with DER requirements, check its length forms, primitive or constructed representation, ordering, default values, and any type-specific constraints. A BER parser accepting a file does not by itself prove that the file is DER-compliant.

For related concepts, see [Public Key Infrastructure (PKI) Concepts](130-Public-Key-Infrastructure-PKI-Concepts.md) and [Private Keys, Public Keys, and Certificates](259-Private-Keys-Public-Keys-and-Certificates.md).

## References

- [ITU-T X.690: BER, CER, and DER encoding rules](https://www.itu.int/rec/T-REC-X.690/en)
- [RFC 5280: Internet X.509 Public Key Infrastructure Certificate and CRL Profile](https://www.rfc-editor.org/rfc/rfc5280.html)
