# Symbiont Wallet: Post-Quantum QOGE — Development Update

Qogecoin has always set out to become a quantum-resistant version of Dogecoin.
This post is the first community update on concrete progress toward that
goal: **Symbiont Wallet**, a working implementation of post-quantum
signatures for QOGE, built under the SAOGEN initiative.

## What's been built

Symbiont Wallet implements **SLH-DSA-SHA2-128f**, the stateless
hash-based signature scheme standardized by NIST as **FIPS 205** — one of
the small set of signature algorithms believed to remain secure even
against an attacker with a large-scale quantum computer.

This isn't a paper design. It's a working Go application, with **47/47
tests passing**, including:

- Real SLH-DSA keypair generation, signing, and verification (32-byte
  public keys, 17,088-byte signatures — exactly matching the FIPS 205
  spec)
- Address derivation: `HASH256(public key)` encoded as a Bech32m address,
  so the public key itself stays hidden until the moment of spending
- A single-use address model: every address is used exactly once, then
  its private key is permanently destroyed — closing the window for
  "harvest now, decrypt later" attacks where someone records today's
  blockchain data hoping to break it with tomorrow's quantum computer
- A CLI you can run yourself, end to end: generate a wallet, get an
  address, sign a message with a real post-quantum signature, and watch
  the key get zeroed on confirmation

## What "P2QPK" addresses are

The addresses Symbiont Wallet produces look like `bq1z...` — a new format
we're calling **P2QPK** ("Pay to Quantum Public Key"). Under the hood,
these use SegWit witness version 2, a part of the address space Bitcoin
(and Qogecoin) already reserve for future upgrades.

**Important — these addresses are not yet protected.** Until a future
soft fork activates the new verification rules, witness-version-2 outputs
are spendable by *anyone* under current consensus rules — that's
deliberate, and how Bitcoin's own SegWit and Taproot upgrades bootstrapped
safely too. In practice: the address *format* is finished and tested, but
**do not send real QOGE to a `bq1z...` address yet**. We'll announce
loudly when that changes.

## What's next

The wallet-side work (SIP-QOGE-PQC-01, and Phase A of SIP-QOGE-PQC-02) is
done. What's ahead is the consensus side — teaching Qogecoin Core itself
to recognize and verify these signatures. That's a bigger, slower-moving
piece of work by nature: it touches the rules every node on the network
agrees on, so it gets the scrutiny that deserves. The roadmap (liboqs
integration, a dedicated sighash specification with independent
cryptographic review, regtest testing, then a public testnet) is laid out
in SIP-QOGE-PQC-02 in the repo.

## Follow along

Everything is open and developing in public:

**👉 [github.com/QOGE/symbiont-wallet](https://github.com/QOGE/symbiont-wallet)**

The README tracks milestone status for both SIPs in real time, the test
results are there to run yourself, and the full SIP documents (design
rationale, threat model, and what we explicitly rejected and why) are in
the repo for anyone who wants the details.

This is early, experimental work — that's the point of building it in the
open. Questions, issues, and scrutiny are welcome.
