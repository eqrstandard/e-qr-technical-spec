# Changelog — e-QR Technical Specification

All notable changes to the technical specification are documented in this file.

The format follows a simplified variant of Keep a Changelog and uses semantic versioning
adapted for specification drafts.

## Signed-URL Carrier Profile v0.2 — addendum to v0.1 (2026-11-15)

### Changed
- The signed link now uses the address structure of the specification, section 7.1: `https://<host>/1/m/<OPID>?pi=…&instr=…&mid=…&sig=…`.
- The signed part carries the version (`ver`) and the payment method (`instr`); a verifier refuses a version or method it does not implement.
- Claim names follow the specification: `amt` (minor units), `ccy`, `rmt`, `ref`.
- Payment methods are listed in the Profile; the first and only one is `SCTI`.
- Codes issued under v0.1 stay valid and are read as version 1, method `SCTI`.

### To be folded into v0.2 of the specification
- Reconcile section 7.4 (no account details in a point-of-sale code) with the signed part of the Profile.

## Signed-URL Carrier Profile v0.1 — addendum to v0.1 (2026-11-14, replaced)

### Added
- Signed-URL Carrier Profile v0.1 (`spec/e-QR_Signed_URL_Carrier_Profile_v0.1.md`): a camera-scannable https link
  carrying a signed claim set (MID, verified name, IBAN, optional amount and reference), verified against the signed
  Operator Directory with no resolver call per payment; operator key and revocation-list endpoints; payer-page privacy rule.
- Hyphenated merchant identifier form `CC-OPID-NNNNNNNN`.

### To be folded into v0.2
- Allow hyphens in the MID (v0.1 section 6.3 says alphanumeric).
- Use "certified" where v0.1 says "accredited".
- State the revocation list for both carriers.

## [v0.1] — 2026-01-12

Initial public consultation draft.

### Added
- QR carrier profile aligned with EPC MSCT QR standardisation.
- Proxy (MID) and token-based payload profiles for merchant-presented POI payments.
- Resolver interface for merchant and transaction resolution.
- Signed Operator Directory as trust anchor.
- Mandatory signed resolver responses using JWS (ES256) and JSON canonicalization.
- Security, parsing, and conformance requirements for payer applications and Operators.

### Notes
- Draft intended for early technical and security feedback only.
- Governance, certification, and rollout details are explicitly out of scope.
