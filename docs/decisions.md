# Decisions

Short notes on what was chosen, what was rejected, and why. Newest first.

## 2026-11-16 · Scope extended: account deletion, payouts and AML

**Chose:** bring account deletion, payouts (withdrawals to a linked bank account) and simulated AML checks into scope. AML is split in two: limits checked synchronously inside wallet-service before money moves, and transaction monitoring in a new `aml-monitoring-service` that consumes Kafka events and raises alerts.

**Rejected:** keeping the original scope, which left withdrawals out and had no compliance features.

**Why:** a banking-sector story needs more than moving money between wallets. Deletion, payouts and AML bring up GDPR erasure vs record retention, KYC-based limits and asynchronous monitoring, all of which come up in fintech interviews. Details in [wallet-service/docs/requirements.md](../wallet-service/docs/requirements.md).
