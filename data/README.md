# Supporting data — `arv_catalog.silveroaks` (E2)

*Optional, but it grounds the story:* the demo is driven live in the Omnigent web UI, but if a customer asks "what does the real data look like?", these synthetic tables live in the **E2** workspace under **`arv_catalog.silveroaks`**. All data is **fictional** (fictional company "SilverOaks Fintech").

| Table | Rows | What it is |
|---|---|---|
| `upi_transactions` | 6,000 | UPI transactions. **622** failed *after* debit (money left, credit/settlement didn't complete) — the auto-reversal candidates. **32** later `SETTLED_LATE` (the race the debate surfaces). |
| `upi_reversals` | 327 | The reversal **ledger / source of truth**. One row per reversal, deduped on **`rrn`**. **17** in `HELD` state (awaiting reconciliation of a late settlement); **7** where NPCI's URCS auto-reversal *also* fired (the double-credit risk). |
| `reversal_policy` | 5 | Reference rules: RBI TAT = 24h, ₹100/day penalty, idempotency key = `rrn`, 90-min reconciliation hold, URCS dedup on. |

### Failure-reason mix (of the 622 failed-after-debit)
| reason | count | NPCI code |
|---|---|---|
| `BENEFICIARY_CREDIT_TIMEOUT` | 337 | U69 |
| `SWITCH_DECLINE` | 167 | U30 |
| `DEEMED_APPROVED_TIMEOUT` | 100 | U68 |
| `BENEFICIARY_BANK_DECLINE` | 18 | U66 |

### Handy queries
```sql
-- Reversal candidates still exposed to an RBI TAT breach
SELECT txn_id, rrn, amount_paise, failure_reason, settlement_status, tat_deadline
FROM arv_catalog.silveroaks.upi_transactions
WHERE credit_status = 'FAILED' AND settlement_status = 'UNSETTLED'
ORDER BY tat_deadline;

-- The idempotency / double-credit risk: reversal fired AND late settlement, or URCS also fired
SELECT r.rrn, r.reversal_state, r.npci_urcs_also_fired, t.settlement_status
FROM arv_catalog.silveroaks.upi_reversals r
JOIN arv_catalog.silveroaks.upi_transactions t USING (rrn)
WHERE t.settlement_status = 'SETTLED_LATE' OR r.npci_urcs_also_fired;
```

> Regenerate anytime from `scripts/silveroaks_data.sql` in the SA's scratch (not shipped in this public repo — synthetic generator only).
