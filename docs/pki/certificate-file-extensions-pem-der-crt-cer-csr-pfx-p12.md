---
title: "Certificate and CMS File Extensions: PEM, DER, PFX, P7B, P7M and P7S"
description: "Learn PEM, DER, CRT, CSR, PFX, PKCS#7 and CMS file extensions, including P7B, P7M and P7S, and inspect them with OpenSSL."
---

# Certificate and CMS File Extensions: PEM, DER, PFX, P7B, P7M and P7S

When working with certificates and private keys, you will encounter extensions such as `.pem`, `.der`, `.crt`, `.cer`, `.csr`, `.pfx`, `.p12`, `.key`, `.p7b`, `.p7m`, and `.p7s`. They are useful hints, but **a filename extension alone does not reliably tell you what a file contains or how it is encoded**.

<!-- more -->

The key distinction is between the **object**—such as a certificate, private key, or certificate signing request—and its **encoding or container**. A certificate, for example, can be stored as binary DER or as PEM text.

## Certificate and key file formats at a glance

| Extension | Common use | What to check |
| --- | --- | --- |
| `.pem` | PEM-encoded certificate, key, request, or other object | Read the `BEGIN` and `END` labels; one file can contain multiple blocks. |
| `.der` | Binary DER-encoded ASN.1 object | Determine whether it is a certificate, key, or another object. |
| `.crt`, `.cer` | Certificate | Either can contain PEM text or binary DER; verify the actual content. |
| `.csr` | Certificate signing request | Usually a PKCS #10 request in PEM or DER; it is not an issued certificate. |
| `.pfx`, `.p12` | PKCS #12 container | May contain a private key, certificates, and additional certificates; contents vary. |
| `.key` | Usually a private key | May be PEM or DER, encrypted or unencrypted. |
| `.p7b`, `.p7c` | PKCS #7 certificate collection | Can contain certificates and CRLs; inspect the actual structure and encoding. |
| `.p7m`, `.p7s`, `.p7z` | CMS / S/MIME message, signature, or compressed data | The suffix is a hint; check the MIME type, CMS content type, and encoding. |

## What is PEM?

PEM is a text representation commonly used for PKI objects. A PEM block has a Base64-encoded body between labeled boundary lines. The label identifies the kind of object inside. The encoding itself provides **no confidentiality**: Base64 is not encryption. [RFC 7468](https://www.rfc-editor.org/rfc/rfc7468)

A PEM certificate looks like this:

```text
-----BEGIN CERTIFICATE-----
... Base64-encoded certificate data ...
-----END CERTIFICATE-----
```

A PEM file can also contain multiple blocks, such as a certificate followed by intermediate certificates. Applications differ in which objects and ordering they accept.

### PEM private key labels

You may see several labels:

```text
-----BEGIN PRIVATE KEY-----
-----BEGIN ENCRYPTED PRIVATE KEY-----
-----BEGIN RSA PRIVATE KEY-----
```

`PRIVATE KEY` commonly denotes an unencrypted PKCS #8 private-key structure, while `ENCRYPTED PRIVATE KEY` denotes an encrypted PKCS #8 structure. `RSA PRIVATE KEY` identifies a traditional RSA-specific structure; some older encrypted PEM files also contain `Proc-Type` and `DEK-Info` headers. Do not assume a key is encrypted just because it is in PEM format. Keep private keys confidential regardless of extension.

## What is DER?

DER (Distinguished Encoding Rules) is a binary ASN.1 encoding. A DER-encoded certificate is not normally readable in a text editor; use a certificate or ASN.1 viewer to inspect it. DER can encode objects besides certificates, so a `.der` extension does not identify the object type by itself.

PEM commonly wraps an encoded ASN.1 object in labeled Base64 text. Converting a certificate between PEM and DER changes its representation; it does not create a new certificate or change its validity period.

## CRT and CER: certificate extensions

`.crt` and `.cer` usually indicate certificate files. Both extensions can be used for PEM or DER encoding; conventions differ across tools and platforms. Check the contents or use a certificate inspection tool instead of assuming the encoding from the suffix.

A PEM file starts with a visible `-----BEGIN CERTIFICATE-----` line. A DER certificate is binary. A `.crt` or `.cer` file can also contain something unexpected, so successful parsing matters more than its name.

## CSR: certificate signing request

A certificate signing request (CSR) asks a certificate authority to issue a certificate. A PKCS #10 CSR contains a public key and requested subject information, and is signed using the corresponding private key. It can also contain requested attributes or extensions; the CA decides what to include in the issued certificate.

A CSR is **not a certificate** and should not contain the private key. A common PEM label is:

```text
-----BEGIN CERTIFICATE REQUEST-----
... Base64-encoded request data ...
-----END CERTIFICATE REQUEST-----
```

Older tools may use `NEW CERTIFICATE REQUEST`. RFC 7468 specifies the `CERTIFICATE REQUEST` label for conforming generators and allows parsers to recognize the older label. [RFC 7468 §7](https://www.rfc-editor.org/rfc/rfc7468#section-7)

## PFX and P12: PKCS #12 containers

`.pfx` and `.p12` commonly refer to PKCS #12 files. A PKCS #12 container can package a private key and its certificate with additional certificates, making it useful for importing or exporting an identity. The exact contents depend on how the file was created. [RFC 7292](https://www.rfc-editor.org/rfc/rfc7292)

PKCS #12 supports password-based protection, but protection depends on how the file was created and which algorithms and password were used. Treat a container that might hold a private key as sensitive; use a strong password where applicable and transfer it securely.

## PKCS #7 and CMS: certificate collections and secure messages

**PKCS #7** describes a syntax for cryptographic messages. **CMS (Cryptographic Message Syntax)** builds on PKCS #7 and defines structures such as `SignedData` and `EnvelopedData`; CMS and older PKCS #7 formats are related but are not interchangeable in every case. CMS is used by S/MIME and other applications for signed, encrypted, or other protected content. [RFC 5652](https://www.rfc-editor.org/rfc/rfc5652)

Common extensions include:

| Extension | Typical use |
| --- | --- |
| `.p7b` | A PKCS #7 certificate bundle, often containing one or more certificates and possibly CRLs. [Microsoft: certificate export](https://learn.microsoft.com/en-us/sharepoint/administration/export-certificate) |
| `.p7c` | An S/MIME certificates-only message, represented as a degenerate `SignedData` structure. |
| `.p7m` | An S/MIME CMS message, such as attached signed data or enveloped encrypted data. The extension alone does not say which. |
| `.p7s` | An S/MIME detached signature; verification also requires the original signed MIME content. |
| `.p7z` | S/MIME compressed data. |

The S/MIME names above are conventions specified in [RFC 8551](https://www.rfc-editor.org/rfc/rfc8551#section-3.2). Other applications may use different names. A `.p7b` certificate bundle generally does not include a private key; PKCS #12 is commonly used when an identity with its private key must be transferred. Neither an extension nor a certificate bundle establishes trust.

A PKCS #7 or CMS object can be encoded as binary ASN.1 or represented as labeled PEM text; a complete S/MIME message is MIME formatted and can include a Base64 transfer-encoded attachment. The extension by itself does not identify which representation you have. CMS also permits BER encoding, including forms that are not DER.

## KEY: a private key file

A `.key` extension generally indicates a private key, but it does not establish whether the file is PEM or DER, which key algorithm it uses, or whether it is encrypted. Inspect it with a suitable tool and avoid copying its contents into tickets, logs, or public repositories.

## How to inspect certificate files with OpenSSL

These examples assume the named file contains the indicated object. They display information without changing the original file.

```bash
# Inspect a PEM certificate
openssl x509 -in certificate.pem -inform PEM -noout -text

# Inspect a DER certificate
openssl x509 -in certificate.der -inform DER -noout -text

# Inspect a PEM certificate signing request
openssl req -in request.csr -inform PEM -noout -text -verify

# Inspect a PKCS #12 container without outputting its keys or certificates
openssl pkcs12 -in identity.p12 -info -noout

# Print a binary DER CMS structure without verifying its signature
openssl cms -cmsout -print -noout -inform DER -in message.p7m

# Print certificates from a DER-encoded PKCS #7 certificate bundle
openssl pkcs7 -print_certs -inform DER -in certificates.p7b -noout
```

For PEM-encoded PKCS #7 or CMS input, use `-inform PEM`; for a complete MIME-formatted S/MIME message, the CMS command supports `-inform SMIME`. Parsing a CMS structure or printing certificates does not verify signatures, certificate chains, or trust. Detached signature verification also needs the exact signed content. See the OpenSSL [`cms`](https://docs.openssl.org/3.0/man1/openssl-cms/) and [`pkcs7`](https://docs.openssl.org/3.0/man1/openssl-pkcs7/) documentation.

The PKCS #12 command may prompt for a password. A verification result for a CSR checks its signature; it does not mean a CA has issued or approved a certificate. If a command fails, check the object type, encoding, password, and OpenSSL version. See the OpenSSL documentation for [`x509`](https://docs.openssl.org/3.0/man1/openssl-x509/), [`req`](https://docs.openssl.org/3.0/man1/openssl-req/), and [`pkcs12`](https://docs.openssl.org/3.0/man1/openssl-pkcs12/).

### Convert a certificate between PEM and DER

```bash
# PEM to DER
openssl x509 -in certificate.pem -inform PEM -out certificate.der -outform DER

# DER to PEM
openssl x509 -in certificate.der -inform DER -out certificate.pem -outform PEM
```

Choose output filenames that do not overwrite files you need. These commands are for certificates; use the appropriate command for keys or requests.

## Frequently asked questions

### Is a `.crt` file always PEM?

No. A `.crt` certificate can be PEM or DER. Check its contents or parse it with the appropriate input format.

### Are `.pfx` and `.p12` different formats?

Both are commonly used for PKCS #12 containers. Their extensions alone do not reveal the contents, password protection, or algorithms used.

### Is a `.p7m` file always encrypted?

No. It can contain signed data, encrypted enveloped data, or another S/MIME content type. Inspect its MIME and CMS structure.

### Is a PEM private key secure because it is Base64 encoded?

No. Base64 is an encoding, not encryption. Use appropriate key protection and restrict access to the file.

