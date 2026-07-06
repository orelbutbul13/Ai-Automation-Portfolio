# AI Legal Document Translation Pipeline

End-to-end automation for certified legal document translation built in n8n with GPT-5 Vision, PII tokenization, glossary-aware AI translation, human-in-the-loop review (with edit and reject/retranslate paths), and automated certificate delivery as a formatted Google Doc, DOCX, and PDF.

Client project for a US-based certified legal translation firm (NDA).

**[→ View Live Project Page](https://orelbutbul13.github.io/Ai-Automation-Portfolio/AI-Legal-Document-Translation-Pipeline/demo.html)** Full pipeline walkthrough: architecture, all 3 AI prompts, bug table, business analysis, and real pipeline output screenshots

**[→ View Case Study](https://docs.google.com/document/d/16NSTzLTynZVfyxZReohpFcyNgvYT1bg8Ncot4laQRlk/edit)** Full written case study including all project links, privacy architecture decision, and outcomes

## Overview

This pipeline replaces a manual certified translation process. A scanned legal document arrives in a Google Drive folder. GPT-5 Vision extracts the text. A PII detection step replaces all client names, addresses, and ID numbers with placeholder tokens before any translation call, so the AI model never sees real client identities. The tokenized text is translated in formal legal register using a glossary of known term pairs, which grows with every document processed. Execution then pauses at a human review step, where a certified reviewer chooses one of three decisions via an email form: **Approve** as-is, **Approve with Edits** (paste a corrected translation, which flows straight into the final certificate), or **Reject** with correction notes, which triggers an automatic retranslation incorporating those notes and a second review round before falling back to manual escalation if rejected again.

On approval, the translation is assembled into a certified document: a bordered metadata table (certificate ID, language pair, document type, hash, reviewer) built directly via the Google Docs API, the firm's real header/footer/signature branding images, the original scanned source pages embedded for reference, and a certification statement — delivered as a Google Doc, DOCX, and PDF, with the source file archived and a processed-document log updated automatically.

## Business Problem

Certified legal translation firms handle documents that demand accuracy at a level where a single inconsistent term can cause a court filing to be rejected. A case may span 10 or more documents: a divorce decree, death certificate, marriage certificate, bank statements. Every one must use identical legal terminology. Manual translation is slow, inconsistency across translators is a real risk, and sending raw client documents containing passport numbers and home addresses to third-party AI services raises serious compliance concerns. This pipeline addresses all three.

## Pipeline Architecture

**[→ View Workflow Screenshot](screenshots/workflow.png)** Full n8n canvas showing all pipeline stages.

```
Drive Trigger → Download → Hash Check → Move to Processing
→ GPT-5 Vision OCR → Parse Output
→ GPT-5 PII Detection → Parse Output → Tokenize
→ Glossary Lookup → Build Translation Input → GPT-5 Translation
→ Parse Output → Save New Glossary Terms
→ Detokenize → Send Review Notification → Wait Node (pipeline suspended)
                                                 ↑
                                     SLA Monitor (hourly poll, separate workflow)
                                     One reminder/escalation email per tier,
                                     never repeated: 4h → 12h → 24h → 48h
→ Apply Review Decision
    Approve            → Assemble Certificate (Docs API table + branding +
                          source pages) → PDF/DOCX export → Drive Upload
                          → Log + Archive → Send Completion Email
    Approve with Edits  → same as above, using the reviewer's corrected text
    Reject              → Retranslate with reviewer notes → second review
                          → Approve path above, or escalate to manual
                          translation if rejected again (max 1 retry)
    Any error            → Log to Sheet → Notify Admin → Move to Errors Folder
```

The pipeline has three visual sections in n8n. The top row is the main happy path. The right section handles the reviewer reject-and-retranslate loop, including a second review step. The bottom row is the error branch that every node routes to on failure. A separate companion workflow (SLA Monitor) polls hourly for stuck reviews and escalates through four tiers without ever repeating a notification.

## The Three AI Prompts

**Prompt 1: Document Reading**
Sent to GPT-5 Vision with the scanned document image. Instructs the model to extract all text in reading order, detect page boundaries, and explicitly flag stamps, seals, and handwritten annotations rather than silently dropping them. Returns structured JSON per page. Works on any source language without configuration changes.

**Prompt 2: PII Detection**
Sent on the extracted text before any translation call. Identifies every name, address, phone number, ID number, date of birth, and case number. Returns a structured entity list with a stable sequential token assigned to each value. The same name always receives the same token across all pages. A Code node performs the actual substitution and the token map lives only in execution memory, never written to any file or log.

**Prompt 3: Legal Translation**
Sent on the tokenized text with glossary term pairs injected. Instructs the model to produce a complete translation without summarizing or omitting anything, preserve all placeholder tokens verbatim, use formal legal register appropriate for court filings, follow the injected glossary terms exactly, and flag any term where no established legal equivalent exists. Returns the translated pages, a list of flagged terms, and any new glossary terms to be saved back to the sheet.

## Tech Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n (self-hosted) |
| OCR | OpenAI GPT-5 Vision |
| PII Detection and Translation | OpenAI GPT-5 |
| Document Intake and Output | Google Drive API |
| Certificate Assembly | Google Docs API (bordered metadata table, inline images, batchUpdate) |
| DOCX/PDF Conversion | pdf.co |
| Glossary and Logs | Google Sheets API |
| Notifications | Gmail API |
| Token Logic and Assembly | JavaScript (n8n Code nodes) |

Self-hosted n8n means all client documents stay on local infrastructure with no per-execution cost at scale. The visual workflow canvas makes every step auditable without reading code.

## Pipeline Screenshots

**[→ View Workflow Screenshot](screenshots/workflow.png)** Full n8n canvas showing all pipeline stages.

**[→ View SLA Monitor Screenshot](screenshots/sla-monitor-workflow.png)** Companion workflow that monitors the human review queue and escalates through 4 tiers (4h/12h/24h/48h), each sent exactly once.

**[→ View Review Email Screenshot](screenshots/review-email-v2.png)** Pipeline paused waiting for human decision.

**[→ View Completion Email Screenshot](screenshots/completion-email-v2.png)** Certified translation approved and delivered.

### Review Decision Examples

Each of the three reviewer decisions verified end-to-end during a QA pass covering 3 languages and 3 document types:

**[→ Approve](screenshots/review-decision-approve.png)** — Turkish divorce decree, approved as-is. [Final certificate](https://drive.google.com/file/d/1_8NP24xB2Yz-AJaO4OoTBbdtLkd4x0N0uBhECoLWewY/view) · [PDF](https://drive.google.com/file/d/1Yuu1DEXswiVXW2ynPgAuMZ7XOkSCzqVR/view)

**[→ Approve with Edits](screenshots/review-decision-approve-with-edits.png)** — Japanese birth certificate. The AI masked names as PII before translation, so restored names came back in Japanese script in the English certificate; the reviewer romanized all four names and the correction flowed straight into the final document. [Final certificate](https://drive.google.com/file/d/1Bl-IEKgqcTOdZfGMeDW6UOlbjNTzrcQ5kLBghrdqWY8/view) · [PDF](https://drive.google.com/file/d/16rh_jX79Oe_sqjRwo-MK-FofQzOUqW6q/view)

**[→ Reject → Retranslate → Approve](screenshots/review-decision-reject-retry.png)** — Spanish power of attorney, rejected for inconsistent terminology, automatically retranslated using the reviewer's notes, approved on second review. [Final certificate](https://drive.google.com/file/d/15cxTV1WW2C_D6xrXDr64oIa1QX_Pn8Nk73MeeXds9-c/view) · [PDF](https://drive.google.com/file/d/1a_92O1ACaj-gBZuGpuTzHrXZkjiw0s7Y/view)

## Challenges Solved

| Problem | Root Cause | Fix |
|---|---|---|
| Google Doc opened empty | Extra line break in multipart MIME boundary broke Drive API parsing, creating a 0-byte unnamed file | Switched from raw string body to binary Buffer; n8n reads the mimeType from the binary field automatically |
| Completion email showed N/A for all reviewer fields | After Move to Archive, the node output is Drive metadata and all pipeline data was lost | Added a Restore Pipeline Code node that re-reads from the earlier Merge node using a $() reference |
| Crash on certificate assembly | fileHash.slice(0,8) threw when hash was undefined after the review form failed to carry pipeline state | Changed to (fileHash \|\| 'NOCERT').slice(0,8) |
| Drive trigger ignored files after workflow restart | Reactivating the workflow resets the trigger checkpoint to now, making files already in the Inbox invisible | Set staticData.lastTimeChecked via the n8n REST API to a time before the upload |
| $helpers unavailable in Code nodes | n8n Code nodes run in an external task runner where $helpers is not defined | Replaced the Code node API call with a dedicated HTTP Request node using predefined OAuth2 credential |
| Certificate assembly failed only on edited translations | Reviewer-pasted text from an HTML form textarea uses `\r\n` line breaks, but the Docs API collapses `\r\n` into a single character — the running index used for text-styling requests drifted out of sync and referenced a position past the real document end | Normalize `\r\n` → `\n` on all reviewer-submitted text before it reaches the certificate-assembly Code node |
| Reject path silently fell into generic error handling instead of the retry loop | The reject-notification email node had a literal line break inside a plain JS string expression, which is invalid syntax — the node's error-continuation output fired instead of proceeding normally | Rewrote the expression using `\n` escape sequences instead of literal newlines |
| Retry loop couldn't find the file to move, and later steps saw an incomplete pipeline object | A Google Sheets "append row" node returns the row it just wrote as the item's new data, discarding every other field from the incoming item — three downstream nodes were still reading fields via the now-empty `$json` | Reference `$('Apply Review Decision').first().json` explicitly instead of `$json` in each downstream node that needs the original pipeline data |
| Retry-limit check routed backwards | The "max retries exceeded" condition's true/false outputs were wired to the wrong branches — reaching the limit triggered another retry instead of escalating, and staying under the limit escalated instead of retrying | Swapped the two output connections on the retry-limit If node |
| Second review form never showed the retranslated text | The review form's comparison-table expression referenced an undefined variable, a copy-paste leftover from the first review form that was never adapted for the retry context | Defined the missing variable against the correct upstream node |
| SLA monitor sent the same reminder dozens of times | The monitor polled hourly and re-sent a notification every time an execution was still over a time threshold, with no memory of what had already been sent | Track sent-tier state per execution in n8n workflow static data (`$getWorkflowStaticData`), so each of 4 tiers (4h/12h/24h/48h) fires exactly once |

## QA Validation

Tested end-to-end with realistic (not placeholder) documents across 3 languages and 3 document types, covering all three reviewer decisions:

| Document | Language Pair | PII Entities Caught | Reviewer Decision |
|---|---|---|---|
| Divorce decree | Turkish → English | 8/8 (names, national ID, address, case numbers) | Approve |
| Birth certificate | Japanese → English | 9/9 (names, personal number, registration number, mixed-era dates) | Approve with Edits |
| Power of attorney | Spanish → English | 7/7 (names, ID numbers, addresses) | Reject → Retranslate → Approve |

Every run correctly auto-detected source language and document type from real document content, tokenized 100% of the PII entities present with zero leakage into the translation model, and produced a properly formatted, downloadable DOCX and PDF certificate.

## Key Outcomes

Full pipeline from scanned PDF to certified Google Doc, DOCX, and PDF with zero manual steps in the standard approval path. Client PII is never exposed to the translation model, enforced by the architecture rather than by trust or policy. The glossary compounds in quality with every document processed, automatically enforcing terminology consistency across every document in a case file. Any language pair is supported with no configuration changes, proven across 3 languages during QA. A full reject-and-retranslate loop lets a reviewer send a translation back with correction notes and get a revised version for a second look, with automatic escalation to manual translation if the retry is rejected too. Every failure is caught, logged, and escalated. Nothing fails silently.

## Skills Demonstrated

Privacy-preserving AI pipeline design with in-memory PII tokenization. Prompt engineering for the legal domain with structured output enforcement, register constraints, glossary injection, and uncertainty flagging. Multi-model AI orchestration across three isolated prompt stages. Human-in-the-loop architecture using the n8n Wait node for cost-free execution suspension, including a full reject/retranslate/re-review loop with max-retry escalation. Google Docs API document assembly: bordered tables, precise inline image sizing, and index-safe batchUpdate ordering. Tiered, state-tracked notification design to prevent alert fatigue. Workflow-wide error handling and operational logging. Systematic root-cause debugging across a multi-stage async pipeline, verified with real evidence (execution data, not just success/fail status) rather than assumed fixes.
