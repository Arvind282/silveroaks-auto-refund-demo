# Reversal Contract — DRAFT (Payments ↔ Ledger & Settlement)

> Status: **DRAFT** — the "Open questions" below are deliberately unresolved. In the demo, Act 1's two-model debate surfaces them and Act 2's Ledger & Settlement teammate answers them live.

## Context
SilverOaks Fintech runs a high-volume UPI rail. A meaningful fraction of debits fail *after* the customer's account is debited — the money left, but the beneficiary credit or the NPCI settlement did not complete. Today the customer must raise a complaint and wait; we want to **auto-reverse**.

Two teams touch this:
- **UPI / Payments (owner of this repo):** operates the UPI switch integration, **detects** the failed transaction, and decides a reversal is **owed**. Emits `upi_txn_failed`.
- **Ledger & Settlement / Treasury:** **moves the money**, owns the double-entry ledger and reconciliation with the NPCI switch, and is the **source of truth** for the reversal record.

## The event (Payments → Ledger)
Payments emits `upi_txn_failed` — see [`../schemas/upi_txn_failed.example.json`](../schemas/upi_txn_failed.example.json). Minimum fields: `txn_id`, `rrn` (NPCI Retrieval Reference Number), `amount_paise`, `payer_vpa`, `payee_vpa`, `failure_reason`, `debit_status`, `credit_status`, `failed_at`.

## What's agreed
- A failed-after-debit UPI txn should be reversed to the payer **without a customer complaint**.
- We must stay inside the **RBI Harmonisation-of-TAT** window (auto-reversal by **T+1**; else ₹100/day to the customer).
- The reversal must be **idempotent** — the customer is credited back **exactly once**.

## Open questions for the Ledger & Settlement team  ⟵ *Act 2 hook*
1. **Idempotency key.** Do we key the reversal on **`rrn`**, `txn_id`, or a composite? NPCI's own auto-reversal (URCS) may also fire — how do we dedup so we don't credit twice?
2. **Reversal instrument.** Credit-**reversal** of the original leg vs. a **fresh credit** — which, and does it change the reconciliation/ledger entries?
3. **Late settlement race.** A "failed" txn can **settle late** at the beneficiary bank. Do we hold for **one reconciliation cycle** before reversing? What's the hold window?
4. **Loss ownership.** If the beneficiary bank later confirms the credit landed *and* we've reversed, who eats the loss and how do we claw back?
5. **Fraud dedup.** How do we guard against a payer who both gets our auto-reversal **and** files an NPCI chargeback for the same RRN?
6. **Source of truth & partials.** Ledger owns the reversal record; how are partial/duplicate-debit cases represented? What's the state machine (`owed → held → reversed → reconciled`)?
7. **TAT clock.** Does the T+1 clock start at `failed_at`, at switch-decline receipt, or at settlement-file time?

## Acceptance (to be finalized after Act 2)
- Given a `upi_txn_failed`, the service produces **at most one** reversal per `rrn`.
- A reversal that later collides with a late settlement or an NPCI URCS reversal is **detected and not double-applied**.
- Every decision is written to the reversal ledger with a state and an audit trail.
