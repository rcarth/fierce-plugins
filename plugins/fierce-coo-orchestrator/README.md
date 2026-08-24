# Fierce COO Orchestrator

> **3.7.0** (2026-08-19): Added the `fierce-business-development` sub-agent, owning pipeline visibility and prospect tracking before a deal becomes a signed event: lead/contact status, stale-lead (14+ days no next step) and stalled-proposal (10+ business days no response, event within 60 days) flags, win/loss capture, and outreach/proposal draft content handed to fierce-communications, never sent. A live check of Monday.com found no active BD pipeline: the only candidate board, "Contacts" (Client Services workspace, id 2608257609), is a dormant default template untouched since 2022 with 4 placeholder items and no deal-stage, value, source, or proposal-date columns. Rather than build against data that doesn't exist, the agent reports what the board actually has and recommends a schema upgrade (Stage / Estimated Value / Lead Source / Proposal Sent Date / Next Step) before Ryan approves changing the board. coo-orchestrator's routing table and escalation rules were updated accordingly. Logged in `memory/decisions.md`.
>
> **3.6.2** (2026-08-19): Confirmed via a live test (3.6.1 bump) that Cowork desktop's per-plugin "Update" button does not work for personal git-marketplace plugins: stays permanently grayed out even after the marketplace repo has a newer version, a marketplace "Sync," and a full app restart. Matches Anthropic issue #65426 (closed "not planned"). The marketplace install path itself DOES work (installing fresh from the Directory pulls the real current version), only the in-place update is broken. New standing process: when a new version is pushed to this repo, remove the existing plugin in Cowork and reinstall fresh from the marketplace, instead of clicking Update. Logged in `memory/decisions.md`.
>
> **3.6.1** (2026-08-19): No functional change. Version-only bump used as a test marker for the above.
>
> **3.6.0** (2026-08-19): Corrected `procedures/fierce-internal-doc-template.md` against a real screenshot of the "ABCD & Company — Client Profile" document: the color palette was wrong (guessed blue instead of the real near-black navy `#14152B` + gold `#C9A961` + gray `#595959`), and banner text should be left-aligned, not centered. Also documented a build-technique fix: a banner's stacked lines must be separate table rows, not multiple paragraphs in one table cell, or they collapse onto one line on read-back. The JazzHR Candidate Status Guide is rebuilt against the corrected template. Logged in `memory/decisions.md`.
>
> **3.5.0** (2026-08-19): Added `procedures/fierce-internal-doc-template.md`, a single reusable design standard (banner-bar section headers, fixed color set: `#1B3A6B` primary / `#5B7FA6` secondary / `#D9E0EB` table tint) for every internal staff-facing Drive document, modeled on the existing "Client Profiles" documents. Ryan's rule: all internal documents must be consistent in design, format, and color scheme, not styled ad hoc per document. The JazzHR Candidate Status Guide was rebuilt against this template. Exact source colors couldn't be safely verified (same base64-corruption risk as the docx path), so this ships as an explicitly Ryan-approved best guess using the established Fierce brand blue, correctable in one place if wrong. `coo-orchestrator` now points to this template for all internal documents. Logged in `memory/decisions.md`.
>
> **3.4.0** (2026-08-19): Fixed team-facing Drive documents rendering with literal `##`/`-`/`**` markdown syntax instead of real formatting. The "JazzHR Candidate Status Guide" was rebuilt with real HTML (headings, bold, bullet lists, a quick-reference table) after Ryan flagged it as not matching Fierce's internal document format. `procedures/google-doc-edit-SOP.md` and `coo-orchestrator` now draw a hard line: `text/html` for anything a human outside the agents will read, `text/plain` stays for agent-only Second Brain memory files (decisions.md, lessons.md, patterns.md, open-loops.md, FIERCE.md, client files). Logged in `memory/decisions.md`.
>
> **3.3.0** (2026-08-19): Finalized and baked in the JazzHR status pipeline after a process review: fixed the 2-vs-3 attempt count on AI Interview Reminder Sent, split "Headshot Received - Ready for Review" into "Pending Event Approval" and "Headshot and Interview Completed - Needs Additional Review", renamed "Decision Making" to "Low Score - Platform Review Needed" and narrowed its scope, disambiguated Not Hired vs. Not Qualified, added Waitlist's auto-email + recruiter follow-up, and required GoCo termination before flipping JazzHR on a failed Contractor. Rules added to `fierce-recruiting` and `fierce-hr-onboarding`; full guide lives in the Drive doc "JazzHR Candidate Status Guide", reasoning in `memory/decisions.md`.
>
> **3.2.0** (2026-08-19): Added an explicit Outlook access rule to `fierce-communications` and `fierce-intelligence`: use the Claude in Chrome extension when the desktop app is connected, fall back to the Microsoft 365 Graph MCP only for unattended/scheduled runs. Codifies a Second Brain decision logged the same day in `memory/decisions.md`.
>
> **3.1.0** (2026-08-18): Wired the orchestrator to the Fierce Second Brain (Google Drive `FIERCE.md` + `memory/`): read before every delegation, checked for a client file on new events, and kept alongside the local `fierce-agents/` system rather than replacing it. Wired weekly-retro to mirror promoted lessons into `memory/lessons.md` and recurring (2+) corrections into `memory/patterns.md`, using the existing Google Doc edit SOP since Drive has no in-place body-edit path.
>
> **3.0.1** (2026-06-26): Bundled `procedures/google-doc-edit-SOP.md` and wired the Staffing Ops & Sync agent prompt to reference it, so the full Google Doc edit procedure ships inside an installed copy. All v3 lessons remain folded into the agent prompts.


A primary-agent system for Fierce Staffing Services and Consulting LLC. Ryan (COO) talks to one Executive Assistant orchestrator, which delegates to 10 specialist sub-agents, quality-checks their deliverables, and reports back in a single COMPLETED / FLAGS / RECOMMENDATIONS / STAGED format.

## Components

### Skills

| Skill | Purpose |
|---|---|
| `coo-orchestrator` | The primary agent. Routes any Fierce operational request to the right sub-agent(s), runs multi-agent requests in parallel, quality-checks output, tracks open delegations, escalates per rules. |
| `weekly-retro` | Friday self-improvement routine. Converts the week's corrections into per-agent LESSONS files, promotes stable rules, prunes conflicts, trends error rates. |

### Sub-Agents

| Agent | Owns |
|---|---|
| `fierce-intelligence` | Morning Coffee briefing, Weekly COO report, ad-hoc data pulls |
| `fierce-event-production` | Full event launch kit as ONE pipeline (JD, Event Snapshot, orientation deck, KBYG); staff profile CVs on demand |
| `fierce-communications` | All email drafts; everything stops in Outlook Drafts |
| `fierce-recruiting` | JazzHR pipeline, screening summaries, time-to-fill, chase lists |
| `fierce-hr-onboarding` | GoCo workflows, policy/acknowledgment language, conditional offers, background check verbiage |
| `fierce-staffing-sync` | LOVB Monday-to-Drive sync and roster hygiene, with anomaly flags |
| `fierce-meeting-accountability` | Fireflies commitments vs Monday cards; surfaces UNTRACKED promises |
| `fierce-payroll-compliance` | Gusto/JazzHR/When I Work reconciliation; report-only |
| `fierce-finance` | Unbilled events, AR aging, payroll deadlines, estimates without contracts; report-only |
| `fierce-business-development` | Prospect/lead pipeline visibility, stale-lead and stalled-proposal flags, win/loss capture, outreach/proposal draft content |

## Setup

1. Install the plugin.
2. Connect a work folder in Cowork and let the orchestrator create `fierce-agents/` inside it (delegation log, corrections log, LESSONS files). This is per-device memory.
3. Set up the Fierce Second Brain in Google Drive: a root `FIERCE.md` doc plus a `memory/` folder (`decisions.md`, `patterns.md`, `lessons.md`, `open-loops.md`, `clients/<name>.md`). This is company-wide memory that both the orchestrator and weekly-retro read/write, and it's what makes unattended scheduled runs work without a device connected.
4. Connectors used: JazzHR, Monday.com, Gusto, GoCo (via browser), When I Work (via browser), Google Drive, Microsoft 365/Outlook, Fireflies, Canva, QuickBooks (via browser where no MCP exists).
5. Related scheduled tasks (managed separately in Cowork): `morning-coffee-coo` (daily 6 AM), `lovb-staffing-sheet-sync` (8 AM and 4 PM daily), `finance-exception-sweep` (Mondays 7 AM), `weekly-retro` (Fridays). Give scheduled task prompts the Drive file/folder ids for FIERCE.md and memory/ so they're read on unattended runs.

## Usage

Talk to the orchestrator naturally: "We signed the Nike event", "run morning coffee", "did we bill Xponential", "what did we commit to on the LOVB call". It routes automatically. The only things it will always stop and confirm: sending external communications, processing payroll, committing money.

## Guardrails

- Communications: drafts only, never sends
- Payroll & Compliance and Finance: report-only, never modify financial data
- Escalations fire immediately for money already moved, events within 14 days understaffed, client deadlines at risk, blocked pipelines, unbilled completed events

## Claude Code Transfer

The `agents/*.md` files use the shared Claude Code agent schema and work as-is in `.claude/agents/`. The two skills carry over to `.claude/skills/`. Keep the `fierce-agents/` memory directory in the project repo so the retro system transfers unchanged.
