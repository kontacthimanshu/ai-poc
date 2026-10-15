# Phase 0 Research: HCIF English-First Cognitive Conversation and Action Framework

**Feature**: `001-hcif-english-cognition` | **Date**: 2026-10-15 | **Spec**: [spec.md](./spec.md)

This document resolves every Technical Context unknown in [plan.md](./plan.md). Each entry has
three parts: Decision, Rationale, and Alternatives considered. R3 and R4 record **observed facts**
from inspecting the official Open English WordNet 2025 JSON archive on 2026-10-15. The spec requires
this inspection to happen before any mapping is defined (spec § Deferred to Planning, P60).

---

## R1. Language, runtime and solution shape

- **Decision**: C# 14 on **.NET 10 (LTS)**. The local SDKs 10.0.302 and 10.0.401 are installed. A
  multi-project solution (`Hcif.sln`) uses Central Package Management (`Directory.Packages.props`)
  with nullable reference types and warnings-as-errors enabled. Outputs are a class-library core, an
  ASP.NET Core minimal-API host (`Hcif.Api`), and a CLI (`hcif`).
- **Rationale**: P41 names modern C#/.NET (SHOULD). P43 requires an API-first, UI-independent core.
  .NET 10 is the current LTS and is already installed. Strong typing, records and immutable
  collections fit P41's "immutable processing results where practical".
- **Alternatives considered**: Python offers a richer NLP ecosystem, but it conflicts with P41 and
  gives weaker typing for the 18 core entities. Java (Stanford CoreNLP) has the same P41 conflict
  plus GPL licensing (see R7).

## R2. Hierarchical finite-automata runtime

- **Decision**: Build a **custom HFA runtime** (`Hcif.Automata`) with **declarative automaton
  definitions** (states, initial/terminal states, transitions, guards, entry/exit operations,
  failure/recovery transitions, I/O event types, limits, cancellation). It uses **UML-statechart
  run-to-completion semantics restricted to sequential composition**: a parent state may *invoke* a
  child automaton, wait for its typed outcome, and then transition. There are no orthogonal regions
  in this feature. Guards and actions are external delegates registered by ID, so they can never
  add or remove states at runtime (P2). The runtime enforces **depth, transition-count, wall-clock
  and per-run step budgets** (P40). It emits a structured transition trace and supports
  suspend/snapshot/resume with versioned JSON snapshots (P44).
- **Rationale**: P2 and P25 require a contract that no off-the-shelf .NET library provides as a
  whole: versioned definitions, typed I/O, child invocation with outcomes, budgets, cancellation,
  persistence and traces. The core is small (~1–2 kLOC) and fully testable against P45's checklist.
- **Alternatives considered**:
  - `Stateless`: flat or substate machines with no child-run invocation, budgets, typed outcomes or
    persistence model, so it would need a wrapper bigger than the runtime itself.
  - Akka.NET / Orleans actor FSMs: distributed-actor overhead and no statechart contract.
  - Workflow engines (Elsa, Temporal): heavyweight, and they don't preserve finite-state semantics.

## R3. Open English WordNet 2025 JSON — observed source schema

The JSON is **not** documented as a formal schema. Everything below was observed on the real
archive. Per P60 / FR-018, the importer validates against an **observed-schema manifest**
(`schema/oewn-2025-json.observed.json`) generated from this inspection. It fails, rather than
guessing, on any unknown top-level shape or unknown relation key; unknown optional keys are reported
as warnings.

**Archive**: `english-wordnet-2025-json.zip`, 9,986,555 bytes. Contains 73 JSON files (72.4 MB
uncompressed). There are **no license files and no subdirectories** inside the archive.

| File group | Count | Top-level shape |
|---|---|---|
| `entries-0.json`, `entries-a.json` … `entries-z.json` | 27 | `{ "<lemma>": { "<posKey>": Entry } }` |
| Lexicographer synset files `noun.*`, `verb.*`, `adj.all/pert/ppl`, `adv.all` | 45 | `{ "<synsetId>": Synset }` |
| `frames.json` | 1 | `{ "<frameId>": "<template string>" }` |

**Totals observed**: 128,009 lemma keys; 135,969 entries (lemma × posKey); 185,129 senses;
107,519 synsets (n 71,864 · v 13,821 · s 10,717 · a 7,502 · r 3,615).

**Entry** (`entries-*.json`), with property frequencies:
- `sense` (135,969, required): array of Sense
- `pronunciation` (32,473): `[{ "value": IPA, "variety"?: "GB"|"US"|… }]`
- `form` (3,084): array of **irregular/variant inflected forms only**, e.g. `go/v → ["gone","went"]`,
  `abacus/n → ["abaci"]`. Regular inflections such as `plays` are **not** listed.

**posKey values**: `n`, `v`, `a`, `r`, `s`, plus **homograph-disambiguated** keys `n-1`, `n-2`,
`v-1`, `v-2`, `a-1`, `a-2` (e.g. `bass → a, n-1, n-2`). The importer splits posKey into
`(pos, homographIndex)`.

**Lemma keys** keep case and spaces (`"musical instrument"`, `"11 November"`). Multi-word
expressions use spaces, not underscores. Proper-name casing matters: `hamlet` (lower case) exists
only as *settlement* / *community* senses.

**Sense**: `id` (sense key, e.g. `play%2:36:01::`), `synset` (synset ID), both required. Optional
**sense-level relation arrays whose targets are sense IDs**: `derivation`, `antonym`, `pertainym`,
`also`, `similar`, `participle`, `exemplifies`, and the morphosemantic links `agent`, `event`,
`result`, `undergoer`, `instrument`, `uses`, `state`, `property`, `location`, `material`,
`vehicle`, `body_part`, `by_means_of`, `destination`. Non-relation keys are `subcat` (verb frame
IDs → `frames.json`), `sent` (example sentences) and `adjposition`.

**Synset**: `definition` (array of strings, required), `members` (lemma strings, required),
`partOfSpeech` (required), `ili` (Interlingual Index, 104,335), `example` (array of strings),
`wikidata` (Q-IDs, *external* references, not synset IDs), `source` (string, e.g.
`"Colloquial WordNet"`). **Synset-level relation arrays whose targets are synset IDs**: `hypernym`,
`similar`, `domain_topic`, `domain_region`, `mero_part`, `mero_substance`, `mero_member`,
`exemplifies`, `also`, `attribute`, `entails`, `causes`.

**Inverse relations are not stored.** There are no `hyponym`, `holo_*`, `has_domain_*` or
`instance_*` keys. **Decision**: the importer materialises inverse edges (`hyponym` from
`hypernym`, `holo_part` from `mero_part`, and so on) and flags them `derived_inverse = true` with
the source edge as provenance. This satisfies P58's hyponym/holonym retrieval without claiming the
source supplied them.

**Referential integrity**: every `sense.synset` and every sense- and synset-level relation target
resolves (0 dangling). The only non-resolving values are 1,140 `wikidata` Q-IDs, which are external
by design. SC-002 "no unresolved internal references" is achievable as-is.

**Baseline grounding verified on real data**:
- `guitar%1:06:00::` → `03472449-n` → hypernym `stringed instrument` (04345614-n) → **`musical
  instrument` (03806455-n)** → device → … → entity. So `Guitar → MusicalInstrument` holds at
  hypernym depth 2 (P64). `sitar` → stringed instrument → musical instrument, which is used by
  US6 corrections.
- `play/v` has 35 senses, including `play%2:36:01::` "perform music on (a musical instrument)",
  `play%2:36:12::` "play on an instrument", `play%2:36:02::` "play a role or part" (example
  *"Gielgud played Hamlet"*) and `play%2:33:00::` "participate in games or sport".
- `football%1:04:00::` → field game → … → **sport**. `actor%1:18:00::` → **performer** →
  entertainer.
- `Seeta` and `blorptar` are absent, as expected: unknown-lexeme handling and NER apply (P61, P70).

## R4. Dataset acquisition, integrity and licensing

- **Decision**:
  1. Primary URL: `https://en-word.net/static/english-wordnet-2025-json.zip`, as the constitution
     requires. Integrity is checked against a **pinned SHA-256**,
     `7d749f6e2c39e6970e4997839dcf6e42fd281f3c2fae0171d2192bae8cfa4b51`. This value was observed on
     download and is **identical to the digest GitHub publishes** for the same asset on the
     `globalwordnet/english-wordnet` release `2025-edition` (published 2025-12-31). A mismatch aborts
     the import.
  2. Pre-fetched copies (`--archive <path>`) must match the same digest (FR-016).
  3. The archive ships **no license text**, so acquisition separately fetches `LICENSE.md` and
     `WNDB_License.txt` from the repository at tag `2025-edition` (both verified to exist, HTTP
     200). Their SHA-256 digests are recorded and they are stored with the original archive.
     `LICENSE.md` requires **attribution to both Princeton WordNet and the Open English WordNet
     team**: OEWN under CC BY 4.0, plus the Princeton WordNet License for underlying data. The
     `hcif dictionary attribution` command generates `THIRD-PARTY-NOTICES` from these (P69,
     FR-026, SC-012). Offline installs must supply the license files alongside the archive.
  4. The **original archive is preserved unmodified** under
     `data/dictionary/originals/oewn-2025/` (P57), separate from the internal SQLite
     representation.
- **Rationale**: P57, P60 step 2 and P69. Pinning a digest that two independent official channels
  agree on gives tamper detection without a signature infrastructure.
- **Alternatives considered**: trust-on-first-use (rejected: no verification of the first fetch);
  using the GitHub asset as primary (rejected: the constitution names en-word.net, so GitHub is only
  a corroborating digest source); the `-plus-json` variant (rejected: not the constitutionally named
  dataset).

## R5. Lexical repository storage, staged publication and idempotency

- **Decision**: Use **SQLite** (`Microsoft.Data.Sqlite`, WAL mode) with one database file per
  imported release: `data/dictionary/releases/oewn-2025.<importId>.db`. The importer writes to a
  `*.staging` file. After full validation (reference resolution, counts versus the source, sample
  lookups) it closes the file, atomically renames it, and atomically rewrites
  `data/dictionary/active-release.json`, which points to the active file. A failure at any point
  deletes the staging file and leaves the active pointer untouched (FR-019, US2-4). Source IDs (sense
  keys, synset IDs) are natural primary keys with `UNIQUE` constraints. Re-import of an identical
  archive (same digest) is detected and becomes a no-op that reports "already imported". A forced
  re-import produces a content-identical database, verified by an ordered content hash (SC-002).
- **Rationale**: Embedded storage, zero ops, offline (P57) and fast indexed lookups. A file per
  release gives trivially safe rollback and keeps the previous release queryable (SC-002).
- **Alternatives considered**: PostgreSQL (operational overhead for a single-user PoC); a graph
  database such as Neo4j (licensing and operational overhead; WordNet traversal is shallow and
  bounded per P66, so recursive CTEs or in-memory adjacency are enough); in-memory only (no
  persistence, and slow cold start).

## R6. English morphology

- **Decision**: Use a rule-based **Morphy-style analyser** in the English language pack:
  1. Look up the exception tables first. These come from WordNet `form` arrays plus a curated
     irregular table in the pack (≈200 irregular verbs, plus irregular plurals and comparatives)
     that carries **feature bundles** (e.g. `went → go +PAST`, `gone → go +PTCP`). WordNet `form`
     lists forms without features.
  2. Apply suffix-detachment rules (`-s/-es/-ies`, `-ed`, `-ing` with consonant doubling and e-drop,
     `-er/-est`).
  3. Validate candidate lemmas against the lexicon. Each analysis yields lemma + POS + features
     (person, number, tense, aspect, degree).
- **Rationale**: FR-015 / P63 require morphology that does not depend on WordNet alone. The
  inspection (R3) confirms WordNet lacks regular forms (`plays`) and feature labels.
- **Alternatives considered**: statistical lemmatisers (non-declarative and weaker for the
  irregular-feature bundles SC-005 needs); a Hunspell affix grammar (gives lemmas, not syntactic
  features).

## R7. Parsing approach (syntax, dependencies, roles)

- **Decision**: Use a **declarative feature-based context-free grammar** in the English pack
  (`grammar/english.hgr`). It extends the P9 EBNF baseline with agreement features, auxiliaries,
  negation, interrogatives, imperatives, passives, modals, coordination, subordinate and relative
  clauses, and prepositional phrases. It is parsed by an **Earley chart parser** with feature
  unification, producing a **packed parse forest**. Head rules derive dependencies. Semantic-role
  mapping rules in the pack turn syntax into canonical predicate–argument structures; passive
  `by`-phrases map to the agent, which gives FR-013 compatibility. Parses are ranked by deterministic
  preferences (lexical-sense fit, attachment heuristics). **All surviving parses within a configured
  margin are retained** as alternatives (FR-014). When no full parse exists, the parser returns the
  maximal partial chart edges plus an explicit diagnostic (P5).
  - POS candidates come from three sources: WordNet (open classes n/v/a/s/r); the pack's
    **closed-class lexicon** for determiners, pronouns, auxiliaries, modals, prepositions,
    conjunctions and wh-words, which **WordNet does not contain**; and capitalisation and shape
    features for proper-noun candidates.
- **Rationale**: P8 and P9 require declarative, language-pack-resident grammars that are
  independent of the automata. Earley handles any CFG including ambiguity and recursion, and
  fails explicitly rather than inventing a parse. The approach is fully deterministic and testable
  (P46).
- **Alternatives considered**:
  - Stanford CoreNLP: Java and **GPL-licensed**, which fails P69-style licence review for
    distribution.
  - spaCy/Stanza via a Python sidecar: statistical, non-declarative, and an extra runtime. It can
    be added later as a pluggable `IParser` (P39) and compared on the evaluation set.
  - Catalyst (C# spaCy port): statistical models with no declarative grammar.
  - An LLM as parser: P1/mission says no exclusive dependence on an LLM, and the spec puts it out of
    scope.

## R8. Named entities, coreference and gender

- **Decision**:
  - **Contract placement**: language identification, NER and coreference are exposed through
    `ILanguagePack` (`ILanguageIdentifier`, `IEntityRecognizer`, `ICoreferenceResolver`), so the
    Information stage in `Hcif.Cognition` calls them without referencing the English pack (P56).
    The default coreference resolver marks pronouns `Unresolved`. The salience resolver replaces it
    by registration, not by editing the stage.
  - **NER**: rule-based. Capitalised tokens not at sentence start, or sentence-initial capitalised
    tokens with no lexicon entry, become `ProperName` candidates with type `Unknown`, refined by
    syntactic role and selectional context. This is how `Seeta` and the `Hamlet` role are handled.
    Mid-sentence capitalised `Hamlet` takes priority as a proper name over the lower-case
    `hamlet` settlement senses.
  - **Coreference**: a deterministic salience-ranked resolver over the `ConversationContext`
    window. A pronoun resolves only when **exactly one** salient antecedent is compatible in person
    and number. Grammatical gender is compatible when the referent gender is `Unknown`. With zero
    or several compatible antecedents, the result is `Unresolved` and the Intelligence stage asks a
    clarification question (US6-1).
  - **Gender**: `GrammaticalGender` (on pronouns) and `ReferentGender` (on entities) are separate
    types. `ReferentGender` stays `Unknown` unless explicit user-provided context states it (P6,
    FR-012). A speaker's use of "she" is recorded as an Information-level *pronoun-usage
    observation*. In this feature it is **never promoted** to referent-gender Knowledge (P65,
    SC-007). No external name-gender lexicon is used.
- **Rationale**: P6, P12, P65 and US1-3 forbid name-based gender inference. Unique-compatible
  resolution meets US6-1 ("only when context supports").
- **Alternatives considered**: neural coreference (non-deterministic, opaque, and tends to use
  name–gender priors).

## R9. Word-sense disambiguation (WSD)

- **Decision**: Use **knowledge-based WSD** in `WordSenseDisambiguationAutomaton`. It scores
  candidate senses on four signals:
  - (a) **selectional fit**: the hypernym closure of syntactic arguments, e.g. a `play` object
    under `musical instrument` (03806455-n) favours `play%2:36:01::`; an object under `sport`
    favours `play%2:33:00::`; a subject under `performer` with a proper-name object favours
    `play%2:36:02::`.
  - (b) gloss/example overlap (simplified Lesk).
  - (c) `subcat` frame compatibility with the parse.
  - (d) sense order as a weak prior.
  The scores are deterministic and explainable: each feature contribution is recorded as evidence.
  If the top-2 margin is below the configured threshold, **all candidates above the floor are
  retained** and the result is marked `Ambiguous`, which is the abstention case for SC-006.
- **Rationale**: P62 requires grammatical structure, surrounding words, semantic relations and
  context, and preserved competition. The real-data chains in R3 support the three baseline
  disambiguations.
- **Alternatives considered**: supervised WSD (needs sense-annotated corpora with licence review,
  and is opaque); always taking the first sense (violates P62).

## R10. Rule language and inference engine

- **Decision**: Implement the P27 grammar **extended** with variables (`?x`), negation (`NOT`),
  quantification (`EXISTS`/`FORALL` over bounded collections), temporal constraints, `SCOPE`
  (stage/domain), `VERSION`, and `EVIDENCE` / `EPISTEMIC` annotations on consequences (P27 SHALL).
  The grammar is in [contracts/rule-language.ebnf](./contracts/rule-language.ebnf). Rule files
  (`*.hrl`) are parsed into an AST and evaluated by a **forward-chaining agenda engine** with these
  properties:
  - conflict resolution by priority, then specificity, then rule ID;
  - de-duplication by canonical proposition hash;
  - a **justification graph** (truth-maintenance style) that records rule ID, bindings and
    supporting knowledge/lexical-relation IDs for every derived item (FR-033);
  - budgets for max firings, max derivation depth, max agenda size and wall-clock time; budget
    exhaustion produces a diagnostic and no partial actions (FR-036);
  - cycle detection by (rule, binding-set) re-firing checks.

  `WordNet.IsA(?x, synset)` and related built-in predicates **invoke the bounded
  `SemanticRelationshipTraversalAutomaton` as child runs**. The engine receives the calling
  automaton's `IAutomatonExecutionContext` (from `KnowledgeConstruction → LogicalInference`), and
  `OntologyReasoning` invokes `LexicalIntelligenceAutomaton`. The lexical automata therefore operate
  under both the Information and Knowledge stages (P67, spec §5). Every traversal carries P66's full
  control set (`TraversalOptions`):
  - an allowed-kind allowlist, depth limit, node cap, cycle detection and duplicate suppression;
  - **relevance filtering** guided by the processing objective: for `CheckIsA(ancestor)`, branches
    whose precomputed `hypernymDepth` cannot reach the ancestor are pruned, and an optional `lexFile`
    allowlist applies;
  - **evidence requirements**: allowed `RelationEvidenceClass` set, no `derivedInverse`-only paths for
    IsA, and a maximum path length;
  - an execution budget.

  Pruned or rejected counts are reported in the trace.
- **Rationale**: P26–P27 and P47. A justification graph makes retraction, correction and erasure
  cascade correctly (FR-035, FR-058).
- **Alternatives considered**: NRules (a .NET Rete engine, but rules are C# classes, not a
  declarative DSL, and it doesn't capture provenance as needed); Datalog engines (no .NET-native
  mature option with provenance).

## R11. Epistemic status, certainty and confidence

- **Decision**: Use `EpistemicStatus` with exactly the 10 P12 classes. Use a separate
  `Certainty` (qualitative: `Categorical`, `Probable`, `Possible`, `Hypothetical`,
  `Counterfactual`, `Unknown`) derived from modality and conditionals: "might" → `Possible`.
  Numeric `Confidence` is **nullable** and populated only by components that declare a calibration
  dataset. The deterministic baseline never populates it (P13, FR-034). Speaker assertions are
  `UserAsserted` with source attribution. Dictionary-grounded derivations are `LogicallyDerived`
  with an `EvidenceKind` of `Taxonomic`, `Entailment`, `HeuristicAssociation` or `Hypothesis`
  (FR-032). For example, `PlaysMusicalInstrument(Seeta)` is `LogicallyDerived` **from** a
  `UserAsserted` premise, so it inherits *qualified* status: it carries `dependsOnUnverified = true`
  and is never presented as verified.
- **Rationale**: P12, P13 and P65. This keeps the qualified-derivation semantics of US3-2 explicit.

## R12. Dynamically compiled rules (P28) and isolation

- **Decision**: Dynamic C# rules implement a versioned `IDynamicRule` contract and are compiled with
  **Roslyn** (`Microsoft.CodeAnalysis.CSharp`). Protection has four layers:
  1. **Compile-time policy**: a semantic analyser rejects references to symbols in `System.IO`,
     `System.Net*`, `System.Diagnostics.Process`, `System.Reflection*`,
     `System.Runtime.InteropServices`, `System.Threading` (except `CancellationToken`),
     `Environment`, `AppDomain` and `Assembly` loading, as well as `unsafe`, `dynamic`, `extern`
     and P/Invoke. Compilation uses a minimal reference set.
  2. **Execution**: rules run in a separate **`Hcif.RuleHost` worker process** that talks over a
     stdio JSON-RPC channel. Rules receive only immutable fact snapshots and return proposed
     consequences, so they have no handles to repositories.
  3. **Resource limits**: a Windows Job Object (Linux: cgroups or `setrlimit`) caps CPU time,
     memory and process count. A per-call timeout kills the worker on expiry.
  4. **Audit**: every compilation and invocation result is recorded (FR-031, US7-2).

  **Residual risk**: .NET has no in-process sandbox, and OS-level network denial depends on
  deployment (firewall rule or container `--network none`). This is documented in the quickstart
  and checked by an integration test.
- **Rationale**: P28 and P51. `AssemblyLoadContext` is not a security boundary, so process
  isolation is required.
- **Alternatives considered**: in-process ALC (rejected: not a security boundary); WASM-compiled C#
  rules (immature tooling); scripting languages such as Lua via MoonSharp (P28 permits C#, and a
  second language adds review burden).

## R13. Intelligence synthesis and decision policy

- **Decision**:
  - **Intent classification** is rule-based from speech act (declarative, interrogative,
    imperative), modality and predicate frames. Requests are classified as
    Explicit/Implicit/Suggested/Unsolicited per P17 / FR-039.
  - **Candidate generation** uses pack-resident response/action templates keyed by intent.
  - The `DecisionEvaluationAutomaton` uses a **transparent multi-criteria score**. The P16 factors
    (relevance, intent alignment, evidence, contextual suitability, expected utility, risk, cost,
    authorization, preference, feasibility) each have a recorded value and weight in configuration.
    **Hard constraints** (authorization, policy) filter before scoring. Candidates within a tie
    margin are all retained (FR-038, US4-4).
  - **No-op, Abstain, Clarify and Answer** are first-class outcomes.
- **Rationale**: P15–P17. Each factor value is inspectable for FR-055.

## R14. Response generation and personality

- **Decision**: Use **template-based English NLG** in the language pack (`responses/*.yaml`). It
  realises the selected decision plus grounded propositions. Hypotheses are realised with hedging
  templates ("You said…", "That may mean…"). Unknown answers produce "I don't know" or a
  clarification template. A `PersonalityProfile` selects among template variants (tone) and adjusts
  suggestion ranking weights only. It **cannot** change epistemic status or authorization (P34,
  FR-044); this is enforced by type (the profile has no access to those fields).
- **Rationale**: Deterministic, grounded output (P33) with no LLM dependency.

## R15. Actions, authorization, confirmation and idempotency

- **Decision**: `ActionExecutionAutomaton` implements the P18 lifecycle exactly (see
  [data-model.md § Action lifecycle](./data-model.md#action-lifecycle-state-machine)).
  - The `ActionPolicyEngine` evaluates typed policies: consequential-class actions (external
    recipient, private data, irreversible, device) require `Confirmation`.
  - A **confirmation token** is bound to the SHA-256 of the canonical action (type + resolved
    destination + parameters) and to the principal. Any change invalidates it.
  - The **idempotency key** is a hash of (action canonical form, intelligence ID). Handlers record
    keys in an outbox table, so a retry after an unknown-status timeout cannot double-send.
  - Baseline handlers: `ConversationalReply`, `ClarificationRequest`, `Computation`,
    `KnowledgeRetrieval`, `KnowledgePersistence`, `MessageDraft`, and `MockMessagingSend`. The
    mock records sends to an inspectable outbox table, which is how SC-008 counts side effects.
  - The handler contract covers future e-mail, API, DB, OS, device, scheduled and composite types
    (FR-047). Device/machine handlers must additionally declare an independent `ISafetyInterlock`,
    which the registry refuses to register without (FR-048).
  - **Asynchronous and long-running actions (P18, P44, FR-049)**: handlers declare
    `IsAsynchronous`. An async `ExecuteAsync` returns `Accepted(externalOperationId)`, which moves
    the action to `AwaitingExternalCompletion`. Completion arrives through `PollStatusAsync` (a
    hosted poller, default 5 s) or through an authenticated completion notification
    (`POST /actions/{id}/external-completion`, re-verified with the handler). A deadline (default
    10 min) leads to `OutcomeUnknown`, then reconciliation.
  - **Suspension and resumption**: `ConfirmationRequested`, `AwaitingExternalCompletion` and
    `RetryScheduled` are suspendable. The action run is snapshotted via `ISnapshotStore` (R17), and
    `ActionResumptionService` resumes pending runs on startup and periodically.
  - **Retry policy**: `RetryPolicy(maxAttempts=3, base=2 s, ×2, max=60 s)`. It applies only to
    `Transient` errors from handlers with `SupportsIdempotency`, and every attempt reuses the same
    idempotency key, so a retry can never duplicate a side effect. Attempts are audited.
  - The mock messaging handler has a configurable async mode so these paths are testable.
- **Alternatives considered**: synchronous-only handlers (violate P18/P44); a durable workflow engine
  (excessive for the PoC; snapshots plus a hosted resumption service are enough).
- **Rationale**: P18–P20, P44, P52 and US5.

## R16. Conversation context, memory layers, privacy and erasure

- **Decision**: Memory has five categories (`Working`, `Conversational`, `SemanticKnowledge`,
  `Episodic`, `Procedural`), each with a configured `RetentionPolicy` (TTL, max items, erasable)
  and `PrivacyClassification` (`Public`, `Personal`, `Sensitive`, `Restricted`). **Erasure** removes
  the target records. It then walks the justification graph: derived knowledge whose **every**
  support is erased is retracted and purged, while knowledge with other supports is re-justified.
  Audit records keep only redacted references (FR-042, FR-058, US6-3). Corrections (`I meant X, not
  Y`) create a superseding assertion and mark the prior one `Retracted`, with a `supersededBy` link.
  History is preserved subject to retention (US6-2).
- **Rationale**: P35, P36 and P50.

## R16a. Feedback and user preferences (P37, FR-038, FR-043)

- **Decision**:
  - **Feedback records**: `Feedback` records (DM §4) are captured through `IFeedbackService`. Users
    submit `ParseAccuracy`, `InferenceCorrectness`, `DecisionRelevance` and `UserAcceptance`
    feedback via `POST /turns/{id}/feedback` or the REPL `:feedback`. The action automaton writes
    `ActionOutcome` and `ExecutionFailure` automatically, and the correction processor writes
    `Correction`. Feedback is stored in Episodic memory and never changes epistemic status.
  - **Ranking influence (optional, P37 MAY)**: aggregated `DecisionRelevance`/`UserAcceptance`
    feedback can adjust per-principal preference weights. Each adjustment is a
    `PreferenceAdjustment` that cites its feedback IDs and can be reverted.
  - **Preferences**: `UserPreference` records (Procedural memory) are seeded from configuration in
    this feature. With no applicable preference, the factor is 0.
  - **Consumption of adjustments**: `ConfiguredUserPreferenceProvider` returns the configured
    preferences with every active (non-reverted) `PreferenceAdjustment` applied. Each returned
    preference lists the adjustment IDs that changed it, so the decision factor table can trace a
    ranking change back to feedback. Reverting an adjustment removes its effect from the next turn
    onward.
- **Rationale**: Meets P37's recording obligation without introducing unbounded learning.

## R16b. Language per turn (P31) and English-first scope (P56)

- **Decision**:
  - **Per-turn detection**: language is detected for every turn and every sentence
    (`Information.detectedLanguage`). `ConversationContext.language` is updated each turn and is
    never fixed by the first turn.
  - **Non-English input**: only the English pack is installed. Non-English sentences produce
    `LANGUAGE_UNSUPPORTED` and are kept as Data, with partial Information covering any English
    sentences. A turn written **entirely** in an unsupported language is **not rejected**: it
    completes with outcome `Clarify` ("only English is understood for now") and updates the context
    language. Rejection is reserved for input with no letters or digits in any script, empty input
    or over-limit input (FR-005).
  - **Recorded deviation**: full multilingual comprehension (P29–P31) is deferred. This is recorded
    in the plan's Complexity Tracking.

## R17. Persistence of runs and knowledge

- **Decision**: A second SQLite database, `data/hcif.db`, holds conversations, Data,
  Information snapshots (JSON column, schema-versioned), the knowledge graph (`entity`,
  `proposition`, `knowledge`, `justification` and `edge` tables with temporal-version columns),
  automaton-run snapshots and traces, the action outbox and audit, and memory records. It is
  accessed through repository interfaces (`IKnowledgeRepository`, etc.) so it can be swapped later
  (P14, P39). Schema migrations are forward-only and versioned.
- **Rationale**: One embedded store keeps the PoC operable offline. Knowledge persistence stays
  independent of the automata engine (P14).

## R18. API, CLI and authentication for the PoC

- **Decision**:
  - **API**: ASP.NET Core minimal API, REST/JSON, with an OpenAPI 3.1 document (see
    [contracts/http-api.openapi.yaml](./contracts/http-api.openapi.yaml)).
  - **CLI**: `hcif` built with `System.CommandLine` (see [contracts/cli.md](./contracts/cli.md)).
    It covers dictionary administration, the chat REPL, trace viewing, evaluation and the
    benchmark.
  - **Principal**: a local API-key header (`X-Hcif-Principal-Key`) mapped to a configured
    principal. Unauthenticated requests are rejected.
  - **Roles**:
    - `User`: conversations and own memory/feedback.
    - `Admin`: dictionary, erasure-of-others and component registration.
    - `Integration`: may call **only** `POST /actions/{id}/external-completion` for asynchronous
      handlers. It cannot hold conversations or read traces.
  - **Principal directory**: principals and roles are exposed to libraries through
    `IPrincipalDirectory` (Abstractions), implemented in `Hcif.Hosting` from configuration.
  - **Bind address**: the API listens only on localhost by default.
  - **Configuration ownership**: each library owns its options type, bound by `Hcif.Hosting`
    through `services.Configure<T>(section)`. `HcifOptions` only aggregates sections; libraries
    never reference `Hcif.Hosting`, which keeps the P42 layer contracts intact:
    - `LexiconOptions`: traversal defaults and cache sizes.
    - `MemoryOptions`: retention defaults and preference seeds.
    - `ActionOptions`: retry policy, poll interval and completion deadline.
    - `IntelligenceOptions`: decision factor weights, tie margin and personality.
    - `InputOptions` (Cognition): input limits.
- **Rationale**: P43 (API-first, presentation-independent) and FR-053. The spec doesn't require
  multi-user authentication; a principal model is still required for authorization (P20).

## R19. Observability and the developer trace

- **Decision**: Use `System.Diagnostics.ActivitySource`/`Meter`, with OpenTelemetry exporters as
  optional configuration, for spans and metrics: per-stage timing for SC-010. A **domain
  `ProcessingTrace`** is persisted per turn: stage outputs, automaton transitions, dictionary
  release, token offsets, rule firings with bindings, evidence and action outcome. It is served at
  `GET /turns/{id}/trace` and `hcif trace show` (FR-055). Logging uses
  `Microsoft.Extensions.Logging` JSON console output, with protected parameters redacted (P52).

## R20. Testing and evaluation

- **Decision**:
  - **Unit tests**: xUnit v3.
  - **API integration tests**: `Microsoft.AspNetCore.Mvc.Testing`.
  - **Mini-lexicon fixture**: a deterministic fixture **extracted from the real OEWN 2025
    archive** by `hcif dictionary extract-fixture`. It holds the closure of the baseline lemmas
    (play, guitar, sitar, football, actor, hamlet, instrument, …) plus hypernym chains, so unit and
    acceptance tests run fast without the full import. The fixture is checked in with its
    provenance and the CC BY 4.0 attribution.
  - **Full-import integration tests**: run when `HCIF_OEWN_ARCHIVE` is set.
  - **Network-disabled tests (SC-003)**: the lexicon assembly has no `HttpClient` dependency, which
    an architecture test enforces. The lookup suite runs with a `NetworkGuard` that fails on any
    socket use (`SocketsHttpHandler` replaced, plus an `AppContext` switch). A CI job optionally
    runs it in a container with `--network none`.
  - **Architecture tests** (reflection-based dependency rules) enforce no grammar→action reference
    (P55) and no `Hcif.Cognition` dependency on `Hcif.Language.English` (P56).
  - **Evaluation**: `Hcif.Evaluation` runs the versioned annotated datasets in `data/eval/`
    (≥300 sentences, ≥100 morphology items, ≥100 WSD items) and emits a JSON report with
    per-category scores (FR-057, SC-004–SC-006).
  - **Benchmark**: `hcif bench` for SC-010.
- **Alternatives considered**: FluentAssertions (rejected: v8 commercial licence; plain xUnit
  asserts are enough).

## R21. Performance budget

- **Decision**: Target p95 ≤ 2 s per short utterance, offline, after warm-up (SC-010). The design
  levers are:
  - SQLite prepared statements and covering indexes on `lemma_lower`, `sense_id` and `synset_id`;
  - an in-memory LRU for lexical lookups and hypernym closures;
  - a precomputed hypernym-depth table for WordNet's ~107k synsets (cheap to build at import);
  - Earley parsing bounded by sentence length (≤ 40 tokens by default; longer input is segmented);
  - inference budgets.
  Per-stage timings are emitted. The import time is measured and reported but excluded from SC-010.

---

**All Technical Context items are resolved. No NEEDS CLARIFICATION remain.**
