---
description: Compact the current session into a self-contained handoff prompt for a new agent. 5 modes — (none) | audit | codex | gemini | compact. Captures decisions w/ rejected alternatives, owner-split open items, pre-publish flags, state-verify shell commands, STATUS tag. Saves to memory/session-handoffs/. Modes stack (e.g. `audit codex`).
---

Generate a self-contained session handoff prompt that lets a new agent (Claude / Codex / Gemini) resume this work without my context. Save it to `~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_<topic-slug>.md`.

## Argument detection (first thing)

Read the slash-command arguments. Route per this table. Multiple arguments stack — run each section in order.

| Argument   | What it does                                                                                  |
|------------|-----------------------------------------------------------------------------------------------|
| (none)     | Standard handoff for resuming this work in a fresh Claude session                             |
| `audit`    | ALSO produce sibling `AUDIT_REPORT.md` for an independent reviewer (see AUDIT MODE)           |
| `codex`    | Scope handoff for Codex execution — file allowlist + paste-ready `codex exec` block (CODEX MODE) |
| `gemini`   | Scope handoff for Gemini CLI — convert Claude tool names + `activate_skill` invocation (GEMINI MODE) |
| `compact`  | Skip decision archaeology — produce a lightweight 50-line summary only (COMPACT MODE)         |

Aliases accepted: `with audit`, `+ audit`, `audit + codex`, etc. Order-independent.

If no argument: just produce the standard handoff (Steps below).

## Steps (standard handoff)

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

- **Self-contained.** A fresh agent reading the handoff + workspace CLAUDE.md should know everything needed to continue. No "as we discussed", no "the X from earlier".
- **Absolute paths.** Future-you can't infer the working directory from "_master/scheduler/postiz_push.py" — write the full path.
- **No secrets.** Reference `.env.master` keys by name only (e.g. `POSTIZ_API_KEY`). Never paste values.
- **Every line load-bearing.** No conversational filler. No "we did some great work today." Cut anything that won't help the next session decide what to do.
- **Cap ~200 lines** if possible. If the session was huge, prioritize: decisions > file paths > open items > everything else.
- **No new files invented.** Only reference what's actually on disk — verify file paths exist before writing them into the handoff.
- **If session is conversational only** (no real artifacts shipped): push back. Tell Dainer there's nothing to hand off; offer to log a quick note in `daily-logs/` instead. Or suggest `/handoff compact` if he just wants a context-clear.

## Filename pattern

```
~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_<topic-slug>.md
```

Example: `2026-05-04-1900_apex-linkedin-pipeline.md`

Mode-specific filename infixes:
- `codex` mode → `..._codex_<slug>.md`
- `gemini` mode → `..._gemini_<slug>.md`
- `compact` mode → `..._compact_<slug>.md`

So future-you can tell at a glance whether a file is a real handoff, a Codex/Gemini handoff, or just compaction.

## After saving

End your reply with one line telling Dainer the next-chat command:

```
Next chat: open new session, paste the contents of <filename> as your first message. Or reference it: "Pick up from <filename>".
```

That's it. Tight, honest, resumable.

---

## CODEX MODE (only when `codex` argument was passed)

Codex is better than Claude for: multi-file refactors, filesystem ops, test runs, ledger audits, send-prepare scripts, verification against live state. Codex is not better for: voice work, brand decisions, fuzzy strategy. Don't hand voice or strategy to Codex.

When `codex` is passed, adjust the standard handoff:

1. **File allowlist (mandatory).** Explicit list of files Codex is permitted to touch. No wildcards. No "anywhere in the repo." Without this, Codex audit-balloons — touching unrelated files and breaking shared interfaces.

2. **Acceptance criterion.** One sentence Codex can grep for. Pass / fail clean. Example: `tests/test_preflight.py::test_blank_template_rejected PASSES + sent_log.jsonl count unchanged`.

3. **STATUS tag requirement.** Codex must end its run with `STATUS: VERIFIED / UNVERIFIED / BROKEN` per the falsification rule. State this in the handoff explicitly.

4. **Cost cap (if relevant).** Max-token or max-minute budget. Codex respects this.

5. **Read these first** — include workspace `CLAUDE.md` + business `CLAUDE.md`. Codex reads these via `AGENTS.md` siblings that `@import` `CLAUDE.md`.

6. **Skip** the standard "pre-publish flags" section — Codex doesn't publish; that's a Claude/Dainer lane.

7. **End the file with a paste-ready block:**

```
codex exec --skip-git-repo-check '<one-sentence task + file allowlist + acceptance criterion>'
```

Dainer copies that one line, pastes into his terminal, Codex starts. No handoff-to-paste-to-prompt translation step.

### CODEX MODE filename

```
~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_codex_<topic-slug>.md
```

---

## GEMINI MODE (only when `gemini` argument was passed)

Same shape as CODEX MODE, retargeted for Gemini CLI. Gemini activates skills via `activate_skill` (not Claude's `Skill` tool). Tool names differ.

When `gemini` is passed, adjust the standard handoff:

1. **Tool-name mapping note.** Short translation table at the top of the handoff if any Claude-specific tools were used. Common mappings:

   | Claude tool       | Gemini equivalent              |
   |-------------------|--------------------------------|
   | `Read`            | `read_file`                    |
   | `Write`           | `write_file`                   |
   | `Edit`            | `replace`                      |
   | `Bash`            | `shell`                        |
   | `Skill`           | `activate_skill`               |
   | `Grep` / `Glob`   | `search_file_content` / `glob` |

2. **Read these first** — include `GEMINI.md` if the project has one. Otherwise note: "Gemini reads `AGENTS.md` which `@imports` `CLAUDE.md`."

3. **Skip** Codex-specific blocks. Don't mention `codex exec`.

4. **End the file with a paste-ready prompt** Dainer can run in the Gemini CLI (or paste as the first message in a Gemini session).

### GEMINI MODE filename

```
~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_gemini_<topic-slug>.md
```

Gemini handoff is the least common in my own workflow but the format is generic enough to keep parity.

---

## COMPACT MODE (only when `compact` argument was passed)

Lightweight save-state. NOT a real handoff. Use when:

- The session was conversational only (research, brainstorming, chatting)
- You just want to clear context before continuing in the same model
- You don't need to capture decisions / rejected alternatives / owner-split next-actions

Output is shorter (~50 lines, not ~200):

1. **Header** — date + one-line topic
2. **What we talked about** — bullet list, 5-15 items
3. **Open threads** — what was left mid-thought (if any)
4. **Files referenced** — paths discussed but not modified

Skip: decisions w/ rejected alternatives, owner split, pre-execution flags, verify-on-resume commands, STATUS tag, TODO.md pointer logic.

### COMPACT MODE filename

```
~/Desktop/DAINER OS/memory/session-handoffs/YYYY-MM-DD-HHMM_compact_<topic-slug>.md
```

If the session DID have decisions or shipped artifacts and Dainer accidentally called `compact`: push back. Say "session had real artifacts — want a full handoff instead?"

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
11. **STATUS tag** — VERIFIED / UNVERIFIED / BROKEN.

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

---

## Stacked modes

Arguments combine. Order doesn't matter.

- `/handoff audit codex` → standard handoff + AUDIT_REPORT.md, AND open-items section scoped for Codex (file allowlist + `codex exec` block at end).
- `/handoff codex gemini` → don't. Pick one execution target. If you really need both, run twice with different slugs.
- `/handoff audit gemini` → standard handoff + AUDIT_REPORT.md, AND open-items section scoped for Gemini.
- `/handoff compact + anything else` → `compact` wins. Compaction skips the rest of the modes.

---

## Workflow examples

**Resume tomorrow in a fresh Claude session** (most common):

```
/handoff
```

Tomorrow: open a new chat, paste the contents of the saved file as your first message. Or reference it: "Pick up from `<filename>`."

**Hand to Codex for execution after Claude designed the approach**:

```
/handoff codex
```

Claude designed; Codex executes the file edits. The handoff bakes in a file allowlist so Codex doesn't audit-balloon. End of file = paste-ready `codex exec` line.

**Independent review before shipping a decision**:

```
/handoff audit
```

Produces standard handoff + `AUDIT_REPORT.md`. Pipe the audit straight to Codex with the paste-ready block at the bottom:

```
codex exec "Read <audit-report-absolute-path>. Answer §10. Write responses to sibling AUDIT_RESPONSE_<slug>.md."
```

Codex writes back. Review the response before committing.

**Cross-tool handoff to Gemini CLI**:

```
/handoff gemini
```

Same structure, retargeted for Gemini's tool names and `activate_skill` invocation pattern.

**Quick context-clear after a long conversational session**:

```
/handoff compact
```

Lightweight 50-line save-state. Filename gets `compact_` prefix. NOT a real handoff. Use when the session had no real artifacts.

**Stacked — audit + scoped for Codex**:

```
/handoff audit codex
```

Standard handoff + `AUDIT_REPORT.md` + Codex-scoped open-items. One slash command, three artifacts.

**Resume a 3-week-old thread**:

```bash
ls ~/Desktop/DAINER OS/memory/session-handoffs/ | grep apex-linkedin
```

Pick the file. Paste it as the first message in a new session. The verify-on-resume shell commands at the bottom of the file confirm nothing drifted while you were away.
