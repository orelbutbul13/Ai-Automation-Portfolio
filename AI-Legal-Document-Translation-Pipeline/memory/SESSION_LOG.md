# Project Memory / Session Log

Running history of major architecture decisions, bugs found, and fixes made on this project, kept alongside the code so context isn't lost between sessions. Newest entries at the top.

---

## 2026-07-05 — Rebuild + comprehensive QA pass

**What changed:**
- Certificate metadata switched from plain text to a real bordered table, built directly via the Google Docs API (`insertTable` + per-cell `insertText`, filled in descending index order to avoid index drift).
- Real client-provided branding (header, footer, signature images from the client's Dropbox) wired in with explicit `objectSize` so images render at the correct physical size instead of native pixel dimensions.
- Original scanned source pages now get rasterized (via pdf.co) and embedded into the certificate for reference, as the spec requires.
- Actual DOCX and PDF export added (previously only a native Google Doc was produced).
- SLA Monitor companion workflow rebuilt: was re-sending the same reminder/escalation email every single hourly poll once a threshold was crossed (one stuck test execution generated 20+ duplicate emails). Now tracks which of 4 tiers (4h / 12h / 24h / 48h) have already fired per execution using n8n's workflow static data, so each tier notifies exactly once.

**Bugs found and fixed during this session's QA pass** (all confirmed via direct execution-data inspection, not just "it ran without erroring"):

1. **`\r\n` index drift in certificate assembly.** Reviewer-submitted text via an HTML form textarea comes back with `\r\n` line endings (per the HTML form-submission spec). The Google Docs API collapses `\r\n` into a single character internally, but the assembly Code node was counting `text.length` in JS (which counts `\r\n` as 2 characters), so its running index drifted out of sync after enough line breaks and eventually referenced a position past the real end of the document. Fixed by normalizing `\r\n` → `\n` on all reviewer-submitted text as soon as it's captured.
2. **Reject-path notification email silently misrouted.** The email node's message template had a literal newline character embedded inside a plain JS string expression — invalid syntax. n8n's `continueErrorOutput` caught the failure and routed to the generic error-cleanup branch instead of the intended retry path, without ever surfacing an error to the user. Fixed by using `\n` escape sequences instead of literal line breaks.
3. **Google Sheets append node silently dropped the pipeline object.** After logging a rejected review to the Review Queue sheet, the node's output became just the row that was written (per how Sheets append nodes work in n8n) — every other field from the incoming pipeline data was gone. Three downstream nodes (a Drive move, a Code node, and an email node) were still reading fields off the now-narrowed `$json` and broke in different ways (file-not-found, undefined values in an email, incomplete data reaching the retry logic). Fixed by having each of those nodes reference `$('Apply Review Decision').first().json` explicitly instead of relying on implicit passthrough.
4. **Retry-limit check wired backwards.** The If node checking "has the retry limit been exceeded?" had its true/false branches connected to the wrong targets — exceeding the limit triggered another retry attempt, and staying under the limit escalated to manual translation. Fixed by swapping the two connection targets.
5. **Second review form showed a blank comparison table.** The HTML field on the retry review form referenced a variable (`pipe`) that was never defined in that expression — a copy-paste leftover from the first review form's version of the same field, which does define it. This meant a real reviewer could never actually see the retranslated text before deciding whether to approve or reject it again. Fixed by defining `pipe` against the correct upstream node (`Detokenize Retry`).

**QA results** (realistic, non-placeholder documents, not just smoke tests):

| Document | Language Pair | PII Entities | Decision Path |
|---|---|---|---|
| Divorce decree | Turkish → English | 8/8 caught | Approve |
| Birth certificate | Japanese → English | 9/9 caught | Approve with Edits |
| Power of attorney | Spanish → English | 7/7 caught | Reject → Retranslate → Approve |

All three review decision paths (Approve / Approve with Edits / Reject-and-retranslate) verified working end-to-end across three languages and three document types, with zero PII leakage into the translation model in every run.

**Known open items:**
- n8n canvas sticky notes still need repositioning (cosmetic only, from being dragged around during development).
- SLA Monitor's 12h/24h/48h tiers are fixed but have not yet been observed firing in real time (only the 4h tier has actually been seen firing live) — the logic was traced through and is symmetric with the 4h tier, but real-time confirmation of the longer tiers is still pending.

---

## Earlier (pre-2026-07-05)

Initial build: 7-stage n8n pipeline (Drive intake → OCR → PII tokenization → glossary-aware translation → detokenization → human review via Wait node → certificate delivery). At that point the certificate was a plain-text Google Doc with placeholder branding, no DOCX/PDF export, and no source-page images — all fixed in the 2026-07-05 rebuild above. See the main README's Challenges Solved table for the bugs found and fixed during that initial build (MIME boundary bug, pipeline-data-loss-after-archive bug, undefined fileHash crash, Drive trigger checkpoint reset, `$helpers` unavailability in Code nodes).
