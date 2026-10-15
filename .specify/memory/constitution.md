<!--
SYNC IMPACT REPORT
==================
Version change: 1.1.0 → 1.1.1 (PATCH — structural reformat to the GitHub Spec Kit constitution
layout; no principle added, removed, or semantically redefined)

Note on prior version: the source document's header declared 1.0.0, while its appended English /
WordNet amendment (Principles 56–70) declared the resulting version to be 1.1.0. 1.1.0 is treated
as the effective prior version.

Modified principles:
  - All 70 principles retained with their original numbers (P1–P70) and normative text.
  - P39 title normalised: "Cognitive Processing SHALL Support Pluggable Engines"
    → "Cognitive Processing Shall Support Pluggable Engines" (capitalisation only).
  - P54 consolidated: the amended 18-step development sequence (former amendment section VIII)
    is now the authoritative sequence in "Development Workflow & Implementation Sequence";
    the superseded v1.0.0 10-step sequence is retained there for traceability.
  - Lower-case "SHALL not" normalised to "SHALL NOT" (P4, P28, P50, P55); no semantic change.

Added sections (structural only):
  - "Mission and Purpose" (former section I + former "Constitutional Objective")
  - Grouping headings (### I–XXIV) inside "Core Principles"
  - "Reference Cognitive Processing Scenario & Acceptance Baseline" (Spec Kit SECTION_2)
  - "Development Workflow & Implementation Sequence" (Spec Kit SECTION_3), incl.
    "Constitution Check Gates" derived from P53, P55 and Governance → Compliance

Removed sections: none. Relocated:
  - Former amendment sections "VIII. Amendment to the Development Sequence" → SECTION_3
  - Former amendment section "IX. Constitutional Compatibility" → Governance
  - Duplicate amendment roman numerals (II–IX) replaced by continuous grouping I–XXIV

Templates requiring updates:
  - ✅ .specify/templates/plan-template.md — "Constitution Check" reads gates from this file at
       runtime; the new "Constitution Check Gates" subsection supplies them. No edit required.
  - ✅ .specify/templates/spec-template.md — no change required; specs MUST cite principles by
       number (P1–P70) per Governance → Compliance.
  - ✅ .specify/templates/tasks-template.md — no change required.

Follow-up TODOs: none. No placeholders deferred.
-->

# Hierarchical Cognitive Intelligence Framework (HCIF) Constitution

**Full title:** Constitution: Hierarchical Finite Automata-Based Artificial Intelligence and
Action Framework
**Status:** Foundational Constitution

## Mission and Purpose

The Hierarchical Cognitive Intelligence Framework (HCIF) SHALL implement an artificially
intelligent conversational and action-taking system based on the following cognitive processing
lifecycle:

**Data → Information → Knowledge → Intelligence → Action**

- The framework SHALL accept natural-language inputs, comprehend their grammatical and semantic
  composition, derive contextual knowledge, perform intelligent reasoning, and execute
  appropriate actions.
- The system SHALL use hierarchical finite automata as its primary execution and orchestration
  mechanism.
- Each cognitive processing stage SHALL be implemented as one or more finite-state machines
  capable of activating subordinate finite-state machines, deterministic algorithms, domain
  rules, knowledge-retrieval procedures, and external computational services.
- The system SHALL support natural-language conversation, contextual learning, knowledge
  accumulation, decision-making, and interactions with external applications and physical
  devices.
- The framework SHALL remain extensible across languages, domains, operating environments, and
  communication channels.
- The framework SHALL NOT depend exclusively on a large language model for its cognitive
  processing.

### Constitutional Objective

The ultimate objective of HCIF is to construct an extensible artificial intelligence framework in
which natural-language communication is systematically transformed into structured information,
evidence-grounded knowledge, contextual intelligence, and authorized actions.

Hierarchical finite automata SHALL provide the execution discipline, grammatical systems SHALL
provide linguistic comprehension, knowledge engines SHALL provide inference, and decision engines
SHALL provide context-sensitive behavior.

The architecture SHALL make every processing stage independently understandable, testable,
replaceable, and extensible across languages and application domains.

## Core Principles

Principles are numbered P1–P70. Numbers are stable identifiers and MUST be used when feature
specifications, plans, and tasks reference this constitution.

### I. Foundational Cognitive Architecture (P1–P3)

#### P1. The Cognitive Lifecycle Is Mandatory

Every natural-language input SHALL enter a formally defined cognitive processing pipeline.

The primary lifecycle SHALL consist of the following stages:

1. Data acquisition
2. Information extraction
3. Knowledge construction
4. Intelligence synthesis
5. Action selection
6. Action execution
7. Feedback assimilation

- The framework SHALL permit transitions between stages according to formally specified
  conditions.
- Processing SHALL NOT assume that every input necessarily produces new knowledge, a decision, or
  an external action.
- An input MAY be rejected, deferred, clarified, or retained without advancing to the next
  cognitive stage.
- Feedback from subsequent processing SHALL be permitted to update prior interpretations.

#### P2. Hierarchical Finite Automata Shall Govern Execution

- Each primary cognitive stage SHALL be implemented using a finite-state automaton.
- A parent automaton SHALL be permitted to activate subordinate automata.
- Subordinate automata MAY activate additional automata recursively, subject to configured
  execution-depth and resource limits.

An automaton SHALL contain:

- A finite set of states
- An initial state
- A set of accepting or terminal states
- A transition function
- Input and output event definitions
- Transition guards where applicable
- Entry and exit operations
- Defined failure and recovery transitions

- The finite-state control model SHALL remain independent of the potentially unbounded data held
  in semantic objects, knowledge repositories, and runtime execution contexts.
- Hierarchical composition SHALL follow explicitly documented statechart or
  hierarchical-state-machine semantics.
- Conditions evaluated through dynamically compiled code SHALL operate as externally evaluated
  guards or actions rather than silently redefining the formal states of the underlying
  automaton.

#### P3. Cognitive States and Cognitive Objects Shall Be Separate

- Finite automata SHALL control the execution lifecycle.
- Typed objects SHALL contain the results of cognitive processing.

The framework SHALL represent each level using distinct models:

- `Data`
- `Information`
- `Knowledge`
- `Intelligence`
- `Action`

- Objects at successive levels SHALL reference their originating objects.
- The system SHALL preserve traceability across all transformations.
- An intelligence object SHALL be traceable to its supporting knowledge, information, and
  original data.

### II. Data Acquisition and Representation (P4)

#### P4. Natural-Language Data Shall Be Preserved

- The system SHALL treat an incoming sentence or utterance as an input data object.
- The original input SHALL be preserved without semantic modification.

A data object SHALL contain, where available:

- Unique identifier
- Original text
- Source identity
- Source application or channel
- Input timestamp
- Language identifier
- Script identifier
- Encoding
- Conversation identifier
- Speaker identity
- Audience or recipient
- Communication modality
- Contextual metadata
- Source reliability metadata
- Privacy classification

The system SHALL support inputs from:

- Keyboard-based conversations
- Speech-to-text interfaces
- Application programming interfaces
- Messaging applications
- Stored textual documents
- Event notifications
- Machine-generated textual output

The data acquisition stage SHALL NOT automatically consider user-supplied assertions to be
verified facts.

### III. Information Construction (P5–P7)

#### P5. Grammar-Driven Comprehension Is Mandatory

- The framework SHALL transform natural-language data into structured information through
  grammatical and semantic analysis.
- Language comprehension SHALL operate through language-specific grammar definitions and reusable
  language-neutral processing stages.

The information extraction pipeline SHALL support:

1. Language identification
2. Unicode normalization
3. Sentence segmentation
4. Tokenization
5. Lexical analysis
6. Morphological analysis
7. Part-of-speech identification
8. Syntactic parsing
9. Dependency analysis
10. Semantic role identification
11. Named-entity recognition
12. Coreference resolution
13. Tense, aspect, and modality identification
14. Negation and uncertainty identification
15. Semantic interpretation
16. Canonical information-object generation

- Each operation SHALL be implemented as a subordinate automaton or a computational service
  activated by an automaton.
- A failed or ambiguous parsing operation SHALL generate explicit diagnostics rather than
  silently manufacturing a unique interpretation.

#### P6. Information Shall Be Represented as Typed Semantic Objects

The framework SHALL produce detailed, strongly typed information objects.

Each information object SHALL represent, where applicable:

- Subject
- Predicate
- Verb
- Direct object
- Indirect object
- Complements
- Attributes
- Modifiers
- Named entities
- Pronouns
- Grammatical gender
- Referent gender, if known
- Number
- Person
- Tense
- Aspect
- Mood
- Voice
- Negation
- Modality
- Semantic roles
- Temporal relationships
- Spatial relationships
- Causal relationships
- Conditional relationships
- Grammatical dependencies
- Discourse relationships
- Speaker intent
- Speech-act classification
- Certainty and ambiguity metadata
- Original token offsets

- The distinction between grammatical gender and a person's actual gender SHALL be preserved.
- A gender value SHALL NOT be inferred solely from a person's name unless an explicitly
  identified source or user-provided context supports the inference.

#### P7. Information Objects Shall Support Arbitrary Linguistic Complexity

The information model SHALL support:

- Simple sentences
- Compound sentences
- Complex sentences
- Multiple clauses
- Nested clauses
- Direct and indirect speech
- Imperative sentences
- Interrogative sentences
- Conditional sentences
- Passive constructions
- Negation
- Hypothetical assertions
- Counterfactual statements
- Elliptical expressions
- Ambiguous statements
- Multi-sentence discourse

Information objects SHALL use compositional structures rather than relying on a fixed number of
grammatical fields.

### IV. Natural-Language Grammar Framework (P8–P10)

#### P8. Grammars Shall Be Declarative and Extensible

The framework SHALL define language grammars independently from the cognitive execution engine.

A grammar module SHALL contain:

- Language metadata
- Tokenization rules
- Lexical categories
- Morphological rules
- Phrase-structure rules
- Dependency rules
- Agreement rules
- Word-order rules
- Semantic mapping rules
- Pragmatic interpretation rules
- Ambiguity-resolution strategies
- Language-specific exceptions

- The system SHALL initially support English (see P56).
- The framework SHALL provide extension points for Hindi and other languages without modifying
  the cognitive automata.
- The architecture SHALL accommodate languages with substantially different word orders,
  inflection patterns, grammatical cases, and writing systems.

#### P9. English Grammar Shall Support a Formal Baseline

- The English grammar module SHALL implement a formal foundational grammar.
- The initial grammar MAY use a context-free or extended context-free representation.

An illustrative grammar is:

```ebnf
Sentence         = Declarative
                 | Interrogative
                 | Imperative
                 | Conditional ;

Declarative      = NounPhrase, VerbPhrase ;

NounPhrase       = ProperNoun
                 | Pronoun
                 | Determiner, Noun
                 | Determiner, AdjectivePhrase, Noun
                 | NounPhrase, PrepositionalPhrase ;

VerbPhrase       = Verb
                 | Verb, NounPhrase
                 | Verb, Complement
                 | Verb, NounPhrase, Complement
                 | Auxiliary, VerbPhrase
                 | VerbPhrase, Adverbial ;

Complement       = NounPhrase
                 | AdjectivePhrase
                 | PrepositionalPhrase
                 | Clause ;

AdjectivePhrase  = Adjective
                 | Adverb, Adjective ;

PrepositionalPhrase
                 = Preposition, NounPhrase ;

Clause           = Sentence ;

Interrogative    = QuestionPattern ;

Imperative      = CommandPattern ;

Conditional     = ConditionalPattern ;

ProperNoun       = LexicalProperNoun ;

Pronoun          = LexicalPronoun ;

Verb             = LexicalVerb ;

Noun             = LexicalNoun ;

Adjective        = LexicalAdjective ;

Adverb           = LexicalAdverb ;

Auxiliary        = LexicalAuxiliary ;

Determiner       = LexicalDeterminer ;

Preposition      = LexicalPreposition ;
```

- This grammar SHALL serve only as a foundational example.
- The production implementation SHALL support recursion, agreement, ambiguity, language-specific
  exceptions, and constructions not captured by the baseline.
- Grammatical recognition SHALL be followed by semantic interpretation rather than being treated
  as sufficient for comprehension.

#### P10. Parsing Shall Produce a Canonical Semantic Representation

Different linguistic expressions conveying equivalent meanings SHOULD map to compatible canonical
representations. For example:

- "Seeta plays guitar."
- "Seeta plays the guitar."
- "The guitar is played by Seeta."

These expressions SHOULD yield a compatible primary semantic proposition, subject to their
linguistic context.

- The system SHALL preserve meaningful differences in tense, emphasis, scope, negation, modality,
  and certainty.
- Canonical semantic representations SHALL be language-neutral wherever practical.

### V. Knowledge Construction (P11–P14)

#### P11. Information Shall Be Transformed Into Explicit and Inferred Knowledge

The knowledge-processing stage SHALL accept one or more information objects. It SHALL construct
knowledge using:

- Explicit assertion extraction
- Existing knowledge retrieval
- Logical inference
- Domain-specific rules
- Semantic relationships
- Ontological relationships
- Contextual association
- Temporal reasoning
- Causal reasoning
- Analogical reasoning
- Probabilistic reasoning
- Contradiction detection
- Evidence reconciliation

- Knowledge SHALL be represented as structured objects capable of supporting future reasoning.
- The framework SHALL distinguish direct assertions from derived conclusions.

#### P12. Knowledge Shall Preserve Epistemic Status

Every knowledge object SHALL include an epistemic classification. Supported classifications SHALL
include:

- User-asserted
- Externally asserted
- Verified
- Logically derived
- Probabilistically inferred
- Hypothesized
- Disputed
- Contradicted
- Retracted
- Unknown

The framework SHALL NOT automatically promote assumptions or cultural stereotypes into
established facts.

For example, from "Seeta plays guitar." the framework MAY derive:

- A user has asserted that Seeta plays guitar.
- Seeta is described as participating in a guitar-playing activity.
- Seeta may possess some guitar-playing ability.

The framework SHALL NOT conclude without additional evidence that:

- Seeta is necessarily a professional musician.
- Seeta is necessarily a music lover.
- Seeta is necessarily a nice or calm person.
- Seeta is necessarily female.
- Seeta necessarily wants to be contacted.

Such propositions MAY be represented as hypotheses when supported by appropriately justified
contextual evidence.

#### P13. Knowledge Shall Support Confidence and Provenance

Each knowledge object SHALL contain:

- Unique identifier
- Proposition
- Subject
- Predicate
- Object
- Supporting information references
- Supporting knowledge references
- Inference rule identifiers
- Evidence identifiers
- Epistemic classification
- Confidence or calibrated probability, where meaningful
- Validity interval
- Creation timestamp
- Last verification timestamp
- Source attribution
- Contradiction status
- Domain classification

- The framework SHALL differentiate logical certainty from statistical confidence.
- Numerical confidence SHALL NOT be arbitrarily assigned to unsupported statements.
- Every inferred conclusion SHALL have an inspectable justification.

#### P14. Knowledge Shall Be Stored in an Extensible Knowledge Repository

The system SHALL support persistent accumulation of knowledge. The knowledge repository SHALL
provide:

- Entity storage
- Property storage
- Relationship storage
- Fact and assertion storage
- Provenance records
- Contradiction tracking
- Temporal versions
- Domain taxonomies
- Ontological hierarchies
- Inference history
- Efficient semantic retrieval

- A graph representation SHOULD be supported.
- Knowledge persistence SHALL be independent of the automata execution engine.
- The repository SHALL support knowledge retraction, correction, expiry, and explicit deletion.

### VI. Intelligence Synthesis (P15–P17)

#### P15. Intelligence Shall Be Represented as Contextual Decision Capability

The intelligence-processing stage SHALL transform available knowledge into contextual
interpretations, goals, decisions, and candidate actions.

An intelligence object SHALL contain, where applicable:

- Current user intent
- Interpreted conversational context
- Relevant knowledge references
- Active goals
- Candidate responses
- Candidate actions
- Expected outcomes
- Action prerequisites
- Uncertainty
- Potential risks
- Policy constraints
- Selected decision
- Selection rationale
- Execution requirements

- Intelligence synthesis SHALL support both conversational and non-conversational objectives.
- The framework SHALL NOT define intelligence solely as the production of external actions.
- A decision to answer, request clarification, abstain, or perform no action SHALL be valid.

#### P16. Intelligence Shall Support Progressive Decision Refinement

The system SHALL progressively evaluate candidate actions according to:

- Relevance
- Intent alignment
- Supporting evidence
- Contextual suitability
- Expected utility
- Confidence
- Risk
- Cost
- Authorization
- User preferences
- Operational feasibility

- The intelligence stage SHOULD prioritize a small number of well-supported candidate actions.
- However, the system SHALL NOT assume that fewer candidate actions always imply greater
  intelligence.
- The framework SHALL support retaining multiple candidates when uncertainty or legitimate
  trade-offs require them.

#### P17. Intelligence Synthesis Shall Distinguish Intent From Opportunity

The system SHALL distinguish:

1. An explicitly requested action
2. An implicitly intended action
3. A suggested action
4. A possible but unsolicited action

For example, receiving "Seeta plays guitar." SHALL NOT automatically authorize sending Seeta a
message.

- The system MAY respond conversationally or propose a contextually relevant next step.
- Sending a message SHALL require an appropriate authorized request, recipient resolution, and
  applicable confirmation.

### VII. Action Selection and Execution (P18–P20)

#### P18. All Actions Shall Be Governed by Finite Automata

The Action Execution Engine SHALL use finite-state automata. The action lifecycle SHALL include:

- Candidate identified
- Preconditions evaluated
- Policy evaluated
- Authorization checked
- Confirmation requested, if required
- Ready
- Executing
- Completed
- Failed
- Compensation or recovery
- Audit recorded

- Action execution SHALL NOT bypass authorization or policy states.
- The framework SHALL support both synchronous and asynchronous external operations.

#### P19. Actions Shall Be Typed and Extensible

Actions SHALL be represented as typed objects. The framework SHALL support action categories
including:

- Conversational reply
- Clarification request
- Computation
- Knowledge retrieval
- Knowledge persistence
- E-mail drafting
- E-mail sending
- Messaging application interaction
- API invocation
- Database operation
- Application command
- Operating-system command
- Device activation
- Machine-control command
- Scheduled operation
- Composite workflow

- The system SHALL provide a standardized action-handler interface.
- New action types SHALL be registerable without modifying the cognitive lifecycle.

#### P20. External Actions Shall Require Explicit Security Controls

The system SHALL distinguish between reasoning about an action and actually performing it.

External actions SHALL be subject to:

- Identity verification
- Permission validation
- User consent where required
- Destination validation
- Parameter validation
- Data sensitivity policies
- Rate limiting
- Execution logging
- Error handling
- Cancellation where possible

- Operations affecting external recipients, financial resources, private information, physical
  devices, or irreversible state SHALL require appropriate safeguards.
- Actions affecting physical machinery SHALL additionally require independent operational safety
  controls and fail-safe behavior.
- No dynamically generated inference SHALL override execution permissions.

### VIII. Hierarchical Automata Organization (P21–P25)

#### P21. The Root Cognitive Automaton Shall Coordinate All Stages

The root automaton SHALL be named `CognitiveLifecycleAutomaton`. Its primary states SHALL be:

```text
Idle
  |
  v
DataAcquisition
  |
  v
InformationConstruction
  |
  v
KnowledgeConstruction
  |
  v
IntelligenceSynthesis
  |
  v
ActionSelection
  |
  v
ActionExecution
  |
  v
FeedbackAssimilation
  |
  v
Idle
```

Error, clarification, cancellation, and recovery transitions SHALL be supported throughout the
lifecycle.

#### P22. Information Processing Shall Have Subordinate Automata

The `InformationConstructionAutomaton` SHALL coordinate subordinate processing units including:

- `LanguageIdentificationAutomaton`
- `LexicalAnalysisAutomaton`
- `MorphologicalAnalysisAutomaton`
- `SyntacticAnalysisAutomaton`
- `SemanticAnalysisAutomaton`
- `PragmaticAnalysisAutomaton`
- `InformationValidationAutomaton`

Each subordinate automaton SHALL expose a standardized execution contract.

#### P23. Knowledge Processing Shall Have Subordinate Automata

The `KnowledgeConstructionAutomaton` SHALL coordinate:

- `AssertionExtractionAutomaton`
- `EntityResolutionAutomaton`
- `KnowledgeRetrievalAutomaton`
- `LogicalInferenceAutomaton`
- `ProbabilisticInferenceAutomaton`
- `OntologyReasoningAutomaton`
- `ContradictionDetectionAutomaton`
- `KnowledgeValidationAutomaton`
- `KnowledgePersistenceAutomaton`

#### P24. Intelligence Processing Shall Have Subordinate Automata

The `IntelligenceSynthesisAutomaton` SHALL coordinate:

- `IntentUnderstandingAutomaton`
- `ContextEvaluationAutomaton`
- `GoalIdentificationAutomaton`
- `CandidateGenerationAutomaton`
- `DecisionEvaluationAutomaton`
- `RiskAssessmentAutomaton`
- `ActionRecommendationAutomaton`

#### P25. Automata Shall Be Composable and Reusable

Every automaton SHALL expose a standard contract containing:

- Automaton identifier
- Version
- Input type
- Output type
- State definitions
- Transition definitions
- Entry conditions
- Exit conditions
- Error semantics
- Execution limits
- Cancellation support

- The framework SHALL permit the reuse of the same automaton across multiple cognitive stages.
- All automata SHALL be testable independently.

### IX. Rule Engine and Inference Grammar (P26–P28)

#### P26. A General-Purpose Rule Engine Shall Govern Derivation

- The framework SHALL implement a domain-neutral rule engine.
- Rules SHALL be expressible independently of the finite-state control mechanism.

A rule SHALL define:

- Rule identifier
- Applicable stage
- Applicable domain
- Input patterns
- Preconditions
- Evaluation logic
- Generated assertions or conclusions
- Candidate actions, if applicable
- Confidence and evidence semantics
- Priority
- Conflict-resolution strategy
- Version

#### P27. Rules Shall Support Declarative Expressions

The framework SHOULD implement a formal rule grammar. An illustrative grammar is:

```ebnf
Rule             = "RULE", Identifier,
                   "WHEN", Conditions,
                   "THEN", Consequences,
                   [ "PRIORITY", Integer ] ;

Conditions       = Condition,
                   { LogicalOperator, Condition } ;

Condition        = FactPattern
                 | RelationPattern
                 | ContextPattern
                 | TemporalPattern
                 | PredicateInvocation ;

Consequences     = Consequence,
                   { ",", Consequence } ;

Consequence      = AssertFact
                 | InferHypothesis
                 | RecommendAction
                 | RequestClarification
                 | EmitEvent ;

LogicalOperator  = "AND" | "OR" ;

FactPattern      = Entity, Predicate, Value ;

RelationPattern  = Entity, Relation, Entity ;

ContextPattern   = ContextKey, Operator, Value ;

TemporalPattern  = TemporalExpression ;

PredicateInvocation
                 = Identifier, "(", Arguments, ")" ;

AssertFact       = "ASSERT", Proposition ;

InferHypothesis  = "INFER", Proposition ;

RecommendAction  = "RECOMMEND", ActionExpression ;

RequestClarification
                 = "CLARIFY", QuestionExpression ;

EmitEvent        = "EMIT", EventExpression ;
```

The grammar SHALL be extended to support negation, quantification, variable binding, temporal
constraints, rule scopes, and inference provenance.

#### P28. Domain Rules Shall Be Dynamically Extensible

- The rule engine SHALL support rules implemented through dynamically compiled code.
- For the initial implementation, C# MAY be used as the rule implementation language.
- Dynamically loaded rules SHALL operate through versioned interfaces.
- Rules SHALL NOT be permitted to access unrestricted operating-system, network, or storage
  functionality.
- Untrusted dynamically compiled code SHALL execute within appropriate process or service
  isolation, using resource and permission restrictions.
- Rules SHALL be independently versioned, testable, and auditable.

### X. Multilingual Architecture (P29–P31)

#### P29. Natural-Language Grammars Shall Be Modular

Each supported language SHALL have an independently configurable language pack. A language pack
SHALL contain:

- Language identification rules
- Script information
- Tokenizer
- Morphological analyzer
- Lexicon
- Syntactic grammar
- Semantic mappings
- Pragmatic interpretation rules
- Response-generation rules
- Language-specific test corpus

The framework SHALL permit multiple versions of the same language pack.

#### P30. Canonical Knowledge Shall Be Language-Neutral

The framework SHALL separate linguistic surface forms from underlying semantic concepts.

For example, "Seeta plays guitar." and "सीता गिटार बजाती है।" SHOULD be convertible into
semantically compatible internal propositions.

- Knowledge inferred from one supported language SHALL be available when processing another.
- The system SHALL preserve language-specific differences that cannot be translated without loss.

#### P31. Multilingual Conversations Shall Be Supported

The framework SHALL support:

- Language switching between turns
- Mixed-language input
- Multilingual entity names
- Language-specific pronouns
- Cross-language entity resolution
- Multilingual response generation
- Localized action descriptions

A conversational session SHALL NOT be permanently bound to its initial language.

### XI. Conversational Intelligence (P32–P34)

#### P32. Conversation Shall Be Context-Aware

The application SHALL maintain a structured conversational context. Conversational context SHALL
include:

- Current topic
- Speaker references
- Identified entities
- Previous assertions
- Previous questions
- Resolved and unresolved references
- User intentions
- Pending actions
- Clarification requests
- Relevant historical knowledge

The framework SHALL distinguish temporary conversational context from persistent knowledge.

#### P33. Conversational Output Shall Be Governed by Knowledge

A conversational response SHALL be grounded in available information, knowledge, or explicitly
identified uncertainty. The system SHALL support:

- Questions
- Answers
- Explanations
- Recommendations
- Clarifications
- Follow-up questions
- Contextual suggestions
- Action proposals
- Statements of uncertainty

The framework SHALL NOT present hypotheses as verified facts.

#### P34. Personality and Response Style Shall Be Separately Configurable

- The system MAY include a configurable conversational personality.
- Personality SHALL influence presentation, tone, and preference-sensitive choices.
- Personality SHALL NOT alter the epistemic status of facts or bypass safety and authorization
  rules.

### XII. Memory and Learning (P35–P37)

#### P35. Cognitive Memory Shall Be Layered

The system SHALL distinguish:

- Working memory
- Conversational memory
- Semantic knowledge memory
- Episodic memory
- Procedural or rule memory

Each memory category SHALL define its retention, retrieval, and deletion policies.

#### P36. Learning Shall Be Evidence-Based

- The system SHALL support updating its knowledge from new observations.
- A new statement SHALL NOT automatically invalidate existing knowledge.
- Conflicting statements SHALL undergo contradiction analysis.
- The framework SHALL support incremental learning without requiring modification of the core
  automata.

#### P37. Feedback Shall Improve Future Decisions

The framework SHALL record permitted feedback concerning:

- Parse accuracy
- Inference correctness
- Decision relevance
- User acceptance
- Action outcome
- Execution failures
- Explicit user corrections

- Feedback MAY influence future rule selection, context interpretation, and recommendation
  ranking.
- Changes to learned behavior SHALL remain traceable and reversible where practical.

### XIII. Extensible Computational Intelligence (P38–P40)

#### P38. Deterministic and Probabilistic Processing Shall Coexist

The framework SHALL support deterministic finite-state orchestration and both deterministic and
probabilistic computational services.

The following processing strategies MAY be incorporated:

- Symbolic inference
- Rule-based deduction
- Ontological reasoning
- Graph traversal
- Statistical classification
- Machine-learning inference
- Neural language models
- External knowledge retrieval
- Numerical optimization
- Planning algorithms

- The selected processing strategy SHALL be declared and auditable.
- The finite-state orchestration layer SHALL remain responsible for execution progression, error
  handling, and authorization boundaries.

#### P39. Cognitive Processing Shall Support Pluggable Engines

- Language and intelligence processing components SHALL be replaceable through defined
  interfaces.
- The framework SHALL permit replacing one parser, inference engine, or reasoning implementation
  without rewriting the root cognitive lifecycle.
- External intelligence services SHALL be optional dependencies rather than mandatory
  architectural foundations.

#### P40. Reasoning Shall Be Bounded and Observable

The system SHALL protect against:

- Infinite inference loops
- Circular rule activation
- Unbounded automata recursion
- Repeated action execution
- Excessive computation
- Unbounded context expansion
- Duplicate knowledge generation

Processing budgets and termination criteria SHALL be configurable.

### XIV. Technical Implementation (P41–P44)

#### P41. Core Framework Shall Be Strongly Typed

- The initial implementation SHOULD use modern C# and a supported .NET runtime.
- The architecture SHALL favor well-defined interfaces, immutable processing results where
  practical, and strongly typed domain models.

The following foundational abstractions SHOULD be provided:

- `IAutomaton`
- `IAutomatonState`
- `ITransition`
- `IAutomatonExecutionContext`
- `ILanguageGrammar`
- `IParser`
- `ISemanticInterpreter`
- `IInformationExtractor`
- `IKnowledgeRepository`
- `IInferenceRule`
- `IInferenceEngine`
- `IIntelligenceSynthesizer`
- `IDecisionPolicy`
- `IActionHandler`
- `IActionAuthorizationService`
- `IConversationMemory`

(Lexical abstractions are defined in P59.)

#### P42. Architecture Shall Separate Responsibilities

The framework SHALL use distinct architectural layers:

1. Input and communication
2. Language recognition and parsing
3. Canonical semantic representation
4. Knowledge and inference
5. Intelligence synthesis and decision-making
6. Action orchestration and execution
7. Persistence and memory
8. Observability and administration

Each layer SHALL communicate through explicit contracts.

#### P43. Interfaces Shall Support Multiple Client Types

- The system SHALL support an API-first architecture.
- The cognitive framework SHALL remain independent of its user interface.

Potential clients SHALL include:

- Console applications
- Web applications
- Desktop applications
- Mobile applications
- Chat interfaces
- Voice interfaces
- Autonomous application integrations

#### P44. Long-Running Operations Shall Be Supported

- The system SHALL accommodate external actions that complete asynchronously.
- Automata execution contexts SHALL support suspension, persistence, resumption, cancellation,
  and recovery.
- State persistence SHALL be explicit and version-aware.

### XV. Verification and Testing (P45–P49)

#### P45. All Automata Shall Be Testable

Each automaton SHALL have automated tests covering:

- State initialization
- Valid transitions
- Invalid transitions
- Transition guards
- Subordinate automata invocation
- Successful completion
- Failure handling
- Cancellation
- Execution limits
- Recovery behavior

#### P46. Grammar Correctness Shall Be Measurable

Each language pack SHALL maintain a versioned grammatical test corpus. Tests SHALL include:

- Valid simple sentences
- Compound and complex sentences
- Ambiguous sentences
- Negated statements
- Questions
- Commands
- Conditional statements
- Pronoun resolution
- Morphological variations
- Cross-language equivalents

The system SHALL evaluate parsing and semantic extraction against annotated expected results.

#### P47. Knowledge Inference Shall Be Explainable and Verifiable

Every inference rule SHALL have:

- Positive test cases
- Negative test cases
- Contradiction test cases
- Missing-evidence test cases
- Provenance verification tests

The framework SHALL test that unsupported conclusions are not promoted to established facts.

#### P48. Actions Shall Be Tested Independently

- Action handlers SHALL be testable without invoking real external applications.
- Mock integrations SHALL be used for automated testing.
- Real external actions SHALL require explicit integration-test authorization.

#### P49. Acceptance Criteria Shall Be Quantifiable

Every feature specification SHALL define measurable acceptance criteria. Applicable metrics
SHOULD include:

- Language identification accuracy
- Parsing accuracy
- Semantic role extraction accuracy
- Coreference resolution accuracy
- Knowledge inference precision
- Contradiction detection accuracy
- Confidence calibration
- Decision relevance
- Action selection accuracy
- Unauthorized action count
- End-to-end response latency

- Metrics SHALL be measured against explicitly identified evaluation datasets.
- No arbitrary accuracy claim SHALL be treated as an implemented capability.

### XVI. Security, Privacy, and Trust (P50–P52)

#### P50. User Data Shall Be Governed by Explicit Policies

- The framework SHALL classify and protect data according to its sensitivity.
- Personal information SHALL NOT be shared with external systems without an appropriate legal
  basis and authorization.
- Persistent user memories SHALL support inspection and deletion.
- Knowledge derived from private communications SHALL retain applicable privacy restrictions.

#### P51. Untrusted Input Shall Not Become Executable Authority

- Natural-language input, retrieved documents, generated rules, and external messages SHALL be
  treated according to their trust levels.
- No untrusted text SHALL directly modify execution permissions, system policies, or action
  authorization requirements.
- The system SHALL defend against instruction injection and unauthorized tool invocation.

#### P52. Every External Action Shall Be Auditable

External action records SHALL include:

- Initiating intent
- Supporting knowledge
- Selected action
- Input parameters, appropriately protected
- Authorization result
- Execution timestamp
- Execution status
- Error information
- External operation identifier, where available

Audit data SHALL follow applicable retention and privacy policies.

### XVII. Development Discipline (P53–P55)

#### P53. Development Shall Be Specification-Driven

Development SHALL follow the GitHub Spec Kit workflow. Every implementation feature SHALL have:

- A written specification
- Functional requirements
- Explicit assumptions
- Acceptance scenarios
- An implementation plan
- A task breakdown
- Automated validation

#### P54. Foundational Components Shall Be Implemented First

- The initial implementation SHALL prioritize foundational components in the order defined in
  **Development Workflow & Implementation Sequence → Mandatory Development Sequence**.
- Advanced features SHALL build upon validated foundational components.

#### P55. Architectural Boundaries Shall Be Maintained

- No feature SHALL directly couple grammatical parsing to an external action handler.
- No feature SHALL bypass the knowledge validation and action authorization boundaries.
- Core automata SHALL NOT contain hard-coded application-specific behavior.
- Domain-specific behavior SHALL reside in pluggable grammars, rules, policies, and action
  handlers.

### XVIII. English-First Language Scope (P56)

#### P56. English Shall Be the Initial Natural Language

- The first production implementation of HCIF SHALL target English exclusively.
- All foundational natural-language comprehension, information extraction, knowledge
  construction, intelligence synthesis, and conversational response mechanisms SHALL first be
  implemented and validated using English.
- The architecture SHALL preserve language-neutral abstractions to accommodate additional
  languages in subsequent releases.
- Other language grammars SHALL NOT be required for initial acceptance.
- The implementation SHALL NOT embed English-specific assumptions into the core finite-state
  automata.
- English-specific rules SHALL reside within the English language module.

### XIX. Mandatory English Lexical Knowledge Base (P57–P60)

#### P57. Open English WordNet 2025 Shall Be the Foundational Dictionary

The system SHALL use **Open English WordNet 2025 — JSON Edition** as its primary English
lexical-semantic knowledge base.

| Item | Value |
|------|-------|
| Official source | https://en-word.net/ |
| Official JSON dataset | https://en-word.net/static/english-wordnet-2025-json.zip |
| Source repository | https://github.com/globalwordnet/english-wordnet |
| License | Creative Commons Attribution 4.0, subject to applicable underlying resource notices |

- The dictionary SHALL be downloaded, validated, imported, and indexed as part of the project's
  reproducible data-initialization process.
- The application SHALL NOT require live internet access for ordinary dictionary lookup after
  installation.
- The original downloaded dataset SHALL be preserved separately from the application's internal
  lexical representation.

#### P58. Dictionary Coverage Shall Include Lexical and Semantic Knowledge

The lexical knowledge base SHALL support retrieval of:

- Words and lexical entries
- Lemmas
- Parts of speech
- Word senses
- Synsets
- Definitions
- Synonyms
- Antonyms
- Hypernyms
- Hyponyms
- Meronyms
- Holonyms
- Verb relationships
- Lexical and semantic relationships
- Available usage examples
- Other relationships provided by the source dataset

- The implementation SHALL preserve source identifiers and relationship semantics.
- The system SHALL NOT assume that every lexical entry contains every relationship type.
- Dictionary data SHALL be treated as a source of lexical and semantic evidence, not as a
  complete representation of real-world knowledge.

#### P59. A Dictionary Abstraction Layer Is Mandatory

The framework SHALL implement a dictionary abstraction independent of Open English WordNet's
physical JSON representation. The abstraction SHALL expose operations for:

- Word lookup
- Lemma lookup
- Part-of-speech retrieval
- Sense retrieval
- Synset retrieval
- Definition retrieval
- Semantic relationship traversal
- Inflection lookup, when available
- Semantic category identification
- Lexical ambiguity analysis

The foundational interfaces SHOULD include:

- `ILexicalDictionary`
- `ILexicalEntryRepository`
- `IWordSenseResolver`
- `ISynsetRepository`
- `ISemanticRelationshipRepository`
- `ILemmatizationService`
- `IDictionaryImportService`

The dictionary provider SHALL be replaceable without modifying the hierarchical automata.

#### P60. JSON Import Shall Be Schema-Aware and Versioned

The framework SHALL implement a dedicated JSON dictionary importer. The importer SHALL:

1. Download the specified dictionary release.
2. Verify archive integrity and source metadata.
3. Safely extract the archive.
4. Inspect and validate the actual source JSON schema.
5. Parse lexical entries, senses, synsets, and relationships.
6. Resolve references between lexical entities.
7. Import the data into the internal lexical repository.
8. Create lookup indexes.
9. Record the imported dictionary version.
10. Generate an import validation report.

- The importer SHALL NOT assume undocumented JSON property names or structures.
- Imported lexical entities SHALL preserve their source identifiers.
- Dictionary import SHALL be repeatable and idempotent.
- Failures SHALL be recoverable without corrupting an existing validated dictionary.

### XX. Dictionary-Assisted Grammatical Comprehension (P61–P63)

#### P61. Lexical Lookup Shall Be Integrated into Information Construction

The `InformationConstructionAutomaton` SHALL invoke the lexical dictionary during language
comprehension. The following processing sequence SHALL be supported:

1. Acquire English sentence.
2. Tokenize the sentence.
3. Normalize tokens.
4. Retrieve candidate lexical entries.
5. Identify candidate parts of speech.
6. Perform morphological analysis.
7. Parse sentence structure.
8. Resolve lexical ambiguity.
9. Assign semantic roles.
10. Construct canonical information objects.

- The framework SHALL support tokens absent from the dictionary, including personal names, newly
  coined words, abbreviations, and domain-specific terminology.
- Unknown vocabulary SHALL NOT automatically terminate grammatical comprehension.

#### P62. Word-Sense Disambiguation Shall Be Context-Aware

The system SHALL recognize that a word may have multiple meanings. For example:

- "She plays guitar."
- "He plays football."
- "The actor plays Hamlet."

The verb "play" occurs in different semantic contexts.

- The framework SHALL use grammatical structure, surrounding words, semantic relations, and
  conversational context to identify candidate meanings.
- The system SHALL preserve competing meanings when reliable disambiguation is not possible.
- A dictionary's available senses SHALL NOT be treated as proof that a particular interpretation
  is correct.

#### P63. Morphological Analysis Shall Complement WordNet

The system SHALL support recognition of word inflections, including:

- Singular and plural nouns
- Verb tense forms
- Verb participles
- Comparative and superlative adjectives
- Irregular forms
- Common derivational constructions

For example: `plays → play`, `played → play`, `playing → play`.

- The system SHALL employ grammatical and morphological rules where the primary dictionary does
  not provide sufficient inflectional information.
- Morphological analysis SHALL remain distinct from semantic inference.

### XXI. Dictionary-Assisted Knowledge Construction (P64–P66)

#### P64. Semantic Relationships Shall Support Knowledge Inference

The `KnowledgeConstructionAutomaton` SHALL use WordNet's semantic relationships when applying
inference rules.

For example, given "Seeta plays guitar." the framework MAY derive the following sequence:

| Step | Result |
|------|--------|
| Explicit assertion | `PlaysInstrument(Seeta, Guitar)` |
| Lexical-semantic grounding | `Guitar → MusicalInstrument` |
| Derived proposition | `PlaysMusicalInstrument(Seeta)` |

The derived proposition SHALL reference:

- The original assertion
- The dictionary's supporting semantic relationship
- The inference rule used
- The inference timestamp
- The epistemic classification

The system SHALL distinguish taxonomic relations from logical implications, correlations, and
hypotheses.

#### P65. Dictionary Knowledge Shall Not Justify Unsupported Personal Conclusions

WordNet relations SHALL NOT be used to infer personal attributes that lack sufficient evidence.

For example, from "Seeta plays guitar." the framework SHALL NOT automatically conclude:

- Seeta is female.
- Seeta is kind.
- Seeta is calm.
- Seeta is a professional musician.
- Seeta wants to receive a message.

- The framework MAY generate qualified hypotheses where relevant and supported by additional
  information.
- Hypotheses SHALL be explicitly distinguished from verified or logically established knowledge.

#### P66. Lexical Semantic Expansion Shall Be Bounded

Knowledge inference SHALL support semantic graph traversal through WordNet. The system SHALL
implement configurable controls for:

- Maximum relationship traversal depth
- Permitted relationship types
- Cycle detection
- Duplicate inference elimination
- Relevance filtering
- Evidence requirements
- Inference execution budgets

- The inference engine SHALL NOT indiscriminately expand every word into every reachable synset.
- Semantic expansion SHALL be guided by the current processing objective.

### XXII. Lexical Processing Automata (P67–P68)

#### P67. A Lexical Intelligence Automaton Shall Be Introduced

The framework SHALL implement `LexicalIntelligenceAutomaton`. This automaton SHALL coordinate
dictionary-assisted lexical processing.

Its subordinate automata SHOULD include:

- `DictionaryLookupAutomaton`
- `LemmaResolutionAutomaton`
- `PartOfSpeechResolutionAutomaton`
- `WordSenseDisambiguationAutomaton`
- `SemanticRelationshipTraversalAutomaton`
- `LexicalValidationAutomaton`

- The lexical automaton SHALL operate as a subordinate component of the information and knowledge
  processing stages.
- It SHALL communicate through typed objects and versioned interfaces.

#### P68. Lexical Processing Results Shall Be Persistable

The system SHALL support caching or persisting resolved lexical interpretations. A lexical
processing result SHOULD include:

- Original token
- Normalized token
- Resolved lemma
- Candidate parts of speech
- Candidate word senses
- Selected word sense, if any
- Synset references
- Grammatical context
- Supporting evidence
- Ambiguity status
- Source dictionary version

Cached results SHALL be invalidated or re-evaluated when relevant dictionary versions or
interpretation rules change.

### XXIII. Dictionary Licensing and Legal Compliance (P69)

#### P69. Dictionary Acquisition Shall Respect Intellectual Property Rights

- The framework SHALL obtain Open English WordNet through its officially published distribution.
- The system SHALL retain applicable license notices, attribution, and source information.
- The application SHALL NOT scrape copyrighted dictionary portals to reproduce proprietary
  dictionary content without appropriate authorization.
- Additional lexical datasets SHALL be accepted only after their redistribution, modification,
  attribution, and commercial-use terms have been reviewed.
- The dictionary import process SHALL record licensing metadata alongside the imported dataset.
- The framework SHALL support generating attribution notices for distribution with the
  application.

### XXIV. Dictionary Testing and Validation (P70)

#### P70. Dictionary Integration Shall Be Fully Testable

The English dictionary subsystem SHALL be validated using automated tests. Tests SHALL cover:

- Dataset download and extraction
- JSON schema validation
- Lexical entry import
- Lemma resolution
- Part-of-speech classification
- Polysemous word lookup
- Synset retrieval
- Semantic relationship traversal
- Missing vocabulary handling
- Dictionary version changes
- Inference provenance
- Duplicate prevention
- Query performance

- The phrase "Seeta plays guitar" SHALL be included in the baseline acceptance corpus.
- The framework SHALL successfully identify "plays" as a form of "play" and identify "guitar" as
  a musical instrument using available lexical and semantic knowledge.
- The proper noun "Seeta" SHALL be handled through named-entity processing, regardless of whether
  it exists in the dictionary.

## Reference Cognitive Processing Scenario & Acceptance Baseline

This scenario is normative: it defines the expected end-to-end behavior for the baseline
acceptance sentence (see P70).

### Input

"Seeta plays guitar."

### Stage A: Data

The system SHALL store the original sentence and associated source metadata.

### Stage B: Information

The information processor SHALL identify:

| Field | Value |
|-------|-------|
| Subject | Seeta |
| Subject category | Person or named entity, subject to resolution |
| Predicate | plays guitar |
| Verb lemma | play |
| Verb tense | simple present |
| Grammatical person | third person |
| Grammatical number | singular |
| Object | guitar |
| Object category | Musical instrument |
| Semantic actor | Seeta |
| Semantic activity | playing a guitar |
| Polarity | affirmative |
| Sentence type | declarative |

Person-specific gender SHALL remain unknown unless provided by contextual evidence.

### Stage C: Knowledge

- The knowledge processor SHALL create the explicit proposition `PlaysInstrument(Seeta, Guitar)`.
- The proposition SHALL initially be attributed to the speaker.
- The framework MAY derive `Instrument(Guitar)` from established domain knowledge.
- It MAY derive a qualified hypothesis such as `PossiblyHasSkill(Seeta, GuitarPlaying)`.
- The knowledge processor SHALL preserve supporting evidence and inference provenance.

### Stage D: Intelligence

The intelligence processor SHALL evaluate possible interpretations and responses. Potential
candidate actions MAY include:

- Acknowledge the statement
- Ask whether Seeta plays professionally
- Ask whether the user also plays an instrument
- Suggest a musical collaboration if relevant
- Offer to draft an invitation if context warrants it

- The system SHALL rank or filter candidates according to available context.
- It SHALL NOT assume that an external action is desirable merely because one is technically
  possible.

### Stage E: Action

- For an ordinary informational statement with no additional request, the system MAY produce a
  conversational response such as: "Does Seeta play guitar professionally, or is it a hobby?"
- If the user explicitly requests a message invitation, the action processor MAY generate a draft
  and, with the necessary authorization, send the message through an installed messaging
  integration.

### Stage F: Feedback

The system SHALL record permitted conversational feedback and update contextual understanding.

## Development Workflow & Implementation Sequence

### Specification-Driven Workflow

Development SHALL follow the GitHub Spec Kit workflow (P53): specification → clarification (as
needed) → implementation plan → task breakdown → implementation → automated validation.

### Mandatory Development Sequence

Per P54 (as amended in v1.1.0), the development order SHALL be:

1. Core finite-state automata framework
2. Hierarchical automata composition
3. Open English WordNet 2025 JSON acquisition
4. Dictionary importer and internal lexical repository
5. Dictionary lookup and indexing services
6. English tokenization and morphological analysis
7. English grammatical parsing
8. Canonical information object generation
9. Word-sense disambiguation
10. Semantic relationship traversal
11. Knowledge repository
12. Deterministic knowledge inference
13. Intelligence synthesis
14. Conversational response generation
15. Controlled action execution
16. Conversational memory and context retention
17. Advanced inference and probabilistic reasoning
18. Additional language packs

<!-- Superseded v1.0.0 sequence, retained for traceability (P54 original):
     1 Finite-state automata framework; 2 Hierarchical automata composition; 3 Data and
     information models; 4 English-language parsing; 5 Canonical semantic representations;
     6 Structured knowledge storage; 7 Deterministic knowledge inference; 8 Basic intelligence
     synthesis; 9 Conversational responses; 10 Controlled action execution. -->

### First Functional Milestone

The first functional milestone SHALL demonstrate:

**English Sentence → Dictionary Lookup → Grammar Analysis → Information Object → Knowledge
Inference → Intelligence Object → Conversational Response**

All stages SHALL provide inspectable intermediate results.

### Constitution Check Gates

Every implementation plan's Constitution Check SHALL verify, at minimum:

- **Spec completeness (P53, P49):** written specification, functional requirements, explicit
  assumptions, acceptance scenarios, and measurable acceptance criteria against identified
  evaluation datasets.
- **Sequence (P54):** the feature does not depend on an unvalidated earlier step of the Mandatory
  Development Sequence.
- **Automata discipline (P2, P3, P25, P45):** execution is governed by finite automata exposing
  the standard contract; cognitive state is separate from typed cognitive objects; automata
  tests cover the P45 checklist.
- **Architectural boundaries (P42, P55):** no parsing-to-action coupling; no bypass of knowledge
  validation or action authorization; no application-specific behavior in core automata.
- **Epistemic integrity (P12, P13, P33, P65):** provenance and epistemic status preserved;
  hypotheses never presented or stored as verified facts.
- **Security (P20, P28, P50–P52):** authorization, untrusted-input handling, rule isolation, and
  audit requirements satisfied; real external actions only under explicit integration-test
  authorization (P48).
- **Bounded reasoning (P40, P66):** processing budgets, depth limits, and cycle detection
  configured.
- **English-first scope (P56):** no English-specific assumptions in core automata.

## Governance

This constitution supersedes conflicting practices, conventions, and documents within HCIF.

### Amendment Procedure

Changes to this constitution SHALL require:

1. A documented amendment proposal
2. Identification of affected principles
3. Architectural impact analysis
4. Compatibility assessment
5. Updated validation requirements
6. Version increment
7. Amendment approval

### Versioning Policy

The constitution SHALL use semantic versioning:

- **MAJOR:** Removal or incompatible modification of foundational principles
- **MINOR:** Addition of new principles or substantial extensions
- **PATCH:** Clarifications that do not materially change architectural obligations

### Compliance

- Feature specifications, implementation plans, and task definitions SHALL reference applicable
  constitutional principles (by number, P1–P70).
- Any deviation SHALL be explicitly justified and documented (in the plan's Complexity Tracking
  section).

### Foundational Priority

When implementation convenience conflicts with these principles, correctness, explainability,
security, and extensibility SHALL take precedence.

### Constitutional Compatibility

- The English-language / Open English WordNet amendment (v1.1.0) supplements the original HCIF
  Constitution v1.0.0.
- Principles P1–P55 remain effective unless explicitly modified.
- Principles P56–P70 SHALL be mandatory for the initial English-language implementation.
- The framework SHALL remain architecturally capable of supporting other languages in future
  releases without requiring redesign of the cognitive lifecycle.

**Version**: 1.1.1 | **Ratified**: 2026-10-15 | **Last Amended**: 2026-10-15
