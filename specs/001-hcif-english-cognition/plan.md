# Implementation Plan: HCIF English-First Cognitive Conversation and Action Framework

**Branch**: `001-hcif-english-cognition` | **Date**: 2026-10-15 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-hcif-english-cognition/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Build an English-first, offline-capable cognitive framework in **C#/.NET 10**. A **custom
hierarchical finite-automata runtime** drives each utterance through
`Data → Information → Knowledge → Intelligence → Action`. Each stage produces typed, immutable,
lineage-linked objects and an inspectable trace.

- **Lexical knowledge**: **Open English WordNet 2025 JSON**, acquired from en-word.net and verified
  against a pinned SHA-256. That digest matches the GitHub `2025-edition` release asset. Its license
  files come from the source repository. The data is imported through a schema-validated, staged
  and idempotent importer into a per-release **SQLite** lexical repository. Inverse relations, which
  the source does not store, are materialised with provenance.
- **Comprehension**: comes from a declarative **English language pack**. It contains a
  closed-class lexicon (WordNet has no function words), Morphy-style morphology with
  irregular-feature tables, a feature-CFG grammar parsed by an **Earley** parser that keeps
  ambiguity, rule-based NER/coreference with strictly separated referent gender, and knowledge-based
  WSD.
- **Knowledge**: a forward-chaining engine over a **declarative rule DSL** (P27 extended). It
  keeps a justification graph, explicit epistemic status, and budgets.
- **Intelligence**: a transparent multi-criteria decision policy.
- **Actions**: policy- and confirmation-gated through `ActionExecutionAutomaton`, with idempotent
  handlers and append-only audit. The baseline includes a mock messaging handler.
- **Delivery**: an API-first service (ASP.NET Core minimal API) plus the `hcif` CLI, delivered in
  milestones M1–M4 that follow the constitution's 18-step sequence.

## Technical Context

**Language/Version**: C# 14 on .NET 10 LTS (local SDKs 10.0.302 / 10.0.401). Nullable enabled,
warnings as errors, Central Package Management.

**Primary Dependencies**:
- `Microsoft.Data.Sqlite`: lexical and knowledge stores.
- `System.Text.Json`: streaming import, snapshots.
- ASP.NET Core minimal API + `Microsoft.AspNetCore.OpenApi`.
- `System.CommandLine`: CLI.
- `Microsoft.CodeAnalysis.CSharp` (Roslyn): dynamic rules, in `Hcif.RuleHost` only.
- `System.Diagnostics` `ActivitySource`/`Meter`, with optional OpenTelemetry exporters.

Built in-house, with no third-party NLP or automata libraries (see research R2, R7, R10): the
automata runtime, Earley parser, morphology, WSD and rule engine. Package versions are pinned in
`Directory.Packages.props` at implementation time.

**Storage**: SQLite (WAL).
- `data/dictionary/releases/oewn-2025.<importId>.db`: one file per release; staged and atomically
  published through `active-release.json`.
- `data/hcif.db`: conversations, cognitive objects, knowledge graph, justifications, automaton
  runs/snapshots/traces, action outbox/audit, memory.
- Original archive and license files are preserved under `data/dictionary/originals/oewn-2025/`.

**Testing**:
- xUnit v3 and `Microsoft.AspNetCore.Mvc.Testing`.
- Contract test kits per interface family.
- Reflection-based architecture tests (P55/P56 boundaries).
- An OEWN-derived mini-lexicon fixture for fast tests; full-dictionary tests are gated by
  `HCIF_OEWN_ARCHIVE`.
- `Hcif.Evaluation` harness and `hcif bench`.

**Target Platform**: Windows 11 (dev) and Linux x64 (server/CI). Cross-platform .NET 10. Fully
offline after dictionary initialization.

**Project Type**: Multi-project .NET solution. It has class-library core layers, a web-service host
(`Hcif.Api`), a CLI (`Hcif.Cli`) and an isolated worker process (`Hcif.RuleHost`).

**Performance Goals**: p95 ≤ 2 s per short English utterance, offline, after warm-up, on a
documented reference machine, with p50/p95 and per-stage timing reported (SC-010). The import is a
measured one-off and is excluded. Dictionary query performance (P70) on the full release after
warm-up: p95 ≤ 20 ms per lemma lookup and p95 ≤ 50 ms per `IsA` traversal (e.g. guitar → musical
instrument). These are provisional engineering targets, asserted in the full-dictionary test
(T044) and revisable with measured evidence.

**Constraints**:
- Offline lookup (P57, SC-003).
- Deterministic baseline with no LLM dependency (mission, spec out-of-scope).
- Bounded reasoning: depth, transitions, rule firings, traversal and wall-clock budgets (P40, P66).
- Inputs of ≤ 40 tokens per sentence by default; longer inputs are segmented.
- Untrusted text is never authority (P51).
- Zero external side effects without authorization (SC-008).

**Scale/Scope**:
- Single-user/small-team PoC.
- Dictionary of 128,009 lemmas, 185,129 senses and 107,519 synsets (≈72 MB source JSON).
- Evaluation sets of ≥300 sentences, ≥100 morphology items and ≥100 WSD items.
- 7 user stories and 60 FRs across milestones M1–M4.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The gates come from the constitution's **Development Workflow → Constitution Check Gates**, plus the
spec's plan-stage checklist (§ Constitutional Traceability).

| # | Gate | Principles | Pre-research | Post-design | Evidence |
|---|---|---|---|---|---|
| G1 | Spec completeness: requirements, assumptions, acceptance scenarios, measurable criteria on identified datasets | P53, P49 | ✅ | ✅ | spec.md (60 FRs, 7 stories, SC-001–013); datasets `data/eval/*-v1` defined in quickstart §9 |
| G2 | Sequence: no dependency on an unvalidated earlier step of the 18-step order | P54 | ✅ | ✅ | Milestone mapping below; M1 builds steps 1–8 before M2's steps 9–14 |
| G3 | Automata discipline: FSA per stage, standard contract, state separate from typed objects, P45 test coverage | P2, P3, P21–P25, P45 | ✅ | ✅ | R2; contracts/automaton-definition.schema.json; data-model §3; `IAutomaton` contract kit |
| G4 | Architectural boundaries: no parse→action coupling; no bypass of knowledge validation/authorization; no app-specific behaviour in core automata | P42, P55 | ✅ | ✅ | Project dependency graph below; architecture tests (R20); action lifecycle guards (data-model) |
| G5 | Epistemic integrity: provenance and status preserved; hypotheses never presented or stored as facts; no stereotype/gender inference | P12, P13, P33, P65 | ✅ | ✅ | R8, R11; Knowledge validation rules; rules may not emit `Verified`/`UserAsserted` (rule-language.ebnf) |
| G6 | Security: authorization, untrusted input, rule isolation, audit; real external actions only with integration-test authorization | P20, P28, P48, P50–P52 | ✅ | ✅ | R12, R15, R16; confirmation tokens; outbox; append-only audit; mock-only handlers in tests |
| G7 | Bounded reasoning: budgets, depth limits, cycle detection, relevance filtering, evidence requirements | P40, P66 | ✅ | ✅ | `ExecutionBudget` in every contract; R10 engine budgets; `TraversalOptions` (kinds, depth, nodes, objective-guided `RelevanceFilter`, `EvidenceRequirement`, budget) |
| G7a | Lexical automata subordinate to both the Information and Knowledge stages | P67 | ✅ | ✅ | `OntologyReasoning` invokes `LexicalIntelligenceAutomaton`; rule built-ins invoke `SemanticRelationshipTraversalAutomaton` via the passed execution context (DM §3, R10) |
| G7b | Synchronous + asynchronous actions; suspension/resumption; bounded retries | P18, P44 | ✅ | ✅ | DM action lifecycle (`AwaitingExternalCompletion`, `RetryScheduled`, suspendable states); `IActionResumptionService`; R15 |
| G7c | Feedback recorded for all P37 categories | P37 | ✅ | ✅ | DM `Feedback`; `IFeedbackService`; `POST /turns/{id}/feedback`; R16a |
| G8 | English-first scope: no English assumptions in core automata; English rules in the pack | P56 | ✅ | ✅ | `Hcif.Cognition` has no reference to `Hcif.Language.English` (arch test); every language service (incl. language ID, NER, coreference, parser) reachable through `ILanguagePack`; FeatureBundle in core types |
| G9 | Dictionary: official source, preserved originals, offline use, schema-aware idempotent import, licensing | P57–P60, P69 | ✅ | ✅ | R3–R5; observed schema inspected on real data *before* defining mappings; license files fetched and hashed |
| G10 | Lexical processing automata and persistable lexical results | P61–P68 | ✅ | ✅ | `LexicalIntelligenceAutomaton` + 6 children; `LexicalAnalysis` cache keyed by release + rule version |
| G11 | Testing: dictionary, grammar corpus, inference, action tests; baseline "Seeta plays guitar" | P45–P48, P70 | ✅ | ✅ | quickstart §1–§5; contract kits; mini-lexicon fixture extracted from the real archive |
| G12 | Not LLM-dependent; pluggable engines; optional external services | Mission, P38, P39 | ✅ | ✅ | All baseline engines deterministic and in-process; `IParser`, `IDecisionPolicy` and providers replaceable |

**Result: PASS with documented deviations.** Under Governance → Compliance, the phase-scoped
deferrals of P19 (most action categories), P29–P31 (multilingual comprehension) and P38
(probabilistic engines) are explicitly justified in **Complexity Tracking → Constitutional
deviations**. None of them is a removal. Each has a target milestone, and the architecture keeps
the extension points they need. There are no other violations.

**Notes raised by research that the implementation must honour**:
1. The archive contains **no license text**. Acquisition MUST fetch `LICENSE.md` and
   `WNDB_License.txt` from the `2025-edition` tag (P69, SC-012).
2. WordNet lacks **regular inflections, function words and inverse relations**. Morphology tables,
   the closed-class lexicon and importer-derived inverses (flagged `derivedInverse`) fill these
   gaps, and nothing is presented as source data that the source doesn't contain (P58).
3. `hamlet` (lower case) exists only as a settlement. Proper-name precedence for mid-sentence
   capitals is required to meet the `The actor plays Hamlet.` corpus row.

## Project Structure

### Documentation (this feature)

```text
specs/001-hcif-english-cognition/
├── plan.md              # This file
├── research.md          # Phase 0 output (R1–R21)
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output (validation guide)
├── contracts/
│   ├── http-api.openapi.yaml
│   ├── cli.md
│   ├── automaton-definition.schema.json
│   ├── rule-language.ebnf
│   ├── component-interfaces.md
│   └── language-pack.md
├── checklists/requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks — NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
Hcif.sln
Directory.Build.props                # nullable, warnaserror, LangVersion
Directory.Packages.props             # central package versions

src/
├── Hcif.Abstractions/               # language-neutral contracts & canonical types (P41, P42) — no dependencies
├── Hcif.Automata/                   # HFA runtime: definitions, validator, runner, budgets, traces, snapshots (P2, P25, P40, P44)
├── Hcif.Lexicon/                    # ILexicalDictionary & repositories over SQLite; traversal; WSD; LexicalIntelligenceAutomaton (P59, P61–P68)
├── Hcif.Lexicon.Import/             # OEWN acquisition, integrity, safe extraction, schema validation, staged import, attribution (P57, P60, P69)
│   └── schema/oewn-2025-json.observed.json
├── Hcif.Language.English/           # English pack: tokenizer, morphology, closed-class lexicon, grammar (.hgr), Earley parser, NER, coref, semantics, NLG
│   └── Pack/                        # declarative resources (contracts/language-pack.md)
├── Hcif.Knowledge/                  # knowledge graph repository, HRL parser, forward-chaining engine, justifications, contradictions (P11–P14, P26, P27)
│   └── Rules/baseline/*.hrl
├── Hcif.Intelligence/               # intent, context evaluation, candidates, decision policy, risk (P15–P17, P24)
├── Hcif.Actions/                    # action registry, ActionExecutionAutomaton, authorization/confirmation, handlers, outbox, audit (P18–P20, P52)
├── Hcif.Memory/                     # conversation context, memory layers, retention, erasure cascade (P32, P35, P50)
├── Hcif.Cognition/                  # CognitiveLifecycleAutomaton + stage coordinator automata + definitions JSON (P1, P21–P24)
│   └── Definitions/*.json
├── Hcif.Hosting/                    # composition root: DI wiring, configuration, pack/provider binding
├── Hcif.Api/                        # ASP.NET Core minimal API (contracts/http-api.openapi.yaml)
├── Hcif.Cli/                        # `hcif` CLI (contracts/cli.md)
├── Hcif.RuleHost/                   # isolated worker for Roslyn-compiled dynamic rules (P28, R12)
└── Hcif.Evaluation/                 # eval harness + benchmark (FR-057, SC-004–SC-006, SC-010)

tests/
├── Hcif.Automata.Tests/             # P45 checklist, budgets, snapshots
├── Hcif.Lexicon.Tests/              # lookup, traversal bounds, WSD, cache invalidation (fixture)
├── Hcif.Lexicon.Import.Tests/       # integrity, zip-slip, schema drift, idempotency, rollback (P70)
├── Hcif.Language.English.Tests/     # morphology, grammar corpus, NER/coref, gender separation
├── Hcif.Knowledge.Tests/            # HRL parsing, inference, provenance, contradiction, erasure cascade (P47)
├── Hcif.Intelligence.Tests/
├── Hcif.Actions.Tests/              # lifecycle, authorization-negative, idempotency, OutcomeUnknown (P48)
├── Hcif.Contract.Tests/             # contract test kits + component registration (SC-013)
├── Hcif.Architecture.Tests/         # dependency rules (P55, P56), no-network in lexicon (SC-003)
├── Hcif.Integration.Tests/          # API + full pipeline; FullDictionary/Offline categories
├── Hcif.Acceptance.Tests/           # baseline corpus table (SC-001), FR-060
├── Hcif.Memory.Tests/               # retention defaults, TTL, erasure eligibility, feedback storage
├── Hcif.Cognition.Tests/            # contract tests for automaton definitions owned by Hcif.Cognition (root, Information, Knowledge, Intelligence)
├── Hcif.TestSupport/                # shared helpers: temp data roots, NetworkGuard, contract test kits, fixture release
└── fixtures/
    ├── oewn-mini/                   # extracted from the real archive + provenance + CC BY 4.0 attribution
    ├── rules/                       # cyclic.hrl, EvilFileAccess.cs, …
    └── handlers/

data/
├── eval/                            # semantic-core-v1, morphology-v1, wsd-v1, bench-100.txt (versioned, annotated)
└── dictionary/                      # runtime (git-ignored): originals/, releases/, active-release.json
```

**Persistence layout**: each library that owns tables embeds forward-only scripts in
`src/Hcif.<Layer>/Migrations/NNNN_*.sql` (exposed via `IMigrationSource`). `Hcif.Hosting` runs
them through the BCL `DbConnection` abstraction (`IDbConnectionFactory`), so no separate
persistence project is needed.

**Configuration ownership**: each library owns its options type (`LexiconOptions`,
`MemoryOptions`, `ActionOptions`, `IntelligenceOptions`, `InputOptions`). `Hcif.Hosting` binds
them with `services.Configure<T>(section)`, and `HcifOptions` only aggregates the sections.
Principals and roles reach libraries through `IPrincipalDirectory` (Abstractions). No library
references `Hcif.Hosting` (research R18).

**Test placement**: each test project references only its subject project plus `Hcif.TestSupport`.
Contract tests for automaton definitions owned by `Hcif.Cognition` live in `Hcif.Cognition.Tests`.
Tests that need the full pipeline or HTTP host live in `Hcif.Integration.Tests` or
`Hcif.Acceptance.Tests`.

**Dependency rules** (enforced by `Hcif.Architecture.Tests`):
- `Hcif.Abstractions` depends on nothing.
- No `src` library other than `Api`, `Cli` and `Evaluation` references `Hcif.Hosting`.
- `Hcif.Automata` depends only on Abstractions.
- Stage libraries (`Lexicon`, `Language.English`, `Knowledge`, `Intelligence`, `Actions`,
  `Memory`) depend on Abstractions and Automata only. **`Language.English` and `Lexicon` MUST
  NOT reference `Actions`** (P55).
- `Hcif.Cognition` depends on Abstractions and Automata only. It obtains stage implementations
  through DI and MUST NOT reference `Language.English` (P56).
- `Hosting` is the only composition root. `Api` and `Cli` reference `Hosting`.
- `Hcif.Lexicon` has no HTTP dependency (SC-003). Only `Lexicon.Import` may use the network.

**Structure Decision**: Use a single .NET solution with one project per architectural layer from
P42 (input/communication = `Api`/`Cli`; language = `Language.English` + `Lexicon`; canonical
semantics = `Abstractions`; knowledge = `Knowledge`; intelligence = `Intelligence`; actions =
`Actions`; persistence/memory = `Memory` + repositories; observability/admin = traces in
`Automata`/`Hosting` + admin endpoints). This makes the constitutional boundaries compile-time
facts that architecture tests can verify.

### Milestone mapping to the mandatory development sequence (P54)

| Milestone | Sequence steps | User stories | Exit criteria |
|---|---|---|---|
| **M1** Core and lexical foundation | 1 FSA core · 2 hierarchical composition · 3 OEWN acquisition · 4 importer + lexical repository · 5 lookup/indexing · 6 tokenization + morphology · 7 English parsing · 8 canonical information objects | US2, US1 | SC-002, SC-003, SC-005 (evaluation harness + morphology dataset built in US1), SC-011; US1 scenarios pass |
| **M2** Knowledge and conversation (**first functional milestone**) | 9 WSD · 10 semantic traversal · 11 knowledge repository · 12 deterministic inference · 13 intelligence synthesis · 14 conversational response | US3, US4 | Milestone chain demonstrable with inspectable intermediates; SC-001 (statement/question rows), SC-004 (dataset in US4), SC-006 (dataset in US3), SC-007, SC-009, SC-010 (benchmark in US4) |
| **M3** Controlled actions and conversational memory | 15 controlled action execution · 16 conversational memory and context retention | US5, US6 | SC-001 (all rows), SC-008; async/retry/resumption tests; feedback tests (P37); correction/erasure scenarios |
| **M4** Advanced cognition and languages (this feature: contract groundwork only) | 17 advanced/probabilistic reasoning (stub automaton returns `NotConfigured`) · 18 additional language packs (pack contract only) | US7 | SC-012, SC-013; dynamic-rule isolation tests |

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

### Constitutional deviations (Governance → Compliance)

These are phase-scoped deferrals, justified by P56 (English-first initial release) and P54
(mandatory sequence: step 17 advanced/probabilistic reasoning, step 18 additional language packs).
None removes an obligation.

| Principle | Deferred in this feature | What this feature still guarantees | Target |
|---|---|---|---|
| P29–P30 (modular language packs, language-neutral knowledge) | Language packs other than English; cross-language proposition mapping | Pack contract + manifest (`contracts/language-pack.md`); language-neutral canonical types; WordNet `ili` stored on synsets for future cross-lingual linking | Sequence step 18 (separate feature) |
| P31 (multilingual conversation) | Comprehension of non-English and mixed-language input; multilingual response generation | Language detected **per turn and per sentence**; `ConversationContext.language` updated every turn and never fixed to the first turn; non-English sentences kept as Data with `LANGUAGE_UNSUPPORTED`; English sentences in mixed input still processed (R16b) | Step 18 |
| P19 (action categories) | Real e-mail, API, DB, OS, device, machine, scheduled and composite handlers | All categories declared as `ActionType` with the full handler contract; unimplemented types are `Blocked` (`HandlerUnavailable`); device/machine registration requires `ISafetyInterlock` | Separate integration features |
| P38 (probabilistic/ML strategies) | Probabilistic inference, calibrated confidence, ML/neural services | `ProbabilisticInferenceAutomaton` slot returns `NotConfigured`; `confidence` stays null; pluggable `IInferenceEngine`/`IParser` contracts | Sequence step 17 |

### Sequencing note (P54)

| Item | Why | Mitigation |
|---|---|---|
| Minimal `ConversationContextStore` built in US4 (steps 13–14) before step 16 | Question answering and salience need a working context within a conversation | Only working-context persistence is built in US4. Layered memory, retention, coreference and erasure (step 16) remain in US6 |

### Design choices that add complexity

These deliberate design choices add complexity and are recorded for review:

| Choice | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| 15 `src` projects | P42 requires eight distinct layers with explicit contracts. Separate assemblies make P55/P56 boundaries compiler- and test-enforced. | One project with folders cannot prevent forbidden references (e.g. grammar → action handler). |
| Custom HFA runtime instead of a library | P2/P25 need versioned definitions, typed child invocation, budgets, persistence and traces together. | `Stateless` and workflow engines lack child-run contracts, budgets or finite-state semantics (R2). |
| Custom Earley parser + feature grammar instead of an NLP library | P8/P9 require declarative, pack-resident grammars, and P5/FR-014 require explicit ambiguity and failure. | CoreNLP is GPL (licence review fails); spaCy/Stanza are statistical, non-declarative and add a Python runtime (R7). They remain possible later as pluggable `IParser`s. |
| Separate `Hcif.RuleHost` process for dynamic rules | P28 requires isolation for untrusted compiled code, and .NET has no in-process sandbox. | In-process `AssemblyLoadContext` is not a security boundary (R12). |
| Importer-materialised inverse relations | P58 requires hyponym/holonym retrieval, and the source stores only one direction. | Computing inverses per query needs full scans; storing them unflagged would misrepresent the source. |
| Two SQLite databases (per-release lexicon + application) | Atomic per-release publication and rollback (P60, SC-002) independent of knowledge persistence (P14). | One database complicates staging/rollback and couples dictionary versions to user data. |
