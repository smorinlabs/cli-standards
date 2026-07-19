# cli-standards

An agent skill that applies this repo's [CLI Design Standard](../../cli-design-standard.md)
to any CLI work, scaled to what the tool actually needs. Four modes: **plan**
(greenfield interview → profile, tier, and a rule-cited interface spec),
**check** (mid-build lookups answered with rule IDs), **review** (design
review of a spec or plan), and **audit** (conformance sweep of an existing
CLI, including sandboxed live execution checks — exit codes, stream
discipline, `--version` format). Depth scales on two dials: tier
(minimal / standard / publishable) and feature applicability (config,
networked, destructive ops, …). Deviations are never silent: waived SHOULDs
land in a `CONFORMANCE.md` note, and deviations that beat the rule become
proposed amendments to the standard itself. The skill reads
`cli-design-standard.md` live from this repo — no bundled copy, no drift.

**Triggers on:** "new CLI", "build a CLI", "add a subcommand", "review this
CLI design", "is this CLI conformant", "audit this CLI", "CLI standards",
"what does the standard say about \<exit codes / flags / config paths\>"
**Arguments:** none

## Install

**In this repo — nothing to install.** Claude Code auto-discovers
`.claude/skills/cli-standards/`; Codex discovers it through the committed
symlink `.agents/skills/cli-standards`.

**Copy into your own setup** (no dependencies):

    git clone https://github.com/smorinlabs/cli-standards
    cp -R cli-standards/.claude/skills/cli-standards ~/.claude/skills/cli-standards   # Claude Code
    cp -R cli-standards/.claude/skills/cli-standards ~/.agents/skills/cli-standards   # Codex

> Note: a copied install loses the live link to `cli-design-standard.md`
> (the skill resolves the standard relative to its own real path). Prefer
> dev mode below so the skill always reads the canonical, current standard.

**Dev mode** (edits in the clone are live next session):

    ln -s "$(pwd)/cli-standards/.claude/skills/cli-standards" ~/.claude/skills/cli-standards   # Claude Code
    ln -s "$(pwd)/cli-standards/.claude/skills/cli-standards" ~/.agents/skills/cli-standards   # Codex

## Example session

> "I'm starting a new CLI for managing deploy previews — help me design it."
> → plan mode: interviews for shape (noun-verb vs small-CLI profile), tier,
> and applicable feature axes, then writes a rule-cited interface spec
> (`docs/cli-interface.md`) and a seeded `CONFORMANCE.md` into the new
> tool's repo, pinned to the standard version it used.

> "Is `acme` ready to publish?"
> → audit mode at publishable tier: static pass over the source, sandboxed
> live probes of the binary, findings table with rule IDs (blockers first),
> then offers the conformance note, CI fixtures (R9.14), and a fix plan.
