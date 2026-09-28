# Receipt profile schema files

## What this folder is

This folder holds the two proposed JSON rules that define the permitted shape of a version-one interaction receipt and its verification bundle. They close the documents to unknown fields, while leaving timing, ordering, cryptographic, size, and cross-field checks to the runtime verifier.

## Files

- `bundle.schema.json` — Defines a five-member bundle with eight explicit proof slots, 1–32 verification keys, and closed structures for provider signatures, notary quorums, quote appraisals, retention evidence, and auxiliary commitments. Status: Reference only. Current state: proposed C2-A structural schema on main; runtime checks remain required; never deployed.
- `README.md` — This plain-language guide describes the two schema files and their role. Status: Finished. Current state: on the product documentation branch, with no deployment.
- `receipt.schema.json` — Defines the receipt fields, including query-free HTTPS endpoints, eleven retention-class values, canonical UTC-second timestamps, and the four witness modes `mpc-tls`, `proxy-tls`, `tee-quote`, and `provider-signature`. Status: Reference only. Current state: proposed C2-A structural schema on main; runtime checks remain required; never deployed.

## Folders

There are no subfolders.

## Where this fits

These are the profile-local schema inputs used by the version-one conformance corpus and verifier before a later schema admission ceremony.
