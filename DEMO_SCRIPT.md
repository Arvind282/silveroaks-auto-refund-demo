# 💸 UPI Auto-Reversal — The Omnigent Demo (run sheet)

**The whole demo is this file.** You drive the Omnigent **web UI** and narrate; the agents generate everything live. The UI mechanics below mirror a verified end-to-end Omnigent deployment; the talk track is tailored to **SilverOaks Fintech**, a (fictional) Indian digital-lending + UPI + insurance fintech.

**The setup (one breath):** You're a **PM on the UPI / Payments team at SilverOaks Fintech**. You're scoping one feature — **Auto-Reversal**: when a UPI payment fails *after* the customer is debited (money left their account, but the beneficiary credit or settlement didn't complete), the money is returned automatically and immediately, instead of the customer raising a complaint and waiting days. You'll (1) use two AI models from different labs to pressure-test it and draft a light PRD, (2) hit a question only the **Ledger & Settlement** team can answer and pull in a teammate on the live session, and (3) hand the build to agents that run in the cloud overnight.

**The arc:** more brains → more people → no laptop. **Meta wink (open or close):** *"The assistant that helped build this demo runs inside Omnigent. Everything you're about to see, it does to itself."*

- ⏱️ ~10–12 min · 👥 Customer-facing · 🧑‍🤝‍🧑 Teammate on standby for Act 2
- Legend: 🎤 = say · 🖱️ = click · ⌨️ = type · ✅ = expected · ✳️ = illustrative / CLI alt

> **Pre-flight:** Logged into your workspace's Omnigent (`https://<your-databricks-workspace>/omnigent`). Context repo public & handy: **`https://github.com/arvind-kumar_data/silveroaks-auto-refund-demo`**. Teammate ready to open a link. Optionally pre-run one session as a fallback (Act 1 takes ~3–5 min).

**Why UPI Auto-Reversal (the stakes, for the room):** In India a failed UPI debit is not a "nice to have" refund — under the **RBI Harmonisation of TAT** rules the money must be auto-reversed by **T+1**, or SilverOaks pays the customer **₹100/day**. It's high-volume, it's real money, and getting it subtly wrong is a fraud hole or a double-credit incident. Perfect thing to have two models argue about before a line of code is written.

---

## Act 1 — Two models pressure-test the feature, then draft a light PRD
**Capability:** Omnigent's built-in **Debby** agent sends every prompt to a **Claude** model 🟠 *and* a **GPT** model 🔵 (different labs), compares them, and can have them debate. **Why a customer cares:** reversals move real money on a regulated rail — one model's blind spot is a fraud hole or a double-credit incident. A builder + a skeptic catch more, on one governed bill.

🎤 *"I'm a PM on the UPI team at SilverOaks. We want to ship Auto-Reversal — if your UPI payment fails after we've debited you, you get your money back automatically, no complaint, no waiting, and we stay inside the RBI T+1 window. Reversals are easy to get subtly wrong, so before I write anything I'm going to have two models — one from Anthropic, one from OpenAI — stress-test it."*

**Set up the session (home screen):**
- 🖱️ Agent picker (defaults to "Claude Code") → **Debby — Multi-agent debate**.
- 🖱️ Host button → **Databricks Sandbox**. 🎤 *"This runs in a Databricks-managed serverless sandbox, not on my laptop — that matters in act three."*
- 🖱️ **Repository** → paste **`https://github.com/arvind-kumar_data/silveroaks-auto-refund-demo`**, branch **`main`** (the button then reads `silveroaks-auto-refund-demo#main`). ✅ Omnigent clones it into the sandbox as the working directory, so the agent starts with our context — the app, the **UPI-Payments vs. Ledger & Settlement** boundary, the `upi_txn_failed` schema, and the draft contract. 🎤 *"I'm pre-loading my team's context as a repo, so I don't paste a wall of background — and the agents build into it later."*
- ⌨️ Now the opener is one line — start the session:
> Read the repo for context (README + docs/reversal-contract.DRAFT.md), then have your two partners pressure-test our UPI Auto-Reversal feature and flag the open questions for the Ledger & Settlement team.
> ✅ Debby `ls`'s the repo, reads the README/schema/contract ("I have full context now"), then dispatches the Claude 🟠 + GPT 🔵 partners. (Files panel shows "No files" until the clone finishes.)
> 💬 **No repo? Fallback brief:** *"I'm a PM on the UPI team at a fintech. We want to ship Auto-Reversal: when a UPI txn fails after debit — money left the customer's account but the beneficiary credit/settlement didn't complete — reverse it automatically inside the RBI T+1 window instead of a manual complaint + multi-day wait. Pressure-test and scope it."*

> ✅ **What happens:** Debby fans your brief to a **Claude** head 🟠 and a **GPT** head 🔵, posts both independent takes + a **"where they agree / differ"** read, then offers next steps (a one-page PRD and/or a `/debate`). Narrate the two takes — your real "two labs, two blind spots" moment:
> - 🟠 **Claude** tends to flag the **demand-side fraud**: unfalsifiable "money never reached me" claims, auto-reversal *plus* an NPCI chargeback (URCS) = double-dip, collusion between payer and payee, and "who eats the loss" when the beneficiary bank later confirms the credit did land.
> - 🔵 **GPT** tends to bring the **settlement-rail / infra** angle: NPCI switch and beneficiary-bank variance, **deemed-approved** timeouts, a "failed" txn that actually **settles late** (so you reverse *and* it settles → double money out), RRN-based reconciliation, and concrete fixes — a short **reconciliation hold** window, an **idempotent reversal ledger keyed on RRN**, and dedup against NPCI's own auto-reversal.
>
> Then ask for the light doc — **don't rely on the (a)/(b) labels, they vary run-to-run; just type what you want:**
> ⌨️ **Turn this into a light one-page PRD I can read on screen, under ~250 words: the problem, the 3 most important features, 1–2 success metrics, and end with "Open questions for the Ledger & Settlement team."**
> ✅ **Expected output:** a ~230-word PRD — **Problem · 3 Features · Success Metrics · Open Questions for Ledger & Settlement** — legible on one screen, and that last section *is* your Act 2 hook (it surfaces loss-ownership, reversal instrument, the idempotency key, NPCI/URCS dedup, and the TAT clock).
> 🎁 **Want them to visibly argue (not just give parallel takes)?** Lead with `/debate … for 1 round` instead — same two models, but you watch them critique each other and converge before the PRD. More drama, ~2 min longer.

🎤 **Why two models (the proof point):** *"The skeptic forced the questions a single model glosses over — who eats it when someone fakes a 'money not received,' how we avoid reversing a txn that later settles, what our idempotency key even is so NPCI's auto-reversal and ours don't both fire. The builder kept it on the customer-trust and RBI-TAT win. That tension is the value, and it's one session, one bill."*

**Governance (for a customer audience):**
- 🖱️ **Agent tools and policies** → 🎤 *"One session, two vendors, one live running cost — and I can attach a **Session Cost Budget** that hard-caps spend and auto-downgrades the model, **Deny PII in LLM requests** — this is customer bank-account and VPA data — or require approval on shell/file ops."* (Show the **Add policy** list.) *"Governed, audited, guardrails the agent can't talk its way around — the same story your risk and compliance teams will ask about."*

---

## Act 2 — One question is Ledger & Settlement's, not mine → pull in a teammate, live 📲
**Capability:** share the *live* session; a teammate joins and steers it. **Why a customer cares:** governed, real-time collaboration on the same running session — not screenshots in Slack.

🎤 *"Here's the honest part: my team owns **detecting** the failed UPI txn and deciding a reversal is owed. But actually **moving the money** — credit-reversal vs. fresh credit, never double-crediting when NPCI's own auto-reversal also fires, reconciling against the switch RRN, owning the reversal ledger as source of truth — that's the **Ledger & Settlement** team. Those open questions at the bottom of the PRD aren't mine to answer. So I'll bring in their lead on this exact session instead of starting a thread."*

**Share it:**
- 🖱️ **Share session** → *"invite others to view or collaborate."*
- Flip **"Any workspace user can view this session,"** or add the teammate by **User ID**, set **Level → Edit**, **Grant**.
- 🖱️ **Copy link** → Slack it. (It shows under their **"Shared with me."**)

🎤 *"Same live session, on their device, authenticated through our Databricks workspace, with the permission I set — Read to watch, Edit to drive."*

**Teammate drives (types into the shared session):**
> ⌨️ Ledger & Settlement here. Reversals touch the money path and the transaction record, which we own. Let's pin the contract: UPI emits a `upi_txn_failed` event (txn id + RRN + amount + failure reason + debit/credit status). Settlement decides credit-reversal vs. fresh-credit, guarantees idempotency **keyed on RRN** so we never double-credit (including when NPCI's URCS auto-reversal also fires), holds for one reconciliation cycle to catch late settlement, and stays source of truth for the reversal ledger. Add that to the PRD and note what each team owns.

🎤 *"That's the answer, written into the PRD by the team that owns it — live, in context. The cross-team gap is closed before a single line of code, and before we risk an RBI TAT breach."*

---

## Act 3 — Hand the build to agents in the cloud, overnight 🌙🤖
**Capability:** the session already runs on a managed **Databricks Sandbox (Lakebox)** — so it keeps going with your laptop closed, and you reconnect from any device. **Why a customer cares:** no babysitting; isolated, serverless, governed.

🎤 *"We're aligned, the contract's in the PRD. Now the build. Remember the host is Databricks Sandbox — this has been running in the cloud the whole time, not on my machine."*

**Kick off the build:**
- 🖱️ **New session** → agent picker → **Polly — Multi-agent coding** (or Claude Code) → host **Databricks Sandbox** → attach the same **Repository** (`…/silveroaks-auto-refund-demo`) so the build lands in `services/reversals/`.
- ⌨️ Hand over the build:
> Build the UPI Auto-Reversal MVP in `services/reversals/` per the repo's context and `docs/reversal-contract.DRAFT.md`: consume `upi_txn_failed` (`schemas/upi_txn_failed.example.json`), implement an idempotent reversal decision keyed on RRN (credit-reversal vs. fresh-credit + a one-cycle reconciliation hold + NPCI/URCS dedup), a Ledger client stub, the customer notification, and write tests. Commit as you go.

🎤 **(Close the laptop. Pause.)** *"Serverless compute, no credentials on the box, sandbox policies still enforced. Closing the lid."*

🎤 *"Tomorrow, from any device, I reopen the session URL:"*
- 🖱️ Reopen → **Files** tab (what it built) + **Shells** tab (what it ran). 🎤 *"It worked the PRD while I slept — an idempotent reversal service keyed on RRN, with tests."* (Bonus: a long run is still shareable — Ledger can watch progress too.)

> ✳️ **CLI alternative (automation / power-user audience):** `databricks sandbox create --name reversal-build`, `databricks sandbox config <id> --no-autostop`, `databricks sandbox ssh <id> -- '…'`, `databricks sandbox status <id>`.

---

## Close 🎬
🎤 *"More brains, more people, no laptop — a Claude and a GPT pressure-testing the feature into a tight PRD, the Ledger & Settlement lead closing the cross-team gap on the same live session, and agents finishing the build in the cloud overnight. And the assistant that helped me build this has been inside Omnigent the whole time. Same platform — swap the model, share the session, run it serverless — no rewrite."*

---

### Verified facts cheat-sheet (for stage Qs)
- **Agents in the picker:** Claude Code · Codex · Cursor · Pi · Polly (multi-agent coding) · Debby (multi-agent debate).
- **Debby:** default = **side-by-side** (fans to a **Claude** 🟠 + **GPT** 🔵 sub-agent, shows both takes + "where they agree/differ," then offers debate-or-PRD); `/debate … for N rounds` adds visible cross-critique → synthesis; sub-agents show in the **Agents** panel.
- **Host:** Databricks Sandbox (managed/serverless) or Connect new host; session shows "Provisioning sandbox…" then runs server-side.
- **Repository attach:** paste a Git URL + branch on the launch screen → cloned into the sandbox as the working dir (public repos need no creds; private needs Databricks git credentials). Demo context repo: `github.com/arvind-kumar_data/silveroaks-auto-refund-demo`.
- **Share:** workspace-view toggle, or per-user **Read/Edit** grant + Copy link; you're **Owner**; teammate sees it under "Shared with me."
- **Governance:** live **Session cost** + **Token usage**; **Add policy** incl. *Session Cost Budget* (hard-cap → forces model downgrade), *Per-User Daily Cost Budget*, *Deny PII in LLM Requests*, *Block Dangerous Shell Commands*, *Require Approval for File & Shell Ops*, *Deny Trivial Tasks on Expensive Models*, *Enforce Sandbox on Agent Start*, GitHub/Google/Gmail access controls.

### Fallbacks
- **Debate/takes slow or long:** ask for "1 round" and keep the ~250-word cap; or fall back to a pre-run session — the PRD is the payoff, not the wait.
- **Teammate can't join:** flip the workspace-view toggle and screen-share, or read their message and narrate it.
- **Composer "Send" looks stuck after a paste:** edit one character in the box so the UI registers the text, then Send.
- **Don't rely on Debby's (a)/(b) labels** — they vary run-to-run; just type what you want.
- **Act 1 timing:** the two model takes are the long pole (~3–5 min) — narrate them as they stream, or pre-open a pre-run session.
- **Files panel shows "No files" at first:** it populates once the host finishes cloning the repo.
- **"Sandbox launch failed" / "runner didn't come online":** transient managed-sandbox hiccup — retry, or start a fresh session.
- **Polly skill chips can auto-prepend** (`/cross-review`, `/fanout`, `/investigate`): clear the box before typing the plain build prompt if you don't want a skill.

*Fictional company "SilverOaks Fintech"; all data synthetic.*
