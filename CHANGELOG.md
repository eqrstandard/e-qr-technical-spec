# Changelog — e-QR Technical Specification

## v0.1 — 2026-11-18

Edited in place to match the scheme in use. The Signed-URL Carrier Profile, published separately on 14–16 November, is folded into the specification and withdrawn.

### Changed
- The code (s. 7–8): every code uses a scheme payer host listed in the Directory's `payer_hosts` (today `pay.eqrstandard.com`) and carries the operator's signed part with the payee's verified name and IBAN. Section 7.4 allows account details only inside that signature.
- Merchant ID (s. 6.3): `CC-OPID-NNNNNNNN`.
- Operator Directory (s. 10): published at `https://eqrstandard.com/.well-known/eqr/operators.json` and signed with the Association's directory key; adds `payer_hosts` and, per operator, `name`, `country` and `registry` in place of `hosts`.
- Directory signature (s. 10.1): the signed part carries the Directory as a JSON object, in no prescribed member order, and payer applications read the Directory from the decoded signed part. The earlier text required the JCS (RFC 8785) form, which the published Directory does not use.
- "Association" and "certified" replace "Governance Authority" and "accredited".

### Added
- The payer page (s. 5.2), payment methods (s. 7.5), verifying a code (s. 8.6), operator endpoints including the revocation list (s. 9.6) and payer host conformance (s. 12.3).
- Test vectors for the signed code (s. 13).

## v0.1 — 2026-01-12

Initial public consultation draft.
