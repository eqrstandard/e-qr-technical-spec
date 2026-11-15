# e-QR Technical Specification

This repository contains the **e-QR Technical Specification**, a draft interoperability specification for
merchant-presented account-to-account (A2A) payment initiation using **SEPA Instant Credit Transfers (SCT Inst)**.

The specification defines:
- a QR carrier profile aligned with EPC MSCT guidance,
- payload encoding rules,
- a resolver-based merchant verification model, and
- security and conformance requirements enabling interoperable implementations across PSPs.

The specification **does not** introduce a new payment system, clearing or settlement infrastructure,
consumer wallet, or PSP APIs. Execution, authentication (including SCA), risk controls, and settlement
remain fully within existing regulated PSP and SCT Inst frameworks.

## Status

- **Current version:** v0.1 (Draft for consultation)
- **Purpose:** early technical and security feedback
- **Scope:** merchant-presented POI initiation (`/m/`) using QR as carrier

This draft is intended for review by banks, PSPs, and payment infrastructure experts to inform
subsequent versions (v0.2+) and a proof-of-concept phase.

## Governance

The specification is developed and stewarded by **e-QR Payment Standard MTÜ**
(Estonian non-profit association).

## Repository contents

/spec/e-QR_Technical_Spec_v0.1.pdf

/spec/e-QR_Signed_URL_Carrier_Profile_v0.1.md — draft addendum: the camera-scannable signed-link carrier used in the pilot


## Feedback

Feedback is currently solicited **off-repository** via direct engagement with banks and PSPs.
Public issues and pull requests may be enabled in a later phase.

## More information

- Project website: https://eqrstandard.com
- Contact: info@eqrstandard.com
