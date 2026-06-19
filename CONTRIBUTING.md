# Contributing to C-Address Onboarding Bridge

Thanks for your interest in contributing. This project is part of the Stellar Wave program, and contributions are welcome when they are scoped, tested, documented, and easy to review.

## Quick Start

```bash
git clone https://github.com/C-Address-Onboarding-Bridge/C-Address-Onboarding-Bridge-Backend.git
cd C-Address-Onboarding-Bridge-Backend
npm install
cp .env.example .env
npm run build
npm run test --workspaces
```

## Contribution Flow

1. Pick an open issue or open a proposal before starting broad work.
2. Comment on the issue so maintainers and other contributors can see that work is in progress.
3. Fork the repository and create a focused feature branch.
4. Keep the change as small as practical and avoid unrelated formatting or dependency churn.
5. Add or update tests for behavior changes.
6. Update documentation when API, SDK, contract, configuration, or operational behavior changes.
7. Run the relevant validation commands before opening a pull request.
8. Open a pull request with a clear summary, validation notes, and linked issue.

## Branch Naming

Use short, descriptive branch names:

- feat/<issue-number>-short-topic for new functionality.
- fix/<issue-number>-short-topic for bug fixes.
- docs/<issue-number>-short-topic for documentation-only changes.
- test/<issue-number>-short-topic for test-only improvements.
- chore/<short-topic> for maintenance that has no user-facing behavior change.

Examples:

- feat/66-positive-amount-guard
- docs/92-troubleshooting-guide
- fix/health-soroban-rpc-check

## Commit Messages

Use conventional commits. Keep the subject imperative and under roughly 72 characters when possible.

Accepted prefixes:

- feat: new user-facing or integration behavior.
- fix: bug fix.
- docs: documentation-only change.
- test: tests-only change.
- refactor: code structure change without behavior change.
- chore: build, dependency, or repository maintenance.
- ci: CI workflow changes.

Examples:

- feat: add Soroban RPC health check
- fix: reject zero amount bridge transfers
- docs: add wallet integration guide

## Pull Request Template

Copy this structure into PRs when the repository template is not shown automatically:

```md
## Summary
- What changed?
- Why is it needed?
- Which issue does it address?

## Type of Change
- [ ] Contract
- [ ] API server
- [ ] SDK
- [ ] CEX/off-ramp integration
- [ ] Documentation
- [ ] Tests or tooling

## Validation
- [ ] npm run build
- [ ] npm run test --workspaces
- [ ] Relevant Rust/Soroban tests
- [ ] Manual verification notes added below

## Security and Operations
- [ ] No secrets, API keys, private keys, seed phrases, or provider credentials are committed
- [ ] Logs mask credentials and personal data
- [ ] Error handling covers expected failure paths
- [ ] API or contract behavior changes are documented

## Notes for Reviewers
Add screenshots, logs, transaction hashes, testnet contract IDs, or known limitations here.
```

## Code Review Checklist

Reviewers and authors should check the following before merge:

- TypeScript follows strict mode expectations and avoids unnecessary any types.
- Rust contract code preserves no_std compatibility where required.
- Privileged contract paths use appropriate authorization checks.
- New behavior has unit, integration, or contract tests at the right level.
- Error handling covers invalid inputs, external provider failures, network timeouts, and retry/idempotency concerns.
- No console.log, dbg!, temporary comments, fixture secrets, or debug-only code remains.
- API request and response changes are documented and validated with schemas where applicable.
- SDK public methods have JSDoc when they are intended for integrators.
- Migration scripts or ADRs are included for architecture, storage, or deployment changes.
- Logs and support diagnostics mask API keys, webhook secrets, private keys, and personal data.
- Documentation, examples, and README references stay consistent with the code.

## Code Style

### TypeScript

- Keep strict typing enabled.
- Prefer explicit domain types for bridge requests, quote responses, provider events, and SDK public APIs.
- Validate external input at the API boundary with Zod or an equivalent schema already used by the codebase.
- Keep provider-specific code behind adapter boundaries instead of leaking provider details into generic client code.
- Avoid broad catch blocks that discard error context; preserve correlation IDs and safe diagnostic metadata.

### Rust and Soroban

- Preserve no_std contract constraints where applicable.
- Use integer math for token amounts and fees.
- Validate positive amounts, supported assets, authorization, and destination identifiers before state changes.
- Keep storage keys, TTL behavior, and emitted events deliberate and documented.
- Add focused tests for contract invariants and failure paths.

## Testing Expectations

Choose the narrowest test level that proves the behavior, then add broader coverage when cross-module behavior changes.

- Unit tests: pure helpers, validation, fee calculations, SDK request builders, and provider error normalization.
- Integration tests: API routes, provider adapters, SDK/API interactions, webhook handling, and idempotency behavior.
- Contract tests: authorization, transfer behavior, storage behavior, fees, event emission, and failure paths.
- End-to-end or manual verification: wallet flows, provider redirects, testnet transaction submission, and webhook callbacks.

Run the full workspace validation before large PRs:

```bash
npm run build
npm run test --workspaces
```

If a validation command cannot be run, state why in the PR and include the strongest available substitute evidence.

## Documentation Requirements

Documentation must be updated when a change affects:

- Public API routes, request fields, response fields, or error codes.
- SDK method signatures, examples, or expected runtime behavior.
- Contract methods, authorization requirements, fees, supported assets, events, or storage.
- Environment variables, deployment steps, provider setup, webhook setup, or operational runbooks.
- Architecture decisions, trust boundaries, or security assumptions.

Use ADRs for durable architecture decisions and guides under docs/ for integration, troubleshooting, security, and operations content.

## Review Turnaround and Escalation

Maintainers and contributors should aim for practical review loops:

- Small documentation or test PRs: first review within 2 business days when maintainers are available.
- API, SDK, or provider changes: first review within 3 business days.
- Contract, security, or architecture changes: allow extra time for deeper review.

If a PR is stuck:

1. Check whether requested changes are still pending from the author.
2. Rebase or update the branch if it has merge conflicts.
3. Leave a concise comment summarizing what is ready for review and what remains uncertain.
4. If there is no response after several business days, ping the issue or PR once with a clear status update.
5. For security-sensitive issues, avoid public details and follow the security reporting process instead.

## Good First Issues

The following areas are good entry points for Wave contributors:

### Soroban Contract

- Add amount > 0 guards to fund_c_address in contracts/onboarding-bridge/src/lib.rs.
- Add Stellar asset contract integration tests that deploy the bridge, mint tokens, and call fund_c_address end to end.

### API Server

- Add a Soroban RPC health check to the /health endpoint.
- Add a short TTL cache for deterministic GET /api/v1/quote responses.

### TypeScript SDK

- Add pagination helpers to BridgeClient.
- Add bounded retry logic with exponential backoff for transient network failures.

### Documentation

- Add JSDoc comments for public BridgeClient methods.
- Write wallet integration guides and troubleshooting examples.

## Baseline Standards

- TypeScript: strict mode, no unnecessary any types, and schema validation for external input.
- Rust: no_std where required and authorization checks for privileged functions.
- Tests: every new feature or bug fix should include relevant tests or a clear explanation of why tests are not applicable.
- API: request validation and documented error behavior are required for public routes.
- Commits: use conventional commit prefixes such as feat:, fix:, docs:, and test:.

## Need Help?

Join the Stellar Wave Discord server and ask in the #c-address-bridge channel. Include the issue link, what you tried, and any non-secret logs or error messages that explain where you are stuck.
