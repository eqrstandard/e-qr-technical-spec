# e-QR Technical Specification

This repository holds the **e-QR Technical Specification**, the interoperability specification of the e-QR payment standard: merchant-presented codes for initiating SEPA Instant Credit Transfers (SCT Inst).

The specification defines:
- the code: an HTTPS link in the format of EPC024-22 v2.10, which CEN published as EN 18184:2025, signed by the certified operator that issued it,
- how a PSP's app, or the scheme payer page, verifies a code without calling the operator,
- the resolver interface and the operator endpoints, including the revocation list,
- the signed Operator Directory, and
- security, privacy and conformance requirements.

e-QR does not introduce a new payment system, clearing or settlement infrastructure, consumer wallet or PSP API. Execution, authentication (including SCA), risk controls and settlement remain with regulated PSPs and the SCT Inst scheme.

## Status

- **Current version:** v0.1, published 18 November 2026
- **In use:** Estonia. Certified operators: https://eqrstandard.com/operators

## Contents

`spec/e-QR_Technical_Spec_v0.1.pdf`

The Scheme Rules, the Operator Certification Criteria and the Governance Note are published at https://eqrstandard.com/documents.

## Governance

Maintained by **e-QR Payment Standard MTÜ**, an Estonian non-profit association (registry code 80666291).

## Contact

info@eqrstandard.com
