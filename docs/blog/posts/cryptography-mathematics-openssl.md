---
date: 2023-03-30
categories:
  - Cryptography
title: "Cryptography, Mathematics, and OpenSSL: A PKI Developer's Perspective"
description: "A PKI developer reflects on cryptography, the mathematics behind it, and using OpenSSL to compare results while building security tools."
---

# Cryptography, Mathematics, and OpenSSL: A PKI Developer's Perspective

Working on public key infrastructure (PKI) often brings me face to face with cryptography and mathematics. Cryptographic algorithms depend on mathematical ideas that can be challenging for a developer whose background is in electronics or computer science. My experience has taught me to understand the concepts and behavior of an algorithm while relying on well-reviewed libraries for its implementation.

<!-- more -->

## Using cryptography without implementing every algorithm

PKI software needs cryptographic operations: generating keys, creating signatures, and checking certificates, among others. In practice, I usually use an established cryptographic library rather than implement the underlying algorithms myself.

That does not mean the mathematics is irrelevant. I need to understand what an algorithm is supposed to do, its inputs and outputs, and the properties that matter when using it. But understanding an algorithm at that level is different from being able to diagnose a subtle error in its mathematical implementation.

## Why comparing results matters

When a calculation produces an unexpected result, limited knowledge of the underlying mathematics can make the cause difficult to find. Is the issue in the algorithm, the input, the data format, or the way the operation was called?

One way I investigate is to compare the same operation against a trusted, independently implemented library. A difference gives me a starting point for checking the inputs, parameters, and output formats. A match is useful evidence, although it does not by itself prove that an implementation is secure or correct in every case.

## OpenSSL as a reference for PKI development

I often use [OpenSSL](https://www.openssl.org/) as a reference when working with cryptographic operations. Its widely used implementation gives me a practical way to inspect data and compare results while developing PKI software.

Of course, no library is completely free of bugs. OpenSSL still needs appropriate configuration, maintenance, and security updates. For my work, its value is that it provides a well-established point of comparison when I am investigating an unexpected result.

## From learning to building tools

Working through these questions and checking results with OpenSSL led me to build several tools of my own. They help me inspect and experiment with the formats and operations used in PKI.

For me, developing cryptographic software is a continuing process of learning: understand the concepts, use established implementations carefully, and investigate differences rather than assuming the first result is correct.

---

*Originally published in Korean on March 30, 2023: [암호학과 수학 그리고 OpenSSL에 관해](https://jykim74.tistory.com/98).*
