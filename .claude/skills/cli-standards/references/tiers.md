# Tiers & applicability — how much of the standard applies

Two orthogonal dials decide depth. **Tier** is ambition (how far along the
SHOULD space and lifecycle rules go). **Applicability** is features (which
rule groups are even in play). Settle both before plan, review, or audit.

## Settling the dials

One AskUserQuestion for tier — recommendation first, derived from context
(existing repo signals: published package metadata, CI, README install
instructions ⇒ lean publishable; single-file script ⇒ lean minimal).
Applicability is usually inferable from the code or the plan; confirm the
inference inside the same gate's option descriptions rather than asking a
second question. Honor notes ("standard, but it will go public next quarter"
⇒ audit at standard, flag the publishable delta).

## Tiers

### minimal — every CLI, even a one-evening tool

The floor. Applicable-MUST subset that makes a tool predictable and safe:

- Profile settled: noun-verb default, or Appendix A verb-first with its entry
  criteria met (§1, Appendix A).
- `--help`/`-h` and `--version`/`-V` — stdout, exit 0 (R4.1).
- kebab-case commands and long flags; lowercase binary (R1.4, R3.3).
- `--` terminator honored; `-` = stdin/stdout where files are taken (R3.1, R3.2).
- Exit codes: `0` success, `1` error, `2` usage; `130` on SIGINT (R6.1).
- stdout = requested data, stderr = diagnostics; errors to stderr (R7.1, R7.6).
- Bare invocation → help/stdout/0; unknown command → usage/stderr/2 (R7.9).
- Destructive actions confirm on a TTY and have `--force`/`--yes` (R8.1, R8.2) — if any exist.
- No secrets accepted via argv (R5.5).

SHOULDs at this tier: noted when trivially cheap, not pressed.

### standard — the default; team and org tools

Everything in minimal, plus all applicable MUSTs and active SHOULD review:

- Full required global core: `-v` repeatable, `-q`, `--debug`, `--config` (R4.1, R4.4).
- Machine output where anything scripts it: `-o`/`--output`, `--json` ≡ `-o json`,
  single-object JSON error schema (R4.2, R7.2, R7.8).
- Config/env/XDG chain if configurable: precedence, `<tool>_config.toml`
  paths, `TOOL_*` env mapping (R5.1–R5.4).
- Full exit-code semantics `0/1/2/3/4/5/130/143` (R6.1); partial failure → 1 (R6.3).
- Non-interactive path for every prompt; `--no-input` (R8.5); `--dry-run` on mutations.
- Deterministic documented `list` ordering; deprecations warned (R9.2).
- Waived SHOULDs documented in the conformance note — the note becomes mandatory here.

### publishable — production-grade, public or widely depended-upon

Everything in standard, plus the full Appendix C sweep:

- Lifecycle: SemVer interface contract (R9.3), deprecation policy (R9.2),
  UTF-8 + locale-independent machine output (R9.4), clean
  SIGINT/SIGPIPE/SIGTERM (R9.6).
- Operational: telemetry posture (R9.7), release integrity if self-updating
  (R9.9), `doctor` diagnostics (R9.10), error-code catalog (R7.12),
  experimental features marked (R9.12).
- **Conformance fixtures in CI (R9.14)** — the audit's live checks, made
  permanent.
- §10 in full if networked.
- Conformance note committed, standard version pinned, audit history kept.

## Applicability axes

Independent of tier — each "yes" switches a rule group on:

| Axis | If yes, in play |
|---|---|
| Configurable? | §5 config/precedence/XDG (R5.1–R5.4, R5.8) |
| Networked? | §10 auth, targeting, pagination, wait, TLS, rate limits |
| Destructive ops? | §8 confirmation, `--force`/`--yes`, non-interactive mapping |
| Scripted consumers? | `-o`/`--json`, R7.2 output stability, R7.8 error schema |
| Long-running/async? | R10.4 wait-by-default, `--no-wait`, `--timeout` |
| Streaming output? | `jsonl` rules, `--watch`/`--follow` (R7.8, decision 39) |
| Plugins? | R9.11 plugin contract |
| Caching/offline? | R5.9 cache controls |
| Handles secrets? | R5.5 argv ban, R5.6 redaction, R10.1 token precedence |

## N/A discipline

Every rule group switched off is listed once in the deliverable as
`N/A — <axis>: no` (e.g. `§10 N/A — not networked`). A group wrongly marked
N/A is itself a finding. This is what keeps "judgment call" honest: the
judgment is visible, reviewable, and reversible when the CLI grows the
feature later.
