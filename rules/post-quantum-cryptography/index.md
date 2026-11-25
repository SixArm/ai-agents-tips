# Post-quantum cryptography

Guidance for any typical coding project, in any typical programming language. Researched 2026-11-25 from public sources. **No cryptographer, security officer, lawyer or data protection officer has reviewed it.** It says which rules exist and how strong they are, which standards to use, how to find what is at risk, an order of work, and what to use in each language. It is not legal advice, and following it does not make a project compliant with anything.

## The short answer

1. **The risk that grows today is recorded traffic.** Someone may record encrypted connections now and read them when a large quantum computer exists. So key exchange comes first, and signatures after, for any data that must stay secret for years.
2. **For most projects the first step is an upgrade, not code.** Use TLS 1.3 with the hybrid group `X25519MLKEM768`. Current releases of Go, Node.js, Java, OpenSSL and `rustls` prefer it by default, and so do current browsers [CF].
3. **The upgrade fails quietly.** An old runtime, an old linked OpenSSL, a pinned curve list, a different TLS provider or a proxy removes the hybrid, and the connection still succeeds. Check what is negotiated.
4. **What is stored and hashed is mostly symmetric**, which NIST and the UK's cyber security centre say can stay [IR8547] [NCSC24].
5. **No source read for this page requires post-quantum cryptography of every software project.** The dates are guidance, or bind particular organisations. Planning by 2028 is still the sensible reading.

## Common claims, checked

| Claim often made | Found | Verdict |
|---|---|---|
| The UK sets 2028 (inventories and plans) and 2035 (full transition) | Yes, and **2031** between them: the highest-priority migration done and the plan refined [NCSC25] | Right, incomplete |
| These are "strict deadlines" and "mandates" | The UK page is guidance, calls the dates "indicative", and says organisations "should work towards these key dates". It is aimed mainly at large organisations, critical national infrastructure and bespoke IT [NCSC25] | Overstated for the UK |
| The UK recommends ML-KEM-768 | Yes, and ML-DSA-65, "for most use cases" [NCSC24] | Right |
| SLH-DSA, LMS or XMSS for software and firmware signing | Yes, "not suitable for general purpose use"; LMS and XMSS only where state can be managed in a trusted manner [NCSC24] | Right, with a condition left out |
| Only AES-256 is quantum-safe | No. The UK says symmetric algorithms with at least 128-bit keys can continue, and NIST puts approved symmetric primitives of at least 128 bits in category 1 or above [NCSC24] [IR8547] | Narrower than the rule |
| Hybrids are a temporary bridge | The UK: if a hybrid is chosen, as "an interim measure" [NCSC24]. NIST allows one when at least one part is approved [IR8547] | Right |
| NIST deprecates RSA and ECC after 2030 and disallows them after 2035 | Yes. The report is **final since 27 October 2026**; the 2024 draft deprecated only the 112-bit ones [IR8547] | Right |
| Executive Order 14412 "mandates" it for US federal contractors by late 2030 | The order (22 June 2026) tells the procurement council to **propose a rule** within 180 days that would require covered contractors to comply by 31 December 2030. Agencies: key establishment by 31 December 2030 and **signatures by 31 December 2031**, for high value assets and high impact systems [EO14412]. Whether the rule has been proposed was not checked | Overstated; the rule is not made |
| EU: strategies by 2026, high-risk systems by 2030, all by 2035 | The Commission's page says Member States "should start" by the end of 2026 and high-risk use cases should move no later than the end of 2030; the roadmap is a set of recommendations aimed at Member States. **2035 is not on the two Commission pages read**; other sources give it for medium risk [EU] | Mostly right; not binding on a supplier |
| DORA requires cryptographic agility, and critical infrastructure must move by 2030 | Two different things. DORA's technical standard, for **financial entities**, requires provisions for changing cryptography as cryptanalysis develops (Article 6(4)), and a recital names quantum threats. It names no post-quantum algorithm and no date [DORA]. The 2030 date is the roadmap's | Merged; corrected |
| PCI DSS 4.0 requires inventories and "quantum readiness plans" | Requirement 12.3.3 asks for an inventory of cipher suites and protocols, a review at least every 12 months and a plan for changes. Two secondary sources agree that it does not name quantum; the standard itself was not read [PCI] | Half right; applies where cardholder data is held |
| "Regulators expect" a privacy assessment that addresses quantum risk, vendor attestations and pilot results | No source was found that says so | Not supported; sensible as practice |

## Rules, and what they mean for a project

| Place | Who it binds | Dates | For a project |
|---|---|---|---|
| United Kingdom, NCSC [NCSC25] | Nobody by law; guidance to large organisations, critical infrastructure and those with bespoke software | 2028, 2031, 2035 | Customers in those groups may plan on it, and need an update or replacement of custom or specialised software on the same timetable |
| United States, EO 14412 and IR 8547 [EO14412] [IR8547] | Federal agencies; covered contractors if the rule is made | 2030, 2031, 2035; TLS 1.3 for agencies by 2 January 2030 | If a federal agency uses it, directly or through a contractor |
| European Union, roadmap [EU] | Member States, as recommendations | end 2026, end 2030, (2035) | A country's own plan decides. "High-risk use cases" are not defined on the pages read |
| European Union, DORA [DORA] | Financial entities | In force; no post-quantum date | If a customer is one |
| Payment cards, PCI DSS [PCI] | Those that hold cardholder data | In force; no post-quantum date | If it stores, processes or transmits cardholder data |
| Sector law, for example health | Those it covers | None found | A 2026 review says HIPAA does not prescribe post-quantum algorithms and that covered entities should document quantum risk where records live long [FRONT] (read in part, through a summary) |

**Other countries and sectors have their own rules.** Ask the customer and the contract.

## Standards

| Need | Standard | Notes |
|---|---|---|
| Key exchange | ML-KEM, FIPS 203 (13 August 2024); ML-KEM-768 as default [NCSC24] [IR8547] | Public key 1,184 bytes, ciphertext 1,088. HQC is selected by NIST as an alternative and not yet published |
| Key exchange in TLS 1.3 | Hybrid `X25519MLKEM768`, RFC 10024 (August 2026, Proposed Standard, the one marked Recommended) [RFC10024] | TLS 1.2 has none |
| Key exchange in SSH | `mlkem768x25519-sha256`, the default since OpenSSH 10.0 (April 2025) [OPENSSH] | OpenSSH 10.1 and later warn when a connection is not post-quantum |
| Encrypting to a public key | HPKE with a hybrid KEM, `MLKEM768-X25519` (also known as X-Wing) | Use it in place of composing a KEM, a KDF and a cipher by hand. Not the same construction as the TLS group with the similar name |
| Signatures | ML-DSA, FIPS 204; ML-DSA-65 as default [NCSC24] | Public key 1,952 bytes, signature 3,309, against 32 and 64 for Ed25519 [RFC9964] |
| Signatures in tokens | RFC 9964 (May 2026, Proposed Standard): `ML-DSA-44`, `-65`, `-87`, key type `AKP` [RFC9964] | PASETO has no post-quantum version [PASETO] |
| Signatures in web certificates | ML-DSA in X.509 | Browsers do not verify it yet; Chrome plans Merkle Tree Certificates instead [CF] |
| Firmware and software signing | SLH-DSA (FIPS 205); LMS, XMSS (SP 800-208) | State must be managed for LMS and XMSS |
| Symmetric, hash, MAC | AES with 128 bits or more, SHA-256 and up, HMAC | No change needed |

Do not use the names or formats from before the standards: Kyber, Dilithium, SPHINCS+ and `X25519Kyber768Draft00` do not interoperate with the final standards.

## Inventory

Fill in one row for each use. "At risk" is risk from a quantum computer only. Record where each use is in the code, the library, its version, the algorithm, and who owns the other end. CycloneDX CBOM is a machine-readable format for this [CBOM].

| Use | Usual algorithm | At risk | Harm if broken |
|---|---|---|---|
| Browser or client to service | TLS: X25519 or ECDHE | **Key exchange, now** | Recorded traffic is read later |
| Service to service, queue, cache, webhook | TLS, mutual TLS | **Key exchange, now** | The same |
| Service to database | TLS of the driver and of the server | **Key exchange, now** | All data in transit |
| SSH, VPN, remote administration | Curve25519, ECDH, RSA | **Key exchange, now** | The same, and any secret typed or copied |
| End-to-end or file encryption, email | RSA, ECDH, age, PGP | **Now** | Stored or intercepted content is read |
| Data at rest, backups | AES | No, unless keys are wrapped with RSA or elliptic curves: then **now** | All stored data |
| Key management service, hardware module | RSA or elliptic curve wrapping | **Now**, if it wraps long-lived keys | The same |
| Sign-in tokens (JWT, PASETO, SAML) | RSA, ECDSA, Ed25519 | Later | Forged sign-in, once a machine exists. Not retrospective |
| Certificates, client certificates, internal authority | RSA, ECDSA | Later; plan long-lived roots early | A false peer |
| Code, package, container, commit and release signing | RSA, ECDSA, Ed25519 | Later; plan long-lived roots early | A forged build |
| Long-lived signatures on documents and records | RSA, ECDSA, Ed25519 | Later; evidence may need renewal before then | A forged or deniable document |
| Firmware and secure boot keys fixed in devices | RSA, ECDSA | Plan now, if the key cannot be replaced in the field | A forged update |
| Passwords | Argon2id, bcrypt, scrypt | No | |
| Random tokens, API keys stored as a hash | SHA-256 | No | |
| Session cookies, audit chains, request signing | AES-GCM, HMAC-SHA-256 | No | |
| Package registries, container images | Theirs | Theirs | |

Three examples of what an inventory finds that a version check does not:

- PostgreSQL 18 sets `ssl_groups` to `X25519:prime256v1` by default, so **the server does not offer the hybrid until it is set**, whatever OpenSSL it uses [PG]. With OpenSSL 3.5 or later, set `ssl_groups = 'X25519MLKEM768:X25519:prime256v1'` and test each client.
- A proxy or content delivery network may offer the hybrid to browsers and not to the origin behind it. Each hop is a separate connection.
- Two libraries in one program may use two different TLS implementations.

## What lives longest

A common rule of thumb (Mosca): if the years that data must stay secret, and the years a migration takes, together exceed the years until a machine exists, it is already late. The last number is not known, and this page does not guess it.

| Data | Must stay secret for | Note |
|---|---|---|
| Health, genetics, identity, biometrics | A lifetime; genetics longer, for relatives | |
| Legal, government, trade secrets, source code | Years to decades | |
| Financial and personal records | As law and contract say, usually years | |
| Root keys, backup keys, key-wrapping keys | As long as the data they protect | Often forgotten |
| Tokens and sessions | Minutes to hours | Forging matters; reading an old one does not |
| Signed releases and firmware | While anyone runs them | Integrity, not secrecy |

## Rules

1. **No cryptography of our own**, as the UK guidance says [NCSC25]. Only a maintained library, in a protocol that has a standard. Calling a raw ML-KEM function and inventing the protocol around it is cryptography of our own.
2. **Key exchange before signatures.** A recorded connection is lost for good; a signature fails only from the day a machine exists.
3. **Hybrid, and then ask again.** `X25519MLKEM768` in TLS 1.3, preferred and not only offered. Not pure ML-KEM, unless a rule requires it: a fault found in a young algorithm or its code would then be the whole protection.
4. **Take the platform's default list of groups.** Do not pin curves or groups in code. Where a list is required, put the hybrid first, keep a classical group for old peers, and review it yearly.
5. **Prefer native, reviewed code to a port.** In order: the language's standard library, a binding to OpenSSL 3.5 or later, AWS-LC or BoringSSL, then a pure-language package. A package that says it is unaudited or experimental is for tests.
6. **Every signed, keyed or encrypted value names its algorithm and key**, and an unknown one is refused or said to be unverifiable, never passed. The verifier chooses which algorithms it allows; the value does not.
7. **Cryptography sits behind one small interface**, so that an algorithm changes in one place and by configuration.
8. **Sizes are not assumed.** No column, header, cookie, packet or buffer is sized for a 64-byte signature or a 32-byte key.
9. **A fallback is seen.** Record the negotiated group as a metric. An offer that a middlebox removes quietly is the likeliest failure, and the UK guidance says to check what is really used [NCSC25].
10. **Availability counts.** Nothing refuses a peer because it is old before that peer's owner has agreed a date. Any refusal is a setting, off by default.
11. **An inventory that cannot go stale.** A test that fails when a cryptographic dependency is added that the inventory does not name.
12. **No claim.** Until it is measured and reviewed, no document says "quantum-safe", "post-quantum compliant" or "quantum-ready".

## Order of work

1. **Inventory** the uses above, with an owner for each.
2. **Upgrade** runtimes, TLS libraries, base images, proxies and database drivers to releases that prefer the hybrid.
3. **Remove** pinned curve lists, TLS 1.2-only settings and old provider choices. TLS 1.3 is the minimum where every peer supports it.
4. **Measure** the negotiated group on every hop, in tests and in production.
5. **Re-wrap** long-lived keys and backups that RSA or elliptic curves protect.
6. **Make signatures replaceable**: algorithm names, key identifiers, rotation, sizes, and verification of two algorithms at once.
7. **Move signatures** when the standard, the library and every verifier are ready. Start with long-lived roots and firmware, and end with short-lived tokens.
8. **Review** yearly, and when a standard, a runtime or a contract changes.

To measure any TLS server, in any language, with OpenSSL 3.5 or later:

```sh
openssl s_client -connect example.com:443 -tls1_3 -groups X25519MLKEM768 </dev/null 2>&1 | grep -i group
```

It prints `Negotiated TLS1.3 group: X25519MLKEM768`, or the handshake fails. This tests the server only; test the project's own client with the checks below.

## Programming language specifics

Versions were read from each package registry on 2026-11-25. They are the state on that day, and not a minimum, except where a minimum is stated. A minimum says when a feature arrived: use the latest supported patch release.

| Language | Hybrid TLS by default | ML-KEM | ML-DSA | Assessment |
|---|---|---|---|---|
| Go | 1.24 and later | Standard library, 1.24 | Standard library, 1.27 | Strongest: complete, and no dependency |
| Java | 27; Oracle 25.0.5 | Standard library, 24 | Standard library, 24 | Strong: upgrade, no code |
| JavaScript (Node.js) | 24.5, 22.20 | Built in, 24.7 | Built in, 24.7 | Strong on the server; browser code is early |
| TypeScript | As JavaScript | As JavaScript | As JavaScript | As JavaScript; types help with agility |
| Rust | `rustls` 0.23.27, `aws-lc-rs` provider only | `aws-lc-rs` | `aws-lc-rs` 1.19 | Strong, if no dependency selects `ring` |
| C# | Set by the operating system | .NET 10 | .NET 10 | Good on Linux; uneven on Windows; absent on macOS |
| Python | Set by the linked OpenSSL, 3.5 or later | `cryptography` 48 | `cryptography` 48 | Good; check the interpreter's OpenSSL |

### Rust

**Assessment.** Strong for TLS. `rustls` with the `aws-lc-rs` provider prefers `X25519MLKEM768` by default: `aws_lc_rs` and `prefer-post-quantum` are both default features [RUSTLS]. The `ring` provider has no post-quantum algorithm, and **a dependency that selects `ring` removes the hybrid without an error.** ML-DSA is stable in `aws-lc-rs`, and in `rustls` it is still experimental [RUSTLS-PQ].

| Need | Use | Notes |
|---|---|---|
| TLS | `rustls` 0.23 with `aws-lc-rs` | Keep default features. With `default-features = false`, add `aws_lc_rs` and `prefer-post-quantum` |
| HTTP | `reqwest`, default `rustls` feature | 0.13.5 selects `aws-lc-rs` by default; check an older version. `native-tls` uses the operating system's library |
| QUIC | `quinn` with `rustls-aws-lc-rs`, without `rustls-ring` | 0.11.12 selects `ring` by default |
| PostgreSQL | `sqlx` with `tls-rustls-aws-lc-rs` | 0.9.1: plain `tls-rustls` means `ring` [SQLX] |
| ML-KEM, ML-DSA | `aws-lc-rs` 1.19: `kem::ML_KEM_768`, `signature::ML_DSA_65` | Native code. ML-DSA has been moved out of the `unstable` module [AWSLCRS] |
| FIPS 140 | `rustls` feature `fips`, `aws-lc-rs` feature `fips` | Check the certificate of the exact module; submitted is not validated |
| OpenSSL already in use | `openssl` crate, linked to OpenSSL 3.5 or later | Control which OpenSSL is linked |
| Tokens | Wait, or sign with `aws-lc-rs` behind an interface | `jsonwebtoken` 12 has no ML-DSA |

**Avoid.** In production: `ml-kem` and `ml-dsa` (RustCrypto), whose pages say they have never been independently audited [RUSTCRYPTO], and `oqs`, whose library liboqs says not to rely on it in production [LIBOQS]. `libcrux-ml-kem` and `libcrux-ml-dsa` are formally verified in part and describe themselves as pre-release; ask the maintainers first [LIBCRUX]. `rustls-post-quantum` is no longer needed for key exchange.

**Check.** Fail the build if `ring` returns, and read the group after a handshake:

```sh
cargo tree -i ring -e features    # says which dependency selects ring
```

```rust
// After the handshake; `conn` is a rustls ClientConnection or ServerConnection.
let group = conn.negotiated_key_exchange_group().map(|g| g.name());
```

Ban `ring` with a `cargo deny` rule, or allow it only where it does no key exchange. Where both providers are linked, install one explicitly at start-up with `CryptoProvider::install_default`.

### Python

**Assessment.** Good, in two halves that are separate. **TLS uses the OpenSSL that the interpreter links**, and needs 3.5 or later; the version of Python does not say which. A Python 3.11 tested for this page linked OpenSSL 3.0.15, which has no hybrid. **The `cryptography` package carries its own OpenSSL**, so its ML-KEM works while TLS in the same process does not have it.

| Need | Use | Notes |
|---|---|---|
| TLS | Standard `ssl`, on an interpreter linked to OpenSSL 3.5 or later | Used by `urllib3`, `requests`, `httpx`, `aiohttp`. Python 3.15 adds `SSLContext.set_groups()`, `get_groups()` and `SSLSocket.group()` [PYSSL] |
| Encrypt to a public key | `cryptography` HPKE, `KEM.MLKEM768_X25519` | Version 48 or later. The hybrid, as a complete protocol [PYCA] |
| ML-KEM, ML-DSA | `cryptography` 48 or later (50.0.2 read) | In `hazmat`. 47 had them only on AWS-LC and BoringSSL, not in the wheels [PYCA] |
| ML-DSA certificates | `cryptography` `x509`, 49 or later | Read the changelog for the release you use |
| SLH-DSA | `pycryptodome` 4.6, with care | New in 4.6.0 (22 November 2026), and part of it is written in Python. `cryptography` lists none |
| Tokens | Wait | `PyJWT` 2.15.1 has no ML-DSA; none was found in `joserfc` |

**Avoid.** In production: `kyber-py` and `dilithium-py` (pure Python, for learning), `liboqs-python` (see liboqs [LIBOQS]), and `pqcrypto` or `quantcrypt` without a review. Do not install `oqs` from PyPI: it is **an unrelated package**. Do not call `set_ecdh_curve()`: it pins one group.

**Check.**

```python
import ssl
assert ssl.OPENSSL_VERSION_INFO >= (3, 5), ssl.OPENSSL_VERSION
# Python 3.15 and later, after the handshake:
#   assert sock.group() == "X25519MLKEM768"
```

Run the assertion inside the production image and on each developer's interpreter. An official image, a system Python, a Conda Python and a `uv` Python may each link a different OpenSSL.

### Java

**Assessment.** Strong, by upgrade. JDK 27 prefers `X25519MLKEM768` in TLS 1.3 without code changes (JEP 527) [JEP527], and **Oracle's JDK 25.0.5 (October 2026) has it too** [JDK25]. OpenJDK 25.0.0 was tested for this page and does not have it: "Java 25" is not enough. Oracle's release notes for JDK 21 list ML-KEM and ML-DSA and no hybrid TLS [JDK21]. ML-KEM (JEP 496) and ML-DSA (JEP 497) are in the standard library since JDK 24, and ML-DSA is not integrated in the JDK's TLS [CF].

| Need | Use | Notes |
|---|---|---|
| TLS | JDK 27, or a JDK 25 update that has JEP 527, with `javax.net.ssl` | Check your vendor: only Oracle's notes were read for 25 |
| TLS on an older JDK | Conscrypt 2.10.0 | Prefers the hybrid since 2.7.0. `Conscrypt.getGroupName()` since 2.9.0. 2.10.0 fixes CVE-2026-132784 [CONSCRYPT] |
| TLS, alternative | Bouncy Castle `bctls-jdk18on` 1.87 | Hybrid support since 1.81; whether it is on by default was not established |
| ML-KEM | `javax.crypto.KEM` with `ML-KEM-768` | JDK 24 and later |
| ML-DSA | `java.security.Signature` with `ML-DSA-65` | JDK 24 and later |
| SLH-DSA, composite signatures | Bouncy Castle, or Conscrypt | |
| High-level API | Tink 1.24 | ML-DSA and SLH-DSA; it needs Conscrypt as provider [TINK] |
| FIPS 140 | `bc-fips`, or your vendor's module | Check the certificate and that it covers the algorithm |
| Tokens | Not established for `nimbus-jose-jwt` | Read its current notes before choosing |

**Avoid.** `jdk.tls.namedGroups` and `SSLParameters.setNamedGroups` with an old list: both replace the default, and so remove the hybrid. Application servers, and Netty with `tcnative`, may not use the JDK's TLS at all: check each.

**Check.** Run with `-Djavax.net.debug=ssl:handshake` and look for `"named group": x25519mlkem768` in the ServerHello. In a test:

```java
var socket = (SSLSocket) SSLContext.getDefault().getSocketFactory().createSocket();
var groups = List.of(socket.getSSLParameters().getNamedGroups());
assert groups.contains("X25519MLKEM768") : groups;   // run with -ea
```

**Key format.** JDK 27 and Oracle 25.0.5 write ML-KEM and ML-DSA private keys as a seed, and **older JDKs do not read them.** In a mixed fleet set `jdk.mlkem.pkcs8.encoding` and `jdk.mldsa.pkcs8.encoding` to `expandedKey` until every reader is upgraded [JDK25] [JDK27].

### JavaScript

**Assessment.** Strong on the server, early in the browser. Node.js 24.5 and 22.20 and later bundle OpenSSL 3.5, and prefer the hybrid in TLS [CF]. Node.js has ML-KEM and ML-DSA in `node:crypto` since 24.7. Browser **TLS** needs no code: Chrome 131, Firefox 132 and Safari 26 and later prefer the hybrid [CF]. Browser **WebCrypto** is not ready: encapsulation is in Chrome 155, a preview in Firefox, and not in Safari [BCD].

| Need | Use | Notes |
|---|---|---|
| TLS, HTTPS, HTTP/2, `fetch` | Node.js 24 or 26 LTS, defaults | Official builds. A build that links another OpenSSL is as old as that library |
| ML-KEM | `crypto.encapsulate()`, `crypto.decapsulate()` with `ml-kem-768` | 24.7 and later |
| ML-DSA | `crypto.sign(null, …)`, `crypto.verify(null, …)` with `ml-dsa-65` | 24.7 and later |
| Same, across runtimes | `crypto.subtle`, after `SubtleCrypto.supports()` | Stability 1.1, active development: it may change [NODE] |
| Hybrid encryption | `crypto.subtle` `MLKEM768-X25519` on recent Node.js, or `@hpke/hybridkem-x-wing` | Pin versions; both are young |
| Tokens | `jose` 6.2 | Lists ML-DSA for JOSE, RFC 9964. Feature-detect: it rests on the runtime |
| Pure JavaScript, last resort | `@noble/post-quantum` 0.7 | Self-audited, **not independently audited, and makes no constant-time claim** [NOBLE]. Verifying with public data is the lowest-risk use |

**Avoid.** `crypto-js`, `node-forge` and `jsrsasign` for new key exchange; `liboqs-node` (last release in 2021); pure-JavaScript signing or decapsulation on a server that an attacker can time; private keys in a web page when a server or the platform can hold them; `ecdhCurve` set to an old list, which removes the hybrid.

**Check.** Node.js 26 prints `X25519MLKEM768` for this, as tested:

```js
import tls from "node:tls";

const socket = tls.connect({ host, port: 443, servername: host, minVersion: "TLSv1.3" }, () => {
  console.log(process.versions.openssl, socket.getEphemeralKeyInfo().name);
  socket.end();
});
```

Deno, Bun, Electron and edge runtimes use other TLS libraries. Test each one that the project runs on.

### TypeScript

**Assessment.** The runtime, the libraries and the risks are those of [JavaScript](#javascript). What TypeScript adds is types, which are good at cryptographic agility and prove nothing about the runtime.

| Need | Use | Notes |
|---|---|---|
| Node.js types | `@types/node`, the same major version as the runtime | Version 26 types `encapsulate`, `encapsulateBits`, `ml-kem-*` and `ml-dsa-*` |
| Browser types | `lib.dom`, checked | TypeScript 6's `lib.dom` has no `encapsulateBits`. Declare a narrow interface rather than using `any` |
| Parsing at the boundary | A schema validator, for example `zod` or `valibot` | Algorithm names that arrive are `unknown`, whatever the type says |
| Tokens, HPKE, pure TypeScript | `jose`, `@hpke/*`, `@noble/post-quantum` | Written in TypeScript; the cautions are the same |

**Practice.** Make the algorithm a closed union, make unhandled ones a compile error, and refuse unknown ones at run time:

```ts
const SIGNATURE_ALGORITHMS = ["Ed25519", "ML-DSA-65"] as const;
type SignatureAlgorithm = (typeof SIGNATURE_ALGORITHMS)[number];

function parseAlgorithm(value: unknown): SignatureAlgorithm {
  const found = SIGNATURE_ALGORITHMS.find((name) => name === value);
  if (found === undefined) throw new Error("unknown signature algorithm");
  return found;
}

function signatureBytes(algorithm: SignatureAlgorithm): number {
  switch (algorithm) {
    case "Ed25519": return 64;
    case "ML-DSA-65": return 3309;
    default: return algorithm satisfies never;
  }
}
```

Brand byte arrays by role, such as `EncapsulationKey`, `Ciphertext` and `SharedSecret`, since all of them are otherwise `Uint8Array` and are easily swapped. Turn on `strict`.

**Avoid.** `as` casts on algorithm names or keys; ambient declarations for a method that the runtime may not have, without a feature check.

**Check.** The same run-time check as JavaScript. A type check is not evidence that a connection is hybrid.

### C#

**Assessment.** Good on Linux, uneven elsewhere. **.NET uses the operating system's cryptography**, so one program differs by platform. .NET 10 added `MLKem`, `MLDsa` and `SlhDsa`, and .NET Framework gets types through `Microsoft.Bcl.Cryptography` [NETX]. In .NET 11's source `MLKem` carries no experimental mark, `MLDsa` carries it on some members (pre-hash and external mu), and `SlhDsa` is still experimental (`SYSLIB5006`) [NETSRC].

| Platform | Primitives | Hybrid TLS (`SslStream`, `HttpClient`, Kestrel) |
|---|---|---|
| Linux | OpenSSL 3.5 or later | From OpenSSL 3.5's defaults; reasoned, not tested here |
| Windows | Documented for Windows 11 Insider builds; check current Windows | Schannel: documented for Insider builds and **disabled by default**; enable `x25519_mlkem768` by group policy or the TLS cmdlets [SCHANNEL] |
| macOS and iOS | None. .NET 11 on macOS was tested: `IsSupported` is false | Not established |

| Need | Use | Notes |
|---|---|---|
| TLS | `SslStream`, `HttpClient`, Kestrel, on a platform that has the hybrid | Or end TLS at a proxy that has it |
| ML-KEM | `System.Security.Cryptography.MLKem` | Program against the base class, and test `IsSupported` |
| ML-DSA | `System.Security.Cryptography.MLDsa` | The same |
| Hybrid encryption | `CompositeMLKem`, `Hpke` in .NET 11 | **Experimental** (`SYSLIB5006`, `SYSLIB5009`): not for data that must stay readable [NET11] |
| Tokens | `Microsoft.IdentityModel.*` 8.19 or later | 8.19.0 added ML-DSA; 8.23.0 resolves `AKP` keys from a key set [MSID] |
| Platforms without support | `BouncyCastle.Cryptography` 2.7, after a review | Its own page says its post-quantum algorithms are **experimental** [BCCS] |

**Avoid.** Suppressing `SYSLIB5006` or `SYSLIB5009` for a whole project: suppress at one call, with a reason. `CipherSuitesPolicy` as a way to choose key exchange: it sets cipher suites, which are not groups. Base images with OpenSSL before 3.5.

**Check.** At start-up, and on every platform in continuous integration:

```csharp
using System.Security.Cryptography;

if (!MLKem.IsSupported || !MLDsa.IsSupported)
    throw new PlatformNotSupportedException("ML-KEM or ML-DSA is missing on this platform");
```

Use that only where the program calls the primitives. It says nothing about TLS: capture a handshake, or use `openssl s_client` against a .NET server.

### Go

**Assessment.** The strongest of the seven: standard library only, with a FIPS 140-3 module. `crypto/tls` prefers `X25519MLKEM768` by default since Go 1.24, and adds `SecP256r1MLKEM768` and `SecP384r1MLKEM1024` since 1.26. `crypto/mlkem` came in 1.24, `crypto/hpke` in 1.26, and `crypto/mldsa`, with support in `crypto/x509` and TLS 1.3, in 1.27 [GO124] [GO126] [GO127].

| Need | Use | Notes |
|---|---|---|
| TLS, HTTP, gRPC | `crypto/tls`, with `CurvePreferences` left `nil` | Go 1.24 or later |
| Encrypt to a public key | `crypto/hpke` with `MLKEM768-X25519` | Go 1.26. The hybrid, as a complete protocol |
| ML-KEM | `crypto/mlkem` | 768 and 1024. Prefer `crypto/hpke` |
| ML-DSA | `crypto/mldsa` | Go 1.27 |
| ML-DSA before Go 1.27 | `filippo.io/mldsa` | Drop-in, and a wrapper of the standard package on 1.27 |
| SSH | `golang.org/x/crypto/ssh`, current | |
| Tokens | `github.com/lestrrat-go/jwx/v4` | Documents ML-DSA [JWX]. `golang-jwt/jwt/v5` 5.3.1 does not have it; a change is open |
| SLH-DSA and others | `github.com/cloudflare/circl` | Its page advises caution in production, as part is experimental [CIRCL] |
| FIPS 140 | `GOFIPS140` at build | Check which module version has a certificate, and whether it has the algorithm [GOFIPS] |

**Avoid.** A `CurvePreferences` list written before 1.24: it removes the hybrid. `GODEBUG=tlsmlkem=0` left over from a workaround. **An old `go` line in `go.mod`**: compatibility settings follow that line and not only the toolchain, so a module that says `go 1.23`, built with Go 1.27.4, negotiated plain `X25519` when tested. `filippo.io/mlkem768` for new code, as the standard package replaces it. `liboqs-go` in production [LIBOQS].

**Check.** Go 1.27.4 prints `X25519MLKEM768` for this, as tested:

```go
conn, err := tls.Dial("tcp", "example.com:443", &tls.Config{MinVersion: tls.VersionTLS13})
if err != nil {
	log.Fatal(err)
}
defer conn.Close()
fmt.Println(conn.ConnectionState().CurveID)
```

In a server, record `r.TLS.CurveID` as a metric. Search for settings that remove it:

```sh
grep -rnE 'CurvePreferences|tlsmlkem|tlssecpmlkem' --include='*.go' --include=go.mod --include=go.work .
```

## Operational risk

- **A connection fails after the change**, and a service stops. Cause: a larger first message, an old TLS library or middlebox, a certificate too big. Roll out in stages, with a way back.
- **A token grows past a limit.** An ML-DSA-65 signature is 4,412 characters in base64url, against 86 for Ed25519. A proxy with an 8 KiB header limit may refuse it and lock every user out. A browser cookie is usually limited to about 4 KiB, so such a token does not fit in one.
- **A field is too short.** A database column, message or barcode was sized for a small signature.
- **Keys that one version writes, another cannot read**, as with Java above.
- **Recorded traffic is read years later**, and a person is harmed by what it says.
- **False reassurance**: the hybrid is thought to be in use and is not.

In a safety-critical or regulated project, put each in the hazard log or risk register.

## What a deploying organisation has to do

A project's code usually cannot do these: certificates and their authority; the proxy; disks, backups and how their keys are wrapped; the identity provider; the network devices between; the other end of each integration; other suppliers; the risk register, the privacy assessment and the date.

## Open questions

Answer these for each project.

1. Which country's and which sector's dates apply, and does a contract say so?
2. How long must each kind of data stay secret?
3. Is FIPS 140 validation required, and does the validated module have ML-KEM and ML-DSA?
4. Is the hybrid required or only preferred, and from when may a connection without it be refused? Recommendation: preferred, and refused only where a deployment says so.
5. Do tokens move to ML-DSA, and when? Recommendation: wait until every verifier supports it; nothing is lost by waiting for short-lived signatures.
6. Who reviews this: is there a cryptographer, and who is the security officer?

## Not done, not verified

- The code samples for Go and JavaScript, and the OpenSSL command, were run against a public server on macOS. The `cryptography` ML-KEM calls, Java's ML-KEM and its default groups, and C#'s `IsSupported` were run locally. The TypeScript sample was compiled with TypeScript 6 in strict mode and run. **The Rust sample was not run**, as its build did not finish; its names were read from the crates' documentation. Python 3.15's methods and the Java assertion were not run. Nothing was run on Windows or Linux.
- The EU roadmap's own PDF, the PCI standard, OMB M-26-15, the FIPS texts, and the DORA text in the Official Journal were not read. The review [FRONT] was read in part. Policy rows were carried over from an earlier version of this page and were not read again.
- No library was audited for this page. "Use" means maintained, documented and the least risky choice found.
- OpenJDK builds other than Oracle's, Bouncy Castle's TLS default, Nimbus, and .NET's hybrid TLS on each platform were not established. Absence of a finding is not absence of support.
- Whether the US procurement rule has been proposed, and any other country's rule, was not established.
- These are the author's reasoning and have no source: key exchange first as a general order; hybrid and not pure ML-KEM; the order of preference among libraries; that old equipment may fail on a larger first message; the header limit and large tokens; and how long each kind of data must stay secret.
- **The library facts go stale in months.** Check each against its source before relying on it.

## Sources

Read 2026-11-25.

Policy and standards:

- [NCSC25] UK National Cyber Security Centre, Timelines for migration to post-quantum cryptography, v1.0, 20 March 2025 (file of August 2026): https://www.ncsc.gov.uk/guidance/pqc-migration-timelines
- [NCSC24] UK National Cyber Security Centre, Next steps in preparing for post-quantum cryptography, v2.0, 14 August 2024: https://www.ncsc.gov.uk/whitepaper/next-steps-preparing-for-post-quantum-cryptography
- [IR8547] NIST IR 8547, Transition to Post-Quantum Cryptography Standards, final, October 2026: https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf
- [EO14412] Executive Order 14412, 22 June 2026: https://www.govinfo.gov/content/pkg/DCPD-202600417/html/DCPD-202600417.htm
- [EU] European Commission, Post-quantum cryptography, and the coordinated implementation roadmap: https://digital-strategy.ec.europa.eu/en/policies/post-quantum-cryptography and https://digital-strategy.ec.europa.eu/en/library/coordinated-implementation-roadmap-transition-post-quantum-cryptography
- [DORA] Delegated Regulation (EU) 2024/1774, Article 6 and recital 9, in an unofficial copy: https://www.springlex.eu/en/packages/dora/rts-rmf-regulation/article-6/
- [PCI] Two secondary accounts of PCI DSS 12.3.3: https://postquantum.com/security-pqc/pci-dss-v4-quantum-cryptographic-requirements/ and https://www.halock.com/are-you-keeping-an-inventory-of-cipher-suites-and-certificates-for-the-new-pci-dss
- [FRONT] Wu and others, a review in Frontiers in Health Services, 16 July 2026: https://www.frontiersin.org/journals/health-services/articles/10.3389/frhs.2026.1901282/full
- [RFC10024] https://www.rfc-editor.org/info/rfc10024
- [RFC9964] https://www.rfc-editor.org/info/rfc9964
- [PASETO] https://github.com/paseto-standard/paseto-spec/tree/master/docs/01-Protocol-Versions
- [CBOM] https://cyclonedx.org/capabilities/cbom/

Platforms:

- [CF] Cloudflare, post-quantum support, as read; some of its library rows are behind the libraries' own pages: https://developers.cloudflare.com/ssl/post-quantum-cryptography/pqc-support/
- [LIBOQS] https://github.com/open-quantum-safe/liboqs
- [OPENSSH] https://www.openssh.com/pq.html
- [PG] https://www.postgresql.org/docs/18/runtime-config-connection.html

Rust:

- [RUSTLS] The `rustls` manual, defaults, and the crate's features, 0.23.46: https://docs.rs/rustls/latest/rustls/manual/_05_defaults/index.html
- [RUSTLS-PQ] https://docs.rs/rustls-post-quantum/latest/rustls_post_quantum/
- [AWSLCRS] https://docs.rs/aws-lc-rs/latest/aws_lc_rs/
- [RUSTCRYPTO] https://docs.rs/ml-kem/latest/ml_kem/ and https://docs.rs/ml-dsa/latest/ml_dsa/
- [LIBCRUX] https://github.com/cryspen/libcrux
- [SQLX] https://docs.rs/crate/sqlx/latest/features

Python:

- [PYCA] https://cryptography.io/en/latest/changelog/ and https://cryptography.io/en/latest/hazmat/primitives/hpke/
- [PYSSL] https://docs.python.org/3/whatsnew/3.15.html

Java:

- [JEP527] https://openjdk.org/jeps/527
- [JDK21] https://www.oracle.com/java/technologies/javase/21all-relnotes.html
- [JDK25] https://www.oracle.com/java/technologies/javase/25all-relnotes.html
- [JDK27] https://www.oracle.com/java/technologies/javase/27-relnote-issues.html
- [CONSCRYPT] https://github.com/google/conscrypt/releases
- [TINK] https://developers.google.com/tink/supported-key-types

JavaScript and TypeScript:

- [NODE] https://nodejs.org/api/webcrypto.html, https://nodejs.org/api/crypto.html, https://nodejs.org/api/tls.html and https://nodejs.org/en/blog/release/v24.7.0
- [BCD] MDN browser compatibility data, `SubtleCrypto`: https://github.com/mdn/browser-compat-data/blob/main/api/SubtleCrypto.json
- [NOBLE] https://github.com/paulmillr/noble-post-quantum

C#:

- [NETX] https://learn.microsoft.com/en-us/dotnet/standard/security/cross-platform-cryptography#post-quantum-cryptography
- [NET11] https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-11/libraries#cryptography
- [NETSRC] https://github.com/dotnet/runtime/tree/release/11.0/src/libraries/Common/src/System/Security/Cryptography
- [SCHANNEL] https://learn.microsoft.com/en-us/windows/win32/secauthn/tls-supported-groups-in-windows-11-24h2-and-later
- [MSID] https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/dev/CHANGELOG.md
- [BCCS] https://github.com/bcgit/bc-csharp

Go:

- [GO124] https://go.dev/doc/go1.24
- [GO126] https://go.dev/doc/go1.26
- [GO127] https://go.dev/doc/go1.27
- [GOFIPS] https://go.dev/doc/security/fips140
- [JWX] https://github.com/lestrrat-go/jwx/tree/v4.6.0
- [CIRCL] https://github.com/cloudflare/circl
