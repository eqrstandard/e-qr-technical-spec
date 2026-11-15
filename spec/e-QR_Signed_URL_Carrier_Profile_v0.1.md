# e-QR Signed-URL Carrier Profile — version 0.1

**Addendum to the e-QR Technical Specification v0.1. Draft: not yet adopted by the board of e-QR Payment Standard MTÜ.**

This addendum defines the code format used in the Estonian pilot: an https link that a phone camera opens and that verifies itself in the browser. Where this Profile and Specification v0.1 differ, this Profile governs codes scanned with a phone camera; Specification v0.1 governs the resolver interface used by bank and PSP applications. Both carriers rely on the same signed Operator Directory.

## 1. Why a second carrier

Specification v0.1 defines a code that a bank application scans and resolves with a POST to the operator. No Estonian bank application does that yet. Until one does, the payer scans with the phone camera, so the code must be an ordinary https link that verifies itself in the browser, with no registry call per payment. This Profile defines that link. Both carriers may be printed for the same merchant; both are covered by the same Operator Directory.

## 2. Identifiers

- **OPID**: three characters A–Z, 0–9, issued by the Association (Spec s. 6.1).
- **MID**: `CC-OPID-NNNNNNNN`, pattern `^[A-Z]{2}-[A-Z0-9]{3}-[0-9]{8}$`, e.g. `EE-PCD-48213907`. Country, issuing operator, eight random digits. MIDs issued before this Profile with six to ten digits stay valid. In resolver calls under Spec v0.1 the same MID is sent as written.

## 3. The code

```
https://<payer-host>/<MID>?amount=<EUR>&ref=<text>&sig=<compact JWS>
```

`<payer-host>` is a host listed for the operator in the Operator Directory. `sig` is required; `amount` and `ref` are display hints only.

JWS header: `{"alg":"ES256","kid":"<kid>"}`, signature raw r‖s. JWS payload:

| Claim | Required | Meaning |
|---|---|---|
| `mid` | yes | Equals the MID in the path |
| `name` | yes | Verified receiver name, shown to the payer and passed to the bank |
| `iban` | yes | Receiver account, no spaces |
| `amount` | no | String with two decimals; present means fixed |
| `ref` | no | Payment description |
| `reference` | no | Estonian reference number, 2–20 digits with 7-3-1 check digit; signed only, never in the query |
| `iat`, `exp` | no | Unix seconds; a past `exp` means expired |

Name and IBAN are signed into the code. A change of either therefore needs a new code, and a change of IBAN a new MID.

## 4. What a verifier does

1. Fetch the Operator Directory from `https://registry.eqrstandard.com/.well-known/eqr/operators.json`; verify `sig.jws` with the Association's pinned directory key; reject if past `valid_until`. Refresh at least every 24 hours (Spec s. 10).
2. Take the OPID from the MID; find that operator with `status: active`; check that the code's host is in its `hosts`.
3. Check that the JWS `kid` is one of that operator's `signing_keys`, verify the signature, check `mid` against the path and `exp` against the clock.
4. Fetch the operator's revocation list (s. 5) at most once a day; refuse a listed MID.
5. Show the signed `name`, the IBAN tail and the amount; only then open the payer's bank. Never follow a redirect to obtain payee data.

## 5. Operator endpoints

On a host listed in the Directory:

- `GET /.well-known/eqr/jwks.json` — the operator's active keys as a JWK set. A convenience copy: a verifier trusts a key only if it is also in the signed Directory.
- `GET /.well-known/eqr/revoked.json` — `{"updated_at": "<RFC 3339>", "revoked": ["<MID>", …]}`, every MID that must not be paid, including pending, suspended, revoked and transferred ones. Updated within one hour of a change. `Cache-Control: public, max-age=3600`.

## 6. Payer privacy

A payer page stores no payer data: no IP address, no user agent, no account details. Scan events may be counted per MID without any payer identifier.

## 7. For Specification v0.2

Fold this Profile into the Specification; allow hyphens in the MID (v0.1 s. 6.3 says alphanumeric); say "certified" where v0.1 says "accredited"; state the revocation list for both carriers.
