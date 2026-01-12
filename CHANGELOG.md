# Changelog — e-QR Technical Specification

All notable changes to the technical specification are documented in this file.

The format follows a simplified variant of Keep a Changelog and uses semantic versioning
adapted for specification drafts.

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
