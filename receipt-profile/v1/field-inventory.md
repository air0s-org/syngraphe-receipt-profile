# Profile v1 field inventory

Retrieval date: 2026-09-04 (America/New_York). All external source digests are SHA-256 over the exact retrieved response bytes.

This appendix is informative. It records the fields reviewed and the profile-v1 disposition. A similar name is comparison evidence only; profile v1 does not claim wire compatibility with any source below.

## Source register

| Source | Immutable locator | Digest |
|---|---|---|
| Chueayen, *Enforcement Attestation Receipts for AI Inference Decisions*, revision 00 | <https://www.ietf.org/archive/id/draft-chueayen-attestation-receipts-00.txt> | `sha256:fb3ff6bd8ba3697b11ca37a49ec57d89d511905dceceb972ba67db4492631138` |
| Marques, *Compliance Profile of Signed Action Receipts for AI Agents*, revision 08 | <https://www.ietf.org/archive/id/draft-marques-asqav-compliance-receipts-08.txt> | `sha256:ee3ca5d7c0acc1cb9b8025d29f19a7d73991718ca35d3bf4229f7b4264976ec0` |
| Das, *Beyond Attestation: An Execution-Finality Architecture for Controlling Sensitive OpenAI and Anthropic Claude Model Information*, revision 00 | <https://www.ietf.org/archive/id/draft-das-rats-openai-anthropic-extraction-00.txt> | `sha256:07c6b2662bcffd8c0393b775a18051075b4df8ca71c9897be4139d181ab5c088` |
| GLACIS OVERT v1.1 Markdown source | <https://overt.is/OVERT_v1.1_STANDARD.md> | `sha256:f25c106b3881ec0bf77dc102bb138322896bcb016495f433786a5362b6e0edda` |
| Phala ACI verifier npm package 0.7.2 | <https://registry.npmjs.org/@phala/aci-verifier/-/aci-verifier-0.7.2.tgz> | `sha256:40fd4772e61bb43b461548c38466f7ff276bed6e197af32e23900e6bd806d68c` |

one private estate repository, pinned by commit, informative only, not publicly verifiable

## Field comparison

| Source | Fields or structures reviewed | Profile-v1 use | Gap retained explicitly |
|---|---|---|---|
| Chueayen 00 | `v`, `attestation_id`, `trace_id`, `org_id`, `request_hash`, `model`, `outcome`, `policy_applied`, `cost_prevented_eur`, `timestamp`, `public_key`, `signature` | Request digest, model id, time, closed object, offline signature verification | No response digest, endpoint, retention class, or mandatory trust topology; its embedded key does not by itself establish issuer identity |
| Marques 08 | Envelope `payload`, `signature`, `anchors`; common fields `type`, `issued_at`, `issuer_id`, `payload_digest`, `action_ref`, `sandbox_state`, `iteration_id`, `key_thumbprint`; decision fields `reason`, `policy_digest`; hash-chain linkage, timestamp anchors, signer-outage evidence, counterparty binding, validity, build provenance, environment attestation | Key fingerprint, canonical payload digest, linkage and anchoring slots, explicit counterparty/notary distinction | Its action/compliance receipt is broader than one provider exchange; profile v1 does not import regulatory-retention conclusions or action-policy fields |
| Das 00 | Candidate RATS claims `release_control_profile`, `release_control_enabled`, `release_control_measurement`, `model_identity`, `release_policy_id`, `security_epoch`, `extraction_state_id`, `extraction_state_commitment`, `rollback_protection`, `finality_sink_id`, `finality_sink_class`, `destination_binding_supported`, `authority_consumption_mode`, `alternate_egress_control`, `release_control_assurance` | Separation of environment/session evidence from per-exchange evidence; model identity as an explicit commitment | The draft is a release-control architecture, not a per-response receipt format; profile v1 does not claim release prevention, extraction prevention, or attested model execution |
| OVERT v1.1 | Non-egress commitments, receipt id/hash, epoch, binary and network hashes, counter, time, temporality flags, notary signatures, transparency proofs, parent receipt references, t-of-n topology, operator and jurisdiction disclosures | Digest-only rule, explicit notary set/quorum/trust model, linkage and multi-operator slots | Profile v1 does not claim OVERT conformance, co-epoch measurement, sampling completeness, log inclusion, or organizational independence |
| Phala ACI verifier 0.7.2 | `api_version`, `receipt_id`, `chat_id`, `model`, `workload_keyset_digest`, `endpoint`, `method`, `served_at`, ordered event log, `key_id`, signature; request/response body hashes; session id/evidence; upstream verification record | Per-response request and response commitments, endpoint/model/time, keyset separation, fail-closed offline transcript | Profile v1 is not an ACI verifier and does not import workload attestation, channel binding, upstream-session verdicts, or private receipt retrieval |

## Retention source snapshots

These current first-party pages constrain the closed enum in profile v1. The hashes identify retrieved page bytes; they are not stable provider policy identifiers.

| Provider | URL | Retrieved | Retrieved-byte SHA-256 | Profile interpretation |
|---|---|---|---|---|
| OpenAI | <https://developers.openai.com/api/docs/guides/your-data> | 2026-09-04 | `b6fb4faccca5f78f05c9201c44a3a85e7052ed7429650cac47319e7d914f3d25` | Default abuse-monitoring retention, Modified Abuse Monitoring, Zero Data Retention, endpoint application state, and feature exceptions remain distinct classes |
| Anthropic | <https://platform.claude.com/docs/en/manage-claude/api-and-data-retention> | 2026-09-04 | `7c4bd54acc63db516913d1a3e1eb186b8f51992b013240ce004affec07d26fde` | Zero Data Retention, covered-model 30-day retention, and feature-specific storage remain distinct classes |
| Google | <https://ai.google.dev/gemini-api/docs/zdr> | 2026-09-04 | `6e94614dc557248e62e21244f63c281e400c47aa03bc7b0e54fe82c5c022945a` | Paid-service limited abuse monitoring, approved zero-data-retention, and feature-specific storage remain distinct classes |

No entry in this table is evidence of what happened during an exchange. Profile v1 records the declared class and rejects document-level contradictions only.

## Relying-party inputs

Primary-proof acceptance additionally requires trusted-key pins and a fixed verification time supplied by the caller outside the bundle. The published vector policies contain fixture pins solely for deterministic conformance testing; they are not a trust source for a deployment.
