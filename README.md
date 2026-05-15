# handoff

A Claude Code slash command that produces a real session handoff — not a context compaction.

## the difference

**Compaction** = your last 80%-full chat gets compressed into a tempfile so a fresh agent doesn't start from zero. Disposable. Single-track. Forgets *why* decisions were made.

**Handoff** = a dated, project-versioned markdown file that captures:

- decisions made + their **rejected alternatives** (so the next session doesn't re-litigate)
- open items split by **owner** (you / Codex / Claude / a teammate)
- pre-execution flags (numbers to verify, services that must be up, secrets needed)
- shell commands the new session can run to confirm nothing drifted
- a `STATUS: VERIFIED / UNVERIFIED / BROKEN` tag per a falsification rule

The output lives under `memory/session-handoffs/YYYY-MM-DD-HHMM_<topic-slug>.md` — you can resume a 3-week-old thread.

## audit mode

`/handoff audit` adds a sibling 300–600 line `AUDIT_REPORT.md` for independent review before you commit to a recommendation. It contains:

- full verbatim source content (agent outputs, research, advisor briefs)
- a convergence / conflict matrix when sources disagree
- each decision with its rejected alternatives + risk-if-reviewer-disagrees grade
- fact-checks with cited URLs (re-verifiable)
- 10–20 concrete audit questions for an independent reviewer

I pipe the audit ask straight to Codex with one paste-ready command.

## install

```bash
mkdir -p ~/.claude/commands
curl -fsSL https://raw.githubusercontent.com/Rebelzxr/handoff/main/handoff.md > ~/.claude/commands/handoff.md
```

That's it. The slash command shows up immediately.

## usage

```
/handoff                  → standard handoff
/handoff audit            → handoff + sibling AUDIT_REPORT.md for an independent reviewer
```

## adapt to your project

The file references `~/Desktop/DAINER OS/` paths and a few project codenames (APEX / DNS / Alpha Machine). Find-replace them with your own project root and project names. The structure is generic.

## related

- Matt Pocock's `handoff` (compaction): https://github.com/mattpocock/skills

## license

MIT.
