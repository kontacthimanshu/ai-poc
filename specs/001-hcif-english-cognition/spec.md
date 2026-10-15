# Feature Specification: HCIF English-First Cognitive Conversation and Action Framework

**Feature Branch**: `001-hcif-english-cognition`

**Created**: 2026-10-15

**Status**: Draft

**Input**: User description: "HCIF English-First Cognitive Conversation and Action Framework, derived from
HCIF Constitution v1.1.1 (`.specify/memory/constitution.md`)"

**Constitutional Coverage**: P1–P70, with first-release emphasis on P1–P28, P32–P33, P35–P36,
P40–P56, P57–P70

## Feature Overview

Develop an English-first conversational and action-taking application in which a typed and
inspectable processing lifecycle transforms user statements from
**Data → Information → Knowledge → Intelligence → Action**, coordinated by hierarchical finite
automata (HFA).

The initial implementation MUST use **Open English WordNet 2025, JSON Edition**, downloaded from its
official source and locally imported and indexed. English grammatical processing is separate from
lexical-semantic lookup: WordNet provides lemmas, senses and relations, while the English language
module supplies syntax, morphology, named-entity processing and context interpretation.

The system shall understand ordinary English input, preserve the original utterance, produce
detailed grammatical and semantic objects, distinguish asserted claims from warranted inferences,
select a contextually appropriate conversational decision, and—only on authorized requests—perform
a controlled action through an extensible action handler. The system shall expose intermediate
artifacts for inspection and testing. Future language packs, engines and channels must be addable
without rewriting core automata.

**First functional milestone** (P54, P70):
`English sentence → Dictionary lookup → Grammar analysis → Information object → Knowledge inference
→ Intelligence object → Conversational response`

Subsequent increments add context persistence, external actions, probabilistic methods and further
language packs in the constitution's mandated sequence.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ingest and understand an English assertion (Priority: P1)

As an administrator, I want to download, verify and import Open English WordNet 2025 JSON, so
English understanding does not depend on a live dictionary website.

**Why this priority**: Dictionary acquisition and indexing precede all English parsing in the
constitution's mandatory development sequence (steps 3–5); every later stage depends on it.

**Independent Test**: Import the official archive into an empty store, turn off network access,
look up `play` and `guitar`, retrieve senses and semantic links, and re-run the import without
duplicates.

**Acceptance Scenarios**:

1. **Given** the configured official archive location, **When** initialization runs, **Then** it
   downloads or uses a verified local copy, validates extraction and JSON structure, indexes
   entries, and records source/version/license metadata.
2. **Given** a completed import, **When** the network is unavailable, **Then** lemma, sense, synset
   and semantic relation queries work locally.
3. **Given** an unchanged dataset, **When** import runs again, **Then** the resulting records and
   source identifiers are unchanged and no duplicates appear.
4. **Given** a failed import, **When** the process aborts, **Then** the previously validated
   dictionary remains usable.

---

### User Story 2 - Install and query the WordNet dictionary (Priority: P1)

As a user, I want to enter a sentence and inspect its structured interpretation so that I can
verify what the system actually understood.

**Why this priority**: It is the first end-to-end proof of Data → Information and is required
before knowledge or action.

**Independent Test**: On a locally initialized dictionary, submit `Seeta plays guitar.` and inspect
original data, tokenization, grammatical analysis, proposition and linked source offsets.

**Acceptance Scenarios**:

1. **Given** the application is initialized, **When** I enter `Seeta plays guitar.`, **Then** it
   preserves the exact original text and records the source, timestamp and conversation identity.
2. **Given** this input, **When** the English grammar stage completes, **Then** it identifies
   `Seeta` as subject/proper-name candidate, `plays` as third-person singular simple-present form of
   `play`, and `guitar` as an object describing a musical instrument; polarity is affirmative and
   sentence type declarative.
3. **Given** no contextual evidence of gender, **When** the system creates information, **Then** it
   leaves referent gender unknown even if an external name lexicon suggests a likely gender.
4. **Given** an unsupported word, **When** lexical lookup fails, **Then** it retains the token and
   attempts a recoverable analysis or returns an explicit partial-parse diagnostic instead of
   inventing a meaning.

---

### User Story 3 - Derive attributable knowledge without unsupported assumptions (Priority: P1)

As a user, I want knowledge derived from what I said and from dictionary relations while clearly
distinguishing what is stated, derived and uncertain.

**Why this priority**: Epistemic integrity (stated vs. derived vs. hypothesized) is a
non-negotiable constitutional obligation and part of the first functional milestone.

**Independent Test**: Ingest the baseline sentence, inspect proposition status, origin, rule
evidence and any optional hypothesis.

**Acceptance Scenarios**:

1. **Given** `Seeta plays guitar.`, **When** knowledge construction runs, **Then**
   `PlaysInstrument(Seeta, Guitar)` is stored as a speaker-attributed assertion, not an
   independently verified real-world fact.
2. **Given** WordNet's valid taxonomic relation for guitar, **When** a permitted rule fires,
   **Then** the system can derive an appropriately qualified `PlaysMusicalInstrument(Seeta)`
   proposition with traceable evidence and rule ID.
3. **Given** no additional evidence, **When** inferences are generated, **Then** the system does
   not assert Seeta is female, nice, calm, a professional musician or interested in being
   contacted.
4. **Given** an opposing statement in a later turn, **When** reconciliation executes, **Then** both
   sources and the disagreement are represented without silently overwriting history.

---

### User Story 4 - Respond intelligently in conversation (Priority: P1)

As a user, I want the application to reply to a statement or question based on relevant
information and knowledge, without taking unintended actions.

**Why this priority**: The conversational response completes the first functional milestone.

**Independent Test**: Submit `Seeta plays guitar.` and then `What instrument does Seeta play?`;
inspect candidate decisions and resulting answers.

**Acceptance Scenarios**:

1. **Given** an informational statement, **When** intelligence synthesis completes, **Then** it
   selects an acknowledgement, relevant question or no-op with a recorded rationale, without
   contacting Seeta.
2. **Given** the later question, **When** context resolution succeeds, **Then** the application
   answers `guitar` and identifies the preceding speaker assertion as the basis.
3. **Given** insufficient evidence, **When** an answer cannot be supported, **Then** it says it
   does not know or asks for clarification rather than inventing a fact.
4. **Given** several relevant candidate responses, **When** selection occurs, **Then** the recorded
   decision uses relevance, uncertainty, user intent and constraints; the algorithm need not
   artificially collapse to one candidate before evidence warrants it.

---

### User Story 5 - Request and authorize external actions (Priority: P2)

As a user, I want the application to offer or carry out useful actions when I request them, with
explicit confirmation and verifiable results for externally consequential operations.

**Why this priority**: Controlled action execution follows the first functional milestone in the
mandatory sequence (step 15, milestone M3).

**Independent Test**: Request a draft invitation, then attempt a simulated send; inspect
permissions, confirmation, outcome and audit record.

**Acceptance Scenarios**:

1. **Given** `Seeta plays guitar.`, **When** the statement is processed, **Then** no message is
   sent and no device is activated.
2. **Given** `Draft a message inviting Seeta to a jam session`, **When** planning runs, **Then** it
   returns a draft through a typed action without sending.
3. **Given** `Send that invitation to Seeta`, **When** recipient identity or permission is missing,
   **Then** execution pauses to resolve the recipient or obtain authorization.
4. **Given** explicit authorization and an available mock handler, **When** the send is approved,
   **Then** the handler executes once and the action result is audited.
5. **Given** a refusal, cancellation, timeout or handler error, **When** execution stops, **Then**
   the state machine records an appropriate terminal/recovery state without falsely reporting
   success.

---

### User Story 6 - Maintain and correct contextual knowledge (Priority: P2)

As a user, I want conversations to remember useful context while allowing corrections and
deletion.

**Why this priority**: Conversational memory and context retention follow controlled actions in
the mandatory sequence (step 16, milestone M3).

**Independent Test**: Make an assertion, ask a pronoun-based follow-up, correct the assertion, and
delete stored conversational knowledge.

**Acceptance Scenarios**:

1. **Given** a preceding statement about Seeta, **When** I ask `What does she play?`, **Then** the
   system resolves `she` only when conversation context supports the reference; otherwise it asks
   whom I mean.
2. **Given** `I meant sitar, not guitar`, **When** correction processing runs, **Then** it records a
   superseding assertion and preserves the correction trail subject to retention policy.
3. **Given** an authorized deletion request, **When** deletion completes, **Then** the targeted
   persistent memory is no longer returned by knowledge queries.

---

### User Story 7 - Extend cognitive components safely (Priority: P3)

As a developer, I want to add a parser, inference rule, action handler or later language pack
without changing the root automaton.

**Why this priority**: Extensibility is an architectural guarantee validated after core behavior
exists; it enables M4 and later features.

**Independent Test**: Register a test parser/rule/handler through a documented contract and run
contract tests with the unchanged coordinator.

**Acceptance Scenarios**:

1. **Given** a new versioned component, **When** it is registered, **Then** the system accepts it
   only if its contract and compatibility checks pass.
2. **Given** a dynamically compiled untrusted rule, **When** it attempts unauthorized file or
   network access, **Then** process isolation/policy prevents it and records a failure.
3. **Given** an inference-cycle rule set, **When** it is evaluated, **Then** cycle detection and
   budgets terminate processing with an inspectable diagnostic.

---

### Baseline Test Corpus and Expected Results

| Input | Information expectation | Knowledge / decision expectation |
|-------|-------------------------|----------------------------------|
| `Seeta plays guitar.` | Proper-name subject; plays→play; simple present; object guitar; affirmative | Speaker assertion; guitar classified as musical instrument; no unsupported personality/gender claim; ordinary conversational reply |
| `The guitar is played by Seeta.` | Passive voice; compatible semantic actor/action/object | Canonical primary proposition compatible with baseline; surface voice retained |
| `Seeta does not play guitar.` | Explicit negation | Opposing assertion; no affirmative fact derived from it |
| `Does Seeta play guitar?` | Interrogative rather than assertion | Answer only from existing evidence; a question is not stored as affirmative fact |
| `Seeta might play guitar.` | Modal uncertainty | Qualified possibility, not a categorical assertion |
| `The actor plays Hamlet.` | Context-sensitive sense of play | No instrument-playing conclusion |
| `He plays football.` | Context-sensitive sense of play | Sports activity, not musical performance |
| `Seeta plays a blorptar.` | Unknown noun preserved | Partial interpretation/ambiguity notice; no invented meaning |
| `Seeta plays guitar. What instrument does she play?` | Pronoun resolved only when context supports it | Answer guitar, attributed to prior speaker assertion |
| `Please draft an invitation to Seeta.` | Imperative/action intent | Draft only; no external send |
| `Send the invitation to Seeta.` | Outbound intent | Require recipient resolution, available handler and authorization/confirmation before send |

Additional corpus categories MUST cover irregular morphology, complex/multiple clauses, ellipsis,
quoted speech, conditionals, contradictory reports, malicious instructions, cyclic rules and action
failures.

### Edge Cases

- The same surface word has several WordNet senses or grammatical roles, with no reliable
  contextual resolution.
- Personal names are missing from WordNet; entity identity is unresolved or conflicts with context.
- Quoted, hypothetical, conditional or negated language, and language flagged as non-literal by an
  explicit marker (e.g. quotation marks, "supposedly", "allegedly", "as if", "/s"), must not become
  unconditional world knowledge. Detecting irony that has no explicit marker is out of scope for
  this feature.
- Multiple speakers disagree about a claim, or a later turn explicitly corrects it.
- A dictionary release contains broken references, a changed undocumented JSON layout, malicious
  ZIP paths or malformed data.
- Offline startup occurs before dictionary initialization, after successful initialization, or
  during a failed update.
- Semantic graph cycles, overexpansion and repeatedly firing rules exhaust computational budgets.
- A conversation mixes English with unknown foreign-language words before additional language packs
  exist.
- A user requests messaging with an ambiguous contact, an unauthorized recipient or a missing
  integration.
- An external action times out after unknown delivery status; retries must avoid duplicate sends
  where possible.
- A conversation is interrupted mid-state, cancelled, resumed, or deleted under the memory policy.
- Prompt injection embedded in user text, dictionary examples or external content requests direct
  tool use; such text must remain data.

## Requirements *(mandatory)*

### Functional Requirements

#### A. Processing lifecycle and finite automata

- **FR-001**: The system MUST expose a `CognitiveLifecycleAutomaton` with `Idle`,
  `DataAcquisition`, `InformationConstruction`, `KnowledgeConstruction`, `IntelligenceSynthesis`,
  `ActionSelection`, `ActionExecution`, and `FeedbackAssimilation` states, plus
  failure/clarification/cancellation/recovery transitions. [P1, P21]
- **FR-002**: Every automaton MUST declare version, input/output contracts, finite states,
  initial/terminal states, transitions, guards, entry/exit behavior, errors, limits and
  cancellation. [P2, P25]
- **FR-003**: Parent automata MUST invoke versioned subordinate automata with well-defined return
  outcomes; the runtime MUST enforce execution-depth, transition-count and time budgets.
  [P2, P25, P40]
- **FR-004**: The system MUST separate finite control state from `Data`, `Information`,
  `Knowledge`, `Intelligence` and `Action` objects and link each derived object to its origins.
  [P3]
- **FR-005**: Processing MUST support partial results, clarification, rejection, deferral and no-op
  without forcing every input to produce a new fact or external action. [P1, P15]
  - **Rejection** applies to input that is empty after normalization, exceeds the configured input
    limits (default 4,000 characters), or contains **no letters or digits in any script** (only
    punctuation, symbols, emoji or control characters). The Data is retained and the turn ends with
    outcome `Rejected` plus a diagnostic. Input limits are enforced by the lifecycle, not by API
    schema validation, so over-limit input is still preserved (P4). Only a transport-level body cap
    (default 64 KB) returns HTTP 413 without retention.
  - **Unsupported language**: a turn written entirely in a language without an installed language
    pack is **not** rejected. It completes with outcome `Clarify` (a reply explaining that only
    English is currently understood) and a `LANGUAGE_UNSUPPORTED` diagnostic, and it updates the
    conversation's current language (P31). In mixed-language turns the English sentences are
    processed normally.
  - **Deferral** applies when a required stage reports a transient unavailability, e.g. the active
    dictionary release is switching or a store is locked. The Data and the run snapshot are retained,
    the turn ends with outcome `Deferred`, and the client can resume it later.
- **FR-006**: Automata MUST emit structured transition traces and bounded failure diagnostics;
  direct grammar-to-action execution is prohibited. [P45, P55]

#### B. Original data and linguistic comprehension

- **FR-007**: The application MUST preserve original input and available
  source/channel/time/language/script/speaker/conversation/privacy metadata. [P4]
- **FR-008**: The English pipeline MUST perform language identification, normalization,
  segmentation, tokenization, lexical lookup, morphology, POS tagging, syntax/dependency analysis,
  roles, NER, coreference, tense/aspect/modality, negation and semantic composition, using declared
  automata or services. [P5, P22, P61]
- **FR-009**: The English language pack MUST define declarative grammar, agreement, inflection,
  lexical categories, semantics, parsing ambiguity policies and a versioned test corpus,
  independent of the root automaton. [P8, P9, P29, P56]
- **FR-010**: Information MUST support arbitrarily nested and multiple clauses and structured
  representation of questions, commands, assertions, passive voice, negation,
  hypothetical/conditional and counterfactual sentences. [P6, P7]
- **FR-011**: Information MUST carry subject/predicate, verb/object roles, modifiers, grammatical
  person/number/gender where applicable, temporal/spatial/causal/conditional relationships,
  modality, certainty and source-token offsets. [P6]
- **FR-012**: The parser MUST distinguish grammatical gender from referent gender; names alone do
  not establish referent gender. [P6, P12, P65]
- **FR-013**: Equivalent active/passive English statements MUST produce semantically compatible
  canonical propositions while preserving relevant tense, scope, negation and emphasis. [P10]
- **FR-014**: Unknown lexemes and unresolved senses MUST produce retained alternatives or explicit
  diagnostics, not fabricated dictionary entries or forced unique parses. [P5, P61, P62]
- **FR-015**: Morphological analysis MUST resolve regular and irregular forms through dedicated
  rules/resources and MUST NOT depend on WordNet alone. [P63]

#### C. Mandatory Open English WordNet 2025 JSON

- **FR-016**: Initialization MUST acquire Open English WordNet 2025 JSON from the
  constitution-specified official dataset URL
  (`https://en-word.net/static/english-wordnet-2025-json.zip`), or from a trusted pre-fetched copy
  of that release. [P57]
- **FR-017**: Acquisition MUST record source URI, release identifier, download date, SHA-256
  digest, archive integrity result, attribution and applicable license notices. [P57, P69]
- **FR-018**: Import MUST inspect and validate the actual JSON schema and preserve source IDs for
  lexical entries, senses, synsets and relations; assumptions about undocumented property names are
  forbidden. [P58, P60]
- **FR-019**: Import MUST use safe extraction, staged publication, rollback on failure,
  deduplication and idempotent re-import; the previous valid dictionary MUST remain available on an
  import error. [P60]
- **FR-020**: A local indexed repository MUST support
  word/lemma/POS/sense/synset/gloss/relation/example queries where the source supplies
  corresponding fields; online services MUST NOT be needed for ordinary lookup. [P57–P59]
- **FR-021**: The system MUST expose the pluggable dictionary interfaces defined by P59, including
  `ILexicalDictionary`, `IWordSenseResolver`, `ISynsetRepository` and
  `ISemanticRelationshipRepository`. [P59]
- **FR-022**: The `LexicalIntelligenceAutomaton` MUST coordinate dictionary lookup, lemma
  resolution, POS resolution, contextual sense selection, semantic traversal and lexical
  validation. [P61–P68]
- **FR-023**: Sense-disambiguation results MUST retain multiple candidates and evidence when
  ambiguous; dictionary membership MUST NOT be mistaken for proof of correct meaning. [P62]
- **FR-024**: Lexical results MUST be cacheable with source release/version and
  interpretation-rule version to support invalidation. [P68]
- **FR-025**: WordNet taxonomic/semantic traversal MUST have configurable relationship allowlists,
  depth limits, visited-node tracking, duplicate suppression, relevance filters guided by the current
  processing objective, minimum evidence requirements and execution budgets. [P66]
- **FR-026**: Import/release procedures MUST retain licensing metadata and produce
  distribution-ready attribution notices; proprietary dictionary-site scraping is out of scope.
  [P69]

#### D. Knowledge, inference and evidence

- **FR-027**: The knowledge stage MUST extract speaker-attributed assertions as distinct from
  independently verified facts. [P4, P11, P12]
- **FR-028**: Knowledge objects MUST carry subject/predicate/object, epistemic class, sources,
  supporting propositions, rule IDs, timestamps, validity and contradiction state. [P12, P13]
- **FR-029**: The knowledge repository MUST support entities, properties, edges, assertions,
  provenance, temporal versions, contradictions, revisions and deletes. [P14, P36]
- **FR-030**: Rule evaluation MUST support declarative patterns, conjunction/disjunction,
  relation/context/temporal predicates,
  assertion/hypothesis/action-recommendation/clarification consequences and rule versioning.
  [P26, P27]
- **FR-031**: The framework MUST permit versioned dynamically compiled rules only through
  restricted execution interfaces; untrusted code MUST run in isolated constrained processes or
  services. [P28, P51]
- **FR-032**: WordNet semantic relationships MUST be available as evidence for inference, with
  explicit distinction between taxonomy, entailment, heuristic association and hypothesis.
  [P11, P64]
- **FR-033**: A derived proposition MUST record a reproducible explanation graph from source
  utterance through information, dictionary source relation and inference rule. [P13, P47, P64]
- **FR-034**: Unsupported personality, gender, occupation and contact-preference inferences MUST
  NOT be promoted to facts; numerical probabilities MUST NOT be invented without calibration or
  evidence. [P12, P13, P65]
- **FR-035**: Contradictions MUST be detected and recorded with competing provenance, without
  silently replacing older claims; corrections/retractions and expiry MUST be supported.
  [P14, P36]
- **FR-036**: The engine MUST bound rule firings, inference depth, graph expansion and retries,
  reporting budget exhaustion without executing unrelated actions. [P40, P66]

#### E. Intelligence, conversation and memory

- **FR-037**: Intelligence objects MUST store interpreted intent, context, candidate
  responses/actions, goals, prerequisites, risks, uncertainties, authorization constraints,
  selected outcome and rationale. [P15]
- **FR-038**: Decisions MUST evaluate relevance, intent, evidence, uncertainty, risk, cost,
  feasibility and user preferences; valid outcomes include answer, question, recommendation,
  abstention and no-op. User preferences come from per-principal preference records (Procedural
  memory, configured for this feature). When a principal has no applicable preference, the
  preference factor is neutral (0). [P15, P16]
- **FR-039**: Action classification MUST distinguish explicitly requested, implied, merely
  suggested and unsolicited possibilities. [P17]
- **FR-040**: Responses MUST ground factual claims in retained information/knowledge and qualify
  hypotheses; unsupported answers MUST trigger uncertainty or clarification behavior. [P33]
- **FR-041**: The conversation context MUST hold previous topics, references, unresolved pronouns,
  assertions, intentions and pending actions, separately from persistent semantic knowledge. [P32]
- **FR-042**: Memory MUST distinguish working, conversational, semantic, episodic and procedural
  categories, with configured retention, retrieval and deletion behavior. [P35, P50]
- **FR-043**: Feedback MUST support user corrections and outcome tracking, preserve a correction
  history where permitted and allow rollback/re-evaluation of affected conclusions. [P36, P37]
  - Users MUST be able to submit permitted feedback on a turn about parse accuracy, inference
    correctness, decision relevance and acceptance of the response.
  - Action outcomes and execution failures MUST be recorded automatically.
  - Feedback is subject to retention policy. It MAY influence later ranking, and any such change
    MUST be traceable and reversible.
- **FR-044**: Configurable personality MAY modify tone or suggestion ranking, but MUST NOT change
  established evidence classifications or authorization controls. [P34]

#### F. Actions, boundaries and extensibility

- **FR-045**: All actions MUST be typed and routed through an `ActionExecutionAutomaton` that
  checks prerequisites, policy, identity, permission, destination, confirmation, execution and
  audit. [P18–P20]
- **FR-046**: A statement about another person MUST NOT trigger messages, OS commands, device
  activation or other external actions in the absence of authorization. [P17, P20]
- **FR-047**: The handler registry MUST support conversational replies, questions, calculations,
  knowledge read/write and mock external messaging at baseline; contracts MUST permit e-mail, API,
  database, OS, devices, scheduled and composite actions later. [P19]
- **FR-048**: Consequential actions MUST require appropriate consent, resolved destination,
  parameter validation and independent safety policy; device/machine controls require independent
  fail-safe mechanisms. [P20]
- **FR-049**: Long-running handlers MUST support idempotency keys where applicable,
  suspension/resumption, cancellation, retry policy, outcome reporting and recovery/compensation
  semantics. [P18, P44]
  - The action lifecycle MUST support both synchronous handlers and asynchronous handlers that
    complete later (by status polling or a completion notification).
  - A pending or awaiting action MUST survive process restart and resume from its persisted state.
  - Retries MUST be bounded (maximum attempts, backoff) and are permitted only for idempotent
    handlers.
- **FR-050**: External action audit MUST record triggering intent, evidence links, selected action,
  authorization result, protected parameters, timestamps, external ID if any and final status.
  [P52]
- **FR-051**: The system MUST treat incoming text, remote content and generated rules as untrusted
  data, not as authorization or executable instructions. [P51]
- **FR-052**: Domain rules, grammar modules, model providers and action handlers MUST be
  replaceable via versioned contracts without modifying the root automata. [P39, P42, P55]
- **FR-053**: The initial API MUST be independent of console/web/mobile/voice presentation,
  although only English text interaction is necessary for first acceptance. [P43, P56]
- **FR-054**: The design MUST preserve language-neutral canonical types and pluggable language
  packs; implementation and acceptance of Hindi or other languages is deferred.
  [P8, P29–P31, P56]

#### G. Quality, validation and administration

- **FR-055**: The application MUST expose, in an administrator/developer trace, the stages,
  automata transitions, dictionary releases, token offsets, semantic objects, rule firings,
  evidence and action outcome for each processed utterance. [P3, P45, P47, P70]
- **FR-056**: Automated unit, integration, grammar-corpus, inference and handler tests MUST cover
  success, ambiguity, failure, cancellation, limits, contradiction, provenance and authorization.
  [P45–P48, P70]
- **FR-057**: Evaluation MUST use versioned annotated datasets with published measurement methods;
  unmeasured correctness claims are prohibited. [P46, P49]
- **FR-058**: Data and derived knowledge MUST enforce privacy classification, user-authorized
  retention/erasure and restrictions on forwarding sensitive information to external services.
  [P50]
- **FR-059**: Each implementation increment MUST pass the constitution's defined gates for
  sequence, automata, epistemic integrity, security, bounded reasoning and English-first
  modularity. [P53–P56]
- **FR-060**: The baseline acceptance test MUST verify `Seeta plays guitar.` at all stages,
  including `plays → play`, guitar instrument classification, unknown referent gender and the
  absence of unauthorized sending. [P70]

### Required State and Evidence Behavior

**Root-state flow**:
`Idle → DataAcquisition → InformationConstruction → KnowledgeConstruction → IntelligenceSynthesis
→ ActionSelection → ActionExecution → FeedbackAssimilation → Idle`.
Intermediate states MAY route to `Clarification`, `Deferred`, `Rejected`, `Failed`, `Cancelled`
and recovery paths as warranted (rejection and deferral triggers per FR-005). No external handler can be reached directly from a parser or dictionary
operation.

**Lexical subordinate automata**: `DictionaryLookupAutomaton`, `LemmaResolutionAutomaton`,
`PartOfSpeechResolutionAutomaton`, `WordSenseDisambiguationAutomaton`,
`SemanticRelationshipTraversalAutomaton` and `LexicalValidationAutomaton` are orchestrated by
`LexicalIntelligenceAutomaton` and invoked in the Information and Knowledge stages.

**Evidence ladder**:

| Level | Meaning |
|-------|---------|
| Data | Original speaker utterance; neither proven nor disproven. |
| Information | Syntactic/semantic interpretation with alternatives when ambiguous. |
| Knowledge | Speaker-attributed assertion, dictionary-supported relation, rule-derived fact or labeled hypothesis; all with provenance. |
| Intelligence | Goal-sensitive selection among conversational responses and potential actions. |
| Action | A typed, policy-checked and authorized operation or explicit no-op. |

### Key Entities *(include if feature involves data)*

| Entity | Meaning and essential fields | Relationships |
|--------|------------------------------|---------------|
| `Data` | Immutable original utterance, source, timestamp, channel, conversation and privacy metadata | Source of one or more Information objects |
| `Token` | Original span, normalized form, offsets, candidate morphology/POS | Member of Information parse |
| `LexicalEntry` | Dictionary source ID, lemma, part of speech, source release | Links to senses |
| `WordSense` | Sense ID, lemma link, gloss, usage, source | Member of synset or lexical relations |
| `Synset` | Source ID, conceptual grouping, definition, available examples | Semantic relationships to other synsets |
| `SemanticRelation` | Typed source/target synsets, provenance and relationship kind | Evidence for derivations |
| `Information` | Parse tree or graph, clauses, dependencies, roles, modality, tense, ambiguity and source offsets | Derived from Data; source of Knowledge |
| `Proposition` | Typed subject/predicate/object, polarity, tense/modality and identity resolution | Carried by Knowledge |
| `Knowledge` | Proposition, epistemic status, evidential origin, inference rule, validity, contradiction | Supports Intelligence and other Knowledge |
| `InferenceRule` | Version, antecedents, conditions, consequences, priority and provenance behavior | Produces derived Knowledge or candidates |
| `Intelligence` | Intent, candidate responses/actions, constraints, justification, selected outcome | Consumes Knowledge; suggests Action |
| `Action` | Type, parameters, authorizing principal, destination, state, idempotency key | Evaluated/executed by handler |
| `AutomatonDefinition` | Versioned states, transitions, guard/action contracts and limits | Defines AutomatonRun |
| `AutomatonRun` | Current/terminal state, events, trace, budget and parent/child links | Produces stage outputs |
| `ConversationContext` | Topic, references, assertions, pending intents, ambiguities | Scope for turns and memory |
| `MemoryRecord` | Category, content reference, retention, privacy and revision | Stores knowledge/episodes/context |
| `DictionaryRelease` | Source URI, digest, schema fingerprint, license, import result | Owns lexical entities |
| `ActionAudit` | Authorization, execution trail, result/error and evidence references | Belongs to Action |
| `Feedback` | Turn reference, feedback category (parse accuracy, inference correctness, decision relevance, acceptance, action outcome, execution failure, correction), rating/comment, principal | Belongs to a turn; may inform later ranking |
| `UserPreference` | Principal, preference key, value, source | Procedural memory; input to the decision preference factor |

Entities MUST have stable identifiers and schema versions. Grammatical and semantic data MUST allow
nested and multiple-clause expressions rather than a flat fixed subject/verb/object record.

## Success Criteria *(mandatory)*

Criteria below are proposed release gates, not claims of achieved accuracy. Each metric MUST be
computed on a frozen, versioned evaluation set published with the implementation. Before
implementation, the team may amend numerical thresholds through normal specification review
without weakening constitutional requirements.

### Measurable Outcomes

- **SC-001**: The baseline sentence and the 10 additional corpus-table scenarios produce expected
  typed outputs and documented states in automated acceptance tests, with 100% of mandatory
  assertions passing.
- **SC-002**: The official WordNet 2025 JSON import completes with no unresolved internal
  references except documented source-data exceptions; a second import produces no extra records;
  a rollback test leaves the previous version queryable.
- **SC-003**: After initialization, 100% of locally tested dictionary lookups execute with network
  access disabled.
- **SC-004**: On a published evaluation set of at least 300 annotated English sentences spanning
  declaratives, questions, negation and compound/complex clauses, at least 90% meet the team's
  documented exact-match or field-level core semantic extraction criterion; per-category scores are
  reported separately. The scoring criterion (fields counted, exact vs. partial match) MUST be frozen
  in the dataset manifest before the first measured run, and any later change creates a new dataset
  version.
- **SC-005**: On a published morphological set of at least 100 regular/irregular English
  inflections, at least 95% resolve to the expected lemma and applicable feature bundle.
- **SC-006**: On an annotated set of at least 100 lexical-ambiguity examples, at least 85% select
  the expected sense or appropriately retain ambiguity under a documented abstention criterion;
  both selection accuracy and abstention frequency are reported. The abstention criterion MUST be
  frozen in the dataset manifest before the first measured run.
- **SC-007**: All automatically derived knowledge in acceptance scenarios retains source and rule
  lineage; 100% of stereotype/unsupported-inference negative tests refrain from marking excluded
  claims as established facts.
- **SC-008**: All authorization-negative integration scenarios perform zero external side-effect
  operations.
- **SC-009**: All bounded-reasoning/cycle tests terminate within configured execution limits and
  return a diagnostic, not a hung process.
- **SC-010**: A local benchmark of 100 short English utterances on a documented reference machine
  reports p50/p95 response latency and per-stage timing; the initial target is p95 ≤ 2 seconds for
  offline, non-LLM, non-network baseline processing after warm-up, excluding import time.
- **SC-011**: All automata unit tests cover initialization, guards, valid/invalid transitions,
  child invocation, failure, recovery, cancellation and configured limits; automated suites pass
  prior to release.
- **SC-012**: All official dataset attributions/license files are preserved and distributable;
  unknown-license datasets are not imported into releases.
- **SC-013**: Registering a replacement parser or mock action handler through an approved contract
  requires zero changes to root automaton state definitions.

## Assumptions

- The provided Constitution v1.1.1 is authoritative; all 70 principle IDs remain unchanged.
- English-only text interaction is the initial operational scope, while other channels and future
  language packs retain architectural extension points.
- Official Open English WordNet 2025 JSON is available for acquisition or installation from a
  verified local package; no HTML dictionary scraping is needed.
- WordNet is a semantic lexicon, not a comprehensive parser, morphological analyzer, named-entity
  directory or world-fact verification source; these functions require additional implementations
  and curated tests.
- The initial implementation may use modern C#/.NET as suggested by P41, but this specification
  does not mandate a particular storage engine, UI toolkit, grammar-parser library or graph
  database.
- External integrations may first be represented by mocks. Real sending, OS/device actions and
  scheduled operations require later handlers and corresponding authorization/safety acceptance
  testing.
- Complex probabilistic reasoning and multilingual language packs are later increments under P54,
  not preconditions for the first English functional milestone.
- A conversational reply is an `Action` in the typed lifecycle but is distinct from an external
  side effect. No-op and clarification are valid results.
- No external message service, device-control endpoint or contact directory is assumed to be
  installed or authorized. Any production connector must be selected, tested and authorized under
  a separate integration feature.

### Out of Scope for the First Functional Milestone

- Production multilingual parsing, code-switching comprehension and Hindi grammar support.
- Live speech recognition/synthesis and a graphical animated character/avatar.
- Unsupervised verification of arbitrary real-world assertions, unrestricted web crawling or
  proprietary dictionary scraping.
- Autonomous messaging, financial transfers, machinery control and other unrestricted external
  actions.
- A requirement to train an LLM or use one as the principal comprehension mechanism.
- Full human-level common-sense reasoning or automatic endorsement of personality/intent hypotheses
  inferred from an activity.

These are phase boundaries, not removal of longer-term constitutional obligations (notably P19,
P29–P31 and P38).

### Deferred to Planning (`/speckit-plan`)

This specification defines what the application must demonstrate. The implementation plan MUST
select and justify the specific parser technology, storage/indexing approach, automata library or
custom runtime, dictionary adapter, isolated rule-execution mechanism, API shape, deployment model
and observability stack. It MUST inspect the actual downloaded WordNet JSON before defining mapping
code or database schemas. No invented WordNet field names are normative here.

## Constitutional Traceability and Delivery Gates

| Requirement group | Constitution principles |
|-------------------|-------------------------|
| Lifecycle, automata and traceability | P1–P3, P21–P25, P40, P45 |
| Data, English grammar and information | P4–P10, P22, P29–P31, P46, P56, P61–P63 |
| Knowledge and inference | P11–P14, P23, P26–P28, P36–P37, P47, P64–P66 |
| Intelligence and response | P15–P17, P24, P32–P34 |
| Action authorization and audit | P18–P20, P44, P48, P50–P52 |
| Extensibility and modular implementation | P25, P38–P43, P55, P67–P68 |
| Open English WordNet acquisition/use | P57–P60, P61–P70 |
| Spec-driven development, evaluation and sequence | P49, P53–P54, P70 |

**Plan-stage Constitution Check (required)**: Verify feature spec completeness; no unvalidated
prerequisite in the 18-step mandatory sequence; automata contracts and tests; language-neutral root
objects; no grammar-to-action bypass; provenance and epistemic integrity; isolated dynamic rules;
consent/audit protections; processing limits; licensing; and English-first scope. Record any
deviation explicitly in the implementation plan's Complexity Tracking section.

### Incremental Milestones (aligned to P54)

| Milestone | Scope | User stories |
|-----------|-------|--------------|
| **M1 — Core and lexical foundation** | HFAs, acquisition, verified WordNet importer, indexing, morphology, grammar and canonical information | US1, US2 |
| **M2 — Knowledge and conversation** | Sense disambiguation, semantic traversal, assertions, deterministic inference, intelligence synthesis and English replies. **First functional milestone acceptance occurs here.** | US3, US4 |
| **M3 — Controlled actions and conversational memory** | Confirmation-gated handlers, audit, corrections, persistence and recovery | US5, US6 |
| **M4 — Advanced cognition and languages** | Calibrated/probabilistic components, additional language packs and UI/channel enhancements as separate features | US7 (contract groundwork); later features |
