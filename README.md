# handoff

A Claude Code slash command that produces a real session handoff — not a context compaction.

5 modes. Args stack.

```
/handoff                  → resume in a fresh Claude session
/handoff audit            → also produce AUDIT_REPORT.md for an independent reviewer
/handoff codex            → scope handoff for Codex execution (file allowlist + paste-ready `codex exec`)
/handoff gemini           → scope handoff for Gemini CLI (tool-name mapping + activate_skill invocation)
/handoff compact          → lightweight 50-line save-state (NOT a real handoff — explicitly labeled)
/handoff audit codex      → stacks: audit report + Codex-scoped open-items, one command
```

## the difference

**Compaction** = your last 80%-full chat gets compressed into a tempfile so a fresh agent doesn't start from zero. Disposable. Single-track. Forgets *why* decisions were made.

**Handoff** = a dated, project-versioned markdown file that captures:

- decisions made + their **rejected alternatives** (so the next session doesn't re-litigate)
- open items split by **owner** (you / Codex / Claude / a teammate)
- pre-execution flags (numbers to verify, services that must be up, secrets needed)
- shell commands the new session can run to confirm nothing drifted
- a `STATUS: VERIFIED / UNVERIFIED / BROKEN` tag per a falsification rule

Output: `memory/session-handoffs/YYYY-MM-DD-HHMM_<topic-slug>.md`. Resumable 3 weeks later.

## audit mode

`/handoff audit` adds a sibling 300–600 line `AUDIT_REPORT.md` for independent review before you commit to a decision:

- full verbatim source content (agent outputs, research, advisor briefs — not summaries)
- convergence / conflict matrix when sources disagree
- each decision with rejected alternatives + risk-if-reviewer-disagrees grade
- fact-checks with cited URLs (re-verifiable)
- 10–20 concrete audit questions for an independent reviewer

Pipe straight to Codex with the paste-ready block at the bottom of the audit report.

## codex mode

`/handoff codex` produces a handoff scoped for Codex execution:

- **explicit file allowlist** — no wildcards, no "anywhere in the repo" (prevents audit-balloon)
- **acceptance criterion** — one sentence Codex can grep for, pass/fail clean
- **STATUS tag requirement** — Codex ends its run with `VERIFIED / UNVERIFIED / BROKEN`
- **paste-ready `codex exec ...` block** at the end of the file — one line, one paste

Filename: `..._codex_<slug>.md` so future-you knows at a glance it's a Codex handoff.

## gemini mode

`/handoff gemini` produces the same shape, retargeted for Gemini CLI — converts Claude tool names (Read/Write/Edit/Bash/Skill) to Gemini equivalents (read_file/write_file/replace/shell/activate_skill) and ends with a paste-ready Gemini invocation.

## compact mode

`/handoff compact` is explicit, labeled compaction. Use it when the session was conversational only (research, brainstorming, chatting) and you just want to clear context. Skips decisions, rejected alternatives, owner split, STATUS tag. Filename gets a `compact_` prefix so future-you doesn't mistake it for a real handoff.

If you accidentally call `compact` on a session with real artifacts, the skill pushes back: "session had real artifacts — want a full handoff instead?"

## workflow examples

**Resume tomorrow in a fresh Claude session** (most common):

```
/handoff
```

Tomorrow: open new chat, paste the file contents as your first message. Or reference: "Pick up from `<filename>`."

**Hand to Codex for execution after Claude designed the approach**:

```
/handoff codex
```

End of file = paste-ready `codex exec` line. One paste, Codex starts with file allowlist + acceptance criterion baked in.

**Independent review before shipping a decision**:

```
/handoff audit
```

Produces handoff + `AUDIT_REPORT.md`. Pipe the audit straight to Codex:

```
codex exec "Read <audit-report-absolute-path>. Answer §10. Write responses to sibling AUDIT_RESPONSE_<slug>.md."
```

Codex writes back. Review the response before committing.

**Resume a 3-week-old thread**:

```bash
ls ~/path/to/memory/session-handoffs/ | grep <topic>
```

Pick the file. The verify-on-resume shell commands at the bottom confirm nothing drifted while you were away.

## install

```bash
mkdir -p ~/.claude/commands
curl -fsSL https://raw.githubusercontent.com/Rebelzxr/handoff/main/handoff.md > ~/.claude/commands/handoff.md
```

That's it. The slash command shows up immediately.

## adapt to your project

The file references `~/Desktop/DAINER OS/` paths and a few project codenames (APEX / DNS / Alpha Machine). Find-replace them with your own project root and project names. The structure is generic.

## related

- Matt Pocock's `handoff` (compaction): https://github.com/mattpocock/skills

## license

MIT.
