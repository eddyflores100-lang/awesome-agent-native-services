# UTA Trust API

> **"The USB-C of agent trust."**

| | |
|---|---|
| **Website** | https://uta-trust-api.eddyflores100.workers.dev |
| **Docs** | [OpenAPI spec](https://uta-trust-api.eddyflores100.workers.dev/openapi.json) · [Spec RFC + conformance](https://github.com/alicelabs-llc/universal-trust-adapter) |
| **GitHub** | https://github.com/alicelabs-llc/universal-trust-adapter |
| **Classification** | `agent-native` |
| **Category** | [Agent Runtime & Infrastructure Services](README.md) |
| **License** | Open-core — SDK [`MIT OR Apache-2.0`](https://www.npmjs.com/package/agent-trust-card); engine AL-1.0; spec CC-BY-NC-ND 4.0; [license matrix](https://marketnow.site/licensing) |

---

## Official Website

https://uta-trust-api.eddyflores100.workers.dev

---

## Official Repo

https://github.com/alicelabs-llc/universal-trust-adapter

---

## How to Use (Agent Onboarding)

**The quickest path for an agent to start using this service:**

```
# URL Onboarding:
Read https://uta-trust-api.eddyflores100.workers.dev/openapi.json and follow the instructions to settle and call.

# SDK / REST:
npm install agent-trust-card        # issue / verify / inspect Agent Trust Cards
# or call the verifier directly — an agent with no pre-paid access receives
# HTTP 402 + payment metadata, settles $0.01 USDC via x402, and retries:
POST https://uta-trust-api.eddyflores100.workers.dev/v1/trust/verify

# Conformance (self-test a credential before submitting it):
npx @marketnow/uta-conformance      # signed vectors + card generator
```

---

## Agent Skills

**Status:** ⚠️ Not yet published

```bash
npx clawhub@latest search uta trust
```

---

## MCP

**Status:** ⚠️ Not yet published

The verifier is a paid REST/x402 resource (the trust decision itself is delivered over HTTP with per-call settlement); there is no MCP transport yet. The `agent-trust-card` npm SDK and the `@marketnow/uta-conformance` CLI cover the full lifecycle from code.

---

## What It Does

UTA Trust API answers one question an agent consumer asks over HTTP: **is this automated requester trustworthy?** The caller presents an Agent Trust Card (ATC) — a self-contained, CA-signed JSON credential binding an agent's identity and capabilities to an Ed25519 public key — and the verifier returns a machine-readable verdict.

Verification is fail-closed and offline-oriented: the Ed25519 signature (RFC 8032) is checked over the JCS-canonicalized (RFC 8785) subtree (ATC-006), the CA public key is resolved from the card's own `attestation.ca_url` so no network call to the CA is required, and revocation is checked against `revocation_check_url` when `revocation_check_required: true` — an unreachable check denies rather than degrades. Verdicts are anchored as public evidence (Sigstore Rekor) so an operator can later bind "verified agent X, verdict Y at time Z" into grant records and audit trails.

The service is x402-native: settlement is $0.01 USDC per single verification (or $0.08 for a 1–10 card batch) on Base (Mainnet `eip155:8453` and Sepolia testnet `eip155:84532`), with no signup, no API key, and no human consent step — an agent that has never heard of the service can pay and call it in one round trip.

---

## Why It Is Agent-Native

| Criterion | Evidence |
|---|---|
| **Agent-first positioning** | "The USB-C of agent trust" / "UTA translates between ALL trust credential formats used by AI agents" — https://github.com/alicelabs-llc/universal-trust-adapter. There is no dashboard, no login, and no human-facing workflow; every surface (API, spec RFC, conformance corpus, SDK) is written for machine consumers. |
| **Agent-specific primitive** | The ATC itself: an Ed25519-signed agent credential over JCS canonicalization, checked by a fail-closed verifier against a 36-vector adversarial conformance corpus before a verdict is issued. The question "should I trust this non-human requester under adversarial behavior vectors?" has no human-facing equivalent — there is no account to create, no OAuth consent, no human identity. |
| **Autonomy-compatible control plane** | An agent without pre-paid access receives HTTP 402 with payment metadata and settles $0.01 USDC via x402 (Base mainnet) with no signup and no per-action human confirmation; all failure modes (unverifiable credential, malformed JCS, unreachable revocation check) deny by default. |
| **M2M integration surface** | REST API with machine-readable [OpenAPI spec](https://uta-trust-api.eddyflores100.workers.dev/openapi.json), npm SDK (`agent-trust-card`), conformance CLI (`@marketnow/uta-conformance`), x402 discovery document at the root URL. |
| **Identity / delegation** | Each verdict is bound to the presented ATC (the agent's identity); delegation is expressible in the credential itself (ATC-009 issuer→agent binding); anchored evidence provides the audit trail. |

---

## Primary Primitives

| Primitive | Description |
|---|---|
| **ATC (Agent Trust Card)** | Self-contained JSON credential binding agent identity, capabilities, and risk policy to an Ed25519 public key; verification is offline once the CA key is resolved from `attestation.ca_url`. |
| **`POST /v1/trust/verify`** | Verify one ATC: Ed25519 signature over the JCS subtree, pinned anchors, revocation check; fail-closed verdict. $0.01 USDC via x402. |
| **`POST /v1/trust/batch`** | Verify 1–10 ATCs in one settlement. $0.08 USDC via x402. |
| **Keypair-to-ATC issuance** | The agent generates its own Ed25519 keypair (RFC 8032) — the private key never leaves the agent; the CA binds identity, capabilities, and risk policy to the 32-byte subject public key and signs the canonical payload (ATC-006), producing a self-contained card. |
| **Rekor-anchored evidence** | Public ledger of conformance/verification anchors ([browse](https://www.marketnow.site/uta/conformance/anchors/)) so verdicts can be bound into audit trails after the fact. |
| **Adversarial conformance corpus** | 36 signed vectors + card generator (`npx @marketnow/uta-conformance`) for cross-language reproducibility of verifier behavior. |

---

## Autonomy Model

1. An agent generates its own Ed25519 keypair; only the 32-byte subject public key is submitted for issuance (private key never leaves the agent).
2. The CA signs the ATC binding identity, capabilities, and risk policy to that key; the agent stores the self-contained JSON card.
3. At request time, the consuming agent presents the ATC to the verifier (or an agent service presents its own card); the verifier checks signature, canonicalization, anchors, and revocation — fail-closed — and returns a machine-readable verdict.
4. Settlement is per-call via x402: HTTP 402 → settle $0.01 USDC on Base → retry with the payment header. No account, no API key, no human step anywhere in the loop.

---

## Identity and Delegation Model

- **Identity:** the ATC binds an agent identity to an Ed25519 subject key; the card is the identity artifact, not an account token.
- **Delegation:** expressible in the credential itself (ATC-009 issuer→agent binding), so a verifier can enforce who vouches for the agent and within which capability scope.
- **Audit:** every verdict is anchored (Sigstore Rekor) to a public ledger; the spec RFC covers key rotation and payTo change scenarios so long-lived grants can be reasoned about after the fact.

---

## Protocol Surface

| Interface | Detail |
|---|---|
| REST API | `POST /v1/trust/verify` (single), `POST /v1/trust/batch` (1–10 cards) — [OpenAPI](https://uta-trust-api.eddyflores100.workers.dev/openapi.json) |
| Payments | x402 (v2) over USDC on Base mainnet (`eip155:8453`) and Base Sepolia testnet; discovery document at the root URL |
| SDK | npm [`agent-trust-card`](https://www.npmjs.com/package/agent-trust-card) — `issue` / `verify` / `inspect` |
| Conformance | `npx @marketnow/uta-conformance` — signed vectors + card generator |
| Evidence ledger | Sigstore Rekor anchors — https://www.marketnow.site/uta/conformance/anchors/ |
| CA public keys | [JWK](https://marketnow.site/api/atc) (pinned trust anchors) · CRL: `/api/atc?action=revocation-list` · OCSP-style: `/api/ocsp?card_id=` |

---

## Human-in-the-Loop Support

None by design, and none required: there is no step in issuance, settlement, or verification that waits on a human. The control-plane constraints live in the credential (capabilities, risk policy, delegation scope, revocation) rather than in approval gates — an operator constrains agents by what the CA will sign and by revoking cards, not by clicking approve.

---

## Why Generic Alternatives Do Not Qualify

| Alternative | Why It Fails |
|---|---|
| **Standard mTLS / API keys** | Bind the caller to a pre-registered account and leak a shared secret per agent; they answer "is this one of my clients?" not "is this autonomous requester trustworthy under adversarial behavior?" and require a human provisioning step per agent. |
| **OAuth/OIDC consent flows** | Assume a human resource owner clicking consent; an agent-to-agent trust decision has no human principal to consent and no session to log into. |
| **Generic JWT verification libraries** | Verify a signature but leave the adversarial semantics (fail-closed revocation, canonicalization pitfalls, delegation scope, evidence anchoring) to each implementer — exactly the surface the 36-vector corpus pins down. |

---

## Use Cases

- **Agent-to-agent commerce** — an agent buying from an unknown seller can require a valid ATC before engaging, with the verdict anchored for later audit.
- **Bounty / marketplace admission** — a marketplace admits agents only when a fail-closed ATC verification passes, without operating its own PKI.
- **Grant and audit trails** — operators bind "verified agent X, verdict Y at time Z" into internal grant records via the public Rekor anchor ledger.
- **Cross-language verifier reproducibility** — implementers self-test against the signed conformance vectors before claiming ATC compatibility.
