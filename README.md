# comos-anchors

Public, append-only anchors for the **ComOS Federation key-binding attestation chain**.

## What this is

The ComOS Federation retains the cryptographic proof of every manager-root key binding (nonce + signature + public key) in a hash-chained, append-only attestation log, disclosed at:

```
https://mcp.comos-gateway.com/.well-known/key-attestation-chain
```

Each entry's `entry_hash` is the hex SHA-256 over `JSON.stringify([seq, manager_id, event, kid, alg, public_key, nonce, signature, at, prev_hash])`; the genesis `prev_hash` is 64 zeros. Anyone can walk the chain, recompute the hashes, and re-verify each retained signature against the retained key over the retained nonce.

A hash chain inside one database detects tampering only if you can trust some copy of the head. **This repo is that copy.** An hourly workflow here fetches the disclosed head and, whenever it has changed, appends one line to [`anchors/key-attestation-chain.jsonl`](anchors/key-attestation-chain.jsonl) and commits it. The commit history is authored by GitHub Actions on GitHub's infrastructure — a privilege domain the federation runtime cannot reach. Rewriting the federation's chain after the fact would also require rewriting this public git history and every clone of it.

## How to verify (no ComOS credentials needed)

1. Fetch the live chain: `curl https://mcp.comos-gateway.com/.well-known/key-attestation-chain`
2. Recompute every `entry_hash` with the formula above and check each `prev_hash` linkage and signature.
3. Compare the live head against the last line of `anchors/key-attestation-chain.jsonl` (or any historical line against the entry at that `seq`). A live chain whose history contradicts an anchored line is a detected rewrite.

## Provenance

Established by ComOS change order **CO 324 / 324-001** (tamper-evident manager-root attestation). The private twin of this anchor runs independently in a second repository — two anchors, two privilege domains. Anchor cadence: hourly, on change only; a quiet chain produces no commits.
