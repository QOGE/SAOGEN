# Symbiont Wallet: Post-Quantum QOGE — Development Update

Qogecoin has always set out to become a quantum-resistant version of Dogecoin. This update summarizes the current progress of **Symbiont Wallet**, the post-quantum wallet and migration path being developed for QOGE under the SAOGEN initiative.

The work has now advanced beyond the first wallet-side implementation stage. The public `symbiont-wallet` repository currently tracks the PQC migration through **SIP-QOGE-PQC-01** and **SIP-QOGE-PQC-02**, including the transition from wallet-side address generation into the consensus-integration candidate phase.

Repository:

https://github.com/QOGE/symbiont-wallet

## What's been built

Symbiont Wallet implements **SLH-DSA-SHA2-128f**, the stateless hash-based signature scheme standardized by NIST as FIPS 205.

This is not only a paper design. It is a working Go implementation with real post-quantum key generation, signing, verification, address derivation, and single-use address lifecycle logic.

The current implementation includes:

* Real SLH-DSA-SHA2-128f keypair generation, signing, and verification
* 32-byte public keys and 17,088-byte signatures
* P2QPK address derivation using `HASH256(public key)`
* Bech32m-encoded QOGE addresses using witness version 2
* Single-use address lifecycle enforcement
* Key zeroing after confirmation
* Taproot rejection through structural witness-version checks
* Change routing to fresh addresses
* Address pre-generation pool support
* 47/47 tests passing across address, signer, keystore, and wallet packages

The core wallet-side architecture is implemented and validated. The project has now moved into the more serious consensus-integration path.

## What P2QPK addresses are

The addresses produced by Symbiont Wallet look like `bq1z...`.

This new format is called **P2QPK**, meaning **Pay to Quantum Public Key**.

Under the hood, P2QPK uses:

* QOGE address HRP: `bq`
* Witness version: `2`
* Address encoding: Bech32m
* Commitment: `HASH256(SLH-DSA public key)`

The public key is not stored openly in the address. It remains hidden behind a hash commitment until the moment of spending, reducing exposure to “harvest now, decrypt later” attacks.

Important warning: these addresses are **not yet safe for real funds** on the current unmodified Qogecoin network.

Until the SIP-QOGE-PQC-02 soft fork activates, witness-version-2 outputs remain anyone-can-spend under current consensus rules. They only become SLH-DSA-protected after Qogecoin Core is upgraded with the new P2QPK verification rules.

Do not send real QOGE to `bq1z...` P2QPK addresses before activation.

## Current migration status

The PQC migration has now reached **Phase D**.

### SIP-QOGE-PQC-01 — wallet-side core

The wallet-side core has been validated.

Completed work includes:

* SLH-DSA-SHA2-128f integration through liboqs-go
* HASH256-based address derivation
* Encrypted keystore persistence
* Address state machine and single-use invariants
* Key zeroing on confirmation
* Taproot rejection
* Fresh-address routing for change
* Address pre-generation pool support

### SIP-QOGE-PQC-02 — consensus integration candidate

The consensus-integration path has advanced through the first major design and validation stages.

Current phase status:

| Phase | Description                                                     | Status                                           |
| ----- | --------------------------------------------------------------- | ------------------------------------------------ |
| A     | Symbiont Wallet address format: witness version 2, Bech32m      | Complete                                         |
| B     | liboqs integration into the Qogecoin Core build path            | Complete                                         |
| C     | P2QPK sighash sub-specification and test vector                 | Complete                                         |
| D     | Consensus implementation in `VerifyWitnessProgram` P2QPK branch | Blocked pending independent cryptographic review |
| E     | Regtest functional testing                                      | Pending                                          |
| F     | Public testnet                                                  | Pending                                          |

Phase D is the actual consensus-level C++ implementation. This makes it naturally higher-stakes than the previous wallet, address-format, and test-vector phases.

However, Phase D is not about guessing the architecture or figuring out the mechanism from zero. The hard design decisions have already been made and verified: the address mechanism, the sighash construction, and the validated cryptographic flow are already defined.

Phase D is therefore about carefully translating a validated design into the harder Qogecoin Core consensus codebase, with maximum caution, review discipline, and respect for the higher security requirements of consensus code.

In simple terms: we are no longer figuring out *what* to build. We are now carefully implementing what has already been validated.

## Why Phase D is gated

Consensus code is different from wallet code.

Wallet code can be tested, replaced, upgraded, and iterated more easily. Consensus code defines the rules every node must agree on. A mistake at this layer can affect the entire network.

For that reason, Phase D is intentionally blocked until independent cryptographic review of the SIP-QOGE-PQC-02a sighash construction is complete.

This is not a cosmetic delay. It is a security gate.

The sighash construction defines exactly what the post-quantum signature commits to. Before that logic is translated into consensus C++, it needs independent review to reduce the risk of hidden signing, replay, malleability, or verification mistakes.

## Test status

The current public repository reports:

```text
go test ./address/...   → 13/13 PASS
go test ./signer/...    →  7/7 PASS
go test ./keystore/...  → 17/17 PASS
go test ./wallet/...    → 17/17 PASS

TOTAL: 47/47 PASS
```

The tests validate the core wallet behavior, address encoding, SLH-DSA signing flow, keystore lifecycle, and full single-use address model.

The address-package tests also validate Bech32 and Bech32m behavior against canonical checksum vectors, reducing the chance of accidental cross-version checksum confusion.

## Security properties

Symbiont Wallet is designed around several core security goals:

| Threat                                  | Mitigation                                                 | Status                    |
| --------------------------------------- | ---------------------------------------------------------- | ------------------------- |
| Public-key harvesting from stored UTXOs | HASH256 hides the public key at rest                       | Implemented and tested    |
| Address reuse                           | Single-use address state machine                           | Implemented and tested    |
| Taproot key-path exposure               | Witness version 1 structurally rejected                    | Implemented and tested    |
| Short mempool exposure window           | One-minute block time plus single-use spend model          | By design                 |
| Plaintext key storage                   | AES-256-GCM encrypted index                                | Implemented and tested    |
| Key persistence after spend             | Key zeroing on retirement                                  | Implemented and tested    |
| Tampered encrypted wallet DB            | GCM authentication                                         | Implemented and tested    |
| Change output reusing exposed key       | Change must route to fresh address                         | Implemented and tested    |
| Cross-version checksum confusion        | Bech32m witness-version binding                            | Implemented and tested    |
| Pre-activation fund loss                | Public warning: P2QPK is anyone-can-spend until activation | Not safe before soft fork |

## What remains open

There are still important open items before any real-value use.

The most important current items are:

1. **Independent cryptographic review**

   SIP-QOGE-PQC-02a must be independently reviewed before the Phase D consensus implementation proceeds.

2. **Consensus C++ implementation**

   After review, the P2QPK verification branch must be carefully implemented in Qogecoin Core consensus code.

3. **Regtest functional testing**

   Symbiont Wallet must interact with a modified Qogecoin node through RPC in controlled regtest conditions.

4. **Public testnet**

   After regtest validation, the next milestone is public testnet deployment.

5. **Production readiness**

   P2QPK addresses must not be treated as safe for real funds until the soft fork activates and the full implementation is reviewed, tested, and deployed.

## What this means for the community

This is a significant step forward for QOGE.

The project is no longer only at the early wallet experiment stage. The wallet-side core is working, the P2QPK address format is defined, liboqs integration into the Qogecoin Core build path has been completed for the development path, and the P2QPK sighash sub-specification has been completed with a test vector.

The next stage is more sensitive because it touches consensus C++.

That is why the project is moving carefully. The goal is not to rush a quantum-resistance headline. The goal is to build a migration path that can actually survive technical scrutiny.

## Follow along

Everything is open and developing in public:

https://github.com/QOGE/symbiont-wallet

The README tracks milestone status in real time, including SIP-QOGE-PQC-01, SIP-QOGE-PQC-02, and SIP-QOGE-PQC-02a.

Questions, issues, review, and technical scrutiny are welcome.

This remains experimental development work until the consensus path is reviewed, implemented, tested, and activated.
