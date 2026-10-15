# CLI Contract: `hcif`

The CLI is a thin client over the same application services as the HTTP API (P43). Exit codes:
`0` success · `1` usage error · `2` validation/integrity failure · `3` not initialized ·
`4` budget exceeded · `5` internal error. Every command supports `--json` for machine-readable
output on stdout. Diagnostics go to stderr.

## Dictionary administration (US2, FR-016–FR-026)

| Command | Behavior |
|---|---|
| `hcif dictionary acquire [--archive <path>] [--offline]` | Downloads the archive from the configured official URL, or takes `--archive`. Verifies it against the pinned SHA-256 and fetches/verifies `LICENSE.md` + `WNDB_License.txt`. Stores the originals under `data/dictionary/originals/oewn-2025/`. `--offline` forbids network and requires local files. Exit 2 on digest mismatch. |
| `hcif dictionary import [--force]` | Runs `DictionaryImportAutomaton` on the stored originals, then stages, validates and publishes. Prints the validation report. Re-running on the same digest prints `AlreadyImported` (exit 0) unless `--force` is given. `--force` must produce an identical `contentHash`. |
| `hcif dictionary init` | `acquire` + `import`. |
| `hcif dictionary status` | Shows the active release, the counts, the digest, the previous releases and the license files. |
| `hcif dictionary verify` | Re-hashes the originals and the active DB `contentHash`, and checks referential integrity. |
| `hcif dictionary rollback --to <importId>` | Re-points the active release to a previous validated import. |
| `hcif dictionary attribution [--out <file>]` | Emits the `THIRD-PARTY-NOTICES` text: Princeton WordNet License + OEWN CC BY 4.0. |
| `hcif dictionary lookup <word> [--pos n\|v\|a\|s\|r] [--relations hypernym,…] [--depth N]` | Offline lookup that shows morphology, entries, senses, synsets and bounded relations. |
| `hcif dictionary extract-fixture --lemmas <file> --out <dir>` | Builds the test mini-lexicon (closure of the given lemmas plus hypernym chains) with provenance. |

## Conversation (US1, US3–US6)

| Command | Behavior |
|---|---|
| `hcif chat [--conversation <id>] [--trace] [--principal <name>]` | Interactive REPL. Each line is one turn. `--trace` prints a stage/transition summary after each reply. REPL meta-commands: `:confirm <actionId>`, `:cancel <actionId>`, `:trace [turnId]`, `:memory`, `:forget <memoryId>`, `:feedback <category> <+\|-\|0> [comment]` (applies to the last turn), `:resume <turnId>`, `:quit`. |
| `hcif say "<text>" [--conversation <id>] [--json]` | Single turn, non-interactive (used by acceptance scripts). |
| `hcif trace show <turnId> [--json]` | Full processing trace (FR-055). |
| `hcif knowledge explain <knowledgeId>` | Explanation graph (FR-033). |

## Quality and evaluation (FR-056, FR-057, SC-004–SC-006, SC-010)

| Command | Behavior |
|---|---|
| `hcif eval run --dataset <data/eval/...> [--out report.json]` | Scores against an annotated, versioned dataset (`manifest.json` + `items.jsonl`). The manifest MUST contain `version`, `sha256`, a frozen `scoringCriterion` and, for WSD datasets, a frozen `abstentionCriterion`. If one is missing or the sha256 mismatches, the command exits **2** without scoring (SC-004, SC-006). Reports per-category metrics, abstention frequency, the criterion text and the dataset version/hash. |
| `hcif bench --utterances <file> --warmup 10 [--out bench.json]` | Reports p50/p95 latency and per-stage timing, plus machine info. Import is excluded. |
| `hcif rules check <file.hrl>` | Parses/validates rules and runs their declared tests. |

## Components (US7)

| Command | Behavior |
|---|---|
| `hcif components register --kind <kind> --path <pkg>` | Runs the contract test kit. Exit 2 with reasons on rejection. |
| `hcif components list` | Lists registered components with versions and contract versions. |
