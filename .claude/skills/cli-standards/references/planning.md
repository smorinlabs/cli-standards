# plan mode — greenfield CLI (or a new command group)

Turn the standard into a concrete interface spec before code exists. Process
skills (brainstorming, writing-plans) own the overall flow when active; this
mode supplies the CLI-domain content inside them.

## Interview — one question at a time, ≤4 total

Skip any question the conversation already answers; fold inferences into the
next gate's options instead of asking extra rounds.

1. **Shape.** What does the tool do, and how many resource types does it act
   on? Decides the profile: noun-verb (default, §1) vs verb-first
   (Appendix A — all three entry criteria MUST hold: ≤~7 fixed commands,
   single implicit resource, unlikely to grow). State the criteria check in
   the recommendation, and note the migration trigger (a second resource type
   ⇒ noun-verb in the next major).
2. **Tier.** minimal / standard / publishable (`references/tiers.md`).
3. **Applicability.** Confirm the inferred axes (config? networked?
   destructive? scripted consumers? async/streaming?) — present as one menu
   with the inference pre-marked, not one question per axis.
4. **Name** (only if unnamed). Hand off availability to
   `claim-package-name-skill`; lowercase binary name per R1.4.

## Deliverable 1 — interface spec

Write `docs/cli-interface.md` in the target repo (location adjustable by
note). Contents, each section citing its rules:

- **Identity**: binary name, profile, tier, standard version pinned.
- **Command tree**: every command/subcommand with one-line purpose; verbs
  from the core set (`list/view/describe/create/delete/update/apply/run/edit`,
  R2.1) or justified domain verbs (R2.x, decision 23); depth ≤3 (decision 11).
- **Flags table**: per command — long name, short (only reserved-safe ones:
  `-f`=force, file input is `--file` with no short, R3.4), type, default,
  repeatable?, negation form (R3.6).
- **Standard options**: which of the Appendix B core apply and any
  deliberately omitted (omitting a required-core option at standard+ tier is
  a waiver).
- **Exit codes**: the `0/1/2/(3/4/5)/130/143` mapping to this tool's failures (R6.1).
- **Config & env** (if configurable): precedence chain, `<tool>_config.toml`
  paths, `TOOL_*` variables exposed (R5.1–R5.4).
- **Output contract**: human default; machine formats offered; error schema
  sample (R7.8) if scripted consumers.
- **Safety**: destructive commands, their confirmation and
  `--force`/`--yes`/`--no-input` behavior, `--dry-run` coverage (§8).

Use Appendix D (worked example) as the shape to imitate; do not restate rules
the spec already satisfies — cite them.

## Deliverable 2 — seeded conformance note

Render `templates/conformance-note.md` into the target repo (default
`CONFORMANCE.md`): standard version, profile, tier, applicability map with
N/A reasons, empty waivers table. This makes deviation tracking start at
day zero, not at first audit.

## Deliverable 3 (publishable tier only) — fixture skeleton

Offer to scaffold the R9.14 conformance fixtures (see
`references/audit.md` § Live probes) as failing-until-implemented CI tests,
in the repo's test framework.
