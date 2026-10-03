# Profile v1.1 schemas

- `receipt.schema.json` is the closed profile v1.1 receipt schema. It adds only the `issuer-operated` notary trust-model value to the v1 receipt shape.
- `bundle.schema.json` is the closed profile v1.1 bundle schema. It accepts the v1 or v1.1 profile identifier and references the v1.1 receipt schema.

Receiver policy, including `issuer_operator`, is outside the artifact schemas because it is trusted input and MUST NOT be supplied by the artifact under test.
