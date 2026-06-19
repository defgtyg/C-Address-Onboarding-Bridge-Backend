# Architecture Decision Records

This directory records backend architecture decisions for the C-Address Onboarding Bridge. ADRs are intentionally short and should explain the context behind a decision, the selected approach, alternatives considered, and consequences for maintainers and integrators.

## Review Process

1. Create a new ADR using the template below and the next sequential number.
2. Mark the ADR as Proposed while discussion is active.
3. Link related ADRs and issues so future readers can follow the decision chain.
4. Update the status to Accepted after maintainer review, or Superseded when a later ADR replaces it.
5. Keep historical ADRs in place even when decisions change; add a Superseded by link instead of deleting them.

## ADR Template

```md
# ADR-XXX: Title

Status: Proposed | Accepted | Superseded

## Context

What problem, constraint, or product requirement forced this decision?

## Decision

What approach did we choose?

## Consequences

What tradeoffs, risks, and follow-up work come from this decision?

## Alternatives Considered

What other options were evaluated, and why were they not selected?

## Related ADRs

- ADR-000: Example
```

## ADR Index

- [ADR-001: Use Soroban for smart contract execution](#adr-001-use-soroban-for-smart-contract-execution)
- [ADR-002: Keep the API server stateless initially](#adr-002-keep-the-api-server-stateless-initially)
- [ADR-003: Use a basis-point fee model](#adr-003-use-a-basis-point-fee-model)
- [ADR-004: Use a monorepo with npm workspaces](#adr-004-use-a-monorepo-with-npm-workspaces)
- [ADR-005: Expose a REST API protected by API keys](#adr-005-expose-a-rest-api-protected-by-api-keys)
- [ADR-006: Keep CEX routing pluggable](#adr-006-keep-cex-routing-pluggable)
- [ADR-007: Use Stellar and soroban-sdk v26](#adr-007-use-stellar-and-soroban-sdk-v26)
- [ADR-008: Use event-driven state updates as the path to realtime status](#adr-008-use-event-driven-state-updates-as-the-path-to-realtime-status)

## ADR-001: Use Soroban for smart contract execution

Status: Accepted

### Context

The project exists to let users fund Soroban smart accounts, also known as C-addresses, without first forcing them through a legacy G-address onboarding flow. The backend needs an execution layer that can represent smart account funding, bridge rules, and future account abstraction behavior on Stellar.

### Decision

Use Soroban smart contracts as the on-chain execution layer for onboarding bridge behavior.

### Consequences

- Contract IDs, network passphrases, XDR validation, and Soroban RPC behavior become first-class integration concerns.
- Backend and SDK code must treat contract interaction as a boundary that can fail independently from provider routing.
- Security review must cover authorization, fees, storage, and transaction construction on the contract side as well as the API side.

### Alternatives Considered

- G-address-only onboarding: simpler but does not solve the C-address adoption problem.
- Off-chain ledger state only: faster to prototype but weaker for transparent settlement and smart account composability.
- A non-Stellar chain: outside the product goal of Soroban dApp onboarding.

### Related ADRs

- ADR-002, ADR-003, ADR-007

## ADR-002: Keep the API server stateless initially

Status: Accepted

### Context

The first backend iteration focuses on routing, quote creation, transaction construction, and provider integration. Adding a database early would introduce migrations, data retention, and operational complexity before persistence requirements are stable.

### Decision

Keep the API server stateless where practical and derive state from requests, provider responses, transaction hashes, and on-chain status. Add durable storage only when a feature has a clear data ownership and retention requirement.

### Consequences

- Deployments are easier to run and scale horizontally.
- Idempotency, auditability, and historical troubleshooting are limited until persistent storage is introduced.
- Webhook and support workflows must preserve enough correlation IDs externally or in logs.

### Alternatives Considered

- Add PostgreSQL immediately: gives strong audit history but increases setup and migration burden.
- Store all bridge state on-chain: transparent but too expensive and awkward for provider and support metadata.
- Use only provider dashboards: insufficient for project-owned audit and incident response.

### Related ADRs

- ADR-005, ADR-008

## ADR-003: Use a basis-point fee model

Status: Accepted

### Context

Bridge fees need to scale with transaction value while remaining clear for users, integrators, and contract logic. Floating point fee calculations are unsafe for currency amounts and make audits harder.

### Decision

Represent percentage fees in basis points and calculate fees using integer amount units.

### Consequences

- Fee configuration is portable across API, SDK, and contract logic.
- UI and API responses must clearly show both basis points and computed fee amounts.
- Rounding rules must be deterministic and documented before production use.

### Alternatives Considered

- Flat fees only: simpler but does not scale across small and large transfers.
- Floating point percentages: easy to display but risky for settlement calculations.
- Provider-specific fee formulas only: delegates too much product behavior to third parties.

### Related ADRs

- ADR-001, ADR-006

## ADR-004: Use a monorepo with npm workspaces

Status: Accepted

### Context

The backend, SDK, provider modules, and contract-adjacent tooling evolve together. Integrators benefit when examples, shared types, and package scripts stay aligned.

### Decision

Use a monorepo layout with npm workspaces for JavaScript and TypeScript packages while keeping contract code in its own clearly named directory.

### Consequences

- Cross-package changes can be reviewed in one PR.
- Shared scripts and dependency versions reduce drift.
- CI must guard against accidental coupling and ensure package-level tests still run independently.

### Alternatives Considered

- Separate repositories: clearer ownership but higher coordination cost for SDK/API changes.
- Single package only: simpler at first but does not reflect API, SDK, and provider boundaries.
- Non-npm workspace tooling: possible later, but npm workspaces are enough for the current package set.

### Related ADRs

- ADR-005, ADR-006

## ADR-005: Expose a REST API protected by API keys

Status: Accepted

### Context

Wallets, dApps, and backend integrators need a predictable interface for quotes, bridge requests, provider routing, and status checks. The first integration model favors server-to-server access over end-user account management.

### Decision

Expose REST endpoints and protect privileged routes with API key authentication.

### Consequences

- REST keeps examples and SDK wrappers straightforward.
- API keys must be scoped, rotated, masked in logs, and separated by environment.
- Future user-facing dashboards may require session or OAuth-style authentication in addition to API keys.

### Alternatives Considered

- GraphQL: flexible but unnecessary for the initial command-style workflow.
- Public unauthenticated API: easier to try but unsafe for provider-backed operations.
- OAuth first: more complete for user accounts but heavier than current server-to-server needs.

### Related ADRs

- ADR-002, ADR-008

## ADR-006: Keep CEX routing pluggable

Status: Accepted

### Context

CEX, card ramp, and off-ramp providers differ in authentication, quote, callback, and settlement behavior. Hard-coding one provider would slow integrations and make outages harder to isolate.

### Decision

Implement provider routing behind pluggable modules with consistent inputs, outputs, errors, and callback handling expectations.

### Consequences

- New providers can be added without rewriting the core API contract.
- Provider-specific errors must be normalized for SDK and support use.
- Security review must validate every provider adapter's signature, replay, and idempotency behavior.

### Alternatives Considered

- Single provider integration: faster but fragile and commercially limiting.
- Fully generic provider DSL: flexible but too abstract before integration patterns are proven.
- Manual operator routing: useful as an emergency fallback but not a scalable product path.

### Related ADRs

- ADR-003, ADR-005, ADR-008

## ADR-007: Use Stellar and soroban-sdk v26

Status: Accepted

### Context

The repository targets current Soroban account and transaction flows. SDK version drift can create subtle XDR, RPC, and contract invocation incompatibilities.

### Decision

Standardize development and examples on Stellar/Soroban SDK v26-compatible behavior until a deliberate upgrade ADR supersedes this decision.

### Consequences

- Examples should mention the expected SDK version when version-specific APIs are used.
- Upgrades require regression checks for XDR encoding, transaction simulation, signing, and submission.
- Integrators using older SDKs may need compatibility notes or migration guidance.

### Alternatives Considered

- Track latest SDK immediately: provides newest features but risks unreviewed breaking changes.
- Support many SDK versions equally: convenient for integrators but costly to test.
- Pin indefinitely: stable but misses security and platform improvements.

### Related ADRs

- ADR-001, ADR-004

## ADR-008: Use event-driven state updates as the path to realtime status

Status: Proposed

### Context

Bridge operations span API requests, providers, Soroban transactions, and webhook callbacks. Users and integrators need status updates, but polling every layer can be inefficient and confusing.

### Decision

Treat provider callbacks, transaction confirmations, and internal bridge state transitions as events. Use these events to support logs and future realtime status channels such as WebSocket or server-sent events.

### Consequences

- Event naming, idempotency, and correlation IDs must be consistent before realtime delivery is exposed.
- The stateless API approach may need durable event storage when realtime status becomes a production feature.
- Support and troubleshooting docs can use the same state transition model as future user-facing status updates.

### Alternatives Considered

- Client polling only: simpler but less responsive and harder to debug.
- Provider webhooks only: misses internal and on-chain transitions.
- WebSocket first: attractive for UX but premature before event semantics are stable.

### Related ADRs

- ADR-002, ADR-005, ADR-006
