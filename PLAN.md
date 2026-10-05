# PLAN — promptpay-go

Status: **M0 scaffolded** · Owner: @Chawn · Last updated: 2026-10-06

## Goal
A correct, well-tested Go library to **generate and parse** Thai QR Payment payloads
(EMVCo Merchant-Presented Mode): PromptPay credit transfer (phone / national ID / tax ID /
e-wallet) and bill payment (Tag 30). Plus an optional QR-image sub-package. Every Thai
e-commerce / POS / invoicing backend in Go needs this.

## Non-goals
- Talking to bank APIs / payment confirmation (slip verification is a different product).
- Storing or logging account identifiers.

## Payload spec summary (verify against the Bank of Thailand Thai QR Code standard doc)
TLV: 2-digit tag, 2-digit length, value.

| Tag | Meaning | Value |
|---|---|---|
| 00 | Payload format indicator | `01` |
| 01 | Point of initiation | `11` static (no amount), `12` dynamic (has amount) |
| 29 | Merchant account — PromptPay credit transfer | nested TLV below |
| 29.00 | AID | `A000000677010111` |
| 29.01 | Mobile | `0066` + number without leading 0 → 13 chars (`0812345678` → `0066812345678`) |
| 29.02 | National ID / Tax ID | 13 digits |
| 29.03 | e-Wallet ID | 15 digits |
| 30 | Merchant account — bill payment | nested: 00 AID `A000000677010112`, 01 Biller ID (tax ID + suffix, 15), 02 Ref1, 03 Ref2 |
| 53 | Currency | `764` (THB) |
| 54 | Amount | e.g. `100.00` — only when dynamic |
| 58 | Country | `TH` |
| 59/60 | Merchant name / city | optional |
| 62 | Additional data | optional (e.g. 62.05 reference label, 62.07 terminal label) |
| 63 | CRC | CRC-16/CCITT-FALSE (poly 0x1021, init 0xFFFF), over the whole payload **including** the literal `6304`, uppercase hex, 4 chars |

Tag order in output must match the reference vectors (`00,01,29,58,53,54,63`) so
outputs are byte-identical to the widely used npm `promptpay-qr`.

## API design (v0.1.0)
```go
package promptpay

type Target struct{ Kind TargetKind; ID string } // Mobile | NationalID | EWallet
func ParseTarget(s string) (Target, error)        // infer kind from length after normalization (10→mobile, 13→ID, 15→e-wallet)

type Options struct {
    HasAmount    bool     // false → static QR (tag 01 = 11)
    AmountSatang int64    // 10050 = 100.50 THB. No float in the core API.
    MerchantName string
    City         string
    Reference    string   // 62.05
}
func Payload(t Target, opt Options) (string, error)
func PayloadBill(billerID, ref1, ref2 string, amountSatang int64) (string, error)

type Decoded struct { Static bool; Target Target; AmountSatang int64; HasAmount bool; Bill *Bill; Raw map[string]string }
func Decode(payload string) (*Decoded, error)   // validates CRC, lengths, nesting; ErrCRC, ErrMalformed
func CRC16(data []byte) uint16

// sub-package promptpay/qrimage (separate go.mod? decide in M4 — keep core zero-dep)
func PNG(payload string, size int) ([]byte, error)
func SVG(payload string) (string, error)
```
Convenience: `promptpay.Amount(100.5)` → satang helper for callers who insist on float.

## Test vectors
`testdata/promptpay_vectors.json` — generated from npm `promptpay-qr@0.5.0`. Include all.
Also: round-trip property test (`Decode(Payload(x)) == x`) and a fuzz test on `Decode`
(must never panic). Add bill-payment vectors by generating them with a second
independent implementation (e.g. a Python lib) — document provenance.

## Milestones
- [x] **M0 — Scaffold**
- [ ] **M1 — TLV encoder + CRC16** with known-answer tests (`CRC16("123456789") == 0x29B1`).
- [ ] **M2 — Credit-transfer payload** — all vectors byte-identical.
- [ ] **M3 — Decoder** + fuzz + round-trip.
- [ ] **M4 — Bill payment (Tag 30)** + additional data (Tag 62).
- [ ] **M5 — qrimage** sub-module (`github.com/Chawn/promptpay-go/qrimage`, own go.mod so the core stays zero-dep).
- [ ] **M6 — CLI** `cmd/promptpay`: `promptpay 0812345678 --amount 100 --png out.png`, `promptpay decode <payload>`.
- [ ] **M7 — Release v0.1.0**, pkg.go.dev examples, awesome-go PR.

## Definition of done
Same as thaiutils-go: vet, race tests, ≥ 90% coverage, lint clean, README example, PLAN ticked.

## Open questions
- Confirm the official BOT spec version and whether 29.01 must always be `0066…` (vs. international formats).
- Whether to expose a float-based API at all.
