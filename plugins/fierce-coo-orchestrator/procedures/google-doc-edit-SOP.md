# Standard Operating Procedure: Editing an Existing Google Doc

Owner: fierce-staffing-sync (any agent editing a Google Doc follows this)
Created: 2026-06-19, from the ECTF SOP update that took 4 corrections and 2 redos.
Updated: 2026-08-18, added the "when NOT to use this" section below after a Second Brain `decisions.md` update silently corrupted via the base64 path.
Updated: 2026-08-19, split the plain-text path into two cases after the "JazzHR Candidate Status Guide" shipped with literal `##`/`-`/`**` characters visible instead of real formatting; see "Plain text vs. HTML" below.

## Why this exists

There is no reliable in-place edit path for a Google Doc from this environment. In-place editing was attempted via Chrome MCP and Control Chrome JS and both fail on docs.google.com. The Google Drive MCP can create and download files but cannot patch the body of an existing Google Doc. Do not rediscover this live. Use the procedure below.

## When NOT to use this (use plain text or HTML instead)

This full download/edit/re-upload/base64 procedure is for documents where formatting actually matters: tables, styling, orientation decks, anything originally built as a real Word document. For anything else, skip the docx round trip entirely and use one of the two lighter paths below, chosen by audience.

Why avoid the docx/base64 path when you don't need it: it requires reproducing a multi-KB (often 10-25KB) blob of binary-as-text inside a tool call. An agent (or model) retyping that blob from a previous tool result is not a mechanical copy; it is regenerated character by character, and long base64 runs can silently drop or duplicate spans with no error thrown. A corrupted upload can still report a plausible non-zero `fileSize` and only fail on `read_file_content` returning empty, or fail to open at all. Confirmed case: 2026-08-18, a `decisions.md` update via this path uploaded successfully (reported `fileSize: 23983` against an actual 7185-byte source) but was unreadable and did not decode as a valid docx.

### Plain text vs. HTML

Both lighter paths follow the same shape: `read_file_content`/`search_files` for the current id (see "IDs drift" below) → compose the full new content → `create_file` with the same `title` and `parentId`, `disableConversionToGoogleType` left unset so Drive converts it to a native Google Doc → `read_file_content` the result and confirm it's complete and correct → `trash_file` the id you started from. Where they differ is `contentMimeType` and what you compose, and that choice matters:

- **Agent-only Second Brain memory logs** (`FIERCE.md`, `memory/decisions.md`, `memory/lessons.md`, `memory/patterns.md`, `memory/open-loops.md`, `memory/clients/*.md`): use `textContent` with `contentMimeType: text/plain`, composed as flat markdown-style text (`## heading`, `- bullet`, etc). These files are read back by agents via `read_file_content`, not primarily viewed by humans in the Docs UI, so it's fine that opening one in Drive shows the literal `##`/`-`/`**` characters rather than real formatting.
- **Any document a human is meant to read in the Docs UI** — team-facing reference guides, policy docs, anything shared outside the agents themselves: use `textContent` with `contentMimeType: text/html`, composed as real HTML (`<h1>`/`<h2>`/`<h3>`, `<p>`, `<b>`/`<i>`, `<ul>`/`<li>`, `<table>` if a table genuinely helps). Drive's converter turns real HTML tags into real Google Doc formatting (actual headings, bold, bullet lists). It does NOT parse markdown syntax inside `text/plain` — a `text/plain` upload containing `## Heading` produces a doc with the literal six characters `## Heading` on the page, not a styled heading. Confirmed case: 2026-08-19, the "JazzHR Candidate Status Guide" (built for the recruiting team, not agents) was created via the plain-text path and rendered with visible `##`, `-`, `**` markdown syntax instead of real formatting; Ryan flagged it as "not in the format of our internal documents" and it had to be rebuilt as HTML. One wrinkle: avoid nesting `<b>` with an inline color style inside a `<td>` in a header row (e.g. `<td><b style="color:#fff;">Status</b></td>`) — this converted to literal `**Status**` text in testing. Plain `<td>Status</td>` with just a background-color on the `<tr>` converts cleanly.

**IDs drift either way.** Every replacement (docx round trip, plain text, or HTML) creates a new file id, since there is no way to patch an existing file's body. Don't hardcode ids for files you expect to edit again; resolve them by `search_files` on `title` + `parentId` first.

## The procedure

1. Download the doc as a Word file.
   Use the Drive MCP `download_file_content` on the file ID, requesting the .docx export. You get base64 back.

2. Decode and edit in the sandbox.
   Decode the base64 to a .docx in the outputs folder. Edit it with python-docx. Edit the actual file on disk. Never edit content carried over from a previous session.

3. Re-upload as a replacement, keeping it a Word file.
   Re-encode the edited .docx from disk to base64 (re-encode from the file, never reuse an old string). Upload with the Drive MCP `create_file` and you MUST set:

   ```
   disableConversionToGoogleType: true
   contentMimeType: application/vnd.openxmlformats-officedocument.spreadsheetml... (use the docx mime for docs:
   application/vnd.openxmlformats-officedocument.wordprocessingml.document)
   ```

   Without `disableConversionToGoogleType: true`, Drive tries to convert the upload and you get an empty 1-byte file. This was the single biggest failure in the ECTF run.

4. Verify before reporting.
   Confirm the uploaded file size is in the expected range (a real doc is kilobytes, not 1 byte). If in doubt, download it back and open it. Only then report the link to Ryan.

## Editing rules that caused redos (apply these)

- Re-encode base64 from the on-disk file every time. Prior-session base64 may be a placeholder stub, not the real file.
- When stripping em dashes, read what each dash means in context first. A lone "—" in a table cell usually means "none" or "no specific location." Replace it with the intended word, not a bare colon or an empty string. Blind character replacement created cells reading ":" and "arrive :".
- Plan all edits, then make them in one pass. Do not run three separate fix passes on the same file.
- Valid base64 stays valid even when the source data has smart quotes. Verify once, then proceed. Do not loop re-reading and re-uploading the same payload out of doubt.

## Quick reference: known-good upload call

create_file with:
- title: same as the existing doc (or the intended name)
- contentMimeType: application/vnd.openxmlformats-officedocument.wordprocessingml.document
- disableConversionToGoogleType: true
- base64Content: freshly encoded from the edited .docx on disk
