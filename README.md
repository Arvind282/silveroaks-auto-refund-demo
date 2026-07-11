# 💸 Omnigent Demo — SilverOaks UPI Auto-Reversal (visual quick start)

A 10–15 min, customer-facing demo of **Omnigent** (Databricks' agent meta-harness) told as one story: a PM scopes a feature → two AI models from different labs pressure-test it → a teammate joins the live session → agents build it in the cloud overnight.

> **Meta wink to open or close with:** *the assistant that helped build this demo runs inside Omnigent — everything you see, it does to itself.*

- **Full talk track + exact prompts + fallbacks:** see [`DEMO_SCRIPT.md`](./DEMO_SCRIPT.md).
- **Pre-staged context repo (attach this in the demo):** this repo — `https://github.com/Arvind282/silveroaks-auto-refund-demo`
- **Scenario:** you're a PM on the **UPI / Payments** team at **SilverOaks Fintech**, shipping **Auto-Reversal** — when a UPI payment fails *after* the customer's account is debited (money left, beneficiary credit/settlement didn't complete), reverse it automatically and immediately, instead of the customer raising a complaint and waiting days for a manual refund.

Why this matters here specifically: failed-transaction reversals are **RBI-regulated** (Harmonisation of TAT — a failed UPI debit must be auto-reversed by **T+1**, or SilverOaks owes the customer **₹100/day** penalty), **high-volume**, and touch **real money** — the exact conditions where one model's blind spot is a fraud hole or a double-credit incident.

---

## Before you start
- Log into your Databricks workspace's Omnigent — **`https://<your-databricks-workspace>/omnigent`**.
- Have this context repo URL handy (above). It's public — clones into the sandbox with no setup.
- Have a teammate on standby who can open a link (Act 2).
- *(Optional, grounds the story in real data)* The supporting SilverOaks tables live in **`arv_catalog.silveroaks`** on E2 — see [`data/README.md`](./data/README.md).

---

## Step 1 — Set up the session
Pick the **Debby** agent (multi-agent debate), set the host to **Databricks Sandbox**, click **Repository** and paste this repo (`…/silveroaks-auto-refund-demo`, branch `main`). Then the opener is one line — *"Read the repo for context… pressure-test our UPI Auto-Reversal feature… flag the open questions for the Ledger & Settlement team."*

🎤 *"I'm pre-loading my team's context as a repo, and I'm using two different models to stress-test this before I write anything."*

---

## Step 2 — Two models pressure-test it (Act 1: many models)
Debby fans the question to a **Claude** head 🟠 and a **GPT** head 🔵 (different labs), then lays out both takes and where they agree/differ. Because the repo is attached, they critique your **actual** `upi_txn_failed` schema and draft contract.

🎤 *"Two labs, two blind spots — one flags the demand-side fraud (fake 'money not received' claims, auto-reversal + NPCI chargeback double-dip); the other brings the settlement-rail variance (a 'failed' txn that actually settles late → reversed AND settled) and a concrete idempotent-ledger contract. Neither alone lands here."*

---

## Step 3 — Turn it into a light PRD
Ask for a one-page PRD (~250 words). It folds the debate's findings into 3 features + metrics, and ends with **Open Questions for the Ledger & Settlement team** — your hand-off to Act 2.

> **Governance (great for customers):** the **Agent tools and policies** panel shows a live **session cost** and a deep **Add policy** catalog (cost budget that forces a model downgrade, **deny-PII** — critical, this is customer bank/VPA data — block dangerous shell commands, and more).

---

## Step 4 — Phone a friend, live (Act 2: many people)
The PRD's open questions belong to **Ledger & Settlement**, not you. Click **Share session** → flip workspace access *or* add a teammate by **User ID** at **Edit** → **Copy link**. They open the **same live session** on their device (authenticated through the workspace) and drive the agent to settle the reversal contract.

🎤 *"Same live session, on her device, with the permission I set — Read to watch, Edit to drive. The cross-team gap closes in context, not over a Slack thread."*

---

## Step 5 — Build it in the cloud, overnight (Act 3: no laptop)
Start a **new session** with **Polly** (multi-agent coding) + the same repo, and hand over the agreed PRD. Because the host is a managed **Databricks Sandbox**, the build runs **serverless**. Polly checks its worker roster and lays out a **cross-vendor** plan: **`claude_code` implements** the MVP in `services/reversals/` and opens a PR, then **`codex` (a different vendor) reviews** the diff against the acceptance contract. Close your laptop; reconnect from any device via the session URL (the Files + Shells tabs show what it built).

🎤 *"It dispatched a Claude implementer and queued a Codex reviewer — multi-vendor, on serverless compute, not my machine. Close the laptop, check back when the implementer's done."*

---

## Close
🎤 *"More brains, more people, no laptop — two labs arguing into a tight PRD, a teammate co-driving the same live session, and agents finishing the build in the cloud. Same platform — swap the model, share the session, run it serverless — no rewrite."*

---

## What's in this repo
| Path | What it is |
|---|---|
| [`DEMO_SCRIPT.md`](./DEMO_SCRIPT.md) | The full run sheet — talk track, exact prompts, governance, fallbacks |
| [`docs/reversal-contract.DRAFT.md`](./docs/reversal-contract.DRAFT.md) | The Payments↔Ledger draft contract with the **open questions** (Act 2 hook) |
| [`schemas/upi_txn_failed.example.json`](./schemas/upi_txn_failed.example.json) | The event the UPI team emits when a debit fails after money leaves |
| [`services/reversals/`](./services/reversals/) | Empty target — the agents build the MVP here in Act 3 |
| [`data/README.md`](./data/README.md) | Pointers to the supporting SilverOaks tables in `arv_catalog.silveroaks` (E2) |

## Gotchas (also in DEMO_SCRIPT.md)
- **Act 1 takes ~3–5 min** (the two model takes are the slow part) — narrate while they stream.
- **Composer "Send" stuck after a paste?** Type/edit one character so the UI registers it.
- **Debby's (a)/(b) offer labels vary** — just type what you want ("turn this into a light one-page PRD…").
- **With Polly selected, the composer shows skill chips** (`/cross-review`, `/fanout`, `/investigate`) — clear the box before typing if you don't want a skill prepended.
- **"Sandbox launch failed" / "runner didn't come online"** is a transient managed-sandbox hiccup — retry, or start a fresh session.
- **Files panel shows "No files" at first** — it populates once the host finishes cloning the repo.

*All names, VPAs, RRNs, and amounts in this repo are **synthetic and fictional**, for a fictional company "SilverOaks Fintech," for demonstration only.*
