# The CLI Design Standard

**A reusable, normative standard for designing command-line interfaces.**

| | |
|---|---|
| **Version** | 1.4.1 |
| **Status** | Active — canonical |
| **Last updated** | 2026-06-27 |
| **Applies to** | All new CLIs, any language |

> **Conformance language.** The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as defined in RFC 2119. **MUST**-level rules are required for conformance; **SHOULD**-level rules are strong defaults that may be waived only with a documented reason; **MAY**-level rules are permitted options.
>
> Every rule has a stable ID (e.g., `R3.4`) so it can be referenced from reviews and the conformance checklist (Appendix C).
>
> Examples throughout use a fictional tool named **`acme`** managing fictional resources (`project`, `widget`, `deploy`). The rules are language-agnostic; framework-specific guidance lives in Appendix E.

---

## Part I — PRD

### Purpose & goals

This document is the single source of truth for how command-line tools in this organization are designed. Its goal is to make every CLI **predictable** (a user who knows one of our tools can guess the others), **scriptable** (automation and agents can depend on stable behavior), and **durable** (the rules are defined once and reused without re-litigation). It exists so that future CLI work starts from settled decisions instead of re-deriving conventions per project.

### Scope

In scope: command and subcommand structure, naming, arguments and flags, standard options, configuration and precedence, environment variables, filesystem locations, exit codes, output streams, help, errors, and the lifecycle concerns (deprecation, versioning, signals, encoding) that distinguish a professional CLI from a script.

### Non-goals

- **Color, theming, and terminal styling** are governed by a separate standard and are deliberately omitted here. This document does not define color usage, `NO_COLOR`/`FORCE_COLOR`, or visual styling. TTY detection appears only where it governs *human-vs-machine* output behavior, not appearance.
- This is not a tutorial on any specific framework; Appendix E is informative only.
- This does not mandate an implementation language.

### Audience

CLI authors, reviewers, and anyone defining a tool's interface. Reviewers can use Appendix C as a checklist.

### Design principles

1. **Stand on the baseline.** POSIX Utility Syntax Guidelines and GNU long-options are the non-negotiable foundation. Modern guidance (clig.dev, the 12-Factor CLI essay, the XDG Base Directory Specification) layers on top. Deviations from the baseline must be deliberate and explained.
2. **Be obvious, then be powerful.** Optimize for what most people do most of the time; expose depth through flags, not through surprising defaults.
3. **Two audiences, one tool.** Every command serves a human at a terminal *and* a script/agent in a pipe. Design for both: readable by default, machine-clean on demand.
4. **Consistency beats cleverness.** Parallel commands behave in parallel ways. A `create` anywhere behaves like a `create` everywhere.
5. **Safe by default.** Destructive actions confirm; automation can opt out explicitly.

### How to use this document & conformance levels

A CLI **conforms** when it satisfies every **MUST** rule and documents any waived **SHOULD**. Two conformance profiles exist:

- **Standard profile** (default) — the noun-verb structure in Part II. Use this unless the small-CLI criteria in Appendix A are met.
- **Small-CLI profile** — the verb-first exception in Appendix A, for bounded single-purpose tools. **Noun-verb is the default norm; verb-first is a bounded exception.**

---

## Part II — Decision Log

Every settled axis, the choice, and why. "Org-locked" entries were decided by the organization and are not re-opened here; "derived" entries were resolved while writing this standard, consistent with the locked ones.

| # | Axis | Decision | Basis | Source |
|---|------|----------|-------|--------|
| 1 | Command ordering | Noun-verb default (`acme project create`); verb-first only under Appendix A criteria | Scales to many resources; models gh/kubectl | org-locked |
| 2 | Language anchoring | Rules language-agnostic; framework syntax confined to Appendix E (Rust/Python/TS) | Reusable across stacks | org-locked |
| 3 | `-v` meaning | `-v`/`--verbose` = verbose; `--version` with `-V` short alias | Verbose is hot path; matches cargo/clig.dev | org-locked |
| 4 | Structured output | `-o`/`--output` enum is canonical; `--json` is shorthand for `-o json` | Enum extensible; `--json` ergonomic | org-locked |
| 5 | Noun number | Singular canonical; plural accepted as undocumented alias | Reads with verbs; models gh | org-locked |
| 6 | Boolean negation | `--no-<foo>` canonical; `--flag=false` accepted silently | Readable, GNU-aligned | org-locked |
| 7 | Env var prefix | `TOOL_*` uppercase, deterministic mapping, curated subset | Predictable; majority convention | org-locked |
| 8 | Exit codes | `0/1/2/130` + semantic `3`/`4`/`5`; reject sysexits.h | Useful signal without BSD baggage; `4` from gh | org-locked |
| 9 | Config format | TOML canonical; YAML permitted for cloud-native tools | Safe to edit, Rust-native; YAML where expected | org-locked |
| 10 | Filesystem paths | XDG Base Directory Specification | Modern standard; predictable | org-locked |
| 11 | Nesting depth | ≤ 3 levels (`tool group noun verb`); deeper requires justification | Discoverability vs. depth | derived |
| 12 | Casing | `kebab-case` for commands, subcommands, and long flags; lowercase binaries | Universal de facto standard | derived |
| 13 | Repeatable flags | Prefer repeating the flag; comma-separated values **MAY** be accepted as a convenience | Unambiguous, composable | derived |
| 14 | Dry-run | `--dry-run` boolean default; tri-state (`=client\|server`) only where a server round-trip differs | Simplicity unless semantics demand more | derived |
| 15 | Verbosity model | `-v` repeatable for levels; `--quiet` wins ties; `--debug` = max diagnostics | Removes verbose/quiet/debug overlap | derived |
| 16 | Code `2` semantics | `2` = usage/parse error (not "cancelled" as in gh) | Matches argparse/shell convention | derived |
| 17 | Stream contract | Ground streams in POSIX fd 0/1/2; requested output (help/version/results) → stdout, all else → stderr | Pipeability; standards-backed | derived |
| 18 | Hyphen operands | `-` = stdin/stdout; `/dev/std*` permitted; destination-file is a distinct flag, never `-o` | POSIX Guideline 13; avoids `-o` format collision | derived |
| 19 | Buffering & flush | Flush all streams before exit; don't rely on stdout/stderr interleaving | ISO C buffering semantics | derived |
| 20 | Read-verb model | `view` (normal detail) + `describe` (expanded); no `get` for resources; `get`/`set` reserved for config key/value | Removes format-vs-content overlap; gh-style naming | derived |
| 21 | Config merge | Per-key merge, nearer scope wins, arrays replace, by default; alternatives allowed if documented | Predictable; matches cargo layering intent | derived |
| 22 | File-input flag | `-f` reserved for `--force`; file input is `--file`, no short | Resolves the `-f` force/file collision | derived |
| 23 | Domain verbs | Industry-established domain verbs (`logs`/`tail`/`exec`/`push`/`pull`/`import`/`export`) allowed where no core verb fits | Real-world verb coverage | derived |
| 24 | `export` semantics | `export` = portable, re-importable bundle (counterpart to `apply`/`import`), distinct from `view -o yaml` | Removes view/export overlap | derived |
| 25 | Output-file short | `--output-file` is long-only (no `-O`) | Avoids the `-o`/`-O` case collision | derived |
| 26 | Authentication | `auth login/logout/status`; token precedence flag-file > env > stored; keychain-first, `0600` file fallback | Core networked pattern; gh-style | derived |
| 27 | Targeting/scope | Persistent `--profile/--context/--project/--namespace`; `-a` in-scope, `-A` all-scopes | Multi-tenant support; kubectl pattern | derived |
| 28 | Pagination | Bounded first page by default; `--all`/`--limit`/`--no-paginate` | Safe default; avoids hammering APIs | derived |
| 29 | Async ops | Wait by default; `--no-wait` returns op id; `--timeout`; `operation view` | Intuitive default | derived |
| 30 | Idempotency / TLS | Auto idempotency key on retry; TLS verified by default, `--insecure` warns loudly | Safe network behavior | derived |
| 31 | Input/stdin/format | `--file` (no short) + auto stdin read; input format auto-detected, `--input-format` override; atomic `--output-file`, silent overwrite + `--no-clobber` | Symmetric with output; ergonomic | derived |
| 32 | No prefix abbreviation | Exact match required for flags/commands; no GNU-style `--ver`→`--verbose` | Protects scripts from drift | derived |
| 33 | Error schema | `-o json` errors are `{"error":{"code","message"}}`, stable `code` | Parseable failures | derived |
| 34 | Bare/unknown invocation | Bare top → help/stdout/0; incomplete group → usage/stderr/2; suggest nearest on unknown | Friendly + strict | derived |
| 35 | Version format | `acme <semver>` one line; `--version -o json` optional | GNU-style, predictable | derived |
| 36 | Secret redaction | Secrets masked in all output incl. debug/logs/errors; `--show-secrets` to reveal | Output-side secret safety | derived |
| 37 | Partial failure | Continue-on-error + per-item report, exit `1` if any failed; `--fail-fast` opt-in | Resilient bulk ops | derived |
| 38 | Signals | Add `SIGTERM` → exit `143` (alongside SIGINT `130`, SIGPIPE) | Container/orchestrator correctness | derived |
| 39 | Watch/follow & ordering | `--watch`/`-w`, `--follow` (no short); deterministic documented `list` order | Streaming + reproducibility | derived |
| 40 | Effective config & misc | `config view`/`--show-origin`; type-to-confirm (MAY); `--concurrency` (MAY); help topics, self-update, plugins (MAY) | Debuggability + ecosystem | derived |
| 41 | v1.4.1 consistency patch | Cheat-sheet verb core synced to R2.1 (drop `get`, split `view`/`describe`); R9.5 trimmed to timeouts/retries, deferring auth/targeting/TLS to §10; disambiguated duplicate titles (R7.4 Pager, R10.5 Idempotency keys) | Cross-pass reconciliation | derived |

---

## Part III — Specification (normative)

### §1 Command structure & naming

**R1.1 — Noun-verb ordering. (MUST, Standard profile)**
A command path **MUST** read *resource then action*: `acme <noun> <verb>`. This is the de facto pattern of the most-praised multi-resource CLIs (gh, kubectl) and scales as resources grow.
✅ `acme project create`, `acme deploy list`
❌ `acme create project`, `acme list-deploys`

**R1.2 — Nesting depth. (SHOULD)**
Command depth **SHOULD NOT** exceed three levels (`acme <group> <noun> <verb>`). Group nouns only when a flat noun set becomes unwieldy. Deeper nesting harms discoverability and tab-completion.
✅ `acme registry image push` ❌ `acme cloud region zone node drain`

**R1.3 — Noun number. (MUST)**
Resource nouns **MUST** be **singular** in their canonical form. The plural form **MAY** be accepted as an undocumented alias but **MUST NOT** appear in help or docs. The verb already carries cardinality, so `list` returning many is not a reason to pluralize.
✅ `acme widget list` (canonical) · `acme widgets list` (silent alias)
❌ documenting `acme widgets list` as the primary form

**R1.4 — Casing. (MUST)**
Binaries, commands, subcommands, and long flags **MUST** be lowercase `kebab-case`. No camelCase, snake_case, or capitals in the interface.
✅ `acme access-token rotate --dry-run` ❌ `acme accessToken Rotate --dryRun`

**R1.5 — Binary naming. (SHOULD)**
The binary name **SHOULD** be short, lowercase, and use only letters, digits, and hyphens (POSIX favors 2–9 characters). It **SHOULD NOT** collide with common system commands.
✅ `acme` ❌ `AcmeCLI`, `test`

**R1.6 — Aliases. (MAY / MUST NOT)**
Built-in short aliases for common verbs and nouns **MAY** be provided (e.g., `ls` for `list`, `rm` for `delete`). Aliases **MUST NOT** be ambiguous, **MUST NOT** shadow a different real command, and **SHOULD** be listed in help. User-defined aliases, if supported, **MUST NOT** override built-in commands.
✅ `acme widget ls` ≡ `acme widget list`
❌ an alias `acme rm` that sometimes means `remove` and sometimes `rmdir`

---

### §2 Command vocabulary

**R2.1 — Canonical verbs. (SHOULD)**
Use the shared verb core below for the stated meaning; introduce a new verb only when none fits. This vocabulary is drawn from the cross-tool consensus (kubectl codifies it most explicitly).

| Verb | Meaning | Notes / avoid |
|------|---------|---------------|
| `list` | enumerate many (alias `ls`); an empty result is success (exit `0`) | avoid `get-all`, `ls-all` |
| `view` | human-readable detail of one resource; `-o`/`--json` switches to machine output | the default read-one verb; avoid `get`/`show` for resources |
| `describe` | *expanded* detail of one — related objects, events, computed/derived state (a superset of `view`) | use only when `view` is insufficient |
| `create` | make a new resource (imperative) | avoid `new`, `add` for resources |
| `delete` | remove (alias `rm`) | avoid `destroy`, `remove` as synonyms |
| `update` / `set` | modify fields | `set` for single field, `update` for many |
| `apply` | declarative create-or-update from a file | reserve for declarative flows |
| `run` | execute a one-off action | |
| `edit` | open in `$EDITOR` | |

✅ `acme project delete demo` ❌ `acme project destroy demo`

> `get`/`set` are **not** resource verbs; they are reserved for key/value accessors on configuration (`acme config get <key>`, `acme config set <key> <value>`), matching git/npm/gcloud `config get`. To read a resource, use `view` (or `describe`).
>
> Beyond the core, a tool **MAY** use an **industry-established domain verb** where no core verb fits — e.g., `logs`, `tail`, `exec`, `push`, `pull`, `import`, `export` — provided its meaning matches the ecosystem norm. Inventing a *novel* verb still requires that no core or established verb fits. In particular, **`export`** produces a *portable, re-importable* artifact (it **MAY** bundle related objects in dependency order and strip secrets) and is the counterpart to `apply`/`import`; it is therefore distinct from `view -o yaml`, which inspects a single resource as-is.

**R2.2 — Resource naming. (SHOULD)**
New nouns **SHOULD** name the domain object as the user thinks of it, singular (R1.3), kebab-cased (R1.4). Avoid abbreviations unless they are industry-standard.
✅ `acme access-token` ❌ `acme atk`

**R2.3 — Positional arguments are identities; flags are modifiers. (MUST)**
Positional arguments **MUST** be the *identity* of the thing acted on (names, IDs, file paths). Everything that *modifies behavior* **MUST** be a flag. A command **SHOULD** take at most one or two positionals; three or more is a strong signal to switch to flags (12-Factor CLI).
✅ `acme deploy create my-service --replicas 3 --region us-east-1`
❌ `acme deploy create my-service 3 us-east-1 rolling true`

---

### §3 Arguments & flags

**R3.1 — `--` ends option parsing. (MUST)**
A bare `--` **MUST** terminate option parsing; everything after it is treated as a positional operand even if it begins with `-` (POSIX Guideline 10).
✅ `acme run -- --not-a-flag ./script` ❌ treating `--not-a-flag` after `--` as an option

**R3.2 — Hyphen operands (stdin/stdout). (SHOULD)**
Where a command reads or writes a *file operand*, a bare `-` **SHOULD** mean stdin (for input) or stdout (for output), per POSIX Guideline 13. A tool **MAY** additionally accept `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` where a real path is required, but **MUST NOT** depend on them existing (they are not POSIX-mandated). A destination-file option, if a command has one, **MUST** be a distinct flag — **`--output-file`, long form only** (no short flag, to avoid an `-o`/`-O` case collision) — **MUST NOT** reuse `--output`/`-o` (which selects *format*, R4.1), and **MUST** honor `-` as stdout.
✅ `acme widget apply -` (manifest from stdin) · `acme report build --output-file -` (write to stdout)
❌ `acme report build -o -` (`-o` selects format, not a file) · erroring on `-` when a pipe is intended

**R3.3 — Long flags. (MUST)**
Every option **MUST** have a `--kebab-case` long form with two dashes. Long flag names **SHOULD** be one to three words.
✅ `--output-format`, `--no-verify` ❌ `--outputFormat`, `--o_f`

**R3.4 — Short flags. (MAY / SHOULD)**
A single-dash, single-character short alias **MAY** be provided, and **SHOULD** be reserved for frequently used options only (clig.dev). Short flags **MUST** map to a long flag of the same meaning. The standard short letters in R4.1 (`-h -V -v -q -o -f -y`) **MUST NOT** be reassigned to a conflicting meaning; in particular `-f` is reserved for `--force`, so a file-input option **MUST** use `--file` with **no short flag** (do not adopt the `-f` = file idiom).
✅ `-o json` ≡ `--output json`; `--file ./manifest.toml` (no short) ❌ a short flag with no long equivalent; `-f ./manifest.toml` for file input

**R3.5 — Combining & value attachment. (MUST / SHOULD)**
No-argument short flags **MUST** be combinable behind one dash (`-abc` ≡ `-a -b -c`, POSIX Guideline 5). A flag's value **MAY** be attached with `=` or given as the next argument; both **MUST** be accepted for long flags.
✅ `acme deploy list -aq`, `--region=us-east-1`, `--region us-east-1`
❌ rejecting `--region=us-east-1` while accepting `--region us-east-1`

**R3.6 — Boolean flags & negation. (MUST)**
A boolean flag is set by presence. For any boolean that **defaults to true**, the tool **MUST** provide an explicit `--no-<foo>` negation as the canonical, documented form. `--flag=false` **MAY** be accepted silently where the framework provides it but **MUST NOT** be the documented interface.
✅ `--no-verify` (documented), `--verify=false` (silently accepted)
❌ documenting `--verify=false` as the only way to disable

**R3.7 — Repeatable / list flags. (SHOULD)**
For options that take multiple values, repeating the flag **SHOULD** be the canonical form; comma-separated values **MAY** also be accepted as a convenience. Document which the tool uses.
✅ `--label a --label b` (canonical) · `--label a,b` (convenience) ❌ silently dropping all but the last `--label`

**R3.8 — Flag / env / config name parity. (MUST)**
A setting exposed in more than one place **MUST** use the same name across flag, environment variable, and config key, via deterministic transforms only: `--some-flag` ↔ `TOOL_SOME_FLAG` ↔ `some-flag` (config key). No bespoke renaming.
✅ `--log-level` ↔ `ACME_LOG_LEVEL` ↔ `log-level`
❌ `--log-level` ↔ `ACME_VERBOSITY`

**R3.9 — Input files & stdin. (SHOULD)**
A command that consumes structured input **SHOULD** accept it via `--file <path>` (long-only, since `-f` is force — R3.4), with `-` meaning stdin (R3.2). When stdin is piped and no `--file`/operand is given, the command **SHOULD** read stdin automatically. Input format **SHOULD** be auto-detected by content/extension, with `--input-format <fmt>` to force it. A destination-file option (`--output-file`) **SHOULD** write atomically (temp + rename) and **MAY** overwrite silently (Unix `>` convention); offer `--no-clobber` to refuse an existing file.
✅ `acme widget apply --file ./w.toml`; `cat w.toml | acme widget apply` ❌ requiring `--file -` when stdin is already piped

**R3.10 — No prefix abbreviation. (MUST)**
Long flags and command names **MUST** require an exact match; automatic unique-prefix abbreviation (GNU-style `--ver` → `--verbose`) **MUST NOT** be enabled, because adding a flag later silently breaks scripts that relied on the shorter form. Provide explicit aliases instead.
✅ only `--verbose` works; declare an alias if a short spelling is wanted ❌ `--ver` resolving to `--verbose` by accident

---

### §4 Standard options

**R4.1 — The universal option set. (MUST)**
Every CLI **MUST** implement the following with exactly these names and meanings. Letters not listed here **SHOULD NOT** be reassigned to conflicting meanings.

| Option | Short | Meaning |
|--------|-------|---------|
| `--help` | `-h` | Print help for the current command and exit `0`. Also `acme help [command]`. |
| `--version` | `-V` | Print version and exit `0`. **`-v` is not version.** |
| `--verbose` | `-v` | Increase diagnostic detail (repeatable; see R4.4). |
| `--quiet` | `-q` | Suppress non-essential output. |
| `--output` | `-o` | Output format enum: `table` (default, human), `json`, `yaml`, `wide`, `name`. |
| `--json` | — | Shorthand for `--output json`. |
| `--dry-run` | — | Show what would happen without doing it (R4.3). |
| `--force` | `-f` | Bypass confirmation/safety checks (R8.1). |
| `--yes` | `-y` | Assume "yes" to prompts (non-interactive consent). |
| `--no-input` | — | Never prompt; fail if input would be required. |
| `--config` | — | Path to a config file, overriding discovery (R5.2). |
| `--debug` | — | Maximum diagnostics, including internal detail intended for bug reports. |

**R4.2 — Structured output. (MUST)**
`-o`/`--output` **MUST** be the canonical machine-output mechanism, with an extensible enum; `--json` **MUST** behave identically to `-o json`. This follows the kubectl/gcloud enum pattern while preserving the ergonomic gh-style `--json`. Field selection, embedded query (`--jq`), and `--template` are **optional** power features and **MUST NOT** be required for conformance.
The standard formats split into two classes: **`table`** (default) and **`wide`** (the same data with more columns) are *human* formats and **MAY** change between releases; **`json`**, **`yaml`**, and **`name`** (bare identifiers, one per line) are *machine* formats and **MUST** stay stable within a major version (R7.2).
✅ `acme widget list -o json`, `acme widget list --json` (identical); `acme widget list -o name | xargs …`
❌ a `--json` that emits a *different* shape than `-o json`; renaming a JSON field in a minor release

**R4.3 — Dry-run. (SHOULD)**
`--dry-run` **SHOULD** be a boolean that prints the intended effect and makes no changes. A tri-state form (`--dry-run=client|server`) **MAY** be used only where a server-validated preview differs meaningfully from a local one (kubectl's case).
✅ `acme deploy delete web --dry-run` ❌ a `--dry-run` that still mutates state

**R4.4 — Verbosity semantics. (MUST)**
`-v`/`--verbose` **MUST** be repeatable to raise the level (`-v`, `-vv`, `-vvv`); `--debug` is equivalent to the maximum level plus internal detail; `--log-level <level>` **MAY** be offered as an explicit alternative. When both `--verbose` and `--quiet` are given, `--quiet` **MUST** win; an explicit `--debug`, however, **MUST** override `--quiet`, since it is a deliberate request for diagnostics. These three controls **MUST NOT** overlap ambiguously — define one ladder. `--quiet`/`-q` suppresses progress and informational diagnostics on stderr only; it **MUST NOT** suppress the requested result on stdout, and **MUST NOT** imply consent to prompts.
✅ `acme deploy create web -vv`, `acme deploy create web --quiet`
❌ `--debug` and `-vvv` meaning different, undocumented things

**R4.5 — Global vs command-local flags. (SHOULD)**
A flag **SHOULD** be global/persistent only if it is meaningful on (nearly) every command — `--help`, `--version`, `--verbose`, `--quiet`, `--output`, `--config`, `--debug`, and connection/identity flags (`--profile`, `--context`). Everything else **SHOULD** be local to the command that uses it. Global flags **MUST** be accepted before the subcommand and **SHOULD** also be accepted after it.
✅ `acme -o json widget list` and `acme widget list -o json` both work
❌ a behavior-shaping flag like `--replicas` declared globally

**R4.6 — Version output format. (SHOULD)**
`--version`/`-V` **SHOULD** print a single line `acme <semver>` (GNU name-and-version style); a build **MAY** append commit/date for pre-release builds, and `--version -o json` **MAY** emit `{"version","commit","built"}`.
✅ `acme 1.4.0` ❌ `Acme CLI — Version Info: v1.4.0 ©2026 …` as the only output

**R4.7 — Concurrency. (MAY)**
A command doing parallelizable work **MAY** expose `--concurrency <n>` (or `-j`/`--jobs` in build/transfer contexts) with a sensible default; it controls client-side parallelism only.
✅ `acme registry image push --concurrency 4` ❌ unbounded parallelism with no way to cap it

---

### §5 Configuration, environment & precedence

**R5.1 — Precedence chain. (MUST)**
Configuration **MUST** resolve in this order, highest priority first, and the tool **MUST** document it verbatim:

1. Command-line flags
2. Environment variables (`TOOL_*`)
3. Project/local config file (nearest, discovered by walking up from the working directory)
4. User config file (`$XDG_CONFIG_HOME`)
5. System config file (`$XDG_CONFIG_DIRS`)
6. Built-in defaults

✅ a `--region` flag overrides `ACME_REGION`, which overrides the project config
❌ a config file value silently overriding an explicit flag

Where more than one config file in the chain sets the same key, they **MUST** merge **per key** by default, with the nearer scope winning and arrays **replaced** (not concatenated). A tool **MAY** use a different merge strategy (e.g., array concatenation, cargo-style) where its domain requires it, but **MUST** document the strategy clearly.

**R5.2 — Config file format & discovery. (MUST / SHOULD)**
The canonical config format **MUST** be **TOML**. **YAML MAY** be used instead *only* for tools that live in the Kubernetes/cloud-native ecosystem, where users expect it; this is the single permitted exception. JSON and INI **MUST NOT** be the canonical format. An explicit `--config <path>` **MUST** **replace** the discovered project/user/system config files — it becomes the sole config file in the chain — while command-line flags and environment variables still layer above it and built-in defaults below it (R5.1). It does **not** merge with the files it overrides.
✅ `~/.config/acme/config.toml`; `--config ./acme.toml`
❌ a primary `config.json` for a general-purpose tool

**R5.3 — Filesystem locations (XDG). (MUST)**
Per-user paths **MUST** follow the XDG Base Directory Specification, namespaced under the tool name, honoring `XDG_*` overrides and falling back to the defaults below.

| Purpose | Variable | Default |
|---------|----------|---------|
| Config | `$XDG_CONFIG_HOME/acme/` | `~/.config/acme/` |
| Data | `$XDG_DATA_HOME/acme/` | `~/.local/share/acme/` |
| State (logs, history) | `$XDG_STATE_HOME/acme/` | `~/.local/state/acme/` |
| Cache | `$XDG_CACHE_HOME/acme/` | `~/.cache/acme/` |
| Runtime (sockets) | `$XDG_RUNTIME_DIR/acme/` | (set by system) |

System-wide config search uses `$XDG_CONFIG_DIRS` (default `/etc/xdg`). Native non-Linux equivalents are permitted deviations (macOS `~/Library/Application Support/acme/`; Windows `%APPDATA%`/`%LOCALAPPDATA%`), but XDG paths are the canonical reference.
❌ writing config to `~/.acme/` (a dotdir in `$HOME`) by default

**R5.4 — Environment variables. (MUST)**
Environment variables **MUST** use the uppercase `TOOL_*` prefix with a deterministic mapping (`--some-flag` → `ACME_SOME_FLAG`). Only a **curated subset** of settings **SHOULD** read from the environment — typically auth/token, host/endpoint, output format, and config path — and the tool **MUST NOT** auto-expose every flag as an env var.
✅ `ACME_TOKEN`, `ACME_ENDPOINT`, `ACME_OUTPUT`, `ACME_CONFIG`
❌ inventing `ACME_VERBOSITY` for `--log-level`; auto-binding every obscure flag

**R5.5 — Secrets. (MUST)**
Secrets (tokens, passwords, keys) **MUST NOT** be accepted as plain command-line flags or positionals, because argv is visible in process listings. Accept them via environment variable, a file path, a credential helper, or stdin.
✅ `ACME_TOKEN=… acme login`, `acme login --token-file ./tok`
❌ `acme login --token sk_live_abc123`

**R5.6 — Secret redaction in output. (MUST)**
Secret values (tokens, passwords, keys) **MUST** be masked in all output — `config view`, `--debug`/request logs, progress, and error messages — e.g. `sk_live_…****`. Revealing a secret in full **MUST** require an explicit flag such as `--show-secrets`.
✅ `acme config view` prints `token = "sk_live_…****"` ❌ `--debug` echoing the full bearer token

**R5.7 — Effective-config introspection. (SHOULD)**
A tool **SHOULD** provide `acme config view` to print the merged, effective configuration, and **SHOULD** offer `--show-origin` (git-style) to reveal which source (flag/env/file) supplied each value — the practical way to debug the precedence chain (R5.1).
✅ `acme config view --show-origin` ❌ no way to see why a setting took the value it did

---

### §6 Exit codes

**R6.1 — Exit-code scheme. (MUST)**
A CLI **MUST** use the following scheme. Unlisted failures collapse to `1`. This is "structured-but-small": enough signal for automation without the BSD `sysexits.h` (64–78) baggage, which we **reject** as over-specified and unavailable on Windows.

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Generic runtime error |
| `2` | Usage / parse error (bad flags, missing args) |
| `3` | Not found (the requested resource does not exist) |
| `4` | Authentication or authorization required |
| `5` | Conflict / precondition failed |
| `130` | Interrupted by `SIGINT` (Ctrl-C) |

Note: code `2` means *usage error* here (matching shell/argparse convention), a deliberate divergence from gh, which uses `2` for "cancelled." Cancellation surfaces as `130`.

**R6.2 — Query & empty-result exit codes. (MUST / MAY)**
By default, a `view`/`describe` of a resource that does not exist **MUST** exit `3` (not found), and an empty `list` **MUST** exit `0` (an empty set is a successful result, not an error). Beyond those defaults, read/query commands **MAY** use exit `1` to mean "ran successfully, found nothing / found differences" (git `diff`/`grep` style), but only for explicit predicate/search commands and only when **documented per command**, so it never collides with `1` = error or with the `3`/`0` defaults above.
✅ `acme widget view gone` → `3`; `acme widget list` (no results) → `0`; `acme widget exists foo` → documented `0`/`1`
❌ an empty `list` exiting `1`; an undocumented `1` scripts can't distinguish from failure

**R6.3 — Partial failure. (SHOULD)**
A command operating on many items **SHOULD** continue on error by default, report a per-item result, and exit `1` if any item failed (no new code); `--fail-fast` **MAY** stop at the first error. The summary **MUST** make clear which items succeeded and which did not.
✅ `acme widget delete a b c` deletes `b`/`c`, reports `a` failed, exits `1` ❌ aborting the whole batch on the first error with no per-item detail

---

### §7 Output, streams & UX

**R7.1 — stdout vs stderr. (MUST)**
Streams follow the POSIX file-descriptor contract — fd 1 (stdout) carries the command's *requested result*; fd 2 (stderr) carries everything else: diagnostics, progress, prompts, warnings, and errors (clig.dev, 12-Factor, gcloud). This split is what makes a tool pipeable and lets shells redirect diagnostics (`2>`) independently of data. Requested output goes to stdout **even when it is informational**: `--help`, `--version`, and query results are the user's requested data and **MUST** go to stdout with exit `0` (R7.5). A warning emitted during a normal data command **MUST** go to stderr so it never contaminates stdout.
✅ `acme widget list -o json | jq .` works because logs went to stderr; `acme --version` → stdout
❌ printing a "Fetching…" spinner — or `--version` — to stdout, corrupting/contaminating the data stream

**R7.2 — Human vs machine output. (MUST / SHOULD)**
The default (`-o table`) is for humans and **MAY** change between releases. Machine formats (`-o json`/`-o yaml`/`-o name`) **MUST** be stable within a major version (R9.3) so scripts can depend on them.
✅ stable JSON keys across `1.x` ❌ renaming a JSON field in a minor release

**R7.3 — TTY-aware behavior. (SHOULD / MUST)**
A tool **SHOULD** detect whether stdout is a TTY and adjust *presentation* accordingly: interactive niceties when attached (spinners, pager, prompts), and none of those when piped. TTY state **MUST NOT** change the selected output *format*: `-o` (default `table`) determines the format regardless of whether output is a terminal or a pipe — a pipe only suppresses the niceties (and, per the separate color standard, color). Scripts wanting machine output **MUST** pass `-o json`/`--json` explicitly rather than relying on the pipe to switch formats. (Color/styling is governed separately and out of scope here.)
✅ `acme widget list | cat` is still `table`; `acme widget list --json | jq .` for machine output
❌ silently emitting JSON instead of the table just because stdout is piped; prompting for confirmation inside a pipe with no TTY

**R7.4 — Pager & progress. (SHOULD)**
Long human output **MAY** invoke a pager when attached to a TTY, and **MUST** provide `--no-pager` (or honor a standard pager variable) to disable it. Progress indicators **MUST** go to stderr (R7.1) and **MUST** be suppressed when not a TTY or under `--quiet`.

**R7.5 — Help text format. (MUST)**
Every command **MUST** print structured help containing, in order: a one-line summary, `Usage:`, then `Commands`/`Arguments`/`Flags`/`Examples` sections as applicable. Help **MUST** go to stdout and exit `0`. At least one realistic **example MUST** be shown for non-trivial commands.

```
$ acme deploy create --help
Create a deployment from the current project.

Usage:
  acme deploy create <name> [flags]

Arguments:
  <name>            Deployment name (required)

Flags:
  -r, --replicas int     Number of replicas (default 1)
      --region string    Target region (default from config)
      --dry-run          Print the plan without applying
  -o, --output string    Output format: table|json|yaml|wide|name (default "table")

Examples:
  acme deploy create web --replicas 3 --region us-east-1
  acme deploy create web --dry-run -o json
```

**R7.6 — Error messages. (MUST / SHOULD)**
Errors **MUST** be written to stderr (R7.1) and **SHOULD** be lowercase-first, free of a trailing period, name the offending input in quotes, and state a next step where possible. When `-o json` is active, errors **SHOULD** also be emitted as structured JSON on stderr so automation can parse them.
✅ `error: project 'demo' not found — run 'acme project list' to see available projects`
❌ `ERROR!! Something went wrong.`

**R7.7 — Buffering & flush. (SHOULD / MUST)**
Because stdout is fully buffered when it is not a terminal (ISO C), diagnostics on stderr **SHOULD** be line- or unbuffered so they appear promptly, and a tool **MUST** flush all streams before exiting on every path — success, error, and signal (R9.6) — so no output is lost. Tools **SHOULD NOT** rely on the interleaved ordering of stdout and stderr, since buffering makes that ordering non-deterministic between a TTY and a pipe.
✅ flushing stdout before `exit(0)`; the final JSON line is never dropped on a panic
❌ losing the last line of output because the buffer wasn't flushed before exit

**R7.8 — Machine-readable error schema. (SHOULD)**
Under `-o json`, an error **SHOULD** be emitted to stderr as `{"error":{"code":"<stable-string>","message":"<human-readable>"}}`; `code` **MUST** be a stable, documented string, and the object **MAY** carry `details` or `request_id`.
✅ `{"error":{"code":"not_found","message":"project 'demo' not found"}}` ❌ a bare unparseable string under `-o json`

**R7.9 — Bare & unknown invocation. (MUST / SHOULD)**
The bare top-level command with no arguments **MUST** print help to stdout and exit `0`. A command *group* invoked with no action (`acme project`) is a usage error: print help to stderr and exit `2`. On an unknown command or flag, the tool **SHOULD** suggest the nearest match.
✅ `acme` → help/stdout/0; `acme projcet list` → "did you mean 'project'?" ❌ `acme project` silently doing nothing and exiting `0`

**R7.10 — Watch / follow streams. (SHOULD)**
Long-running stream commands use `--watch`/`-w` (re-render on change) or `--follow` (stream appended output; long-only, since `-f` is force). Output **MUST** be flushed per record (R7.7), and Ctrl-C **MUST** exit `130` cleanly.
✅ `acme deploy logs web --follow`; `acme widget list --watch` ❌ a buffered `--follow` that shows nothing until exit

**R7.11 — Deterministic list ordering. (SHOULD)**
`list` output **SHOULD** have a stable, documented default order (e.g., name or creation time) so scripts and diffs are reproducible; `--sort-by` **MAY** override it.
✅ a documented "sorted by name" default ❌ rows returned in nondeterministic API order

---

### §8 Interaction & safety

**R8.1 — Confirm destructive actions. (MUST)**
Irreversible or destructive operations **MUST** confirm before proceeding when attached to a TTY, and **MUST** offer a non-interactive bypass. The two bypass flags are distinct: **`--yes`/`-y`** answers the confirmation prompt (consent), while **`--force`/`-f`** *additionally* overrides a guard condition — for example, the resource is protected, is in use, or has drifted from its expected state. Under `--no-input` with no bypass flag, the command **MUST** fail rather than guess.
✅ `acme project delete demo --yes` skips the y/n prompt; `acme deploy delete web --force` deletes even though it is still receiving traffic
❌ deleting without confirmation and without a bypass flag; requiring `--force` merely to skip a routine prompt

**R8.2 — Non-interactive / CI behavior. (MUST)**
A tool **MUST** be fully usable without a TTY. It **MUST NOT** block on a prompt when input is unavailable; instead it uses provided flags/env or fails with a clear message and exit `2`/`4`. `--no-input` forces this mode explicitly.

**R8.3 — Idempotency & declarative flows. (SHOULD)**
Where it makes sense, mutating commands **SHOULD** be idempotent (re-running converges, not errors), and declarative inputs **SHOULD** use `apply` (R2.1). Distinguish imperative (`create`, `delete`) from declarative (`apply`) clearly.
✅ `acme widget apply manifest.toml` converges on re-run ❌ `apply` that fails because the resource already exists

**R8.4 — Stronger confirmation for catastrophic actions. (MAY)**
For catastrophic or wide-blast-radius operations, a tool **MAY** require the user to type the resource's name (not just y/n) when interactive; `--yes`/`--force` still bypass for automation (R8.1).
✅ "type the project name to confirm deletion:" ❌ deleting an entire project on a single keystroke

---

### §9 Lifecycle & ecosystem

**R9.1 — Completion, man pages & help topics. (SHOULD / MAY)**
A CLI **SHOULD** ship shell completion (`acme completion <shell>`) for at least bash, zsh, and fish, and **SHOULD** be able to generate man pages or equivalent reference docs. It **MAY** also provide non-command help *topics* via `acme help <topic>` (e.g., `formatting`, `environment`, `exit-codes`), gh-style.

**R9.2 — Deprecation policy. (MUST)**
When a command or flag is retired, the tool **MUST**: (a) print a deprecation warning to stderr that names the replacement, (b) keep the old name working as a hidden alias for a stated number of releases, and (c) document a removal timeline. Deprecations **MUST NOT** change exit codes or break scripts until removal.
✅ `warning: 'acme deploy rm' is deprecated; use 'acme deploy delete' (removed in 3.0)`
❌ silently deleting a flag in a minor release

**R9.3 — Interface stability & versioning. (MUST)**
The tool **MUST** version itself with SemVer and treat its **interface as the public contract**: command names, flag names, exit codes, env-var names, and machine-readable output shape. Breaking any of these **MUST** be a major-version change. `-o json`/`-o name` and other machine formats **MUST** remain stable across non-major versions (R7.2).

**R9.4 — Encoding & locale. (MUST / SHOULD)**
Input and output **MUST** default to UTF-8. Human-facing messages **MAY** honor locale, but machine-readable output (`-o json`/`yaml`) **MUST** remain locale-independent (e.g., `.` decimal separator, ISO-8601 timestamps, untranslated keys) so scripts are portable.
✅ JSON timestamps as `2026-06-27T12:00:00Z` regardless of locale
❌ localized number formatting (`1.234,56`) inside JSON

**R9.5 — Timeouts & retries. (SHOULD)**
Network-backed CLIs **SHOULD** apply sane request timeouts and bounded retries with backoff on idempotent calls, and **MUST NOT** retry non-idempotent mutations blindly (use idempotency keys, R10.5). Authentication, targeting, pagination, and TLS conventions for networked tools are defined in §10.

**R9.6 — Signals. (MUST)**
A tool **MUST** exit cleanly on `SIGINT` with code `130`, releasing resources, and **MUST** handle `SIGPIPE`/`EPIPE` gracefully — when a reader like `head` or `less` closes the pipe, exit quietly without a broken-pipe stack trace. It **MUST** also treat `SIGTERM` like `SIGINT` — clean up, flush (R7.7), and exit `143` (128+15) — so it behaves correctly under containers, orchestrators, and CI runners.
✅ `acme widget list | head -5` ends silently; `kill <pid>` shuts down cleanly with code `143` ❌ a Python traceback on `BrokenPipeError`; ignoring SIGTERM until killed with `SIGKILL`

**R9.7 — Telemetry & update checks. (MUST / SHOULD)**
Any telemetry **MUST** be opt-out via a documented flag/env and **MUST NOT** transmit secrets or full argv. Update-availability checks, if present, **SHOULD** be non-blocking, cached, and disableable, and **MUST NOT** run in non-interactive/CI contexts by default. A tool **MAY** offer self-update via `acme upgrade`.

**R9.8 — Empty-state & first-run. (SHOULD)**
On first run or empty results, a tool **SHOULD** guide the next step rather than printing nothing or a bare error.
✅ `no projects yet — create one with 'acme project create <name>'`
❌ an empty line, or a stack trace, when there is simply nothing to show

**R9.9 — Plugins. (MAY)**
A tool **MAY** support an extension model that runs `acme-<name>` executables found on `PATH` as `acme <name>` (git/kubectl style); plugins **MUST** follow this same standard.
✅ `acme-foo` on `PATH` is invoked by `acme foo` ❌ a plugin that ignores the exit-code and stream conventions

---

### §10 Networked tools

> This section applies only to CLIs that talk to a remote API or service. Local-only tools may skip it.

**R10.1 — Authentication & identity. (MUST / SHOULD)**
Authentication lives under an `auth` group: `acme auth login`, `acme auth logout`, `acme auth status`. Credentials **MUST** resolve in the order flag-supplied token‑*file* > `ACME_TOKEN` env > stored credential, and a token **MUST NOT** be passed as a plain flag value (R5.5). Stored credentials **SHOULD** use the OS keychain when available, falling back to a `0600` file under `$XDG_STATE_HOME/acme/` (R5.3). Auth failures exit `4` (R6.1).
✅ `acme auth login`; `ACME_TOKEN=… acme deploy list` ❌ `acme deploy list --token sk_live_…` (visible in `ps`)

**R10.2 — Targeting & scope. (MUST / SHOULD)**
Tools spanning tenants/accounts/regions **MUST** expose persistent targeting flags (`--profile`, `--context`, `--project`, `--namespace`, as applicable), resolving flag > env > the config's current context. `--all`/`-a` selects everything *within the current scope*; `-A` (or `--all-<scope>s`) selects across *all* scopes (kubectl pattern). Context switching is a config operation (`acme config use-context <name>`).
✅ `acme deploy list --all`, `acme deploy list -A` ❌ a hidden default account with no flag to see or change it

**R10.3 — Pagination. (SHOULD)**
A `list` over a remote collection **SHOULD** return a bounded first page by default and print a hint that more exist; `--all` fetches the entire set, `--limit N` caps it, and `--no-paginate` disables auto-fetching. This is independent of the terminal pager (R7.4). Bounded-by-default avoids accidentally hammering an API.
✅ `acme widget list` → first page + "… 240 more; use --all"; `acme widget list --all` ❌ silently fetching 50,000 rows by default

**R10.4 — Long-running operations. (SHOULD)**
An operation that does not complete synchronously **SHOULD** *wait* by default, showing progress on stderr; `--no-wait` returns immediately with an operation id, `--timeout <dur>` bounds the wait, and `acme <noun> operation view <id>` polls status. Interrupting a wait **MUST NOT** be assumed to cancel the server-side operation unless documented.
✅ `acme deploy create web` (waits); `acme deploy create web --no-wait` → prints op id ❌ `--no-wait` with no way to check status afterward

**R10.5 — Idempotency keys. (SHOULD)**
Where the API supports it, a mutating command **SHOULD** attach an idempotency key so an automatic retry (R9.5) does not double-apply, and **MAY** expose `--idempotency-key <key>` for caller-supplied dedupe.
✅ a retried `acme billing charge create` does not double-charge ❌ blind retries that duplicate a mutation

**R10.6 — TLS & insecure mode. (MUST)**
TLS certificate verification **MUST** be on by default. An escape hatch (`--insecure` / `--tls-no-verify`) **MAY** exist but **MUST** print a warning to stderr on every use and **MUST NOT** be enabled silently by an environment variable alone.
✅ `acme --insecure deploy list` warns on stderr ❌ disabling verification via a quiet env var with no warning

---

## Appendix A — Small-CLI (verb-first) profile

**Noun-verb (Part II) is the default norm.** This verb-first profile is a **bounded exception**, not an alternative of equal standing.

**Entry criteria (all MUST hold to use this profile):**
1. The tool has a **small, fixed** set of commands (roughly ≤ 7) that act on a **single implicit resource** or no resource at all.
2. The command set is **unlikely to grow** into multiple resource types.
3. The verbs read naturally without a noun (the tool *is* the noun), as with `git`, `cargo`, `npm`.

**What changes:** commands are flat verbs (`acme build`, `acme run`, `acme check`) instead of `noun verb`. **What stays the same:** every other rule in Part III applies unchanged — flags (§3), standard options (§4), config/precedence (§5), exit codes (§6), streams and errors (§7), and lifecycle (§9) are identical.

✅ verb-first fit: `acmefmt fmt`, `acmefmt check`, `acmefmt --version`
❌ verb-first misuse: a platform tool managing `projects`, `deploys`, `tokens`, and `users` — that **MUST** use noun-verb.

> Migration note: if a verb-first tool grows a second resource type, that is the trigger to move to noun-verb (R1.1) in the next major version (R9.3).

---

## Appendix B — Quick-reference cheat sheet

**Standard options**

| Option | Short | Meaning |
|---|---|---|
| `--help` | `-h` | help, exit 0 |
| `--version` | `-V` | version, exit 0 |
| `--verbose` | `-v` | verbose (repeatable) |
| `--quiet` | `-q` | suppress non-essential output |
| `--output` | `-o` | `table\|json\|yaml\|wide\|name` |
| `--json` | — | = `-o json` |
| `--dry-run` | — | preview, no changes |
| `--force` | `-f` | bypass safety |
| `--yes` | `-y` | assume yes |
| `--no-input` | — | never prompt |
| `--config` | — | config path |
| `--debug` | — | max diagnostics |

**Exit codes**

| 0 | 1 | 2 | 3 | 4 | 5 | 130 | 143 |
|---|---|---|---|---|---|---|---|
| success | error | usage | not found | auth | conflict | SIGINT | SIGTERM |

**Precedence (high → low):** flags → `TOOL_*` env → project config → user config (`$XDG_CONFIG_HOME`) → system config (`$XDG_CONFIG_DIRS`) → defaults

**Env vars:** `ACME_*` uppercase; `--some-flag` → `ACME_SOME_FLAG`; curated subset only

**XDG paths:** config `~/.config/acme/` · data `~/.local/share/acme/` · state `~/.local/state/acme/` · cache `~/.cache/acme/`

**Verb core:** `list`(`ls`) · `view` · `describe` · `create` · `delete`(`rm`) · `update`/`set` · `apply` · `run` · `edit`

---

## Appendix C — Conformance checklist

A tool conforms when every **MUST** is satisfied and each waived **SHOULD** is documented.

**Structure & naming**
- [ ] R1.1 Noun-verb (or Appendix A criteria met) · R1.3 singular nouns · R1.4 kebab-case · R1.6 unambiguous aliases · R2.1 canonical verbs (`view`/`describe`, not `get` for resources)

**Flags & args**
- [ ] R3.1 `--` terminator · R3.2 `-` = stdin/stdout (no `-o` overload) · R3.3 kebab long flags · R3.4 reserved short letters (`-f`=force; file input is `--file`) · R3.5 combinable shorts + `=`/space values · R3.6 `--no-<foo>` negation · R3.8 flag/env/config name parity

**Standard options**
- [ ] R4.1 full universal set present · R4.2 `--json` ≡ `-o json` · R4.4 verbose/quiet/debug ladder unambiguous

**Config / env / fs**
- [ ] R5.1 documented precedence chain · R5.2 TOML (or cloud-native YAML) · R5.3 XDG paths · R5.4 `TOOL_*` curated env · R5.5 no secrets in argv

**Exit & output**
- [ ] R6.1 exit-code scheme · R6.3 partial-failure → exit 1 · R7.1 stdout=data/stderr=diagnostics · R7.2 stable machine output · R7.5 structured help with example · R7.6 errors to stderr · R7.7 flush before exit · R7.8 JSON error schema · R7.9 bare→help/0, group→usage/2

**Safety & lifecycle**
- [ ] R8.1 destructive confirm + bypass · R8.2 non-interactive usable · R9.2 deprecation policy · R9.3 SemVer interface contract · R9.4 UTF-8 + locale-independent machine output · R9.6 clean SIGINT/SIGPIPE/SIGTERM · R3.10 no prefix abbreviation · R5.6 secrets redacted in output

**Networked tools (if applicable)**
- [ ] R10.1 auth group + token precedence (no token in argv) · R10.2 scope/targeting flags · R10.3 bounded pagination · R10.4 wait-by-default + `--no-wait` · R10.6 TLS verified, `--insecure` warns

---

## Appendix D — Worked example

A reference tool, `acme`, that obeys the standard end-to-end.

**Command tree (noun-verb, depth ≤ 3):**

```
acme
├── project
│   ├── list           # ls
│   ├── create <name>
│   ├── view <name>
│   ├── describe <name>    # expanded detail (R2.1)
│   ├── export <name>      # portable bundle; --output-file <file>, -o sets format (R2.1, R3.2)
│   └── delete <name>  # rm, confirms unless --yes/--force
├── deploy
│   ├── list
│   ├── create <name>  --replicas --region --dry-run
│   ├── apply --file <f>   # declarative; -f reserved for force, so --file (R3.4)
│   ├── logs <name>    --follow   # domain verb (R2.1)
│   └── delete <name>
├── registry
│   └── image
│       ├── push <ref>     # domain verb (R2.1)
│       └── pull <ref>     # domain verb (R2.1)
├── auth                   # networked tools (§10)
│   ├── login
│   ├── logout
│   └── status
├── config             # get/set/view local + user config
├── completion <shell>
├── help [command]
└── (global: -h/--help, -V/--version, -v/--verbose, -q/--quiet,
            -o/--output, --json, --config, --debug)
```

**Sample top-level `--help`:**

```
$ acme --help
acme — manage projects, deployments, and images.

Usage:
  acme <resource> <action> [arguments] [flags]

Resources:
  project     Manage projects
  deploy      Manage deployments
  registry    Manage the image registry
  config      Read and write configuration

Other commands:
  completion  Generate shell completion scripts
  help        Help about any command

Global flags:
  -h, --help             Show help
  -V, --version          Show version
  -v, --verbose          Increase verbosity (repeatable: -vv, -vvv)
  -q, --quiet            Suppress non-essential output
  -o, --output string    Output format: table|json|yaml|wide|name (default "table")
      --json             Shorthand for --output json
      --config string    Path to config file (default: $XDG_CONFIG_HOME/acme/config.toml)
      --debug            Emit maximum diagnostics

Examples:
  acme project list -o json
  acme project export demo -o yaml --output-file backup.yaml
  acme deploy create web --replicas 3 --region us-east-1
  acme deploy logs web --follow

Exit codes: 0 ok · 1 error · 2 usage · 3 not-found · 4 auth · 5 conflict · 130 interrupted · 143 terminated
```

**Behaviors that demonstrate the rules:**

```bash
acme deploy create web --replicas 3 --region us-east-1   # positionals=identity, flags=modifiers (R2.3)
acme deploy list --json | jq '.[].name'                  # machine output to stdout, logs to stderr (R7.1)
acme project delete demo                                 # prompts (R8.1)
acme project delete demo --yes                           # non-interactive consent (R8.1/R8.2)
acme widget list | head -3                               # clean SIGPIPE exit (R9.6)
ACME_TOKEN=… acme registry image push app:1.2.3          # secret via env, not argv (R5.5)
acme deploy create web --dry-run                          # preview, no changes (R4.3)
acme project export demo -o yaml --output-file backup.yaml  # -o sets format, --output-file the destination (R3.2)
acme project export demo -o json --output-file -            # --output-file - writes to stdout (R3.2)
acme deploy apply --file ./deploy.toml                      # file input is --file; -f stays force (R3.4)
acme auth login                                            # credentials via login flow, not argv (R10.1)
acme widget list --all -o json                             # fetch full set explicitly (R10.3)
acme deploy logs web --follow                              # streaming; Ctrl-C exits 130 (R7.10)
```

---

## Appendix E — Framework mapping (informative)

How to satisfy the standard in common frameworks. This appendix is guidance, not a requirement; the normative rules are language-agnostic.

**Rust — Clap (derive).**
Clap defaults already align well: it generates `-h`/`--help` and `-V`/`--version` (capital `V`), so reserve lowercase `-v` for verbose via `#[arg(short, long, action = ArgAction::Count)]`. Use subcommands (`#[derive(Subcommand)]`) for noun-verb; `ArgAction::Count` gives repeatable `-v`; `value_enum` backs `-o/--output`. Pair with `figment`/`config` crates for the precedence chain and `directories`/`etcetera` for XDG paths. Exit with `std::process::exit(code)` using the §6 codes (note: a panic yields `101`, which you should avoid surfacing as a normal failure).

**Python — argparse / Click / Typer.**
- *argparse*: provides `-h`/`--help` and a `version` action; add `--version` explicitly and keep `-v` for verbose (`action="count"`). It exits `2` on parse error — already matching R6.1. Use subparsers for noun-verb.
- *Click*: `--help` is automatic (no `-h` unless you add it); add `@click.version_option()` and set `-V`. Use `@click.group()`/`@click.command()` for structure, `count=True` for repeatable `-v`, and handle `BrokenPipeError` to satisfy R9.6.
- *Typer*: same engine as Click; lean on enums for `--output`, and set `context_settings` to add `-h`.
For XDG use `platformdirs`; for config, `tomllib` (stdlib read) plus `tomli-w` to write.

**TypeScript — oclif / Commander.**
- *Commander*: defaults to `-V`/`--version` and `-h`/`--help`, which matches the standard; declare subcommands for noun-verb and `.option('-v, --verbose', ..., (_, p) => p+1, 0)` for repeatable verbose.
- *oclif*: topic/command structure maps naturally to noun-verb; it provides help and plugin-based completion. Use `env-paths` for XDG-style locations and `process.exitCode` with the §6 codes; install a `process.stdout.on('error')` handler for `EPIPE` (R9.6).

---

*End of standard. To evolve it, bump the version (SemVer), add a Decision Log row, and update the affected rule and the conformance checklist together so they never drift.*
