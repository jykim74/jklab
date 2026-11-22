---
date: 2023-02-24
categories:
  - PKI
  - Development
title: "Why I Use OpenSSL and Qt to Build PKI Tools"
description: "A developer shares why OpenSSL and Qt are useful for building cross-platform PKI tools, along with their trade-offs and a comparison with mbed TLS."
---

# Why I Use OpenSSL and Qt to Build PKI Tools

When I build public key infrastructure (PKI) tools, I use OpenSSL for cryptographic and PKI functionality and Qt for the user interface. They are not the only choices, but together they give me the capabilities and cross-platform support I need.

<!-- more -->

## Why OpenSSL for PKI development?

Of the PKI libraries I have personally used, OpenSSL has offered the broadest range of functionality. My experience does not cover every available library, but OpenSSL has been a useful foundation for working with cryptography and PKI data.

Its long history, wide use, and availability across platforms also make it a valuable reference when I need to investigate an operation or compare results. PKI relies heavily on standards, and being able to compare behavior with an established implementation helps me understand interoperability problems.

### When I use mbed TLS

For smaller projects, I have also used mbed TLS. I have found it a capable cryptographic library, but in my experience it has offered less of the broad PKI functionality I need for tools that work with complex ASN.1 formats and protocols such as OCSP, TSP, and CMP. The right choice depends on the version and the requirements of the project.

### The trade-off: OpenSSL can be complex

OpenSSL's breadth comes with a learning curve. I have found parts of its code difficult to follow, and some internal APIs less thoroughly documented than I would like. ASN.1 handling also makes extensive use of macros. Understanding a large library takes time, especially when working beyond its public interfaces.

For licensing, check the license for the specific OpenSSL release and all dependencies before distribution. OpenSSL 3.0 and later use the Apache License 2.0; older releases have different terms. [OpenSSL licensing](https://www.openssl-library.org/source/license/)

## Why Qt for cross-platform user interfaces?

I use Qt because it makes user interface development approachable and supports desktop platforms beyond Windows. Earlier in my development work, I used MFC with Visual Studio and focused on Windows. With Qt, I found it easier to write and maintain code intended for Windows, macOS, and Linux.

Qt also supports mobile platforms, although I have not personally used it for iOS or Android development. Cross-platform support does not remove every platform-specific task: building, testing, packaging, and distribution still need attention on each target system.

## Building BerEditor, CertMan, and CryptokiMan

Using Qt made it easier for me to compile and port BerEditor, CertMan, and CryptokiMan to macOS and Linux. At the time I originally wrote this post, I distributed macOS builds alongside the Windows versions. Linux packaging was more complicated and I had not yet made a Linux release available; I hoped to share one when time allowed.

That is a snapshot of my experience in 2023, not a current release list. For availability, check the [JKLab website](https://jykim74.github.io/jklab/).

## Choosing tools for your own project

OpenSSL and Qt suit the tools I build, but another combination may be better for a different application. Consider the cryptographic operations and protocols you need, the platforms you support, resource limits, maintenance effort, documentation, and applicable licenses.

---

*Originally published in Korean on February 24, 2023: [OpenSSL 과 QT로 개발 하는 이유는?](https://jykim74.tistory.com/82).*
