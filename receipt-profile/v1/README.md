## Plain-language guide (added 2026-09-24)

### What this folder is

This folder specifies the first interaction-receipt profile and the material used to test it. The profile records digests and a declared trust arrangement for one request and response, while keeping their contents out of the bundle.

### Files

- `field-inventory.md` — Compares the proposed fields with six sources, pins five OutFox paths by Git blob, and records first-party retention-page digests retrieved on 2026-09-04. Status: Reference only. Current state: informative research appendix on main; never deployed.
- `README.md` — Contains this guide plus the preserved normative profile, including the five top-level bundle members, eight required proof slots, eleven retention classes, and stable `AIR-V1` validation rules. Status: In progress. Current state: proposed C2-A profile on main; its own text says admission has not occurred; never deployed.
- `vectors.json` — Contains 12 accepted and 12 rejected Ed25519 test bundles spanning provider signatures, two-notary quorum modes, quote appraisals, auxiliary commitments, retention contradictions, and the exact expected `AIR-V1` failure rules. Status: Reference only. Current state: Node 24.18.0 conformance corpus with public test keys on main; never deployed.

### Folders

- `schemas/` — Contains the closed receipt and bundle structural schemas plus its guide; runtime-only checks are explicitly outside those JSON files. Status: Reference only. Current state: proposed schema material on main; never deployed.

### Where this fits

This folder is the proposed C2-A receipt contract and offline corpus that later admission and consumer integration would use.

# AI Interaction Receipt — profile v1

Status: profile v1 specification. This document is normative except where a section is marked informative.

## 1. Scope

This profile defines a digest-only verification bundle for one AI request and one AI response. It records the request digest, response digest, HTTPS endpoint without query data, provider-qualified model identifier, declared retention class in force, UTC timestamp, and an explicit trust statement. It does not carry request or response bodies.

The profile extends verification only. It does not define receipt issuance, provider behavior, training behavior, data deletion, or a network service. A `valid` status means the bundle passed this profile's structural, binding, primary-witness, caller-trust, and temporal checks. It does not mean that a provider followed a stated retention class, that an endpoint was reached, or that optional external proof systems accepted their committed artifacts.

Normative terms `MUST`, `MUST NOT`, `REQUIRED`, `SHOULD`, and `MAY` are interpreted as in RFC 2119 and RFC 8174.

## 2. Artifacts

- Proposed receipt schema: `docs/countersign/receipt-profile/v1/schemas/receipt.schema.json`
- Proposed bundle schema: `docs/countersign/receipt-profile/v1/schemas/bundle.schema.json`
- Published corpus: `docs/countersign/receipt-profile/v1/vectors.json`
- Receiver module: `examples/credential-consumer/profile-v1.mjs`
- Standalone runner: `examples/credential-consumer/verify-profile-v1-vectors.mjs`
- Deterministic vector producer: `examples/credential-consumer/scripts/generate-profile-v1-vectors.mjs`
- Tests: `examples/credential-consumer/test/profile-v1.test.mjs`

These are PROPOSED schemas for C2-A admission, not checked-in protocol schemas. Admission is a Stage-2 act under the schema-freeze law and requires the C2-A pin-move ceremony. Until that ceremony completes, the Stage-0 specification and receiver use these documents only as profile-local conformance artifacts.

The proposed schemas are closed: an unknown member is an error. The runtime verifier repeats the semantic and cryptographic checks that JSON Schema cannot express.

Each published vector also carries a `verification_policy` test fixture containing a fixed verification time and fixture-key pins. That wrapper member is not part of the receipt bundle and is not a deployment trust source.

## 3. Data model

Every bundle has exactly five top-level members:

```text
profile
receipt
proofs
retention_evidence
verification_keys
```

`proofs` always contains all eight named slots. A slot that has no evidence is `null`; absence of the slot is invalid. This prevents a serializer from erasing the distinction between an explicitly unavailable proof and an older shape that never considered it.

### 3.1 Mandatory trust statement

Every receipt contains this complete object:

```json
{
  "terminating_party": "cloudflare|provider|google-fe",
  "witness_mode": "mpc-tls|proxy-tls|tee-quote|provider-signature",
  "notaries": [
    {
      "id": "bounded identifier",
      "key_ref": "ed25519:sha256:<64 lowercase hex>",
      "operator": "bounded identifier",
      "jurisdiction": "ISO country or country-subdivision form"
    }
  ],
  "quorum": { "k": 0, "n": 0 },
  "notary_trust_model": "counterparty|threshold|tee|none"
}
```

The following combinations are valid:

| `witness_mode` | Required primary proof | Notary constraints | Required trust model |
|---|---|---|---|
| `mpc-tls` | `notary_quorum` | `n` equals the declared notary count; `1 <= k <= n` | `counterparty` or `threshold` |
| `proxy-tls` | `notary_quorum` | `n` equals the declared notary count; `1 <= k <= n` | `counterparty` or `threshold` |
| `tee-quote` | `tee_quote` | empty notary list and `k = n = 0` | `tee` |
| `provider-signature` | `provider_signature` | empty notary list and `k = n = 0` | `none` |

`none` is a disclosure that no notary trust model is present. It is not a favorable trust result.

### 3.2 WHO BINDS vocabulary

`WHO BINDS` identifies the actor responsible for supplying a value to the receipt or its proof structure. It is exactly one of `client`, `notary`, `provider`, or `none-yet`. It does not substitute for the primary-witness verification rules in Section 5.

### 3.3 Field table

All fields below are required unless marked “required slot; nullable.” An object-valued slot, when non-null, requires every nested field shown.

| Field | Type / closed values | Requirement | Domain or signature input | WHO BINDS |
|---|---|---|---|---|
| `profile` | constant profile identifier | required | receipt-profile selection | `client` |
| `receipt.receipt_id` | bounded identifier | required | `syngraphe.receipt-profile-v1.receipt\0` | `client` |
| `receipt.request_digest` | `sha256:<hex>` | required | receipt domain + provider signature component | `client` |
| `receipt.response_digest` | `sha256:<hex>` | required | receipt domain + provider signature component | `client` |
| `receipt.endpoint` | HTTPS URL, no credentials, port, query, or fragment | required | receipt domain through `receipt_digest` | `client` |
| `receipt.model_id` | bounded provider-qualified identifier | required | receipt domain + provider signature component | `provider` |
| `receipt.retention_class` | Section 4 enum | required | receipt domain + provider signature component | `provider` |
| `receipt.timestamp` | canonical RFC 3339 UTC seconds | required | receipt domain + provider signature component | `provider` |
| `receipt.trust_statement.terminating_party` | `cloudflare`, `provider`, `google-fe` | required | receipt domain | `client` |
| `receipt.trust_statement.witness_mode` | Section 3.1 enum | required | receipt domain | `client` |
| `receipt.trust_statement.notaries` | array, at most 16 | required | receipt domain | `client` |
| `receipt.trust_statement.notaries[].id` | bounded identifier | required per item | receipt domain | `client` |
| `receipt.trust_statement.notaries[].key_ref` | Ed25519 key fingerprint | required per item | receipt domain | `client` |
| `receipt.trust_statement.notaries[].operator` | bounded identifier | required per item | receipt domain | `client` |
| `receipt.trust_statement.notaries[].jurisdiction` | country or subdivision code form | required per item | receipt domain | `client` |
| `receipt.trust_statement.quorum.k` | integer `0..65535` | required | receipt domain | `client` |
| `receipt.trust_statement.quorum.n` | integer `0..65535` | required | receipt domain | `client` |
| `receipt.trust_statement.notary_trust_model` | Section 3.1 enum | required | receipt domain | `client` |
| `proofs.provider_signature` | object or `null` | required slot; nullable | RFC 9421-style signature base | `provider` |
| `proofs.provider_signature.algorithm` | `ed25519` | required when non-null | signature parameter | `provider` |
| `proofs.provider_signature.covered_components` | exact ordered six-member list | required when non-null | RFC 9421-style signature base | `provider` |
| `proofs.provider_signature.created` | Unix seconds | required when non-null | signature parameter; equals receipt time | `provider` |
| `proofs.provider_signature.key_ref` | Ed25519 key fingerprint | required when non-null | signature parameter | `provider` |
| `proofs.provider_signature.receipt_digest` | `sha256:<hex>` | required when non-null | provider signature component | `provider` |
| `proofs.provider_signature.signature` | 64-byte base64url Ed25519 signature | required when non-null | RFC 9421-style signature base | `provider` |
| `proofs.notary_quorum` | object or `null` | required slot; nullable | `syngraphe.receipt-profile-v1.notary-signature\0` | `notary` |
| `proofs.notary_quorum.receipt_digest` | `sha256:<hex>` | required when non-null | notary domain | `notary` |
| `proofs.notary_quorum.signatures` | array, at most 16 | required when non-null | notary domain | `notary` |
| `proofs.notary_quorum.signatures[].notary_id` | declared notary id | required per item | notary domain | `notary` |
| `proofs.notary_quorum.signatures[].key_ref` | declared key fingerprint | required per item | notary domain | `notary` |
| `proofs.notary_quorum.signatures[].algorithm` | `ed25519` | required per item | notary domain | `notary` |
| `proofs.notary_quorum.signatures[].signature` | 64-byte base64url signature | required per item | notary domain | `notary` |
| `proofs.tee_quote` | object or `null` | required slot; nullable | `syngraphe.receipt-profile-v1.tee-appraisal\0` | `notary` |
| `proofs.tee_quote.receipt_digest` | `sha256:<hex>` | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.quote_format` | `aws-nitro`, `sev-snp`, `tdx-v4` | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.quote_digest` | digest of quote bytes | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.appraisal.verifier_id` | bounded identifier | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.appraisal.key_ref` | Ed25519 key fingerprint | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.appraisal.appraised_at` | canonical RFC 3339 UTC seconds | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.appraisal.result` | `pass` | required when non-null | TEE appraisal domain | `notary` |
| `proofs.tee_quote.appraisal.signature` | 64-byte base64url signature | required when non-null | TEE appraisal domain | `notary` |
| `proofs.ots` and nested fields | receipt digest, HTTPS calendar URL, proof digest | required slot; nullable | none-yet; only `receipt_digest` links the slot | `none-yet` |
| `proofs.key_transparency` and nested fields | receipt digest, key ref, log id, checkpoint and inclusion digests | required slot; nullable | none-yet; only `receipt_digest` links the slot | `none-yet` |
| `proofs.lineage` and nested fields | receipt digest, parent digest, fixed relation | required slot; nullable | none-yet; only `receipt_digest` links the slot | `none-yet` |
| `proofs.disclosure_stance` and nested fields | receipt digest, `digest-only`, statement digest | required slot; nullable | none-yet; only `receipt_digest` links the slot | `none-yet` |
| `proofs.multi_operator_inclusion` and nested fields | receipt digest and 2–16 distinct-operator entries | required slot; nullable | none-yet; only `receipt_digest` links the slot | `none-yet` |
| `retention_evidence.provider` | `openai`, `anthropic`, `google` | required | receipt retention-class consistency check | `client` |
| `retention_evidence.source_url` | exact first-party URL for provider | required | source lookup | `client` |
| `retention_evidence.retrieved_on` | calendar date | required | source snapshot metadata | `client` |
| `retention_evidence.source_sha256` | digest of retrieved response bytes | required | SHA-256 | `client` |
| `retention_evidence.declared_class` | equals receipt class | required | receipt retention-class consistency check | `client` |
| `retention_evidence.service_features` | known, unique, sorted feature ids | required | contradiction check | `client` |
| `verification_keys[]` | 1–32 closed key objects | required | role-specific proof verification | `client` |
| `verification_keys[].algorithm` | `ed25519` | required | proof verification | `client` |
| `verification_keys[].public_key` | raw 32-byte base64url key | required | Ed25519 verification | `client` |
| `verification_keys[].key_ref` | `ed25519:sha256:` + SHA-256 of raw key | required | key-substitution check | `client` |
| `verification_keys[].role` | provider, notary, TEE verifier, or log operator | required | proof-role check | `client` |
| `verification_keys[].operator` | bounded identifier | required | disclosure only | `client` |

## 4. Retention-class vocabulary

The vocabulary records the class declared for an exchange. It is not evidence that the provider performed the declared behavior. When a current provider arrangement cannot be mapped without ambiguity, the client MUST emit `undeclared`.

| Provider | Closed classes | First-party source retrieved 2026-09-04 |
|---|---|---|
| OpenAI | `openai-abuse-monitoring-30d`, `openai-application-state-30d`, `openai-modified-abuse-monitoring`, `openai-zero-data-retention`, `undeclared` | <https://developers.openai.com/api/docs/guides/your-data> |
| Anthropic | `anthropic-covered-model-30d`, `anthropic-feature-specific`, `anthropic-zero-data-retention`, `undeclared` | <https://platform.claude.com/docs/en/manage-claude/api-and-data-retention> |
| Google | `google-approved-zero-data-retention`, `google-feature-specific`, `google-paid-service-limited-abuse-monitoring`, `undeclared` | <https://ai.google.dev/gemini-api/docs/zdr> |

The verifier rejects a provider/model-prefix mismatch, a provider/endpoint-host mismatch, a source-URL mismatch, and these named contradictions:

- OpenAI zero-data-retention with conversations state, file storage, Responses storage, or video.
- Anthropic zero-data-retention with a covered 30-day model, code execution, or managed agents.
- Google approved zero-data-retention with explicit caching, file storage, Search or Maps grounding, Interactions storage, or Live session resumption.

The feature list is not inferred from the endpoint. It is supplied by the client and remains subject to independent review.

## 5. Cryptographic processing

### 5.1 Receipt digest

1. Validate the closed receipt shape and the safe-integer subset.
2. Canonicalize the receipt with RFC 8785 JSON Canonicalization Scheme rules. Profile member names are ASCII and numbers are safe integers.
3. Hash UTF-8 bytes of `syngraphe.receipt-profile-v1.receipt\0 || canonical_receipt` with SHA-256.
4. Encode as `sha256:<64 lowercase hex>`.

### 5.2 Provider signature

The provider signature uses the RFC 9421 signature-base construction over this exact ordered list:

```text
syngraphe-request-digest
syngraphe-response-digest
syngraphe-model-id
syngraphe-retention-class
syngraphe-timestamp
syngraphe-receipt-digest
```

The first five are the required exchange tuple. The receipt-digest component additionally covers endpoint, receipt id, and the complete trust statement. The `@signature-params` line repeats the ordered component identifiers and carries `created`, `keyid`, `alg="ed25519"`, and `tag="provider-receipt-v1"`. `created` MUST equal the receipt timestamp in Unix seconds.

### 5.3 Notary quorum

Each notary signs UTF-8 bytes of `syngraphe.receipt-profile-v1.notary-signature\0 || receipt_digest` with Ed25519. The verifier resolves the declared notary id to the exact declared key reference, checks key role `notary`, verifies each distinct signature, and requires at least `k` valid declared notaries.

### 5.4 TEE quote appraisal

The bundle carries only the quote digest. The appraiser signs the ordered UTF-8 tuple under `syngraphe.receipt-profile-v1.tee-appraisal\0`: receipt digest, quote format, quote digest, appraisal time, result, and verifier id. The profile verifier validates that appraisal signature under a key with role `tee-verifier`.

This check verifies a signed appraisal of committed quote bytes. It does not directly parse hardware quote bytes and does not authenticate the appraiser merely because its public key is present. A relying party MUST obtain the accepted appraiser key reference through an authenticated trust channel before treating the appraisal as trusted.

### 5.5 Key references

`key_ref` is `ed25519:sha256:` followed by the lowercase SHA-256 digest of the raw 32-byte Ed25519 public key. A self-contained public key proves cryptographic consistency, not operator identity. Production reliance requires an authenticated key pin, directory, or transparency policy outside this self-contained bundle.

### 5.6 Caller trust and time policy

The module entry point is `verifyReceiptProfileV1Bundle(bundle, verification_policy)`. The caller MUST supply this closed policy object separately from the bundle:

```json
{
  "max_future_skew_seconds": 0,
  "trusted_key_refs": ["ed25519:sha256:<64 lowercase hex>"],
  "verified_at": "2026-09-04T12:05:00Z"
}
```

`trusted_key_refs` MUST come from a relying-party trust configuration or authenticated resolver, never from `bundle.verification_keys`. The verifier cryptographically checks included public keys, then requires the active provider key, TEE appraiser key, or at least `k` notary keys to match caller pins. A valid signature without sufficient caller pins produces `status: "indeterminate"` and `ok: false`.

`verified_at` is the caller's fixed evaluation time. `max_future_skew_seconds` is an integer from 0 through 86,400. A receipt or TEE appraisal later than that evaluation window is invalid. The verifier does not read the ambient clock, which keeps replay and testing deterministic.

The result status is exactly `valid`, `invalid`, or `indeterminate`. `ok` is true only for `valid`. Invalid bundle findings are returned in `errors`; missing external trust is returned separately in `indeterminate`. Untrusted values are never echoed.

## 6. Verification rules

The verifier is fail-closed and never includes untrusted field values in an error. Its stable rules are:

| Rule | Outcome condition | Named invalid vector |
|---|---|---|
| `AIR-V1-R001` | bundle is not acyclic JSON object data | — |
| `AIR-V1-R002` | bundle exceeds 262,144 UTF-8 bytes | — |
| `AIR-V1-R003` | top-level closed shape mismatch | `invalid-01-unknown-field` |
| `AIR-V1-R010` | unsupported profile id | — |
| `AIR-V1-R020` | receipt closed shape or canonical subset mismatch | — |
| `AIR-V1-R021` | request or response digest malformed | — |
| `AIR-V1-R022` | endpoint is not bounded query-free HTTPS | — |
| `AIR-V1-R023` | id, model, or timestamp malformed | — |
| `AIR-V1-R024` | receipt is later than caller verification window | — |
| `AIR-V1-R030` | trust statement incomplete or malformed | `invalid-02-incomplete-trust-statement` |
| `AIR-V1-R031` | witness, trust model, notaries, and quorum inconsistent | `invalid-03-inconsistent-empty-quorum` |
| `AIR-V1-R040` | proof slot container is not closed or complete | — |
| `AIR-V1-R050` | verification key list or metadata invalid | — |
| `AIR-V1-R051` | key reference does not match public key | `invalid-06-key-fingerprint-mismatch` |
| `AIR-V1-R052` | proof key is absent or has the wrong role | — |
| `AIR-V1-R053` | cryptographic proof lacks enough caller-pinned keys; status is indeterminate | — |
| `AIR-V1-R060` | proof receipt digest does not match computed digest | `invalid-04-proof-receipt-digest-mismatch` |
| `AIR-V1-R070` | provider witness mode lacks provider signature | — |
| `AIR-V1-R071` | provider signature metadata or signature invalid | `invalid-05-provider-signature-corrupt` |
| `AIR-V1-R080` | notary witness mode lacks quorum proof | — |
| `AIR-V1-R081` | notary signatures do not meet quorum | `invalid-07-notary-quorum-shortfall` |
| `AIR-V1-R082` | a notary signature or binding is invalid | `invalid-08-notary-signature-corrupt` |
| `AIR-V1-R090` | TEE witness mode lacks quote appraisal | `invalid-09-tee-proof-absent` |
| `AIR-V1-R091` | TEE quote commitment or appraisal invalid | `invalid-10-tee-appraisal-corrupt` |
| `AIR-V1-R092` | TEE appraisal is later than caller verification window | — |
| `AIR-V1-R100` | retention evidence shape or snapshot metadata invalid | — |
| `AIR-V1-R101` | provider and first-party source mismatch | — |
| `AIR-V1-R102` | retention class invalid, cross-provider, or inconsistent | — |
| `AIR-V1-R103` | endpoint host does not match provider | — |
| `AIR-V1-R104` | service feature list invalid | — |
| `AIR-V1-R105` | retention class contradicts a selected feature | `invalid-11-retention-contradiction` |
| `AIR-V1-R106` | model prefix does not match provider | — |
| `AIR-V1-R110` | auxiliary slot metadata invalid | — |
| `AIR-V1-R111` | multi-operator inclusion is not structurally multi-operator | `invalid-12-single-operator-inclusion` |
| `AIR-V1-R120` | caller verification policy is absent or malformed; absence is indeterminate | — |

## 7. STRONGER creation and disclaimer inversions

These rules are mandatory for any product surface that creates, stores, displays, exports, or verifies profile-v1 bundles.

1. **ProviderReceipt and SessionRecord remain different types.** `ProviderReceipt` is per exchange and contains the request/response commitment tuple and its primary witness proof. `SessionRecord` describes a session-scoped channel, keyset, or attestation state. Neither type may be accepted where the other is required. A receipt MAY carry a digest reference to a session record, but the reference does not import the session's claims. This is the profile-v1 proposal for owner decision O-6.
2. **Primary proof is explicit.** Missing, invalid, or below-quorum primary evidence means rejection. A configured witness mode may never fall back to a different witness mode.
3. **Retention is a declaration.** A recognized class means only that the bundle names a class from a retrieved first-party source. `undeclared` means the mapping was not established. Neither result proves provider conduct.
4. **Included keys are not trust anchors.** A matching fingerprint means the included bytes match the reference. It does not establish operator identity. The active proof MUST also meet the caller-supplied pin policy; otherwise the status is `indeterminate`. The inverse statement MUST be displayed wherever a self-contained result is shown.
5. **`ots` is `none-yet`.** `null` means no OpenTimestamps commitment is supplied. A non-null slot means a proof digest and calendar location are recorded; it does not mean the external proof was checked.
6. **`key_transparency` is `none-yet`.** `null` means no key-log commitment is supplied. A non-null slot does not mean log inclusion, checkpoint signature, consistency, or operator identity was checked.
7. **`lineage` is `none-yet`.** `null` means no parent receipt is declared. A non-null parent digest does not prove that the parent exists or that the stated relation is valid.
8. **`disclosure_stance` is `none-yet`.** `null` means no disclosure-stance commitment is supplied. A non-null digest does not prove what the referenced statement says or that it was followed.
9. **`multi_operator_inclusion` is `none-yet`.** `null` means no multi-operator inclusion commitments are supplied. A non-null slot proves only that at least two distinct operator identifiers and well-formed digests were supplied; it does not verify inclusion proofs, checkpoints, or organizational independence.
10. **TEE appraisal is not raw-quote verification.** A valid appraisal signature confirms only the appraiser's signed statement over the quote digest. Hardware endorsement, freshness, revocation, and appraiser trust remain external inputs.
11. **Absence never becomes favorable.** `null`, `undeclared`, an unavailable trust anchor, or an unverified auxiliary slot MUST remain explicit and MUST NOT be promoted to a favorable claim.

## 8. Receiver integration after foundation PR #63

Foundation reference: `feat/consumer-conformance` at `ef98d7f5e966657ee7e87b745692be90eb4e872a`. This lane neither bases on nor merges that branch.

The profile mirrors the foundation's `examples/credential-consumer/` module, script, test, and pinned-vector layout. It requires no service. After #63 merges, one manifest line in `examples/credential-consumer/package.json` may expose `node verify-profile-v1-vectors.mjs` as a named script; the existing `test/*.test.mjs` glob discovers the profile test without receiver changes. The existing closed credential webhook payload MUST NOT be widened to carry this separate bundle.

## 9. Reproducible verification

Use the repository-pinned runtime:

```sh
nvm use
node examples/credential-consumer/verify-profile-v1-vectors.mjs

node --test examples/credential-consumer/test/profile-v1.test.mjs
```

`nvm use` consumes the checked-in `.nvmrc`. The runner is a conformance-corpus tool: it exits non-zero unless the corpus contains exactly 12 valid and 12 invalid vectors, every valid vector is accepted under its fixed test policy, and every invalid vector is rejected for its named rule. Corpus test pins are not deployment authority.

## 10. Security considerations

- The verifier accepts at most 262,144 serialized bytes, 32 keys, and 16 notaries. Unknown fields fail closed.
- Endpoint URLs exclude credentials, explicit ports, query strings, and fragments so secrets and request parameters do not enter receipts.
- Provider lookup and retention-contradiction lookup use own-key maps so hostile names cannot reach object prototypes.
- Canonicalization rejects lone UTF-16 surrogates before hashing.
- The caller supplies trusted key pins and a fixed verification time independently of the bundle; missing authority remains `indeterminate`.
- Error objects contain rule ids, field paths, and fixed descriptions only. They do not echo field values or exchange data.
- Replay detection and receipt-id uniqueness require durable receiver state and are outside this no-service verifier. A deployment MUST add that state before using receipts for replay-sensitive decisions.
- Ed25519 and SHA-256 are fixed for profile v1. Algorithm substitution is rejected. A successor profile is required for algorithm migration.
- Provider terms and feature eligibility change. A client MUST refresh the source snapshot and use `undeclared` when the then-current arrangement is not represented by this closed vocabulary.
- Optional evidence slots do not elevate the primary status. Their verification requires separately specified proof bytes, trust anchors, and verifier procedures.
