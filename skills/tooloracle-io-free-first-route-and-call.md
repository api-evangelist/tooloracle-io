---
generated: '2026-09-19'
method: generated
name: Route an intent through OracleNet and call the tool (free-first)
description: Classify a natural-language need with the free handshake, discover the recommended MCP endpoint, list its tools, and call a free tool — paying nothing and never guessing an endpoint.
api: mcp/tooloracle-io-mcp.yml
operations: [researchMcp, micaMcp]
endpoints_without_operationId: [POST /quantum/mcp, POST /xrpl/mcp/]
mcp_tools: [quantum_intent, quantum_route, quantum_execute, quantum_status, health_check]
source: >-
  Grounded in the provider's own free-first flow (skills/tooloracle-io-oraclenet-mesh-SKILL.md), POST /handshake observed live,
  tools/list on https://tooloracle.io/quantum/mcp/ (mcp/tooloracle-io-tools.json), and the endpoint operations in
  openapi/tooloracle-io-mcp-platform-openapi.yml. `researchMcp` and `micaMcp` exist verbatim in that spec; POST /quantum/mcp and POST /xrpl/mcp/ are in the spec but carry no operationId.
---

# Route an intent through OracleNet and call the tool (free-first)

OracleNet is a router, not a single API: the right first call is never a tool, it is the free handshake that tells you which of ~90 MCP endpoints holds the capability and what it costs.

## Auth
- None for this whole flow. Discovery, handshake, `tools/list` and the free tools run anonymously (20 requests/day per the RFC 9728 document; see `rate-limits/tooloracle-io-rate-limits.yml`).
- Optional: an `X-API-Key` from the `kya_register` tool (on `https://feedoracle.io/mcp`) or an OAuth 2.1 bearer from `https://feedoracle.io/mcp/token` lifts the daily quota. See `authentication/tooloracle-io-authentication.yml`.

## Steps
1. **Reduce the task to one sentence** of intent.
2. **Handshake (free)** — `POST https://tooloracle.io/handshake` with `{"intent": "<sentence>"}`. The response is JSON-LD (`anp:HandshakeResponse`). Read `classification.oracle`, `classification.confidence` (re-phrase on `low`), `classification.source` (whether an LLM or a static keyword rule chose the route), and `routing.interfaces[]` — each interface carries its own `auth` (`none` or `x402-payment`).
3. **Pick the `auth: none` interface** where one is offered (e.g. `https://tooloracle.io/mica/mcp/`), or use the mesh router directly: `POST https://tooloracle.io/quantum/mcp/` with `tools/call` → `quantum_route` (ranked candidates) or `quantum_intent` (LLM parser).
4. **List the tools** — `POST <endpoint>` with `{"jsonrpc":"2.0","id":1,"method":"tools/list"}`. The `inputSchema` returned is the real contract; do not infer arguments from names. (`https://tooloracle.io/uvo/mcp/` is stateful — send `initialize` first and echo `Mcp-Session-Id`.)
5. **Call a free tool** — `tools/call` with `health_check` or the free tool the route names. Results arrive as JSON text inside `result.content[0].text` and carry a `request_id` — keep it for support.
6. **Stop before any 402.** If a `tools/call` answers HTTP 402, the tool is paid: the price is in `denial.required_amount_usd` (v1 routes) or the `PAYMENT-REQUIRED` header (v2 routes). Do not pay without an explicit budget and consent from the calling principal — this is the provider's own rule.
7. **Return the result with provenance**: the oracle name, endpoint, `request_id`, and whether the response was signed (most are not — `verification-policy.json`).

## Errors
- JSON-RPC `-32600 Missing session ID` → the server is stateful; run `initialize`. `-32001 unauthorized` → the server (DealOracle) needs a credential even to list tools. HTTP 429 (nginx HTML, no `Retry-After`) → serialize requests. Full catalog: `errors/tooloracle-io-problem-types.yml`.

## Notes
- Counts in the discovery documents (servers, tools) are live snapshots and disagree with each other; trust `tools/list`, not the marketing numbers.
- Never contact anything listed in `https://tooloracle.io/.well-known/do-not-contact.json`.
