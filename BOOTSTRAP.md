# COS Pilot Bootstrap

**Populating your `status.json` from Cameron's existing leadership data.**

Audience: each pilot COS (Gerald, Clark, CoS-MH1, Vera, and any future onboardees).
Time: ~15 minutes with your principal, mostly approval at /wrap.

---

## Where you start

Your subfolder exists at `<this-matrix-repo>/<your-cos-name>/`. Two files:

- `index.html` — your principal's personal leader-dashboard. No edits.
- `status.json` — a **stub**. Empty `goals[]`, `last_updated: 2026-01-01` so the matrix flags you as "no real report yet."

Your job: replace the stub with your principal's real rocks.

---

## Where the source data lives

Cameron already maintains every leader's rocks in his `dashboard-data.js`, on the deployed goals-dashboard repo:

`https://raw.githubusercontent.com/einsteinadmin/goals-dashboard/main/dashboard-data.js`

Inside, find the `leadership[]` array entry whose `name` matches your principal. Each entry has 4–6 rocks with:

| Field | What it is |
|---|---|
| `name` | Rock name |
| `critical` | Critical Number? (boolean) |
| `status` | Current RAG state (`supergreen` / `green` / `yellow` / `red` / `unknown`) |
| `assessment` | One-paragraph current state |
| `superGreen` | Super Green threshold criteria |
| `green` | Green threshold criteria |

**You don't invent rocks.** You translate Cameron's existing ones into your status.json schema. The rock list and shape come from him — your job is to add the new fields your status.json needs and validate the rest with your principal.

---

## Schema mapping (Cameron's data → your status.json)

| Cameron's `leadership[]` field | Your `status.json` goal field | Action |
|---|---|---|
| `name` | `name` | Copy verbatim |
| `critical` | `critical` | Copy |
| `status` | `self_status` | Copy (same enum) |
| `assessment` | `assessment` | Copy verbatim |
| `superGreen` | `superGreen` | Copy |
| `green` | `green` | Copy |
| — | `id` | **NEW** — stable slug. e.g. `mike-personnel-mgmt`. Used by /wrap to match deltas; never changes even if the rock is renamed. |
| — | `number` | **NEW** — sequential, 1..6 |
| — | `confidence` | **NEW** — ask your principal, 1–5 |
| — | `percent_complete` | **NEW** — ask your principal, 0–100 |
| — | `blockers` | **NEW** — pull from `assessment` + ask |
| — | `milestones` | **NEW** — ask your principal |
| — | `updated` | **NEW** — today's date |

Plus top-level fields:

```json
{
  "schema_version": "1.0",
  "owner": { "name": "<principal>", "title": "<title>", "cos": "<your-cos-name>" },
  "period": "Q2 2026",
  "last_updated": "<now, ISO-8601>",
  "updated_by": "<your-cos-name> (/wrap — bootstrap)",
  "cadence_days": 1,
  "goals": [ ... ],
  "kpis": [],
  "adopted": [
    { "pattern": "template",      "version": "<latest>", "state": "adopted" },
    { "pattern": "wrap-git-push", "state": "adopted" },
    { "pattern": "gm-step-0",     "state": "adopted" },
    { "pattern": "status-file",   "version": "v1.0",     "state": "adopted" }
  ],
  "network_acks": []
}
```

`network_acks[]` starts empty. You'll fill it in as you actually ack SCB announcements going forward.

---

## Worked example — Gerald bootstrapping Mike's first rock

**From Cameron's `dashboard-data.js`** (Mike's Rock #1, abbreviated):

```json
{
  "name": "Personnel Mgmt — Manager Hiring + New Manager Development (Garland + Tampa)",
  "critical": true,
  "status": "green",
  "assessment": "5/18 PM: Pause Tampa hiring sourcing... Combined sales + hiring bandwidth model in design.",
  "superGreen": "All DFWT 100%+ staffing, all 4 new-in-role managers at B+...",
  "green": "DFWT 95%+ staffing, all 4 new managers at B rating by EOQ..."
}
```

**What Gerald writes into `gerald/status.json`** (the full goal object — fields Gerald added bolded):

```json
{
  "id": "mike-personnel-mgmt",
  "number": 1,
  "name": "Personnel Mgmt — Manager Hiring + New Manager Development (Garland + Tampa)",
  "critical": true,
  "self_status": "green",
  "confidence": 4,
  "percent_complete": 50,
  "assessment": "5/18 PM: Pause Tampa hiring sourcing... Combined sales + hiring bandwidth model in design.",
  "blockers": [
    "Texas contingent hiring legality — Mike researching policy draft",
    "Beatty needs to run more trial days"
  ],
  "milestones": [
    { "name": "All 4 new managers at B rating", "date": "2026-07-31", "done": false }
  ],
  "superGreen": "All DFWT 100%+ staffing, all 4 new-in-role managers at B+...",
  "green": "DFWT 95%+ staffing, all 4 new managers at B rating by EOQ...",
  "updated": "2026-06-02"
}
```

`confidence`, `percent_complete`, `blockers`, `milestones`, `id`, `number`, `updated` are all new. The rest is copied straight from Cameron's data.

---

## Required-new-field guidance

**`id`** — `<principal-first>-<short-name>` works. Lowercase, dashes only. Examples: `mike-fleet-safety`, `paul-mover-comp`, `matisen-vp-rollout`. Stable forever.

**`confidence` (1–5)** — Ask your principal. Don't guess. Rough heuristic:
- **5** — recent hard data confirms status
- **4** — on track, no surprises this week
- **3** — directional read, some uncertainty
- **2** — shaky, red flags emerging
- **1** — gut-call only, low signal

**`percent_complete` (0–100)** — Rough estimate from your principal. Doesn't have to be exact; better than blank.

**`blockers[]`** — Pull anything from the `assessment` text that reads like a blocker, then ask your principal: "anything else slowing this down?" Concise sentences only — not paragraphs.

**`milestones[]`** — Ask: "What are the 2–3 milestones between now and EOQ that mark this rock progressing?" Each has `name`, `date` (YYYY-MM-DD), `done` (bool).

---

## Bootstrap flow (the actual sequence)

1. **Fetch** Cameron's `dashboard-data.js` (URL above). Find your principal's `leadership[]` entry.
2. **Draft** the translated `goals[]` array — copy what's there, add the new fields with placeholder values.
3. **Open the conversation with your principal:** *"Cameron's been tracking your rocks — here's how I'd populate your status. Walk through each one with me?"*
4. **Per rock:** confirm `self_status`, ask `confidence`, ask `percent_complete`, pull blockers from assessment + ask for additions, ask for 2–3 milestones. Apply each as you go.
5. **Seed `kpis[]`** if your principal tracks any (most don't at this level — leave empty if so).
6. **Seed `adopted[]`** to mirror what your COS is actually running (see template in the schema-mapping section above). Pull the latest template version from the SCB or ask Albert.
7. **Leave `network_acks[]` empty.** Fill it in as you actually ack things going forward.
8. **Run /wrap.** Step 4.5 fires the approval gate against your filled-in status.json. Since this is your first push, the gate shows the entire delta vs. the empty stub.
9. Your principal approves (or edits item-by-item). File writes, validates against the schema, pushes to the matrix repo's `<your-subfolder>/status.json`.
10. Within ~30 seconds, the deployed matrix and Cameron's org dashboard reflect your principal's state. **You're live.**

---

## What you don't do

- **Don't invent rocks.** Cameron's `leadership[]` is the rock list. If it has 4, you have 4.
- **Don't push without principal review.** Even on bootstrap, /wrap fires the gate.
- **Don't guess confidence or percent_complete.** Ask. Leave blank only if your principal genuinely doesn't have a read yet — they fill in as signal accumulates.
- **Don't write blockers from your own inference.** Pull from existing notes (assessment) + ask. Naming a blocker has weight; your principal owns naming their pain.
- **Don't backfill `network_acks[]` to look caught up.** Only ack threads you've actually read with your principal.

---

## After bootstrap

Bootstrap is one-time. From now on:

- Your COS owns `status.json` updates daily via /wrap (Step 4.5 gate — see your `/wrap` skill).
- `/digest` folds meeting deltas into the file (blockers, milestone hits, status changes).
- The trust ladder relaxes the gate over time: **full gate weeks 1–2 → material-change-only weeks 3–6 → trust-by-default at steady state.** Your principal opts in when ready.
- Friday /wrap does a deeper pass (refresh assessment narratives, sanity-check `adopted[]` and `network_acks[]`).

You're on the network. The matrix shows your row green, Cameron's dashboard shows the "view full →" link working, and your slice publishes itself going forward.

---

**Reference:** Full COS Network v2 architecture lives at `AI Operating System/cos-network-v2-plan.html` in Cameron's workspace. Schema at `_shared/cos-network-v2/schema/status-schema.json`. /wrap gate spec at `_shared/cos-network-v2/wrap-approval-gate.md`.

**Questions?** Drop them on the Shared Context Board (`Active Context` group), tag `@Albert (Cameron's COS)`.
