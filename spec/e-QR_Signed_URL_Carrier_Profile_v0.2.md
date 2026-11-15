# e-QR Signed-URL Carrier Profile — version 0.2

Adopted by the board of e-QR Payment Standard MTÜ (registry code 80666291) on 15 November 2026 as an addendum to the e-QR Technical Specification v0.1.

This Profile defines the code a payer scans with a phone camera: an https link that carries the operator's signature and verifies itself in the browser. Version 0.2 puts that link into the address structure of Specification v0.1 s. 7.1 and adds a version and a payment method to the signed part. It replaces version 0.1. References to Carrier Profile v0.1 in the Scheme Rules v0.1 and the Certification Criteria v0.1 are read as references to this version.

## 1. One code, two readers

Specification v0.1 defines a code that a bank application scans and resolves with a POST to the operator. A phone camera cannot do that: it opens the link in a browser. A code under this Profile serves both. Opened by a camera, it shows the operator's payer page, which verifies the signature with no registry call for the payment. Read by an application that implements Specification v0.1, it is a proxy-mode code (Spec s. 8.2) and is resolved as Spec s. 9 describes, once the operator answers resolver calls on the code's host (s. 7).

## 2. Identifiers

- **OPID**: three characters A–Z, 0–9, issued by the Association (Spec s. 6.1).
- **MID**: country, issuing operator and eight random digits in the form `CC-OPID-NNNNNNNN`, pattern `^[A-Z]{2}-[A-Z0-9]{3}-[0-9]{8}$`. MIDs issued before this Profile with six to ten digits stay valid. In resolver calls the MID is sent as written.
- **PI**: the payload issuer (Spec s. 6.2). Where the operator generates the code itself, the PI is its own OPID.

## 3. The code

```
https://<host>/1/m/<OPID>?pi=<PI>&instr=<method>&mid=<MID>&sig=<compact JWS>
```

Example: `https://eqr.ee/1/m/PCD?pi=PCD&instr=SCTI&mid=EE-PCD-48213907&sig=eyJ…`

| Part | Required | Meaning |
|---|---|---|
| `<host>` | yes | A host listed for the operator in the Operator Directory |
| `/1/` | yes | Version of the code format (s. 4) |
| `/m/` | yes | Payment at the point of sale; the only type in this version |
| `<OPID>` | yes | The operator; equals the OPID inside the MID |
| `pi` | yes | Payload issuer |
| `instr` | yes | Payment method (s. 5) |
| `mid` | yes | Merchant ID |
| `sig` | yes | The signed part (s. 3.1) |
| `amt`, `ccy`, `rmt` | no | Amount in minor units, currency, payment description, as in Spec s. 8.1. Suggestions only: a payer page prefills them and the payer may change them. A signed value always takes precedence |

The other parameters of Spec s. 8.1 may be present. A reader ignores a parameter it does not know. A resolver ignores `sig`.

### 3.1. The signed part

`sig` is a compact JWS. Header: `{"alg":"ES256","kid":"<kid>"}`, signature in raw `r||s` form. Payload:

| Claim | Required | Meaning |
|---|---|---|
| `ver` | yes | Number. Equals the version in the path |
| `instr` | yes | Equals `instr` in the query |
| `mid` | yes | Equals `mid` in the query |
| `name` | yes | Verified receiver name, shown to the payer and passed to the bank |
| `iban` | yes for `SCTI` | Receiver account, uppercase, no spaces |
| `amt` | no | Integer, minor units (Spec s. 8.4); present means the amount is fixed |
| `ccy` | no | `EUR` when absent |
| `rmt` | no | Payment description, 1–140 characters |
| `ref` | no | Structured creditor reference, 1–35 characters; in Estonia the reference number, 2–20 digits with a 7-3-1 check digit. Signed only, never taken from the query |
| `iat`, `exp` | no | Unix seconds; a past `exp` means expired |

A verifier ignores a claim it does not know. Name and IBAN are signed into the code: a change of either needs a new code, and a change of IBAN a new MID (Rules s. 3.2).

The signed part is encoded, not encrypted: whoever scans the code can read the name and the account. Specification v0.1 s. 7.4 requires account details at the point of sale to be obtained by resolution. This Profile permits them inside the operator's signature, because a payer page must verify and pay with no registry call, and the payee is a business whose account every payer sees in the transfer. Spec s. 7.4 continues to apply to a code without `sig`.

## 4. Version

The version is `1`. It is raised only when a reader built for the earlier version would misread the code. Additions that an older reader can safely ignore (a new optional claim or parameter) do not raise it.

## 5. Payment methods

| Code | Method | Signed part must carry | The payer's side |
|---|---|---|---|
| `SCTI` | SEPA credit transfer in euro to the signed account, sent as an instant transfer (SCT Inst) | `iban` | The payer confirms the transfer in their own bank |

A code carries one method. The Association adds a method by a new version of this Profile, which gives it a four-character code, the claims it requires and what the payer's side does. A method code is never reused for another meaning.

## 6. What a verifier does

1. Fetch the Operator Directory from `https://registry.eqrstandard.com/.well-known/eqr/operators.json`; verify `sig.jws` with the Association's pinned directory key; reject if past `valid_until`. Refresh at least every 24 hours (Spec s. 10).
2. Take the OPID from the path; find that operator with `status: active`; check that the code's host is in its `hosts` and that the MID carries the same OPID (Spec s. 7.3).
3. Check that the JWS `kid` is one of that operator's `signing_keys` and verify the signature.
4. Check `ver`, `instr` and `mid` in the signed part against the path and the query; refuse the code if any differs. Check `exp` against the clock.
5. If the version or the method is not one the verifier implements, tell the payer that the code needs a newer application and offer no payment.
6. Fetch the operator's revocation list (s. 7) at most once a day; refuse a listed MID.
7. Show the signed `name`, the tail of the account, the amount and the payment method; only then open the payer's bank. Never follow a redirect to obtain payee data.

## 7. Operator endpoints

On a host listed in the Directory:

- `GET /.well-known/eqr/jwks.json`: the operator's active keys as a JWK set. A convenience copy: a verifier trusts a key only if it is also in the signed Directory.
- `GET /.well-known/eqr/revoked.json`: `{"updated_at": "<RFC 3339>", "revoked": ["<MID>", …]}`, every MID that must not be paid, including pending, suspended, revoked and transferred ones. Updated within one hour of a change. `Cache-Control: public, max-age=3600`.
- `POST` to the address of the code without its query: the resolver of Spec s. 9. The Association sets the date from which every operator answers it on the code's host; until then an operator may serve the resolver on another host listed in the Directory.

## 8. Codes issued under version 0.1

Version 0.1 codes have the form `https://<host>/<MID>?amount=<EUR>&ref=<text>&sig=<compact JWS>`, and their signed part has no `ver` or `instr` and carries `amount` (a string with two decimals), `ref` (the payment description) and `reference` (the Estonian reference number).

- A verifier reads such a code as version 1, method `SCTI`, and verifies it as in s. 6, taking the MID from the path.
- These codes stay valid until they expire or are revoked.
- A certified operator issues new codes only in the form of s. 3.

## 9. Payer privacy

A payer page stores no payer data: no IP address, no user agent, no account details. Scan events may be counted per MID without any payer identifier.

## 10. For Specification v0.2

Fold this Profile into the Specification; allow hyphens in the MID (v0.1 s. 6.3 says alphanumeric); say "certified" where v0.1 says "accredited"; state the revocation list for both carriers; reconcile s. 7.4 with s. 3.1 of this Profile.
