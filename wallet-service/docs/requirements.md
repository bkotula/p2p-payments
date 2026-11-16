# p2p-payments: Requirements

Status: drafted 2026-11-16. Complements `payment-app-milestone-plan.md`.
Changes the plan's scope: account deletion, AML (anti-money-laundering) checks and payouts (withdrawals) are now in scope, because they are needed for a banking-sector story.

## Functional requirements

### v1
- A user can register and log in
- A user has one EUR wallet, created at registration
- A user can top up their wallet through a simulated payment provider
- A user can find another user by their handle
- A user can send money to another user
- A user can see their balance and paged transaction history
- An admin can list all users

### Account deletion (v1)
- A user can request deletion of their account
- Deletion requires an empty wallet: any remaining balance is first paid out to the user's linked bank account through a simulated provider (payout)
- Deletion is blocked while the user has pending top-ups, transfers or payouts, an open AML alert, or a frozen account
- On deletion, personal data (name, email, handle, bank account reference) is anonymized and login is disabled
- Transfers and ledger entries are kept; other users see "Deleted user" in their history
- A deleted user's handle is not given to anyone else (prevents impersonation)

### Anti-money-laundering (v1, simulated)
- **Verification (KYC, simulated):** a user is UNVERIFIED or VERIFIED; an admin can mark a user as verified. Unverified users have lower limits.
- **Limits, checked before a transfer, top-up or payout:** a maximum per transaction and a maximum per rolling 24 hours, higher for verified users. Over the limit → rejected.
- **Transaction monitoring, after the transfer commits:** rules run asynchronously on transfer, top-up and payout events and create an alert when one matches:
  - a single large amount
  - many transfers in a short time (velocity)
  - several amounts just below a limit (structuring)
  - money topped up and sent on almost immediately (pass-through)
- **Review:** an admin can list alerts, close them, and freeze or unfreeze an account. A frozen account can't send, top up or pay out, but can still log in and see its history.
- **Audit log:** admin actions (verify, freeze, unfreeze, close alert) are recorded with who, what and when. Insert-only.
- **Record retention:** transaction records and the audit log are kept for at least 5 years (the usual EU requirement), including after account deletion.

### Later
- Multiple currencies: one wallet per currency, convert before sending, rate locked by a short-lived quote

## Business rules (invariants)
- A wallet balance can never go below zero
- A transfer debits and credits in one transaction, so money is never created or lost
- All ledger entries sum to zero; each wallet's balance equals the sum of its entries
- Ledger entries and audit log entries are never updated or deleted

## Non-functional requirements

### What the app guarantees
- **Consistency over availability** for money movement: when the current balance can't be confirmed, the request is rejected
- **Idempotency:** a retried request (double-click, network retry) never moves money twice
- **Durability:** a committed transfer and its event survive a crash
- **Security:** users can only read and spend from their own wallet; only admins can review alerts and freeze accounts
- **Eventual consistency is acceptable** for side effects: notifications and AML monitoring alerts may arrive seconds after the transfer
- **Privacy:** personal data can be erased without breaking financial records

### What a production system would also need (not built)
- Scale: the app targets fewer than 100 users, plus load tests to measure throughput
- High availability, latency targets, horizontal scaling
- Real KYC (identity documents) and sanctions-list screening
