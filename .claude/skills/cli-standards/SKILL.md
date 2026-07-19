---
name: cli-standards
description: Apply the org's CLI Design Standard (cli-design-standard.md in this repo) to any CLI work, scaled by tier (minimal / standard / publishable) and feature applicability. Four modes — plan (greenfield interview → noun-verb vs small-CLI profile, tier, interface spec), check (mid-build lookups — "what exit code / flag / config path does the standard say"), review (design review of a CLI spec or plan; findings cite rule IDs), audit (conformance sweep of an existing CLI with live execution checks; emits findings, a CONFORMANCE.md waiver note, optional CI fixtures). Fires whenever a CLI is being created, designed, extended, reviewed, or audited — "new CLI", "build a CLI", "add a subcommand", "review this CLI design", "is this CLI conformant", "audit this CLI", "CLI standards". Not for generic non-CLI architecture review (factor-architect), generic quality sweeps (factor-scan), external CLI-practice research (guided-research), or package-name availability (claim-package-name-skill).
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, AskUserQuestion
---

# cli-standards

Apply the CLI Design Standard — this repo's `cli-design-standard.md` — to a CLI
being planned, built, reviewed, or audited, at the depth that CLI actually
needs.

> **NO SILENT DEVIATION.** Every applicable MUST the CLI skips is a blocker
> finding. Every waived SHOULD gets a conformance-note entry with rule ID,
> deviation, rationale, and owner/date. Judgment calls are *recorded*, never
> just made.
>
> No exceptions: not "it's a tiny tool" (that's a tier, and tiers still have
> MUSTs), not "we'll document it later", not "the flag name is close enough".
>
> Violating the letter of this rule is violating the spirit of it.

## Locating the standard

This skill ships beside the canonical standard and reads it live — never from
memory, never from a bundled copy. The skill's base directory may be a symlink
(global placement); resolve it first, then the standard sits three levels up:

    STD="$(dirname "$(dirname "$(dirname "$(realpath "<skill-base-dir>")")")")/cli-design-standard.md"

Read the header and note the **Version** line. Every deliverable — spec,
review, audit, conformance note — pins the standard version it was produced
against. If the file is missing, stop and report; do not proceed from memory.

## Modes

Pick by what the user is doing; ask only when genuinely ambiguous.

| Mode | Fires when | Load | Deliverable |
|---|---|---|---|
| **plan** | Greenfield CLI or new command group: "new CLI", "design the interface" | `references/planning.md` | Interface spec + seeded conformance note |
| **check** | Mid-build lookup: "what exit code / flag / config path / verb…" | The relevant § of the standard | Cited answer with rule IDs; nothing written |
| **review** | A spec, plan, or diff exists but hasn't shipped: "review this CLI design" | `references/review.md` | Findings table with rule IDs |
| **audit** | An existing CLI: "is it conformant", "audit this CLI", pre-publish gate | `references/audit.md` | Findings + conformance note + optional CI fixtures |

Before plan, review, or audit: settle **tier and applicability** via
`references/tiers.md` — one AskUserQuestion, recommendation first, notes
honored. `check` skips tiering; it is a lookup, not a judgment.

## Depth scaling

Not everything in the standard applies to every CLI. Two orthogonal dials
(defined in `references/tiers.md`):

- **Tier** — ambition: `minimal` / `standard` / `publishable`. Sets how much
  of the SHOULD space is in play and how deep an audit goes.
- **Applicability** — features: config, networked, destructive ops, machine
  output, async, streaming, plugins, caching. Each switches whole rule groups
  on or off regardless of tier.

A rule that is off is reported **N/A with a reason**, never silently skipped.
An applicable MUST is never tier-waivable.

## Workflow (every mode)

1. Locate and version-pin the standard (above).
2. Identify the mode; for plan/review/audit, settle tier + applicability.
3. Execute the mode's reference procedure.
4. Cite rule IDs (`R#.#`) on every finding and answer, with MUST/SHOULD/MAY
   severity — MUST violation = blocker, SHOULD deviation = fix or waiver,
   MAY = suggestion.
5. Deviations that stay: record in the target repo's conformance note
   (render `templates/conformance-note.md`; no `{{VARS}}` may survive).
6. **Feed back upstream.** A deviation that seems *right* and generalizable is
   a proposed amendment: draft a Decision Log row + rule edit against
   `cli-design-standard.md` in this repo and offer it to the user. The
   standard evolves by amendment, not by drift.

## Red Flags

| Thought | Reality |
|---|---|
| "Tiny tool — standards are overkill" | The minimal tier is a dozen checks. Tier it, don't skip it. |
| "I remember what the standard says" | It's versioned and amended (54 decisions and counting). Read the section. |
| "That flag name is close enough" | Exact names are the contract (R3.10 bans abbreviation for a reason). Cite the rule, use its name. |
| "I can audit from the source alone" | Half of Appendix C is runtime behavior. Run the binary (sandboxed) or mark those rules unverified. |
| "This deviation is fine, moving on" | Fine → waiver entry. Generalizable → amendment. Never just "moving on". |

## See also

- `factor-architect` — generic (non-CLI) architecture review.
- `factor-scan` — generic code-quality sweeps; conformance is not quality.
- `guided-research` — researching external CLI practice; this skill applies
  the settled internal standard.
- `claim-package-name-skill` — name availability; plan mode hands off to it.
- `repo-please-setup` — release plumbing once a publishable-tier CLI ships.
