# Data Model: HCIF English-First Cognitive Conversation and Action Framework

**Feature**: `001-hcif-english-cognition` | **Date**: 2026-10-15 | **Spec**: [spec.md](./spec.md) |
**Research**: [research.md](./research.md)

## Conventions

- **Identifiers**: every entity has a stable, opaque ID (`<prefix>_<ULID>`, e.g. `dat_01J…`), with
  two exceptions. Lexical entities keep their **source IDs** (sense keys, synset IDs) as natural
  keys (P58, P60). A small number of runtime-only value types have no ID.
- **Schema versions**: every persisted aggregate carries `schemaVersion` (semver). Snapshots that
  can't be read under the current version are migrated or rejected explicitly, never coerced (P44).
- **Immutability**: `Data`, `Token`, `Information`, `Proposition`, lexical entities and
  `ActionAudit` entries are immutable after creation. Changes create new versions or superseding
  records (P41).
- **Lineage**: each cognitive object references its origin(s) (P3). The lineage chain is
  `Data ← Information ← Knowledge ← Intelligence ← Action`.
- **Language neutrality**: core types (`Hcif.Abstractions`) contain no English-specific enums.
  Language-specific feature values are carried as `FeatureBundle` (string-keyed, pack-declared)
  (P56).
- `?` marks optional fields. `[]` marks collections.

---

## 1. Lexical subsystem (source: Open English WordNet 2025 JSON; see research R3)

### DictionaryRelease

| Field | Type | Notes / validation |
|---|---|---|
| `releaseId` | string | e.g. `oewn-2025`; unique together with `importId` |
| `importId` | ULID | one per successful import |
| `sourceUri` | URI | MUST equal the configured official URL or `file://` pre-fetched path (FR-016) |
| `releaseTag` | string | `2025-edition` |
| `downloadedAt` | timestamp | |
| `archiveSha256` | hex(64) | MUST equal pinned `7d749f6e…cfa4b51` (FR-017) |
| `integrityResult` | enum `Verified \| Mismatch \| Unverifiable` | only `Verified` may be published |
| `schemaFingerprint` | hex(64) | hash of the observed key census (file groups, posKeys, relation keys) |
| `licenseFiles[]` | {name, sha256, storedPath} | `LICENSE.md`, `WNDB_License.txt` (P69) |
| `attribution` | text | generated notice: Princeton WordNet + OEWN team, CC BY 4.0 |
| `counts` | {lemmas, entries, senses, synsets, relations, derivedInverseRelations} | compared against the source census |
| `importReport` | ImportValidationReport | P60 step 10 |
| `status` | enum (see state machine) | |
| `contentHash` | hex(64) | ordered hash of all imported rows; identical on re-import (SC-002) |

**Import state machine** (`DictionaryImportAutomaton`):

```text
Requested → Acquiring → Verifying → Extracting → SchemaValidating → Parsing → ResolvingReferences
→ Loading(staging) → Indexing → Validating → Publishing → Published
any state ──error──▶ RollingBack → Failed        (active release unchanged; staging deleted)
Verifying ──same digest already Published──▶ AlreadyImported (no-op)
```

`Extracting` rejects absolute paths, `..` segments, symlinks, and entries outside the
target directory (zip-slip). It also enforces caps on the total uncompressed size and file count.

### LexicalEntry

| Field | Type | Notes |
|---|---|---|
| `entryKey` | string | `<lemma>#<posKey>` (natural key) |
| `lemma` | string | source casing preserved; spaces preserved |
| `lemmaLower` | string | indexed lookup key |
| `pos` | enum `Noun \| Verb \| Adjective \| AdjectiveSatellite \| Adverb` | from posKey `n v a s r` |
| `homographIndex` | int? | from `n-1`, `n-2`, … |
| `forms[]` | string | source `form` (irregular/variant forms only) |
| `pronunciations[]` | {value, variety?} | |
| `releaseId` | string | owning DictionaryRelease |

Relationships: 1 → N `WordSense`.

### WordSense

| Field | Type | Notes |
|---|---|---|
| `senseKey` | string | source `id`, e.g. `play%2:36:01::` (natural key) |
| `entryKey` | string | FK → LexicalEntry |
| `synsetId` | string | FK → Synset (MUST resolve) |
| `subcatFrames[]` | string | FK → VerbFrame IDs |
| `exampleSentences[]` | string | source `sent` |
| `adjPosition` | string? | |
| `ordinal` | int | position within the entry (weak WSD prior) |

### Synset

| Field | Type | Notes |
|---|---|---|
| `synsetId` | string | e.g. `03806455-n` (natural key) |
| `pos` | enum | from `partOfSpeech` |
| `lexFile` | string | source file, e.g. `noun.artifact` (semantic category, P59) |
| `definitions[]` | string | `definition` |
| `examples[]` | string | `example` |
| `members[]` | string | lemmas |
| `ili` | string? | Interlingual Index (future cross-language link, P30) |
| `externalRefs[]` | {kind:`wikidata`, value} | not internal references |
| `sourceNote` | string? | `source` |
| `hypernymDepth` | int | precomputed at import |

### SemanticRelation

| Field | Type | Notes |
|---|---|---|
| `relationId` | ULID | |
| `level` | enum `Synset \| Sense` | |
| `kind` | enum | synset-level: `Hypernym, Hyponym*, Similar, DomainTopic, HasDomainTopic*, DomainRegion, HasDomainRegion*, MeroPart, HoloPart*, MeroSubstance, HoloSubstance*, MeroMember, HoloMember*, Exemplifies, IsExemplifiedBy*, Also, Attribute, Entails, Causes`; sense-level: `Derivation, Antonym, Pertainym, Also, Similar, Participle, Exemplifies, Agent, Event, Result, Undergoer, Instrument, Uses, State, Property, Location, Material, Vehicle, BodyPart, ByMeansOf, Destination` |
| `sourceId` / `targetId` | string | synset IDs or sense keys; MUST resolve |
| `derivedInverse` | bool | `true` for `*` kinds, materialised by the importer |
| `inverseOf` | ULID? | provenance link to the source edge |
| `relationEvidenceClass` | enum `RelationEvidenceClass`: `Taxonomic \| Entailment \| Meronymic \| Lexical \| Morphosemantic \| Associative` | classifies the *relation*. Inference maps it to the distinct `EvidenceKind` (see Knowledge) when it uses the edge (FR-032) |

**Validation**: unknown relation keys in the source cause the import to fail (R3). Relation
semantics are preserved verbatim (P58).

### VerbFrame
`frameId` (e.g. `via`), `template` (e.g. `Somebody ----s`).

### LexicalAnalysis *(cacheable result, P68)*

| Field | Type |
|---|---|
| `originalToken`, `normalizedToken` | string |
| `resolvedLemma` | string? |
| `candidatePos[]` | {pos, source: `Lexicon \| ClosedClass \| Morphology \| Shape`} |
| `candidateSenses[]` | {senseKey, score, featureContributions{}} |
| `selectedSense` | string? |
| `ambiguityStatus` | enum `Resolved \| Ambiguous \| Unknown \| NotApplicable` |
| `synsetRefs[]`, `grammaticalContext`, `evidence[]` | |
| `dictionaryReleaseId`, `interpretationRuleVersion` | cache-invalidation keys (FR-024) |

---

## 2. Cognitive objects

### Data (immutable)

| Field | Type | Notes |
|---|---|---|
| `dataId` | ID | |
| `originalText` | string | byte-exact; never modified (P4) |
| `encoding` | string | |
| `sourceIdentity`, `channel` | string | e.g. `api`, `cli` |
| `receivedAt` | timestamp | |
| `declaredLanguageId`, `declaredScriptId` | string? | **source-declared only** (e.g. a channel header), set at creation. *Detected* language and script live on `Information`/`SentenceAnalysis`, so `Data` stays immutable |
| `conversationId`, `turnIndex` | ID, int | |
| `speakerPrincipalId` | ID | |
| `audience` | string? | |
| `modality` | enum `Text \| SpeechTranscript \| Api \| Message \| Document \| Event \| MachineOutput` | |
| `contextMetadata` | map | |
| `sourceReliability` | enum `Unknown \| UserStated \| Trusted \| Untrusted` | |
| `privacy` | PrivacyClassification | |
| `trustLevel` | enum `Untrusted` (default for all text, P51) | |

### Token
`tokenId`, `dataId`, `sentenceIndex`, `startOffset`, `endOffset` (UTF-16 offsets into
`originalText`), `surface`, `normalized`, `shape` (`Capitalized`, `Upper`, `Lower`, `Numeric`,
`Punct`), `lexicalAnalysisRef`, `morphAnalyses[]` {lemma, pos, features: FeatureBundle}.

### Information

| Field | Type | Notes |
|---|---|---|
| `informationId` | ID | |
| `dataId` | ID | origin (P3) |
| `languagePack` | {id: `en`, version} | |
| `detectedLanguage`, `detectedScript` | {tag, evidence}? | from `ILanguageIdentifier`, per utterance; per-sentence values on `SentenceAnalysis` (P31) |
| `sentences[]` | SentenceAnalysis | multi-sentence discourse (P7) |
| `diagnostics[]` | {code, severity, span, message} | e.g. `UNKNOWN_LEXEME`, `PARTIAL_PARSE`, `AMBIGUOUS_SENSE` |
| `status` | enum `Complete \| Partial \| Failed` | |

**SentenceAnalysis**: `span`, `detectedLanguage?`, `sentenceType` (`Declarative`,
`Interrogative`, `Imperative`, `Exclamative`), `speechAct` (`Assert`, `Question`, `Request`,
`Command`, `Correction`, `Delete`, `Feedback`, `Greeting`, `Other`), `parses[]` (ranked
alternatives, each with a score; ≥1 when `Complete`), `selectedParseIndex?`, `clauses[]`.

**Clause** (recursive, compositional; FR-010):

| Field | Type |
|---|---|
| `clauseId` | ID |
| `kind` | `Main \| Subordinate \| Relative \| Complement \| Conditional \| Coordinate \| ReportedSpeech` |
| `predicate` | {lemma, senseCandidates[], selectedSense?, tokenRefs[]} |
| `arguments[]` | {role: SemanticRole (`Agent`, `Patient`, `Theme`, `Recipient`, `Instrument`, `Location`, `Time`, `Experiencer`, `Stimulus`, …), grammaticalFunction (`Subject`, `DirectObject`, `IndirectObject`, `Complement`, `Oblique`), phrase: Phrase} |
| `modifiers[]` | Phrase |
| `tense`, `aspect`, `mood`, `voice` | enums (`Present`/`Past`/`Future`; `Simple`/`Progressive`/`Perfect`/`PerfectProgressive`; `Indicative`/`Imperative`/`Subjunctive`/`Interrogative`; `Active`/`Passive`) |
| `polarity` | `Affirmative \| Negative` |
| `modality` | {kind: `None \| Possibility \| Necessity \| Permission \| Obligation \| Ability`, marker?} |
| `certainty` | Certainty (R11) |
| `relations[]` | {kind: `Temporal \| Spatial \| Causal \| Conditional \| Discourse`, targetClauseId, marker} |
| `subClauses[]` | Clause |
| `dependencies[]` | {head, dependent, label} |
| `isQuoted`, `isHypothetical`, `isCounterfactual`, `isNonLiteral` | bool. `isNonLiteral` is set only by explicit markers (`supposedly`, `allegedly`, `as if`, `/s`, scare quotes) |

**Phrase**: `kind` (`NP`, `VP`, `PP`, `AdjP`, `AdvP`, `Clause`), `head`, `tokenRefs[]`,
`determiner?`, `modifiers[]`, `entityRef?`, and `grammaticalFeatures` (person, number,
grammaticalGender?).

**EntityMention**: `mentionId`, `tokenRefs[]`, `kind` (`ProperName`, `Pronoun`, `CommonNP`),
`entityType` (`Person?`, `Unknown`, …), `resolvedEntityId?`, `resolution`
(`Resolved \| Unresolved \| Ambiguous`), `candidates[]`, `grammaticalGender?`.
**There is no `referentGender` field on mentions.** Referent gender exists only on `Entity` and
defaults to `Unknown` (FR-012).

### Entity (knowledge graph node)
`entityId`, `canonicalName`, `aliases[]`, `entityType`, `referentGender`
(`Unknown` default; changeable only by an explicit user-provided statement with provenance),
`lexicalGrounding?` (synsetId), `createdFrom` (knowledge refs).

### Proposition (immutable, language-neutral; P10, P30)

| Field | Type | Notes |
|---|---|---|
| `propositionId` | ID | |
| `predicate` | string | canonical predicate symbol, e.g. `PlaysInstrument` |
| `arguments[]` | {role, entityId? \| synsetId? \| literal?} | e.g. `(Seeta, Guitar)` |
| `polarity`, `tense`, `modality`, `certainty` | | preserved distinctions (FR-013) |
| `scope` | `Asserted \| Quoted \| Conditional \| Hypothetical \| Counterfactual \| Questioned \| NonLiteral` | prevents quoted, hypothetical or explicitly non-literal text becoming world knowledge |
| `surfaceVoice` | `Active \| Passive` | retained, not part of identity |
| `canonicalHash` | hex | over (predicate, args, polarity, modality, scope); voice excluded → active/passive compatible |

### Knowledge

| Field | Type | Notes |
|---|---|---|
| `knowledgeId` | ID | |
| `propositionId` | ID | subject/predicate/object come from it |
| `epistemicStatus` | enum (10 P12 classes) | `UserAsserted`, `ExternallyAsserted`, `Verified`, `LogicallyDerived`, `ProbabilisticallyInferred`, `Hypothesized`, `Disputed`, `Contradicted`, `Retracted`, `Unknown` |
| `dependsOnUnverified` | bool | true when any support is not `Verified` (R11) |
| `confidence` | decimal? | **null** unless a calibrated component sets it (FR-034) |
| `sourceAttribution` | {principalId?, dataId?, releaseId?} | |
| `supportingInformation[]`, `supportingKnowledge[]` | ID | |
| `inferenceRuleIds[]`, `evidenceIds[]` | | |
| `justificationId?` | ID | → Justification |
| `validFrom`, `validTo?` | timestamp | validity interval |
| `createdAt`, `lastVerifiedAt?` | timestamp | |
| `contradictionStatus` | `None \| Contradicts \| ContradictedBy` + `contradictionSetId?` | |
| `supersededBy?`, `supersedes?` | ID | corrections (US6-2) |
| `domain` | string | |
| `version` | int | temporal versioning (P14). Starts at 1 and increments on every revision (status change, re-justification, correction, validity change). Prior versions are kept in `knowledge_version` per retention |
| `memoryCategory` | MemoryCategory | |
| `privacy` | PrivacyClassification | inherited from the most restrictive support (P50) |

**Validation**:
- A `LogicallyDerived` knowledge item MUST have ≥1 rule ID and a justification.
- A `UserAsserted` item MUST have `sourceAttribution.principalId` and `dataId`.
- `Verified` MUST NOT be set by any baseline component.

**Epistemic transitions** (allowed):

```text
UserAsserted ──conflicting assertion──▶ Disputed
UserAsserted ──same speaker correction──▶ Retracted (supersededBy → new UserAsserted)
LogicallyDerived ──support retracted/erased──▶ Retracted (or re-justified if other support exists)
Hypothesized ──supporting evidence via rule──▶ LogicallyDerived | ProbabilisticallyInferred
any ──expiry (validTo passed)──▶ (excluded from queries; history retained per retention)
```

Forbidden: any transition to `Verified` from the baseline pipeline; any transition caused directly
by untrusted text content (P51).

### Justification (truth-maintenance node; FR-033)
`justificationId`, `conclusionKnowledgeId`, `ruleId`, `ruleVersion`, `bindings{}`,
`premiseKnowledgeIds[]`, `lexicalRelationIds[]` (e.g. the `Hypernym` edges
guitar→stringed instrument→musical instrument), `informationIds[]`, `dataIds[]`, `firedAt`,
`evidenceKind` (`EvidenceKind`: `Taxonomic \| Entailment \| HeuristicAssociation \| Hypothesis`;
FR-032). This is distinct from the lexical `RelationEvidenceClass`. Together these form the
reproducible explanation graph.

### ContradictionSet
`contradictionSetId`, `memberKnowledgeIds[]` (≥2), `kind` (`PolarityConflict`,
`ValueConflict`, `TemporalConflict`), `detectedAt`, `resolution`
(`Unresolved \| SupersededByCorrection \| RetainedBoth`). Members are never deleted by
detection (FR-035).

### InferenceRule
`ruleId`, `version`, `stage` (`Knowledge`, `Intelligence`), `domain`, `kind`
(`Declarative \| DynamicCompiled`), `sourceText` or `assemblyRef`, `priority`, `conflictStrategy`,
`consequenceKinds[]`, `allowedPredicates[]`, `evidenceSemantics`, `enabled`, `sha256`, and
`testRefs[]`. A rule must have positive, negative, contradiction, missing-evidence and provenance
tests before `enabled = true` (P47).

### Intelligence

| Field | Type |
|---|---|
| `intelligenceId`, `informationId`, `knowledgeRefs[]` | ID |
| `intent` | {kind: `Inform \| AskFact \| RequestDraft \| RequestSend \| Correct \| Delete \| Chitchat \| Unknown`, requestClass: `Explicit \| Implicit \| Suggested \| Unsolicited` (P17)} |
| `contextSnapshotRef` | ID |
| `goals[]` | string |
| `candidates[]` | Candidate |
| `selected[]` | candidate IDs (≥0; several retained when tied, FR-038) |
| `outcome` | `Answer \| Acknowledge \| Ask \| Recommend \| Clarify \| Abstain \| NoOp \| ProposeAction \| ExecuteAction` |
| `rationale` | text + factor table |
| `uncertainty[]`, `risks[]`, `policyConstraints[]`, `executionRequirements[]` | |

**Candidate**: `candidateId`, `kind` (`Response`, `Action`), `actionType?`, `template?`,
`groundingKnowledgeIds[]`, `prerequisites[]`, `expectedOutcome`,
`factors{relevance, intentAlignment, evidence, contextualSuitability, expectedUtility, risk, cost,
authorization, preference, feasibility}` (each `{value, weight}`), `score`,
`filteredBy?` (hard-constraint reason).

### Action

| Field | Type |
|---|---|
| `actionId` | ID |
| `intelligenceId`, `candidateId` | ID |
| `type` | ActionType (`ConversationalReply`, `ClarificationRequest`, `Computation`, `KnowledgeRetrieval`, `KnowledgePersistence`, `EmailDraft`, `EmailSend`, `MessageDraft`, `MessageSend`, `ApiInvocation`, `DatabaseOperation`, `ApplicationCommand`, `OsCommand`, `DeviceActivation`, `MachineControl`, `ScheduledOperation`, `CompositeWorkflow`) |
| `isExternalSideEffect` | bool (from the type registry) |
| `consequenceClass[]` | `ExternalRecipient`, `PrivateData`, `Financial`, `PhysicalDevice`, `Irreversible` |
| `parameters` | typed per handler; `protected` fields are redacted in logs |
| `authorizingPrincipalId?`, `destination?` | resolved recipient / endpoint |
| `idempotencyKey` | hex |
| `confirmationToken?` | bound to the canonical action hash |
| `state` | ActionState (below) |
| `handlerId`, `handlerVersion` | |
| `result?` | {status, externalOperationId?, output?, error?} |

#### Action lifecycle state machine

Implements P18 exactly. Owned by `ActionExecutionAutomaton`.

```text
CandidateIdentified → PreconditionsEvaluated → PolicyEvaluated → AuthorizationChecked
   → [ConfirmationRequested ⇄ (await user)] → Ready → Executing → Completed → AuditRecorded
Executing ──accepted (async handler)──▶ AwaitingExternalCompletion
AwaitingExternalCompletion ──poll/notification: succeeded──▶ Completed
AwaitingExternalCompletion ──poll/notification: failed──▶ Failed
AwaitingExternalCompletion ──completion deadline passed──▶ OutcomeUnknown
Executing ──transient error & handler idempotent & attempts < max──▶ RetryScheduled ──backoff elapsed──▶ Executing
Executing ──error|timeout (non-retryable or attempts exhausted)──▶ Failed → CompensationOrRecovery → AuditRecorded
PreconditionsEvaluated|PolicyEvaluated|AuthorizationChecked ──fail──▶ Blocked → AuditRecorded
ConfirmationRequested ──refused|cancelled|expired──▶ Cancelled → AuditRecorded
AwaitingExternalCompletion|RetryScheduled ──cancel (handler supports cancellation)──▶ Cancelled → AuditRecorded
Executing ──unknown delivery status──▶ OutcomeUnknown → (reconcile via idempotency key) → Completed|Failed
```

Guards: `Ready` requires `authorization = Granted` AND (confirmation not required OR the
confirmation token is valid for the current canonical hash). **No transition bypasses
`PolicyEvaluated` or `AuthorizationChecked`** (P18). `Completed` is reported only after the
handler's positive result.

**Asynchronous and long-running semantics (P18, P44, FR-049)**:
- `ConfirmationRequested`, `AwaitingExternalCompletion` and `RetryScheduled` are **suspendable**
  states. The run is snapshotted to `ISnapshotStore` on entry, and an `ActionResumptionService`
  resumes it after process restart.
- **Completion** arrives by handler polling (`PollStatusAsync`, default interval 5 s) or by an
  authenticated completion notification. A deadline (default 10 min) leads to `OutcomeUnknown`.
- **Retry policy** (`RetryPolicy`): `maxAttempts` (default 3), exponential `backoff` (default 2 s
  base, ×2, max 60 s). It applies only when `Capabilities.SupportsIdempotency = true` and the error
  is classified `Transient`. Each attempt reuses the same idempotency key.

The `Action` record gains these fields: `attempts`, `nextAttemptAt?`, `externalOperationId?`,
`completionDeadline?` and `snapshotRef?`.

### ActionAudit (append-only; P52)
`auditId`, `actionId`, `initiatingIntent`, `supportingKnowledgeIds[]`, `selectedActionType`,
`parametersRedacted`, `authorizationResult`, `confirmation` {requestedAt, respondedAt, response},
`stateTrail[]` {state, at}, `executedAt?`, `finalStatus`, `error?`, `externalOperationId?`,
`retentionPolicyId`.

---

## 3. Automata

### AutomatonDefinition
`automatonId` (e.g. `CognitiveLifecycleAutomaton`), `version`, `inputType`, `outputType`,
`states[]` {name, kind: `Initial \| Normal \| Terminal(Accepting|Failure|Cancelled)`,
entryOps[], exitOps[], invokes?: {automatonId, versionRange}}, `transitions[]` {from, event,
guard?, to, actions[]}, `errorSemantics`, `limits` {maxDepth, maxTransitions, timeout},
`cancellable`, `events{inputs[], outputs[]}`.
**Validation**: exactly one initial state; ≥1 terminal state; every state is reachable; every
non-terminal state has a failure path; guard and action IDs resolve in the registry.
Machine-readable form: [contracts/automaton-definition.schema.json](./contracts/automaton-definition.schema.json).

### AutomatonRun
`runId`, `automatonId`, `definitionVersion`, `parentRunId?`, `childRunIds[]`, `depth`,
`currentState`, `terminalState?`, `outcome` (`Succeeded \| Failed \| Cancelled \| Clarify \|
Deferred \| Rejected \| BudgetExceeded`), `budget` {transitionsUsed, elapsed, limits}, `trace[]`
{seq, from, event, guardResult?, to, at, durationMs}, `diagnostics[]`, `inputRef`, `outputRef?`,
`snapshot?` (versioned JSON for suspend/resume).

**Root automaton** (`CognitiveLifecycleAutomaton`, P21):

```text
Idle → DataAcquisition → InformationConstruction → KnowledgeConstruction → IntelligenceSynthesis
     → ActionSelection → ActionExecution → FeedbackAssimilation → Idle
(any non-Idle) ──needs clarification──▶ Clarification → Idle (turn completes with a clarification reply)
DataAcquisition|InformationConstruction ──reject (empty after normalization | exceeds input limits | no letters/digits in any script)──▶ Rejected → Idle
InformationConstruction ──all sentences LANGUAGE_UNSUPPORTED──▶ IntelligenceSynthesis (outcome Clarify: "only English is understood"; turn completes; context language updated)
(any non-Idle) ──defer (stage reports transient unavailability: dictionary release switching, store locked)──▶ Deferred → Idle   (snapshot retained; resumable via POST /turns/{turnId}/resume)
(any non-Idle) ──error──▶ Failed → Recovery → Idle
(any non-Idle) ──cancel──▶ Cancelled → Idle
InformationConstruction ──partial & no actionable content──▶ IntelligenceSynthesis (with diagnostics)
KnowledgeConstruction ──nothing to construct (e.g. question)──▶ IntelligenceSynthesis
ActionSelection ──NoOp──▶ FeedbackAssimilation
```

**Subordinate automata**:
- `InformationConstructionAutomaton`: LanguageIdentification, Lexical (→
  `LexicalIntelligenceAutomaton`), MorphologicalAnalysis, SyntacticAnalysis, SemanticAnalysis,
  PragmaticAnalysis, InformationValidation (P22).
- `LexicalIntelligenceAutomaton`: DictionaryLookup, LemmaResolution, PartOfSpeechResolution,
  WordSenseDisambiguation, SemanticRelationshipTraversal, LexicalValidation (P67).
- `KnowledgeConstructionAutomaton`: AssertionExtraction, EntityResolution, KnowledgeRetrieval,
  LogicalInference, ProbabilisticInference (stub: `NotConfigured` outcome in this feature),
  OntologyReasoning, ContradictionDetection, KnowledgeValidation, KnowledgePersistence (P23).
  `OntologyReasoning` **invokes `LexicalIntelligenceAutomaton`** to ground arguments.
  `LogicalInference` passes its execution context to the rule engine, so WordNet built-in predicates
  **invoke `SemanticRelationshipTraversalAutomaton`** as a child run rather than reading the
  repository directly. This puts the lexical automata under both the Information and Knowledge
  stages (P67, spec §5).
- `IntelligenceSynthesisAutomaton`: IntentUnderstanding, ContextEvaluation, GoalIdentification,
  CandidateGeneration, DecisionEvaluation, RiskAssessment, ActionRecommendation (P24).
- `ActionExecutionAutomaton`: the action lifecycle above.

---

## 4. Conversation and memory

### ConversationContext
`conversationId`, `principalId`, `turns[]` {turnIndex, dataId, informationId, intelligenceId,
actionIds[]}, `currentTopic?`, `salientEntities[]` {entityId, salience, lastMentionTurn},
`assertionsThisConversation[]`, `openQuestions[]`, `unresolvedReferences[]`,
`pendingActions[]` (awaiting confirmation), `clarificationRequests[]`, `language` (current;
set from each turn's `Information.detectedLanguage` by `FeedbackAssimilation`. It is **never fixed
to the first turn's language** (P31). A turn in an unsupported language yields a
`LANGUAGE_UNSUPPORTED` diagnostic and does not change the pack used for later English turns).
Context is **temporary**: it is stored in the `Conversational`/`Working` memory categories,
separately from `SemanticKnowledge` (P32).

### MemoryRecord
`memoryId`, `category` (`Working`, `Conversational`, `SemanticKnowledge`, `Episodic`,
`Procedural`), `contentRef` (entity type + ID), `retentionPolicyId`, `privacy`, `createdAt`,
`expiresAt?`, `revision`, `erasedAt?`.

### RetentionPolicy
`policyId`, `category`, `ttl?`, `maxItems?`, `erasable`, `keepCorrectionTrail`, and
`auditRedaction`.

**Default policies** (configurable in `MemoryOptions`, bound by Hosting; tests assert these defaults):

| Category | TTL | Max items (per principal) | Erasable | Keep correction trail | Audit redaction |
|---|---|---|---|---|---|
| `Working` | end of turn | 200 | n/a (auto-cleared) | no | n/a |
| `Conversational` | 30 days after last turn | 5,000 turns | yes | yes | references only |
| `SemanticKnowledge` | none | unlimited | yes | yes (90 days after erasure: hashed references only) | references only |
| `Episodic` | 365 days | 50,000 | yes | yes | references only |
| `Procedural` (rules, preferences) | none | 1,000 | preferences: yes; admin-installed rules: no (admin only) | yes | references only |

### UserPreference (Procedural memory)
`preferenceId`, `principalId`, `key` (e.g. `responseStyle`, `suggestFollowUps`, `topicInterest:music`),
`value` (string/number/bool), `source` (`Configuration \| UserStated`), `createdAt`. In this feature
these are seeded from configuration. The decision `preference` factor is 0 when no preference
applies (FR-038).

### Feedback (P37, FR-043)
`feedbackId`, `turnId`, `principalId`, `category` (`ParseAccuracy`, `InferenceCorrectness`,
`DecisionRelevance`, `UserAcceptance`, `ActionOutcome`, `ExecutionFailure`, `Correction`),
`targetRef?` (informationId / knowledgeId / intelligenceId / actionId), `rating`
(`Positive \| Negative \| Neutral`), `comment?` (untrusted text, `Personal` privacy), `source`
(`User \| System`), `createdAt`, `retentionPolicyId` (Episodic).
`ActionOutcome` and `ExecutionFailure` are written automatically by `ActionExecutionAutomaton`.
Feedback never alters epistemic status directly. Any ranking influence is recorded as a reversible
`PreferenceAdjustment` referencing the feedback IDs (P37 "traceable and reversible").

### PreferenceAdjustment
`adjustmentId`, `principalId`, `preferenceKey`, `delta`, `feedbackIds[]`, `createdAt`,
`revertedAt?`. Adjustments are read through `IPreferenceAdjustmentSource`, which is empty until the
US6 store exists. `IUserPreferenceProvider` applies active (non-reverted) adjustments on top of
configured preferences and reports the adjustment IDs used. The decision factor table therefore
shows every feedback-driven ranking change.

### PrivacyClassification
`Public \| Personal \| Sensitive \| Restricted`. Forwarding `Personal`+ data to an external
handler requires an authorization grant with the matching scope (FR-058).

---

## 5. Extension and registration

### ComponentRegistration (US7)
`componentId`, `kind` (`Parser`, `Grammar`, `LanguagePack`, `InferenceRule`, `ActionHandler`,
`DictionaryProvider`, `DecisionPolicy`), `version`, `contractId` + `contractVersionRange`,
`capabilities[]`, `compatibilityResult` (`Accepted \| Rejected(reason)`), `registeredAt`,
`registeredBy` (admin principal). Registration runs the contract test kit for that kind and
rejects the component on failure.

### LanguagePackManifest
`packId` (`en`), `version`, `languageTag` (`en`), `script` (`Latn`), plus resources: tokenizer
rules, closed-class lexicon, morphology tables, grammar (`.hgr`), semantic mappings, pragmatic
rules, response templates, test corpus, and `dictionaryProvider` binding (P29). See
[contracts/language-pack.md](./contracts/language-pack.md).

---

## 6. Processing trace (FR-055)

### ProcessingTrace
`traceId`, `turnId`, `dataId`, `rootRunId`, `stages[]` {stage, startedAt, durationMs, runId,
outputRef, diagnostics[]}, `dictionaryRelease` {releaseId, importId}, `tokens[]` (with offsets),
`lexicalAnalyses[]`, `informationRef`, `ruleFirings[]` {ruleId, version, bindings, conclusionId},
`knowledgeCreated[]`, `intelligenceRef`, `actionOutcomes[]`, and `budgets`.
Protected parameters are redacted. Retrieval requires the trace owner or an admin.

---

## Spec entity coverage

| Spec Key Entity | Defined here |
|---|---|
| `Data` | §2 Data |
| `Token` | §2 Token |
| `LexicalEntry` | §1 LexicalEntry |
| `WordSense` | §1 WordSense |
| `Synset` | §1 Synset |
| `SemanticRelation` | §1 SemanticRelation |
| `Information` | §2 Information / SentenceAnalysis / Clause |
| `Proposition` | §2 Proposition |
| `Knowledge` | §2 Knowledge (+ Justification, ContradictionSet) |
| `InferenceRule` | §2 InferenceRule |
| `Intelligence` | §2 Intelligence |
| `Action` | §2 Action |
| `AutomatonDefinition` | §3 AutomatonDefinition |
| `AutomatonRun` | §3 AutomatonRun |
| `ConversationContext` | §4 ConversationContext |
| `MemoryRecord` | §4 MemoryRecord |
| `DictionaryRelease` | §1 DictionaryRelease |
| `ActionAudit` | §2 ActionAudit |
| `Feedback` | §4 Feedback |
| `UserPreference` | §4 UserPreference |
