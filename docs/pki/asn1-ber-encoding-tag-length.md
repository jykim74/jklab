---
title: "ASN.1 BER Encoding Guide: TLV, Types, and Examples"
description: "Learn ASN.1 BER encoding with TLV, tags, lengths, data types, OIDs, and practical hexadecimal examples."
---

# ASN.1 Basic Encoding Rules (BER): TLV and Encoding Examples

Basic Encoding Rules (BER) define how to encode values described using ASN.1 (Abstract Syntax Notation One). This guide explains the **tag-length-value (TLV)** structure, identifiers, lengths, and common ASN.1 types with hexadecimal examples.

## The BER tag-length-value structure

A BER encoding consists of an identifier, a length, and contents:

```text
+----------------+----------------+----------------+
| Tag            | Length         | Value          |
+----------------+----------------+----------------+
```

- **Tag**, also called the identifier, identifies the class, form, and tag number.
- **Length** specifies the number of content octets in a definite-length encoding, or indicates an indefinite-length encoding.
- **Value**, also called the contents, contains the encoded data.

A constructed value can contain nested TLV encodings. For example, the value of an outer element can contain two encoded elements:

```text
Outer tag | Outer length | Tag 1 | Length 1 | Value 1 | Tag 2 | Length 2 | Value 2
                         |<---------------- outer contents ---------------->|
```

## BER identifier encoding

The first identifier octet contains the tag class, the primitive/constructed bit, and either the tag number or an indication that more identifier octets follow. Bit 7 is the most significant bit in the diagrams below.

```text
Bits:  7  6 | 5 | 4  3  2  1  0
      class | P/C | tag number
```

### Tag classes

| Bits 7–6 | Class |
|---|---|
| `00` | Universal |
| `01` | Application |
| `10` | Context-specific |
| `11` | Private |

### Primitive and constructed forms

Bit 5 indicates the form:

| Bit 5 | Form | Meaning |
|---|---|---|
| `0` | Primitive | The contents represent the value directly. |
| `1` | Constructed | The contents contain encoded components. |

### Tag number encoding

For tag numbers **0–30**, bits 4–0 contain the tag number directly. For tag numbers **31 or greater**, those bits are `11111`, and subsequent octets encode the tag number in groups of seven bits, most significant group first. The high bit of each subsequent octet is `1` if another octet follows and `0` for the last octet.

For tag numbers 31–127, one subsequent octet is sufficient. Larger tag numbers require two or more subsequent octets.

### Identifier examples

| Class and form | Tag number | Binary identifier | Hexadecimal |
|---|---:|---|---|
| Universal, primitive | 2 | `00000010` | `02` |
| Private, constructed | 13 | `11101101` | `ED` |
| Application, primitive | 40 | `01011111 00101000` | `5F 28` |

For example, `5F 28` starts with application class (`01`), primitive form (`0`), and `11111`, indicating a high tag number. The next octet ends in the seven-bit value `0101000`, or 40 in decimal.

## BER length encoding

BER supports definite-length and indefinite-length encodings. Definite length has short and long forms; both encode the number of **content octets**, excluding the identifier and length octets.

### Short form

The short form uses one octet. Its most significant bit is `0`, and its remaining seven bits hold a content length from 0 to 127.

```text
0 | length in 7 bits
```

For a content length of 5 octets, the length octet is `00000101`, or `05` in hexadecimal.

### Long form

In the long form, the most significant bit of the first length octet is `1`. The remaining seven bits specify the number of subsequent length octets. Those octets encode the content length as an unsigned binary integer, most significant octet first.

```text
1 | number of length octets | length octet(s)
```

For a content length of 1,020 octets:

```text
Binary: 10000010 00000011 11111100
Hex:    82       03       FC
```

`82` indicates that two length octets follow; `03 FC` represents 1,020 in decimal.

### Indefinite form

The indefinite form uses the length octet `80` and does not state the content length. Instead, an **end-of-contents** marker (`00 00`) terminates the contents:

```text
Tag | 80 | Contents | 00 00
```

BER permits indefinite length for constructed encodings; primitive encodings use definite length. DER, a more restrictive set of encoding rules based on BER, does not permit indefinite lengths.

## BER encoding examples for ASN.1 types

The following examples use hexadecimal octets. Unless stated otherwise, the examples use primitive encodings and definite lengths. BER can permit other valid encodings; these examples do not enumerate every alternative.

### BOOLEAN

A BOOLEAN has universal tag `01` and one content octet. `00` represents FALSE; any nonzero octet represents TRUE in BER. DER requires `FF` for TRUE.

```text
TRUE:   01 01 FF
FALSE:  01 01 00
        tag length value
```

### INTEGER

An INTEGER has universal tag `02`. Its contents use the minimum number of octets needed for a signed two's-complement value, most significant octet first. A leading `00` may be needed to keep a positive value positive; negative values use two's complement.

```text
72:  02 01 48
-2:  02 01 FE
```

### BIT STRING

For a primitive BIT STRING, tag `03` is followed by the length and contents. The first content octet gives the number of unused bits in the final data octet. The data bits are packed into octets, with padding at the end if necessary.

```text
Value:    '101100001001'B
Encoding: 03 03 04 B0 90
          │  │  │  └───── data: 10110000 10010000
          │  │  └──────── 4 unused bits in the last octet
          │  └─────────── 3 content octets
          └────────────── BIT STRING tag
```

### OCTET STRING and NULL

A primitive OCTET STRING uses tag `04`; its contents are the octets of the value. NULL uses tag `05` and has no content octets.

```text
'A24F'H:  04 02 A2 4F
NULL:     05 00
```

### Explicit and implicit tagging

An **EXPLICIT** tag wraps the complete encoding of the underlying value. An **IMPLICIT** tag replaces its identifier. In this example, `[21]` is context-specific and the underlying BOOLEAN is TRUE:

```asn1
ExplicitPresent ::= [21] EXPLICIT BOOLEAN
ImplicitPresent ::= [21] IMPLICIT BOOLEAN
```

```text
EXPLICIT: B5 03 01 01 FF
          └ outer ┘ └ BOOLEAN ┘
IMPLICIT: 95 01 FF
```

`B5` is constructed because it wraps another TLV; `95` is primitive because it replaces the primitive BOOLEAN identifier. The original example treats `[21] BOOLEAN` as explicit; writing `EXPLICIT` makes that assumption clear regardless of the module's tagging default.

### SEQUENCE

A SEQUENCE encodes its components in their declared order. Its universal constructed identifier is `30`.

```asn1
PersonnelRecord ::= SEQUENCE {
  name     OCTET STRING,
  location INTEGER { homeOffice(0), fieldOffice(1), roving(2) },
  age      INTEGER OPTIONAL
}

rockStar1 PersonnelRecord ::= {
  name '626967'H, location roving, age 26
}
```

```text
30 0B                SEQUENCE, 11 content octets
   04 03 62 69 67    name: '626967'H
   02 01 02          location: roving (2)
   02 01 1A          age: 26
```

### SEQUENCE OF

A SEQUENCE OF encodes an ordered collection of values of the same type. For `{ 10, 12, -2, 8 }`:

```text
30 0C          SEQUENCE OF, 12 content octets
   02 01 0A    10
   02 01 0C    12
   02 01 FE    -2
   02 01 08     8
```

### SET

A SET contains named components whose order is not significant in BER. The following example uses an implicitly tagged SET and context-specific implicit tags for its fields:

```asn1
PersonnelSet ::= [0] IMPLICIT SET {
  name     [0] IMPLICIT OCTET STRING,
  location [1] IMPLICIT INTEGER { home(0), field(1), roving(2) } OPTIONAL,
  age      [2] IMPLICIT INTEGER OPTIONAL
}
```

For location `home`, name `'44617679'H`, and age 44, one BER encoding is:

```text
A0 0C                   context-specific constructed [0], 12 content octets
   81 01 00             location: home (0)
   80 04 44 61 76 79    name: '44617679'H
   82 01 2C             age: 44
```

DER imposes additional ordering rules, so this BER example should not be assumed to be DER.

### CHOICE

A CHOICE represents one alternative; the chosen alternative's encoding identifies it. Here the selected `r-and-d` alternative is an implicitly tagged SEQUENCE:

```asn1
Division ::= CHOICE {
  manufacturing [0] IMPLICIT SEQUENCE {
    plantID INTEGER, majorProduct OCTET STRING
  },
  r-and-d [1] IMPLICIT SEQUENCE {
    labID INTEGER, currentProject OCTET STRING
  }
}

currentAssignment Division ::= r-and-d : {
  labID 48, currentProject '44582D37'H
}
```

```text
A1 09                   context-specific constructed [1], 9 content octets
   02 01 30             labID: 48
   04 04 44 58 2D 37    currentProject: '44582D37'H
```

The source illustration labels the outer length `0C`; the two components shown occupy **9** octets, so the corrected length is `09`.

### OBJECT IDENTIFIER

An OBJECT IDENTIFIER (OID) uses universal tag `06`. Combine the first two arcs as `40 × first arc + second arc`; encode that combined value and each remaining arc as a base-128 subidentifier, most significant group first. Each octet's high bit is `1` if another octet of the same subidentifier follows and `0` on the last octet. The combined first subidentifier can itself require more than one octet.

For OID `1.0.8571.2.1`:

```text
06 05 28 C2 7B 02 01
      │  └───┘ │  └── arc 1
      │   8571 └───── arc 2
      └────────────── 40 × 1 + 0 = 40 (28 hex)
```

`C2 7B` represents `66 × 128 + 123 = 8571`; its first high bit indicates continuation.

### GeneralizedTime and UTCTime

These types encode their textual time representations as character octets. For example, the character `1` is `31` and the decimal point is `2E` in hexadecimal.

```text
GeneralizedTime "19960415203000.0":
18 10 31 39 39 36 30 34 31 35 32 30 33 30 30 30 2E 30

UTCTime "9604152030Z":
17 0B 39 36 30 34 31 35 32 30 33 30 5A
```

These are BER examples. DER and specific profiles can impose stricter requirements on time representations.

## References

Adapted from the Korean series: [Basic Encoding Rules (1/3)](https://jykim74.tistory.com/56), [part 2](https://jykim74.tistory.com/57), and [part 3](https://jykim74.tistory.com/58).
