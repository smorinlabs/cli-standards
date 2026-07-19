# review mode — design review of a spec, plan, or unshipped diff

Review the *interface* against the standard before it hardens into shipped
behavior. Input is whatever exists: an interface spec, a Superpowers plan, a
PR diff, a README draft, or help text. This is not a code-quality review
(that is factor-scan) and not generic architecture review (factor-architect)
— every finding here traces to a rule ID.

## Procedure

1. Tier + applicability settled first (`references/tiers.md`).
2. Sweep the input against the standard **in section order**, skipping
   switched-off groups: §1–2 structure/vocabulary → §3–4 flags/standard
   options → §5 config/env → §6 exit codes → §7 output/streams → §8
   safety → §9 lifecycle → §10 networked. Read each section from the live
   standard as you sweep — do not review from memory.
3. Check the *cross-cutting traps* the Decision Log exists for, common in
   otherwise-clean designs: `-v` means verbose not version (decision 3);
   `get` is not a resource verb — `view`/`describe` (decision 20); `set`
   reserved for config (decision 42); `-f` is force, file input is `--file`
   (decision 22); `-o` is output format, never destination file (decision
   18/25); no prefix abbreviation (decision 32); plural nouns only as silent
   aliases (decision 5).
4. Where the input is silent on an applicable area (no exit-code mapping, no
   non-interactive story), that is a **gap finding**, not a pass.

## Findings format

One table, ordered blockers first:

| Rule | Level | Where | Finding | Fix |
|---|---|---|---|---|
| R3.4 | MUST | `deploy -f <file>` | `-f` used for file input | `--file`; reserve `-f` for `--force` |

- **MUST violation** → blocker; the design does not conform until fixed.
- **SHOULD deviation** → fix, or a waiver entry in the conformance note with
  rationale and owner/date. Offer to write the entry.
- **MAY / style** → suggestion; no tracking obligation.

Close with the N/A list (groups off, with reasons) and the standard version
reviewed against.

## After the table

1. Offer to apply agreed fixes to the spec/plan directly.
2. Waivers accepted → update the conformance note.
3. A deviation that is arguably *better than the rule* → propose a standard
   amendment (SKILL.md workflow step 6) instead of a waiver.
