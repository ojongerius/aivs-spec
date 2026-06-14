# AIVS — Agentic Integrity Verification Specification

**AIVS** is a portable, self-verifiable archive format for cryptographic proof of AI agent sessions. It provides tamper-evident audit logs that can be verified offline without contacting any server, blockchain, or authority.

## Overview

AIVS enables organizations to prove what their AI agents did — and for any party to independently verify that proof. It satisfies regulatory audit trail requirements (EU AI Act Article 19, ISO/IEC 42001:2023, NIST AI RMF) with a concrete, standardized format.

## Specifications

| Spec | IETF Draft | Description |
|------|-----------|-------------|
| [AIVS v1](./draft-stone-aivs-00.txt) | [draft-stone-aivs-00](https://datatracker.ietf.org/doc/draft-stone-aivs/)) | Hash-chained audit logs, Ed25519 signing, self-contained verification |

## Key Features

- **Self-verifiable**: Bundles verify offline using only Python 3 standard library
- **Portable**: Single `.tar.gz` file containing audit log, signature, manifest, and verifier
- **Tamper-evident**: SHA-256 hash chain detects any modification, insertion, deletion, or reordering
- **Cryptographic identity**: Ed25519 signatures prove who produced the bundle
- **AIVS-Micro**: Lightweight 200-byte attestation for continuous monitoring and API responses
- **Zero dependencies**: No external services, blockchains, or PKI required

## How It Works

Each action in an agent session is recorded as a JSON audit row with a deterministic SHA-256 hash. Each row's hash depends on the previous row's hash, forming an unbreakable chain. Modifying any action invalidates all subsequent hashes — making tampering immediately obvious.

```
Row 1: action1 → hash1
Row 2: action2 → hash(hash1 + action2)
Row 3: action3 → hash(hash2 + action3)
...
```

The chain hash (hash of all row hashes) is signed with Ed25519. The bundle includes the signed chain, all audit rows, manifest metadata, and a self-contained `verify.py` script.

## Bundle Structure

```
aivs_proof_SESSIONID_TIMESTAMP.tar.gz
└── session_proof/
    ├── audit_log.jsonl       # SHA-256 hash-chained action log
    ├── manifest.json         # Bundle metadata
    ├── session_sig.txt       # Ed25519 signature over chain hash
    ├── public_key.pem        # Signer's public key
    └── verify.py             # Self-contained verifier (stdlib only)
```

## Verification

Extract and run the verifier:

```bash
tar xzf aivs_proof_*.tar.gz
cd session_proof
python3 verify.py
```

Exit code 0 = verified. Exit code 1 = tampered or signature invalid.

## What AIVS Proves

✅ **Integrity**: Actions have not been modified  
✅ **Completeness**: No actions have been inserted or deleted  
✅ **Ordering**: Actions are in the recorded sequence  
✅ **Identity** (with signature): Bundle was produced by a specific Ed25519 keypair  

## What AIVS Does NOT Prove

❌ **Truthfulness**: AIVS doesn't prove actions actually occurred (agent could fabricate)  
❌ **Timeliness**: Timestamps are self-reported (layer RFC 3161 for trusted time)  
❌ **Key authenticity**: No PKI or certificate chain (public key is self-asserted)  

## Use Cases

- **Regulatory compliance**: Satisfy EU AI Act, ISO 42001, NIST AI RMF audit trail requirements
- **Agent accountability**: Prove to users/regulators what an agent did
- **Incident investigation**: Immutable record of agent actions for security review
- **Escrow/payment**: Attach proof bundle to transaction settlement
- **Supply chain**: Chain bundles together to form session sequences

## Related Specs in SwarmSync

| Protocol | Purpose |
|----------|---------|
| [VCAP](https://datatracker.ietf.org/doc/draft-stone-vcap-00/) | Verified escrow settlement |
| [AP2](https://datatracker.ietf.org/doc/draft-stone-ap2-00/) | Async payment protocol |
| [SwarScore](https://datatracker.ietf.org/doc/draft-stone-swarmscore-v1-00/) | Agent reputation scoring |
| [ATEP](https://datatracker.ietf.org/doc/draft-stone-atep-00/) | Portable trust credentials |

## License

Dual-licensed: [Apache 2.0](LICENSE-APACHE) / [MIT](LICENSE-MIT)

## Author

Ben Stone — [SwarmSync.AI](https://swarmsync.ai) — benstone@swarmsync.ai
