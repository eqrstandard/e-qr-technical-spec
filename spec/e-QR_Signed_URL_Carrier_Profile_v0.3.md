# e-QR Signed-URL Carrier Profile — version 0.3

Adopted by the board of e-QR Payment Standard MTÜ (registry code 80666291) on 16 November 2026 as an addendum to the e-QR Technical Specification v0.1. It replaces version 0.2. References to Carrier Profile v0.1 or v0.2 in the Scheme Rules v0.1 and the Certification Criteria v0.1 are read as references to this version.

This Profile defines the code a payer scans with a phone camera: an https link that carries the operator's signature and verifies itself in the browser. Version 0.3 moves every code to one payer page run for the whole scheme, so a payer sees the same page whichever certified operator onboarded the merchant, and places the Operator Directory on the Association's own domain.

## 1. One code, two readers

Opened by a phone camera, the code shows the scheme's payer page, which verifies the operator's signature with no registry call for the payment. Read by an application that implements Specification v0.1, it is a proxy-mode code (Spec s. 8.2) and is resolved as Spec s. 9 describes (s. 7).

## 2. Identifiers

- **OPID**: three characters A–Z, 0–9, issued by the Association (Spec s. 6.1).
- **MID**: country, issuing operator and eight random digits in the form `CC-OPID-NNNNNNNN`, pattern `^[A-Z]{2}-[A-Z0-9]{3}-[0-9]{8}$`. In resolver calls the MID is sent as written.
- **PI**: the payload issuer (Spec s. 6.2). Where the operator generates the code itself, the PI is its own OPID.

## 3. The code

```
https://pay.eqrstandard.com/1/m/<OPID>?pi=<PI>&instr=<method>&mid=<MID>&sig=<compact JWS>
```

Example: `https://pay.eqrstandard.com/1/m/PCD?pi=PCD&instr=SCTI&mid=EE-PCD-48213907&sig=eyJ…`

| Part | Required | Meaning |
|---|---|---|
| host | yes | A payer host listed in the Directory's `payer_hosts`. Today the only one is `pay.eqrstandard.com`. An operator never uses a host of its own in a code |
| `/1/` | yes | Version of the code format (s. 4) |
| `/m/` | yes | Payment at the point of sale; the only type in this version |
| `<OPID>` | yes | The operator; equals the OPID inside the MID |
| `pi` | yes | Payload issuer |
| `instr` | yes | Payment method (s. 5) |
| `mid` | yes | Merchant ID |
| `sig` | yes | The signed part (s. 3.1) |
| `amt`, `ccy`, `rmt` | no | Amount in minor units, currency, payment description, as in Spec s. 8.1. Suggestions only: the payer page prefills them and the payer may change them. A signed value always takes precedence |

The other parameters of Spec s. 8.1 may be present. A reader ignores a parameter it does not know. A resolver ignores `sig`.

### 3.1. The signed part

`sig` is a compact JWS. Header: `{"alg":"ES256","kid":"<kid>"}`, signature in raw `r||s` form. Payload:

| Claim | Required | Meaning |
|---|---|---|
| `ver` | yes | Number. Equals the version in the path |
| `instr` | yes | Equals `instr` in the query |
| `mid` | yes | Equals `mid` in the query |
| `name` | yes | Verified receiver name, shown to the payer and passed to the payer's payment service provider |
| `iban` | yes for `SCTI` | Receiver account, uppercase, no spaces |
| `amt` | no | Integer, minor units (Spec s. 8.4); present means the amount is fixed |
| `ccy` | no | `EUR` when absent |
| `rmt` | no | Payment description, 1–140 characters |
| `ref` | no | Structured creditor reference, 1–35 characters; in Estonia the reference number, 2–20 digits with a 7-3-1 check digit. Signed only, never taken from the query |
| `iat`, `exp` | no | Unix seconds; a past `exp` means expired |

A verifier ignores a claim it does not know. Name and IBAN are signed into the code: a change of either needs a new code, and a change of IBAN a new MID (Rules s. 3.2).

The signed part is encoded, not encrypted: whoever scans the code can read the name and the account. Specification v0.1 s. 7.4 requires account details at the point of sale to be obtained by resolution. This Profile permits them inside the operator's signature, because the payer page must verify and pay with no registry call, and the payee is a business whose account every payer sees in the transfer. Spec s. 7.4 continues to apply to a code without `sig`.

## 4. Version

The version is `1`. It is raised only when a reader built for the earlier version would misread the code. Additions that an older reader can safely ignore do not raise it.

## 5. Payment methods

| Code | Method | Signed part must carry | The payer's side |
|---|---|---|---|
| `SCTI` | SEPA credit transfer in euro to the signed account, sent as an instant transfer (SCT Inst) | `iban` | The payer confirms the transfer with their own payment service provider |

A code carries one method. The Association adds a method by a new version of this Profile. A method code is never reused for another meaning.

## 6. The Operator Directory

Published by the Association at `https://eqrstandard.com/.well-known/eqr/operators.json` and signed with its directory key, whose public half is on the Specification page for verifiers to pin. No operator holds the directory key. The Directory carries:

- `payer_hosts`: the hosts a code may use (s. 3);
- per operator: `opid`, `name`, `country`, `status`, `signing_keys` (kid and public JWK), and `registry`, the base address of its endpoints (s. 7);
- `valid_until`.

## 7. Operator endpoints

Under the operator's `registry` address:

- `GET /.well-known/eqr/jwks.json`: the operator's active keys. A convenience copy: a verifier trusts a key only if it is also in the signed Directory.
- `GET /.well-known/eqr/revoked.json`: `{"updated_at": "<RFC 3339>", "revoked": ["<MID>", …]}`, every MID that must not be paid, including pending, suspended, revoked and transferred ones. Updated within one hour of a change. `Cache-Control: public, max-age=3600`.
- `POST /1/m/<OPID>`: the resolver of Spec s. 9.
- `POST /1/log`: scan counts from the payer page (s. 9).

A payer host answers a resolver call at the code's own address by forwarding it unchanged to the resolver of the operator named in the path.

## 8. What a verifier does

1. Fetch the Directory (s. 6); verify its signature with the pinned directory key; reject it if past `valid_until`. Refresh at least every 24 hours.
2. Check that the code's host is in `payer_hosts`.
3. Take the OPID from the path; find that operator with `status: active`; check that the MID carries the same OPID.
4. Check that the JWS `kid` is one of that operator's `signing_keys` and verify the signature.
5. Check `ver`, `instr` and `mid` in the signed part against the path and the query; refuse the code if any differs. Check `exp` against the clock.
6. If the version or the method is not one the verifier implements, tell the payer the code needs a newer application and offer no payment.
7. Fetch that operator's revocation list at most once an hour; refuse a listed MID.
8. Show the signed `name`, the tail of the account, the amount, the payment method and the operator's name; only then hand the payment to the payer's payment service provider. Never follow a redirect to obtain payee data.

## 9. The payer page and payer privacy

The Association runs the payer host, or has it run on its behalf under a written agreement. It holds no merchant data, gives no operator preference, and works the same for every operator in the Directory.

It stores no payer data: no IP address, no user agent, no account details. It may count scan events per MID without any payer identifier and sends them only to the operator named in the code, at that operator's `/1/log`.

## 10. Earlier versions

Codes in the forms of Profile v0.1 and v0.2 were never issued to merchants in the field and are not valid. A verifier refuses them.

## 11. For Specification v0.2

Fold this Profile into the Specification; allow hyphens in the MID (v0.1 s. 6.3 says alphanumeric); say "certified" where v0.1 says "accredited"; state the revocation list for both carriers; reconcile s. 7.4 with s. 3.1 of this Profile.
