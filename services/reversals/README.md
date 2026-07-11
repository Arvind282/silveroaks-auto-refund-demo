# services/reversals/ — (intentionally empty)

This is where the **Act 3** build lands. In the demo, Polly / Claude Code implements the UPI Auto-Reversal MVP here:

- consume `upi_txn_failed` (see `../../schemas/upi_txn_failed.example.json`)
- an **idempotent** reversal decision keyed on **`rrn`** (credit-reversal vs. fresh-credit)
- a **one reconciliation-cycle hold** to catch late settlement
- **NPCI / URCS dedup** so we never double-credit
- a Ledger client stub + customer notification
- tests

Leaving it empty on purpose so the agents have a clean target and the "it built into our repo" moment is real.
