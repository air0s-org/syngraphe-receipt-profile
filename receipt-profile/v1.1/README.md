# AI Interaction Receipt — profile v1.1

Status: profile v1.1 specification. This document is normative.

Profile v1.1 is an additive receiver-policy revision beside profile v1. It does not change any profile v1 bytes. A profile v1 receipt remains structurally valid when evaluated with the v1.1 receipt schema.

## Schema delta

Profile v1.1 adds `issuer-operated` to `receipt.trust_statement.notary_trust_model`. The schemas remain closed. The profile identifier accepted by the v1.1 bundle schema is either `syngraphe.ai-interaction-receipt.profile-v1` or `syngraphe.ai-interaction-receipt.profile-v1.1`.

The corresponding wire schemas are:

- `https://syngraphe.org/schemas/ai-interaction-receipt/v1.1`
- `https://syngraphe.org/schemas/countersign-standard-envelope/v1.1`

## Receiver-trusted issuer operator

The receiver policy MAY contain `issuer_operator` beside its trusted keys and status-list snapshots. This value is a stable opaque identifier matching the receipt profile's bounded identifier syntax. The receiver MUST NOT derive, copy, or otherwise read this policy value from the receipt, envelope, bundle, CAWG issuer DID, URL, or another artifact under test.

Comparison with every `receipt.trust_statement.notaries[].operator` value is byte-for-byte exact. Receivers MUST NOT normalize case, parse a DID or URL, or apply aliases.

For a receipt that claims `zktls-bound` evidence, the receiver MUST cap the verified result to `gateway-signed`, set `product_receipt` to `false`, and return `IRV1-C001-ISSUER-OPERATED-NOTARY` when either:

1. `receipt.trust_statement.notary_trust_model` is `issuer-operated`; or
2. any `receipt.trust_statement.notaries[].operator` exactly equals the receiver policy's `issuer_operator`.

The capped result remains `valid` at the capped class. It is not an invalid receipt.

If a notarised receipt claims `zktls-bound`, no declared `issuer-operated` trigger is present, and the receiver policy lacks `issuer_operator`, the receiver MUST return `indeterminate` with `IRV1-E065-ISSUER-OPERATOR-POLICY-UNAVAILABLE`. It MUST NOT silently preserve `zktls-bound`.

## Conformance vectors

`vectors.json` fixes the five required receiver decisions: exact-match cap, distinct-operator no-cap, declared `issuer-operated` cap, missing policy value as indeterminate, and unchanged v1 receipt acceptance under v1.1.
