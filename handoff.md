---
description: Compact the current session into a self-contained handoff prompt for a new Claude chat. Captures decisions, file paths, open todos, pre-publish flags, and an optional Codex audit question. Saves to memory/session-handoffs/. Add `audit` argument (`/handoff audit`) to ALSO produce a sibling AUDIT_REPORT.md for Codex independent review when the session has decision archaeology.
---

Generate a self-contained session handoff prompt that lets a new Claude session resume this work without my context. Save it to `~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_<topic-slug>.md`.

**Argument detection (first thing):** If the user passed `audit` (or `with audit` / `+ audit`) in the slash command arguments, run AUDIT MODE in addition to the standard handoff — see the AUDIT MODE section near the end of this file. If no argument, just produce the handoff.

## Steps

1. **Identify business context** from the conversation: APEX / DNS / Personal / Alpha Machine / AGI / Other. Use it to load the right charter into the handoff "read these first" list.

2. **Pick a topic slug** — short, kebab-case, scope-specific (e.g. `apex-linkedin-pipeline`, `alpha-machine-edition-12`, `dns-leads-q2-cleanup`). Append it to the filename.

3. **Read state**:
   - `memory/TODO.md` (current resumption point)
   - `memory/daily-logs/<today>.md` if exists
   - Files modified in THIS session (use your conversation context — don't re-read what's already in your head)

4. **Produce a single markdown handoff** with these sections, in order:
   - **Header** — business + date + one-line topic
   - **Read these first** — relative or absolute paths to load before doing anything (workspace CLAUDE.md, business CLAUDE.md, project README, the new handoff itself)
   - **What got built this session** — terse, with absolute file paths and line counts. Group by sub-area if multiple.
   - **Decisions made** — numbered list. Each: what was decided + 1-line rationale. Include rejected alternatives so the new session knows what NOT to revisit.
   - **Open items / what's next** — split by owner (Dainer / Codex / Claude). Each item: action + acceptance criterion.
   - **Pre-execution flags** — if applicable (numbers to verify, secrets needed, services that must be up). Skip section entirely if irrelevant.
   - **Audit ask** — if Dainer asked for a second opinion before execution: a specific question block for Codex or another model. Each question concrete and answerable, not "thoughts?". Skip section if not requested.
   - **Coaching note** — pull the binding constraint from the active business charter (e.g. APEX = email lane primary; Sandra block sacred 9-11pm).
   - **How to verify state on resume** — copy-paste shell commands the new session can run to confirm nothing drifted between sessions. Aim for 3-5 commands. Each with expected output one-liner.
   - **STATUS tag** — `STATUS: VERIFIED` if files are confirmed on disk; `STATUS: UNVERIFIED` if some artifacts couldn't be checked.

5. **Save the file**. Print the absolute path back to Dainer.

6. **Print a 6-10 line preview** of the handoff so Dainer can sanity-check before opening a new chat.

7. **TODO.md pointer (conditional).** Read `memory/TODO.md` first. If TODO.md's existing "🔄 In progress" / "⏭️ Exact next steps" sections describe DIFFERENT active work (e.g. TODO is tracking email sends, but this handoff is LinkedIn pipeline) → DO NOT append. TODO.md is single-track, owned by the next user-facing execution. Print a note to Dainer: `TODO.md owned by [other track]; this handoff at [path] — reference it manually when you resume.` Otherwise, if TODO.md is the same track as this handoff → append one line under "📋 Context for next session": `Resume via: ~/Desktop/DAINER OS/memory/session-handoffs/[filename].md`.

## Constraints

- **Self-contained.** A fresh Claude session reading the handoff + workspace CLAUDE.md should know everything needed to continue. No "as we discussed", no "the X from earlier".
- **Absolute paths.** Future-you can't infer the working directory from "_master/scheduler/postiz_push.py" — write the full path.
- **No secrets.** Reference `.env.master` keys by name only (e.g. `POSTIZ_API_KEY`). Never paste values. Per `feedback_secrets_never_in_chat.md`.
- **Every line load-bearing.** No conversational filler. No "we did some great work today." Cut anything that won't help the next session decide what to do.
- **Cap ~200 lines** if possible. If the session was huge, prioritize: decisions > file paths > open items > everything else.
- **No new files invented.** Only reference what's actually on disk — verify file paths exist before writing them into the handoff.
- **If session is conversational only** (no real artifacts shipped): push back. Tell Dainer there's nothing to hand off; offer to log a quick note in `daily-logs/` instead.

## Filename pattern

```
~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_<topic-slug>.md
```

Example: `2026-05-04-1900_apex-linkedin-pipeline.md`

## After saving

End your reply with one line telling Dainer the next-chat command:

```
Next chat: open new session, paste the contents of <filename> as your first message. Or reference it: "Pick up from <filename>".
```

That's it. Tight, honest, resumable.

---

## AUDIT MODE (only when `audit` argument was passed)

When the user runs `/handoff audit` (or `/handoff with audit`), produce a sibling AUDIT_REPORT.md **in addition to** the standard handoff. This is the long-form artifact for Codex / external reviewer / a future Dainer reviewing his own work before committing.

### When to actually do it

Audit mode requires real decision archaeology. If the session is single-track / routine (daily sends, bug fix, content gen, one-shot HTML), **push back**: "Session is routine — handoff alone is enough. Want me to skip audit and just do handoff?" Wait for Dainer's call.

Audit mode applies when the session has any of:

- **3+ decisions with rejected alternatives** (not just "we did X")
- **Fact-checks that informed a decision** (external claims verified with cited sources)
- **Multi-voice / multi-source synthesis** (e.g. multiple advisors → unified plan; multi-agent dispatch → compiled output)
- **Pricing / strategic / hire-fire decisions** before Dainer commits
- **A new playbook / strategy doc** that future-Dainer or Codex needs to defend

### Where to save the audit report

NOT in the handoff folder. Save next to the main artifact being audited:

| Scope | Save to |
| --- | --- |
| APEX work | `~/Desktop/DAINER OS/APEX Agency/AUDIT_REPORT_<topic-slug>.md` |
| DNS work | `~/Desktop/DAINER OS/DNS/AUDIT_REPORT_<topic-slug>.md` |
| Alpha Machine | `~/Desktop/DAINER OS/The Alpha Machine/AUDIT_REPORT_<topic-slug>.md` |
| Personal | `~/Desktop/DAINER OS/PERSONAL OS/AUDIT_REPORT_<topic-slug>.md` |
| Workspace-level | `~/Desktop/DAINER OS/audits/AUDIT_REPORT_<YYYY-MM-DD>_<slug>.md` (mkdir if needed) |

Use the same `<topic-slug>` as the handoff so the pair is obvious.

### Audit report structure (sections in order)

1. **TL;DR for reviewer** — 1 page max. What was decided, what needs verifying, the audit ask.
2. **State at compile time** — facts the decisions rest on. Each with the exact shell command to re-verify.
3. **Full source content (verbatim)** — agent outputs / interviews / research that informed decisions. NOT summaries. If 4 advisors weighed in, paste all 4 briefs in full. If 6 research files were written, table-of-contents them with key findings. The reviewer should not have to chase sources.
4. **Convergence / conflict matrix** — when multiple sources disagree, show the matrix explicitly (e.g. Kennedy yes / Taleb no on E4/E5).
5. **N decisions with rejected alternatives** — for each decision: what was chosen, why, what was rejected, risk-if-reviewer-disagrees (Low / Medium / High).
6. **Fact-checks performed** — each external claim that informed a decision: claim → result (TRUE / FALSE / UNVERIFIED) → cited URLs. The reviewer can re-verify.
7. **Tactical Q&A** — verbatim user questions + answers that informed the work (when the conversation itself was part of the reasoning trail).
8. **Internal cross-checks** — consistency check against existing workspace rules / charters. Flag any rule that might conflict.
9. **File manifest** — everything written this session, with line counts.
10. **Specific audit questions for reviewer** — grouped by category (Sanity/fact-check · Internal consistency · Operational executability · Strategic · Risk · Process). Concrete, answerable. **Aim for 10-20 questions, not 3.** Never "thoughts?". Always "verify X" or "is Y consistent with Z?".
11. **STATUS tag** — VERIFIED / UNVERIFIED / BROKEN per Rule #1.

End with one paragraph telling the reviewer where to write their response (e.g. sibling `AUDIT_RESPONSE_<topic-slug>.md` file).

### Length target

300-600 lines. Audit reports earn length by content, not bloat. If you hit 600+, prioritize: full source content > decisions w/ alternatives > fact-checks > audit questions > everything else.

### Tie the two together

In the standard handoff, under "What got built this session", add one line:

- `Audit report: <absolute path to audit report>` — for Codex / Dainer / contractor review before committing to the recommendations.

If there's an "Open items" → Codex section, add:

- `Read AUDIT_REPORT_<slug>.md and answer the questions in §10. Write responses to sibling AUDIT_RESPONSE_<slug>.md.`

### Codex invocation hint (print at end of reply, AFTER the handoff next-chat line)

```
Codex audit: codex exec "Read <audit-report-absolute-path>. Answer the questions in §10. Write responses to a sibling AUDIT_RESPONSE_<slug>.md. Short direct answers. Cite file paths or URLs for any factual claim. Flag any HIGH-confidence disagreement with the recommendations."
```

That gives Dainer a single copy-paste to fire the audit. He runs Codex when he's ready; the audit response lives next to the audit report; the loop closes.
