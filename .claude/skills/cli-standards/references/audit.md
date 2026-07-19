# audit mode — conformance sweep of an existing CLI

Gap-analyze a real, runnable CLI against the standard: brownfield tools never
built to it, and publishable-tier tools gating a release. Depth scales with
tier; findings follow the same table format as review mode.

## 1. Locate and build

Find the binary or entry point; build if needed (respect the repo's own
`justfile`/`Makefile`/package scripts). If it cannot be run, fall back to a
static-only audit and mark every runtime rule **unverified** — never inferred-
pass.

## 2. Sandbox the live runs

Live probes must not touch real user state:

- Scratch environment: point `XDG_CONFIG_HOME`, `XDG_DATA_HOME`,
  `XDG_STATE_HOME`, `XDG_CACHE_HOME` (and `HOME` if the tool ignores XDG) at
  a temp directory.
- **Never execute mutating or destructive verbs against real resources.**
  Probe them only via `--help`, `--dry-run`, or against scratch fixtures.
  Networked tools: read-only endpoints only, and only with explicit user
  go-ahead.

## 3. Static pass (source + docs)

Grep/read for the declared contract: command tree and verb choices (§1–2),
flag names/shorts/negation (§3), standard options present (§4 vs Appendix B),
config paths and env prefix (§5), exit-code constants (§6), help/README
consistency, secrets in argv or output paths (R5.5/R5.6).

## 4. Live probes (runtime contract)

The core checks — each maps to rules; capture stdout, stderr, and exit code
separately for every probe:

| Probe | Expect | Rules |
|---|---|---|
| `tool --version` | one line `tool <semver>`, stdout, exit 0 | R4.1, decision 35 |
| `tool` (bare) | help on stdout, exit 0 | R7.9 |
| `tool <unknown>` | usage on stderr, exit 2, nearest-command suggestion | R7.9, decision 34 |
| `tool <group>` (incomplete) | usage on stderr, exit 2 | R7.9 |
| `tool --help` / `-h` | structured help with an example, stdout, exit 0 | R7.5 |
| `tool list -o json` | machine-clean stdout (parses), diagnostics on stderr only | R7.1, R7.2 |
| failing cmd with `-o json` | single JSON error object on stderr, stable `code` | R7.8 |
| destructive cmd, stdin not a TTY | no hang; exit 2/4/1 per R8.2 | R8.1, R8.2 |
| `tool <cmd> --no-such-flag` | usage error, exit 2 | R6.1 |
| abbreviated flag (`--ver`) | rejected, not expanded | R3.10 |
| SIGINT mid-run | exit 130; SIGTERM → 143 (standard+ tier) | R6.1, R9.6 |

Tier scaling: minimal runs roughly the first six; standard runs all;
publishable additionally verifies §9 operational rules (deprecation warnings,
telemetry posture and its CI disablement, `doctor`, locale-independence via
`LC_ALL=C`) and §10 if networked.

## 5. Report

Findings table (Rule | Level | Where | Finding | Fix), blockers first, then:
N/A groups with reasons, unverified rules (with why), standard version, and a
conformance summary line — e.g. *"14 applicable MUSTs: 11 pass, 2 fail,
1 unverified"*.

## 6. Artifacts (offer, don't assume)

1. **Conformance note** — render `templates/conformance-note.md` (or update
   the existing one): results, waivers for accepted SHOULD deviations, audit
   history row with date + standard version.
2. **CI fixtures (R9.14)** — port the live probes into the repo's test
   framework so conformance outlives this session. Publishable tier: strongly
   recommended, offered by default.
3. **Fix plan** — hand ordered blockers to the normal planning flow
   (writing-plans) if the user wants them fixed now.
4. **Amendments** — deviations that beat the rule go upstream as Decision Log
   proposals (SKILL.md workflow step 6).
