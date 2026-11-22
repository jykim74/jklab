---
title: "TLS and SSL Certificate Validity Periods: 2026–2029 Policy"
description: "Understand public TLS certificate validity limits: 200 days in 2026, 100 days in 2027, and 47 days in 2029, plus domain validation and renewal guidance."
---

# TLS and SSL Certificate Validity Periods: Policy and 2026–2029 Timeline

**Publicly trusted TLS server certificates issued from March 15, 2026 have a maximum validity of 200 days.** That limit falls to 100 days on March 15, 2027 and 47 days on March 15, 2029. These are maximums: a certificate authority (CA) can issue a certificate for a shorter period. The rules apply according to the certificate's issuance date. [CA/Browser Forum, TLS Baseline Requirements §6.3.2](https://cabforum.org/working-groups/server/baseline-requirements/requirements/)

This guide describes the CA/Browser Forum's policy for **publicly trusted TLS server certificates**, commonly called SSL certificates. It is current as of November 22, 2026. Private PKI certificates, client certificates, code-signing certificates, and CA certificates have different applicable policies; do not assume this schedule governs them.

## Public TLS certificate maximum validity timeline

| Certificate issuance date | Maximum validity |
| --- | ---: |
| Before March 15, 2026 | 398 days |
| March 15, 2026 through March 14, 2027 | 200 days |
| March 15, 2027 through March 14, 2029 | 100 days |
| On or after March 15, 2029 | 47 days |

The limits are specified in [§6.3.2 of the TLS Baseline Requirements](https://cabforum.org/working-groups/server/baseline-requirements/requirements/). For example, a certificate issued on March 14, 2027 falls under the 200-day limit; one issued on March 15 falls under the 100-day limit. A policy change does not, by itself, shorten the validity dates already encoded in an existing certificate. Existing certificates must still satisfy the rules applicable to their issuance and remain subject to revocation and relying-party policies.

The requirements calculate a day as 86,400 seconds, and any extra time counts as another day. They also recommend a one-day margin: for example, certificates issued in the 200-day period should not exceed 199 days by default. The mandatory maximum and recommended margin are distinct. Check the CA's actual certificate lifetime rather than assuming every CA issues for the maximum.

## What is a TLS certificate validity period?

An X.509 certificate contains `notBefore` and `notAfter` timestamps. Its validity period is the interval between them. A relying party checks the time as part of certificate validation, along with other requirements such as the hostname, chain of trust, and applicable policies. A valid date alone does not guarantee trust. [RFC 5280 §4.1.2.5](https://www.rfc-editor.org/rfc/rfc5280#section-4.1.2.5)

A renewed certificate is a newly issued certificate with its own validity period. Buying a multi-year service subscription does not extend an individual public TLS certificate beyond the applicable limit: the service must issue and deploy replacement certificates during the subscription.

## Domain validation reuse is a separate limit

Before issuing a public TLS certificate, a CA must validate authorization for the domain names or IP addresses. The CA/Browser Forum separately limits how long the CA may reuse eligible validation data or a completed validation. These limits are measured **before issuance**, not from the certificate's expiration date. [TLS Baseline Requirements §4.2.1](https://cabforum.org/working-groups/server/baseline-requirements/requirements/)

| Certificate issuance date | Maximum domain/IP validation reuse |
| --- | ---: |
| Before March 15, 2026 | 398 days |
| March 15, 2026 through March 14, 2027 | 200 days |
| March 15, 2027 through March 14, 2029 | 100 days |
| On or after March 15, 2029 | 10 days |

From March 15, 2026, the maximum reuse period for subject identity information validation is 398 days under the same section. Individual validation methods have additional conditions, and a CA may use shorter periods. In particular, **10 days is not the certificate lifetime**: it is the future maximum reuse window for eligible domain/IP validation information. A certificate issued after March 15, 2029 can have a validity period of up to 47 days.

## Why are certificate lifetimes getting shorter?

Shorter lifetimes limit the time a misissued certificate or compromised key can remain usable before the certificate expires. They also encourage automated issuance and replacement. Expiration is not a substitute for revocation: a compromised or incorrectly issued certificate may need to be revoked promptly. [Let's Encrypt: Certificate Lifetime Rationale and Plans](https://letsencrypt.org/docs/cert-lifetimes/)

The trade-off is operational: certificates must be replaced more often, so failed validation, inaccessible challenge endpoints, deployment errors, or missed reloads can cause outages unless the lifecycle is monitored.

## How to prepare for shorter TLS certificate lifetimes

1. **Inventory certificates and endpoints.** Include load balancers, reverse proxies, CDNs, APIs, mail servers, appliances, and internal deployment targets that use public certificates.
2. **Automate issuance and deployment.** Use an ACME client or your CA's supported automation. Ensure that the certificate and corresponding private key reach every endpoint and that services reload them safely. [RFC 8555: ACME](https://www.rfc-editor.org/rfc/rfc8555)
3. **Automate validation.** Check that the chosen domain-control challenge can be completed on every renewal, including any DNS or network permissions it needs. Wildcard certificates require an appropriate validation method, such as DNS-01.
4. **Renew early and monitor the deployed certificate.** Allow time for retries and alert on renewal failures, imminent expiration, and certificates that were issued but not deployed. Verify each externally reachable endpoint, not just the CA's issuance dashboard.
5. **Test failure recovery.** Exercise DNS changes, CA access failures, credential rotation, deployment rollback, and certificate reloads. Keep challenge credentials and private keys protected.
6. **Check vendor and CA policies.** Devices that require manual import, strict change windows, or proprietary APIs may need replacement or a different deployment workflow before the 47-day limit takes effect.

ACME Renewal Information (ARI) lets a CA suggest a renewal window to compatible clients; it can help spread renewal load and respond to CA guidance. Consult the CA and client documentation for support. [RFC 9773](https://www.rfc-editor.org/rfc/rfc9773)

## Example: Let's Encrypt's schedule

As of its July 2026 documentation, Let's Encrypt's default certificates have a 90-day lifetime; it also offers shorter profiles. Its published plan moves the default to **64 days on February 10, 2027** and then **45 days on February 16, 2028**. These are that CA's planned lifetimes, not the industry-wide maximums. Confirm the current plan and the profile used by your ACME client before making changes. [Let's Encrypt: Upcoming Features](https://letsencrypt.org/ca/upcoming-features/), [64-Day Certificate Lifetimes](https://letsencrypt.org/2026/10/07/64-day-certs.html)

## Frequently asked questions

### Are SSL certificates limited to 47 days now?

No. As of November 22, 2026, the CA/Browser Forum maximum for newly issued publicly trusted TLS server certificates is 200 days. The 47-day maximum is scheduled for certificates issued on or after March 15, 2029. A CA may already offer a shorter lifetime.

### Does a 47-day certificate require domain validation every 47 days?

Not necessarily. The validation reuse limit and the certificate validity limit are different. From March 15, 2029, the maximum reuse window for eligible domain/IP validation is 10 days before issuance. A CA may validate again sooner or apply method-specific requirements.

### Can a private CA issue certificates with longer lifetimes?

The public TLS Baseline Requirements describe publicly trusted TLS server certificates; they do not automatically set the same limit for a separately trusted private PKI. Private certificates are still governed by their own CA policy, software, platform, and organizational requirements.

### How can I check a certificate's dates?

For a local PEM certificate, use OpenSSL:

```bash
openssl x509 -in certificate.pem -noout -subject -issuer -dates
```

To inspect the leaf certificate presented by a server, use the correct hostname for both connection and SNI:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates
```

Replace `example.com` with your hostname. This displays the presented certificate's dates; it is not a complete certificate validation or an inventory of every backend.

## Sources

- [CA/Browser Forum TLS Baseline Requirements](https://cabforum.org/working-groups/server/baseline-requirements/requirements/), §§4.2.1 and 6.3.2; version 2.3.1, October 4, 2026.
- [CA/Browser Forum Ballot SC081v3](https://cabforum.org/2025/04/11/ballot-sc081v3-introduce-schedule-of-reducing-validity-and-data-reuse-periods/), the adopted schedule for shorter validity and validation reuse periods.
- [RFC 5280: Internet X.509 Public Key Infrastructure Certificate and CRL Profile](https://www.rfc-editor.org/rfc/rfc5280#section-4.1.2.5), certificate validity.
- [RFC 8555: Automatic Certificate Management Environment (ACME)](https://www.rfc-editor.org/rfc/rfc8555) and [RFC 9773: ACME Renewal Information](https://www.rfc-editor.org/rfc/rfc9773).
- [Let's Encrypt: Certificate Lifetime Rationale and Plans](https://letsencrypt.org/docs/cert-lifetimes/) and [Upcoming Features](https://letsencrypt.org/ca/upcoming-features/).
