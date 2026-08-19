# Fierce Internal Document Template

Owner: any agent creating a Drive document meant for internal Fierce staff to read (recruiting team references, client profiles, policy guides, process docs).
Created: 2026-08-19, after the JazzHR Candidate Status Guide went through three rounds of Ryan's feedback: first for using literal markdown syntax instead of real formatting (see `google-doc-edit-SOP.md`), then for not matching the design of Fierce's existing internal documents (banner-bar section headers, not plain headings), then for using the wrong colors (a guessed blue instead of the real palette). Round 3 was corrected against a screenshot Ryan sent of the actual "ABCD & Company — Client Profile" document. **The colors and alignment below are screenshot-verified, not guessed.**

## Why this exists

Ryan's rule: "All of the documents that we create that are fierce internal staff facing need to be consistent with the design, format and color scheme." One-off HTML per document produces inconsistent results. Use this template's exact structure and colors for every new internal document, and check existing ones against it when you touch them.

Reference examples: the "Client Profiles" Drive folder (id `1aP9Oq2RcScPO6Zhm-APdlDHxrol5n2nw`), e.g. "ABCD & Company — Client Profile" and "Splendid Affairs — Client Profile".

**Note on getting colors right the first time:** don't try to reverse-engineer exact colors by downloading a doc as `.docx` and decoding the base64 to inspect the XML. That path was attempted on 2026-08-19 and failed exactly the way `google-doc-edit-SOP.md` warns about: the docx decoded, but `word/document.xml` (the file with the actual colors) came back corrupted and unreadable while the smaller supporting files decoded fine. If you need to match an existing document's exact look and don't already have verified values in this file, ask Ryan for a screenshot rather than guessing or attempting the risky decode.

## Structure (screenshot-verified 2026-08-19)

1. **Title banner**: two stacked lines, same dark navy background, both **left-aligned** (not centered) with left padding/indent: company name in caps, white, bold, larger size on the first line; document title/subtitle directly below in the gold accent color, bold, slightly smaller. **Build this as two separate `<tr><td>` rows in one table, not two `<p>`/`<div>` tags inside one cell.** Multiple paragraphs inside a single table cell collapse onto one line when read back via `read_file_content` (confirmed 2026-08-19, tried both `<div>` and `<p>`); two separate table rows read back correctly as two lines every time. This may only be a `read_file_content` serialization limit rather than a real rendering bug in the live Doc, but there's no reliable way to tell without a screenshot, so don't risk it: always use one table row per line of text you need to stay visually distinct.
2. **Accent divider bar**: a solid gold bar, no text, directly under the title banner, moderate height (thicker than a hairline, roughly 10-14px).
3. **Byline line**: plain italic paragraph below the divider, small font, gray text, left-aligned, format: `Official [team] reference — [update cadence note] | Last updated [date]`.
4. **Intro paragraphs**: plain bold-label paragraphs (`<b>Audience:</b> ...`, `<b>Purpose:</b> ...` or equivalent) before the first section banner.
5. **Major section banners**: full-width single-cell table, dark navy background, white bold **left-aligned** text (with left padding), all-caps section title (e.g. "OVERVIEW", "KEY CONTACTS", "QUICK REFERENCE").
6. **Status/tag line** (optional, use when a section has a clear at-a-glance state): a short bold line in green, directly under a major section banner, all-caps (e.g. "ACTIVE & EXPANDING"). Only use where a genuine status concept exists; don't force it into every section.
7. **Sub-section banners**: full-width single-cell table, medium-dark gray background, white bold **left-aligned** text (with left padding), all-caps (e.g. a status name, a contact category like "CLIENT SIDE").
8. **Callout / note box**: for editor notes or side information (not primary content), use a light gray background box with a bold header line ("Notes for whoever's updating this doc:") followed by a bullet list, visually distinct from regular body content. Do not use this treatment for primary content, only for asides.
9. **Body content**: plain `<p>` paragraphs and real bullet lists (`<ul><li>`) directly under each banner. Bullets render as standard round bullets, left-indented under the banner's left margin.
10. **Data tables**: for quick-reference tables, use a real `<table border="1">` with a light tint header row. Do NOT put `<b>` or bold styling directly on table `<td>` cells; it round-trips as literal `**text**` when read back via `read_file_content` (a serialization quirk, avoid it regardless of whether it also affects the live Doc's visual rendering). Keep table cell content plain.
11. **Footer note**: small italic gray text at the bottom, noting where the document lives / how to request changes.

## Color reference (screenshot-verified)

| Use | Hex | Notes |
|---|---|---|
| Title banner, major section banners, sub-section banner alternative | `#14152B` | Very dark navy, near-black |
| Accent divider bar, document subtitle text in the title banner | `#C9A961` | Muted gold |
| Sub-section banners | `#595959` | Medium-dark gray |
| Status/tag line text | `#2F7D4F` | Green, bold, all-caps, used sparingly |
| Callout/note box background | `#F2F2F2` | Light gray, with a thin `#DDDDDD` border if a border is needed |
| Byline / footer text | `#555555` | Gray, italic |
| Banner text | `#ffffff` | White, bold |

All banner text is **left-aligned**, not centered. This was the second mistake in the earlier guessed version, along with the wrong colors.

## Minimal HTML skeleton

```html
<table style="width:100%; border-collapse:collapse;">
<tr><td style="background-color:#14152B; color:#ffffff; text-align:left; padding:14px 20px 4px 20px; font-size:14pt; font-weight:bold; letter-spacing:0.5px;">FIERCE STAFFING SERVICES AND CONSULTING</td></tr>
<tr><td style="background-color:#14152B; color:#C9A961; text-align:left; padding:2px 20px 14px 20px; font-size:11pt; font-weight:bold;">[Document Title]</td></tr>
</table>
<table style="width:100%; border-collapse:collapse; margin-bottom:10px;">
<tr><td style="background-color:#C9A961; height:12px; padding:0;"></td></tr>
</table>
<p style="font-style:italic; font-size:10pt; color:#555555;">Official [audience] reference &mdash; [note] &nbsp;|&nbsp; Last updated [date]</p>

<!-- major section -->
<table style="width:100%; border-collapse:collapse; margin-top:20px;">
<tr><td style="background-color:#14152B; color:#ffffff; text-align:left; padding:8px 16px; font-size:12pt; font-weight:bold;">SECTION NAME</td></tr>
</table>

<!-- optional status/tag line, only when a real status concept exists -->
<p style="color:#2F7D4F; font-weight:bold; margin-top:8px;">ACTIVE STATUS LABEL</p>

<!-- sub-section -->
<table style="width:100%; border-collapse:collapse; margin-top:12px;">
<tr><td style="background-color:#595959; color:#ffffff; text-align:left; padding:6px 16px; font-size:11pt; font-weight:bold;">SUBSECTION NAME</td></tr>
</table>
<p>Body text.</p>

<!-- callout / note box -->
<table style="width:100%; border-collapse:collapse; margin-top:12px; background-color:#F2F2F2;">
<tr><td style="padding:12px 16px;">
  <p style="font-weight:bold; margin:0 0 6px 0;">Notes for whoever's updating this doc:</p>
  <ul style="margin:0;"><li>Note item.</li></ul>
</td></tr>
</table>
```

Follow the general write procedure in `google-doc-edit-SOP.md` (search for the current file id, compose full content, `create_file` with `contentMimeType: text/html`, verify with `read_file_content`, `trash_file` the old id). This template only governs the HTML structure and colors, not the write mechanics.

## Applied to

- "JazzHR Candidate Status Guide" — rebuilt 2026-08-19 against this corrected, screenshot-verified version.
- Not yet retrofitted: the two existing Client Profile documents (ABCD & Company, Splendid Affairs) predate this template and were themselves the design reference; they don't need to change since they're already the pattern.
