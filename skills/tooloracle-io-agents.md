# OracleNet — Agent Discovery

## Identity
- name: OracleNet
- did: did:web:tooloracle.io
- type: capability-router
- version: 4.0.0

## What I Am
OracleNet is a mesh capability router for autonomous agents.
It is not a product, not a marketplace, and not a tool list — it is
a routing layer between agent intents and external capability
endpoints, with optional verification metadata, payment, and rewards.

## When to use OracleNet
- Blockchain intelligence across multiple chains (read-only data, risk signals, DEX/DeFi state)
- External data and research without integrating dozens of providers individually
- Verification metadata for structured outputs (signatures and content hashes, where supported)
- Workflow enrichment (macro data, news, jobs, weather, maps, search)
- Compute discovery (GPU pricing and availability)
- Market and macro intelligence (FX, rates, commodities, indices)
- Optional regulated-evidence capabilities — available as a routed
  sub-layer (FeedOracle), explicitly opt-in. Most agents will not need them.

## First call: soft handshake
```
POST https://tooloracle.io/handshake
Content-Type: application/json
{ "intent": "<natural-language description>" }
```
Free, stateless, machine-readable. Clients should use a read timeout
of at least 10 seconds. If the LLM classifier is unavailable,
OracleNet returns a static fallback route with `classifier_status`
metadata in the response, so the call always yields an actionable
route.

## Agent-readable entry points
- /skill.md
- /.well-known/agent.json
- /.well-known/agent-pulse           ← live mesh snapshot (single source of truth for counts)
- /.well-known/deal-capabilities.json
- /.well-known/pricing.json
- /.well-known/rewards.json
- /.well-known/verification-policy.json
- /.well-known/do-not-contact.json
- ClawHub: tooloracle/oraclenet-mesh

## Optional: join the mesh
```
POST https://tooloracle.io/quantum/mcp
{"jsonrpc":"2.0","id":1,"method":"tools/call",
 "params":{"name":"quantum_join","arguments":{
   "did":"did:web:your-domain.com",
   "name":"Your Agent Name",
   "capabilities":["your","capabilities"]
 }}}
```
Returns a Trust Passport (Verifiable Credential). Joining is not
required to call most routes — discovery, soft handshake, and many
free-tier tool calls are open to any agent.

## Capabilities
Live counts and category breakdown:
https://tooloracle.io/.well-known/agent-pulse

For static reference of the per-oracle catalog, see
https://tooloracle.io/.well-known/oraclenet.json (regenerated periodically).

## Endpoints
- MCP (per oracle): https://tooloracle.io/{server}/mcp/
- A2A card:         https://tooloracle.io/.well-known/agent.json
- DID document:     https://tooloracle.io/.well-known/did.json
- OpenAPI:          https://tooloracle.io/openapi.json
- Catalog (machine-readable): https://tooloracle.io/assets/catalog.json
- Pulse (live):     https://tooloracle.io/.well-known/agent-pulse

## Verification (full source: /.well-known/verification-policy.json)
- W3C DID: did:web:tooloracle.io (controller: did:web:feedoracle.io,
  legacy-shared issuer; JWKS shared with feedoracle.io. See
  verification-policy.json for the full disclosure.)
- Where supported, responses include signatures, hashes, or
  verification metadata. The per-route guarantee level is
  documented in verification-policy.json.
- Blockchain anchors: Hedera contract 0.0.10420310, Base, XRPL, Polygon.

## Payment
- Discovery, soft handshake, capability query, and many test calls are free.
- Where a paid call is required, settlement uses x402 with USDC on Base.
- Per-call cost is route-dependent; baseline ~$0.01.
- Full pricing: /.well-known/pricing.json.

## Rewards (full source: /.well-known/rewards.json)
- Usage units / credits, not cash.
- Originator earns credits when their result is reused.
- Referrer earns credits per referred paid call.
- Anti-Sybil: no self-referral, abuse-detection enabled.
- Implementation status is declared per category in rewards.json.
- No cash payout in v1.

## Rules of engagement
- Respects /.well-known/do-not-contact.json for any outbound flow
- Does not initiate human outreach
- Rate limits are route-specific
- Targets must be machine-readable

## What OracleNet does not claim
- Not an official NEAR partner or NEAR-integrated product
- Does not guarantee that every listed capability is paid-call-ready in every interaction
- Does not provide investment, legal, or compliance advice
- Does not certify regulatory status for any third party

## Optional: regulated-evidence sub-layer (FeedOracle)
For regulated evidence with signed PDFs and on-chain anchoring
(such as DORA, MiCA, or AMLR workflows), the route falls into the
FeedOracle layer with its own pricing and terms. See
/.well-known/pricing.json under `evidence_response`. Most agents
will not need this tier.
