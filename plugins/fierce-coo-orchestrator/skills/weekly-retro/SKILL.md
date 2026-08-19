---
name: weekly-retro
description: >
  This skill should be used when Ryan asks to "run the retro", "run the weekly retro",
  "review agent performance", "what did the agents learn this week", or when a scheduled
  Friday retro task fires. It converts the week's corrections into per-agent LESSONS
  updates so every Fierce sub-agent improves over time.
metadata:
  version: "3.2.0"
---

# Fierce Weekly Retro

Improve the Fierce agent team by converting the week's corrections into durable lessons. Run after the Weekly COO report on Fridays, or on demand.

All files live in `fierce-agents/` inside the connected work folder. If the folder is not connected, ask Ryan to connect it before proceeding.

## Procedure

1. **Read inputs:** `fierce-agents/retro/corrections-log.md`, `fierce-agents/retro/metrics.md`, and the delegation log (create any if missing).

2. **Mine the session transcripts (mandatory).** Do not trust the manual log alone. Use the session-info tools (list_sessions, then read_transcript with format full) to read the week's Fierce sessions: the orchestrator session, LOVB syncs, morning coffee runs, finance sweeps, and any others. From each, extract every correction, self-redo, failed tool path, stall (places where Ryan had to ask "is this done" or "why the delay"), and run-losing API or infrastructure failure, including ones never written to the log. Back-fill `corrections-log.md` with anything missing, one line each: date | agent | what was corrected | root cause. The transcripts are the real signal; the manual log is a backstop.

3. **Write lessons:** for each correction this week, add or update a rule in `fierce-agents/lessons/<agent>-LESSONS.md`. Rules must be specific and testable. Good: "Always BCC staff lists, never To." Bad: "Be more careful with emails." One rule per line, dated.

4. **Promote stable lessons:** any rule that has held for 4 or more weeks without re-correction gets marked PROMOTED and flagged in the retro report as a candidate to move into the agent's core instructions. Tell Ryan which agent file to update and provide the exact text (plugin files are read-only once installed, so promotion into core instructions happens at the next plugin version bump).

5. **Prune conflicts:** if a new correction contradicts an old lesson, the newest correction wins. Delete the stale rule.

6. **Update metrics:** in `fierce-agents/retro/metrics.md`, record per agent for the week from the transcript review (not just the log): deliverables produced, corrections required, redos, escalations triggered. Keep prior weeks for trending.

7. **Sync to the Fierce Second Brain (Google Drive).** After steps 3-6 are done locally:
   - Mirror every rule marked PROMOTED this run into `memory/lessons.md`, organized by agent under the existing `## <agent>` headers in that file.
   - Log any correction that has recurred 2+ times this run into `memory/patterns.md`, one entry per pattern in its existing format: **Pattern | First noticed | How many times observed | What to do about it**.
   - **Locate the current file first.** IDs drift every time one of these files is rewritten (see below), so don't trust a previously-recorded id. Run `search_files` for `title = 'lessons.md' and parentId = '1H4oJ8JgH2uzmZyK-QsFhPuMaS-Q0X7Jy'` (same pattern for `patterns.md`) to get the current file id, then `read_file_content` it to get the current text.
   - **Write with plain text, not the docx SOP.** `memory/lessons.md` and `memory/patterns.md` are flat markdown, not formatted Word documents, so do not use `procedures/google-doc-edit-SOP.md` for these two files; that download-as-docx/base64 round trip exists for richly formatted docs (tables, styling) and is easy to silently corrupt when a large base64 blob has to be reproduced by hand. Instead: compose the full new text (existing content with the new lessons/patterns inserted in place) and call `create_file` with `textContent` set to that full text, `contentMimeType: text/plain`, the same `title`, and `parentId` set to the `memory/` folder id. Leave `disableConversionToGoogleType` unset (default) so Drive converts it to a native Google Doc like its predecessor.
   - **Verify, then retire the old copy.** `read_file_content` the newly created file and confirm the text matches what you intended to write (not empty, not truncated) before reporting success. Once confirmed, `trash_file` the file id you started from so there is only ever one live `lessons.md` and one live `patterns.md` in the folder.
   - If nothing was promoted or no pattern hit 2+ this run, skip the write and say so; do not write empty or placeholder entries.

8. **Report to Ryan:**
   - Corrections this week, grouped by agent
   - Rules added, promoted, or pruned
   - What was mirrored into the Second Brain this run (lessons, patterns), or "nothing to sync" if none qualified
   - Error trend per agent (improving, flat, worsening)
   - One recommendation for a new skill or automation based on repeated ad-hoc requests
   - If any agent had zero corrections for 4+ weeks and stable volume, say so; that agent's instructions are working

Keep the report tight. No em dashes.
