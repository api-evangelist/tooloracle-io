---
generated: '2026-09-19'
method: generated
name: Buy one compliance or safety verdict with x402 v2
description: Quote, pay for and verify a single priced /v2 call (sanctions screen, MiCA stablecoin check, CVE lookup, agent preflight) using the x402 v2 challenge/response flow in USDC on Base, with the provider's own safety latches.
api: openapi/tooloracle-io-x402-v2-openapi.yml
operations: [sanctionsScreen, micaStablecoinCheck, cveLookup, agentPreflight, uvoQuick]
source: >-
  Grounded in openapi/tooloracle-io-x402-v2-openapi.yml (every operationId above exists verbatim, each with 200/402/400 responses and an
  x-x402 price block), the live 402 observed on POST /v2/cve_lookup on 2026-09-20, https://tooloracle.io/docs/x402-buyer-quickstart/ and
  https://tooloracle.io/.well-known/x402.
---

# Buy one compliance or safety verdict with x402 v2

The /v2 routes are account-less: no API key, no OAuth. Payment IS the authentication, one call at a time.

## Auth
- `PAYMENT-SIGNATURE` header carrying an EIP-3009 `transferWithAuthorization` for USDC on Base mainnet (`eip155:8453`, asset `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`). The legacy `X-PAYMENT` header is rejected on /v2.
- Wallet needs only USDC, no ETH — the facilitator (`https://api.cdp.coinbase.com/platform/v2/x402`) submits the transfer.

## Steps
1. **Choose the operation and read its price** from the spec's `x-x402` block or `/.well-known/x402` (e.g. `cveLookup` $0.005, `sanctionsScreen` $0.02, `micaStablecoinCheck` $0.01, `agentPreflight` $0.005). Treat these as advisory only.
2. **Quote (free)** — `POST https://tooloracle.io/v2/sanctions_screen` (`sanctionsScreen`) with the request body from the operation's example (`{"name": "...", "country": "..."}`) and NO payment. Expect HTTP 402 with an empty body and a base64 `PAYMENT-REQUIRED` header; decode it and read `accepts[0]` — `scheme exact`, `network eip155:8453`, `amount` in atomic USDC (6 decimals), `payTo`, `maxTimeoutSeconds 300`. **This is the authoritative price.** The tool did not execute.
3. **Validate before signing**: `x402Version == 2`, `scheme == exact`, `network == eip155:8453`, asset and `payTo` match the manifest, `amount` is at or under your ceiling. On any mismatch sign nothing.
4. **Pay once** — sign the authorization for exactly `accepts[0]` from THIS challenge (never a cached one) and retry the same POST with `PAYMENT-SIGNATURE`. Send at most one paid retry; latch so a timeout never re-signs.
5. **Read the result** — HTTP 200 with the operation's `*Output` schema (`hit`, `matched_lists`, `confidence` for `sanctionsScreen`; verdict + grade for `micaStablecoinCheck`; `vulnerabilities[]` for `cveLookup`; GO/CAUTION/STOP for `agentPreflight`). The `PAYMENT-RESPONSE` header carries the settlement and on-chain tx.
6. **Verify the receipt** — paid responses embed `chain.receipt.verification`; recompute the digest over the canonical JSON of `chain.receipt.document.receipt` and check the Ed25519 signature against `https://feedoracle.io/.well-known/nomos-execution-jwks.json` (`receipt_verify.mjs` in the quickstart does exactly this).

## Idempotency and reversibility
- The EIP-3009 nonce makes the PAYMENT unrepeatable, but a second signed POST is a second purchase. There is no `Idempotency-Key`; see `conventions/tooloracle-io-conventions.yml#idempotency`.
- Payments are irreversible and non-refundable once the call executed (Terms §3, §13). Do not retry after `unexpected_settle_error` or `settlement_pending` until the on-chain state is known.

## Errors
- 400 `{error, code, message}` — fix the body against the `*Input` schema.
- 402 codes in the fresh `PAYMENT-REQUIRED.error`: `insufficient_funds`, `invalid_payload`, `invalid_scheme`, `invalid_network`, `invalid_x402_version`, `invalid_payment_requirements` (always copy `accepts[0]` from the fresh challenge), `unexpected_verify_error` (manual retry only).
- 500 `application/problem+json` — no payment was taken; retry later with a fresh challenge. Full table: `errors/tooloracle-io-problem-types.yml`.
