# Component Interface Contracts (`Hcif.Abstractions`)

These are the versioned, language-neutral contracts that pluggable components implement (P39, P41,
P42, P55, P59). The contract version is `1.0.0`. Breaking changes require a major bump plus an
adapter. Signatures are normative at the level shown. Supporting record types are defined in
[data-model.md](../data-model.md). Every async method takes a `CancellationToken`, and every
long-running method honours the budgets in `ExecutionBudget`.

## Automata (P2, P25, FR-001–FR-006)

```csharp
public interface IAutomaton
{
    AutomatonDefinition Definition { get; }                 // id, version, states, transitions, limits
}

public interface IAutomaton<TIn, TOut> : IAutomaton
{
    ValueTask<AutomatonResult<TOut>> RunAsync(TIn input, IAutomatonExecutionContext ctx, CancellationToken ct);
}

public interface IAutomatonState { string Name { get; } StateKind Kind { get; } }
public interface ITransition    { string From { get; } string Event { get; } string? GuardId { get; } string To { get; } }

public interface IAutomatonExecutionContext
{
    string RunId { get; }
    string? ParentRunId { get; }
    int Depth { get; }
    ExecutionBudget Budget { get; }                          // depth, transitions, deadline (P40)
    ITraceSink Trace { get; }
    ValueTask<AutomatonResult<TChildOut>> InvokeAsync<TChildIn, TChildOut>(
        string automatonId, VersionRange range, TChildIn input, CancellationToken ct);
    ValueTask<SnapshotRef> SuspendAsync(CancellationToken ct); // P44
}

public sealed record AutomatonResult<T>(RunOutcome Outcome, T? Output, IReadOnlyList<Diagnostic> Diagnostics, string RunId);
public enum RunOutcome { Succeeded, Failed, Cancelled, Clarify, Deferred, Rejected, BudgetExceeded }

public interface IGuard  { string Id { get; } ValueTask<bool> EvaluateAsync(GuardContext ctx, CancellationToken ct); }
public interface IStateOperation { string Id { get; } ValueTask ExecuteAsync(OperationContext ctx, CancellationToken ct); }
```

## Language (P8, P29, P56)

```csharp
// Every language-specific service the Information stage needs is reachable through ILanguagePack,
// so Hcif.Cognition never references a concrete pack (P56, plan gate G8).
public interface ILanguagePack
{
    LanguagePackManifest Manifest { get; }
    ILanguageIdentifier LanguageIdentifier { get; }
    ITokenizer Tokenizer { get; }
    IMorphologicalAnalyzer Morphology { get; }
    IClosedClassLexicon ClosedClass { get; }
    ILanguageGrammar Grammar { get; }
    IParser Parser { get; }
    IEntityRecognizer Entities { get; }
    ICoreferenceResolver Coreference { get; }       // replaceable; default resolver marks all pronouns Unresolved
    ISemanticInterpreter Semantics { get; }
    IPragmaticInterpreter Pragmatics { get; }
    IResponseRealizer Responses { get; }
}

public interface ILanguageIdentifier
{
    LanguageIdentification Identify(string text);   // per-utterance + per-sentence tag, script, evidence; never mutates Data
}
public interface IEntityRecognizer
{
    IReadOnlyList<EntityMention> Recognize(IReadOnlyList<Token> tokens, ParseTree? tree); // no gender inference from names (P6)
}
public interface ICoreferenceResolver
{
    ValueTask<IReadOnlyList<EntityMention>> ResolveAsync(IReadOnlyList<EntityMention> mentions,
                                                        ConversationContext context, CancellationToken ct);
}

public interface ILanguageGrammar { string Id { get; } string Version { get; } GrammarDefinition Definition { get; } }
public interface IParser
{
    ValueTask<ParseResult> ParseAsync(IReadOnlyList<Token> tokens, ILanguageGrammar grammar,
                                      ParseOptions options, CancellationToken ct);   // returns ranked forest + diagnostics
}
public interface IMorphologicalAnalyzer { IReadOnlyList<MorphAnalysis> Analyze(string surface); }
public interface ISemanticInterpreter
{
    ValueTask<IReadOnlyList<Clause>> InterpretAsync(ParseTree tree, LexicalContext lex, CancellationToken ct);
}
public interface IInformationExtractor   // implemented by Hcif.Cognition InformationExtractor (drives InformationConstructionAutomaton)
{
    ValueTask<Information> ExtractAsync(Data data, ConversationContext context, CancellationToken ct);
}
```

## Dictionary (P59, FR-020, FR-021)

```csharp
public interface ILexicalDictionary
{
    DictionaryReleaseRef Release { get; }
    ValueTask<IReadOnlyList<LexicalEntry>> LookupWordAsync(string surface, LexicalPos? pos, CancellationToken ct); // via morphology
    ValueTask<IReadOnlyList<LexicalEntry>> LookupLemmaAsync(string lemma, LexicalPos? pos, CancellationToken ct);
    ValueTask<IReadOnlyList<LexicalPos>> GetPartsOfSpeechAsync(string lemma, CancellationToken ct);
    ValueTask<IReadOnlyList<WordSense>> GetSensesAsync(string lemma, LexicalPos? pos, CancellationToken ct);
    ValueTask<IReadOnlyList<string>> GetInflectionsAsync(string lemma, LexicalPos pos, CancellationToken ct); // source `form`, when available
    ValueTask<string?> GetSemanticCategoryAsync(string synsetId, CancellationToken ct);                   // lexFile
    ValueTask<AmbiguityProfile> AnalyzeAmbiguityAsync(string lemma, CancellationToken ct);
}
public interface ILexicalEntryRepository { ValueTask<LexicalEntry?> GetAsync(string entryKey, CancellationToken ct); }
public interface ISynsetRepository
{
    ValueTask<Synset?> GetAsync(string synsetId, CancellationToken ct);
    ValueTask<IReadOnlyList<string>> GetDefinitionsAsync(string synsetId, CancellationToken ct);
}
public interface ISemanticRelationshipRepository
{
    ValueTask<TraversalResult> TraverseAsync(string startId, TraversalOptions options, CancellationToken ct);
    ValueTask<IsAResult> IsAAsync(string synsetId, string ancestorSynsetId, TraversalOptions options, CancellationToken ct);
}
// P66 / FR-025 controls. All are configurable, with defaults in `LexiconOptions.Traversal` (bound by Hosting).
public sealed record TraversalOptions(
    IReadOnlySet<SemanticRelationKind> AllowedKinds,   // permitted relationship types
    int MaxDepth,                                      // maximum traversal depth
    int MaxNodes,                                      // expansion cap
    ExecutionBudget Budget,                            // inference execution budget
    ProcessingObjective Objective,                     // e.g. GroundArgument(targetSynsets), CheckIsA(ancestor), ExpandForWsd(contextLemmas)
    RelevanceFilter Relevance,                         // objective-guided pruning: target-ancestor depth bound, lexFile allowlist,
                                                       // min gloss/context overlap; branches that can't reach the objective are pruned
    EvidenceRequirement Evidence);                     // min path evidence: allowed RelationEvidenceClass set, forbid derivedInverse-only
                                                       // paths for IsA, max path length counted as evidence
// TraversalResult: Edges, Visited, Truncated, PrunedByRelevance, RejectedForEvidence, BudgetExhausted, Diagnostics.
// Cycle detection uses the visited set; duplicate (source, kind, target) edges are suppressed.
// IsAResult: IsA, EvidencePath (relation IDs), Depth, plus the same pruning/budget flags.
public interface IWordSenseResolver
{
    ValueTask<SenseResolution> ResolveAsync(SenseQuery query, CancellationToken ct); // retains candidates + evidence (P62)
}
public interface ILemmatizationService { IReadOnlyList<MorphAnalysis> Lemmatize(string surface, LexicalPos? pos); }
public interface IDictionaryImportService
{
    ValueTask<ImportJob> ImportAsync(ImportRequest request, CancellationToken ct);  // staged, idempotent (P60)
    ValueTask<DictionaryReleaseRef> RollbackAsync(string importId, CancellationToken ct);
}
```

## Knowledge and inference (P11–P14, P26–P28)

```csharp
public interface IKnowledgeRepository
{
    ValueTask<Knowledge> AddAsync(Knowledge item, CancellationToken ct);
    ValueTask<IReadOnlyList<Knowledge>> QueryAsync(KnowledgeQuery query, CancellationToken ct);  // excludes erased/expired by default
    ValueTask<ExplanationGraph> ExplainAsync(string knowledgeId, CancellationToken ct);
    ValueTask<RetractionReport> RetractAsync(string knowledgeId, RetractionReason reason, CancellationToken ct);
    ValueTask<ErasureReport> EraseAsync(EraseRequest request, CancellationToken ct);           // cascades via justifications
    ValueTask<ContradictionSet?> FindContradictionsAsync(Proposition p, CancellationToken ct);
}
public interface IInferenceRule
{
    string Id { get; } string Version { get; } RuleScope Scope { get; } int Priority { get; }
    ValueTask<IReadOnlyList<RuleMatch>> MatchAsync(FactView facts, RuleContext ctx, CancellationToken ct);
    IReadOnlyList<Consequence> Fire(RuleMatch match);       // pure; engine records provenance
}
public interface IInferenceEngine
{
    // ctx is the calling automaton's execution context (KnowledgeConstruction → LogicalInference), so
    // WordNet built-ins invoke SemanticRelationshipTraversalAutomaton as child runs (P67, spec §5).
    ValueTask<InferenceResult> RunAsync(InferenceRequest request, IAutomatonExecutionContext ctx,
                                        ExecutionBudget budget, CancellationToken ct);
    // result: derived knowledge + justifications + firings + BudgetExceeded diagnostic (FR-036)
}
public interface IDynamicRule        // compiled in Hcif.RuleHost only; receives immutable snapshots (P28, R12)
{
    string Id { get; } string Version { get; }
    IReadOnlyList<ProposedConsequence> Evaluate(FactSnapshot snapshot);
}
```

## Intelligence, conversation and actions (P15–P20, P32–P35)

```csharp
public interface IIntelligenceSynthesizer   // implemented by Hcif.Intelligence IntelligenceSynthesizer (drives IntelligenceSynthesisAutomaton)
{
    ValueTask<Intelligence> SynthesizeAsync(Information info, IReadOnlyList<Knowledge> knowledge,
                                            ConversationContext context, CancellationToken ct);
}
public interface IDecisionPolicy
{
    string Id { get; } string Version { get; }
    DecisionEvaluation Evaluate(IReadOnlyList<Candidate> candidates, DecisionContext ctx); // factors recorded; ties retained
}
public interface IConversationMemory
{
    ValueTask<ConversationContext> GetAsync(string conversationId, CancellationToken ct);
    ValueTask SaveAsync(ConversationContext ctx, CancellationToken ct);
    ValueTask<IReadOnlyList<MemoryRecord>> ListAsync(string principalId, MemoryCategory? category, CancellationToken ct);
    ValueTask<ErasureReport> ForgetAsync(string memoryId, string principalId, CancellationToken ct);
}
public interface IActionHandler
{
    string Id { get; } string Version { get; }
    ActionType Handles { get; }
    ActionHandlerCapabilities Capabilities { get; } // IsExternalSideEffect, ConsequenceClasses, SupportsIdempotency,
                                                    // SupportsCancellation, IsLongRunning, IsAsynchronous, RetryPolicy?
    ValidationResult ValidateParameters(ActionParameters p);
    // Synchronous handlers return Completed/Failed. Asynchronous handlers return Accepted(externalOperationId)
    // → AwaitingExternalCompletion (P18, P44). Failures carry ErrorClass Transient|Permanent for the retry policy.
    ValueTask<ActionExecutionResult> ExecuteAsync(ActionInvocation invocation, CancellationToken ct); // invocation carries IdempotencyKey, Attempt
    ValueTask<ActionExecutionResult> PollStatusAsync(string externalOperationId, CancellationToken ct); // async handlers only
    ValueTask<ActionExecutionResult?> ReconcileAsync(string idempotencyKey, CancellationToken ct);    // OutcomeUnknown recovery
    ValueTask<CancellationResult> CancelAsync(string externalOperationId, CancellationToken ct);      // when SupportsCancellation
    ValueTask<CompensationResult> CompensateAsync(ActionInvocation invocation, CancellationToken ct);
}
public sealed record RetryPolicy(int MaxAttempts, TimeSpan BaseBackoff, double Multiplier, TimeSpan MaxBackoff); // default 3, 2s, 2.0, 60s
public interface IActionResumptionService
{
    ValueTask<int> ResumePendingAsync(CancellationToken ct);  // on startup + periodically: resumes suspended action runs from ISnapshotStore
    ValueTask<Action> CompleteExternallyAsync(string actionId, ExternalCompletion completion, CancellationToken ct); // completion notification
}
public interface IFeedbackService   // P37, FR-043
{
    ValueTask<Feedback> RecordAsync(FeedbackRequest request, Principal principal, CancellationToken ct);
    ValueTask<IReadOnlyList<Feedback>> ListAsync(string turnId, Principal principal, CancellationToken ct);
}
public interface IUserPreferenceProvider   // FR-038; returns empty when none apply → preference factor 0
{
    ValueTask<IReadOnlyList<UserPreference>> GetAsync(string principalId, CancellationToken ct);
}
public interface IPreferenceAdjustmentSource   // US4 default: empty; US6: SQLite-backed (preference_adjustment table)
{
    ValueTask<IReadOnlyList<PreferenceAdjustment>> GetActiveAsync(string principalId, CancellationToken ct);
}
public interface ISafetyInterlock { string Id { get; } ValueTask<bool> PermitAsync(ActionInvocation i, CancellationToken ct); } // required for PhysicalDevice (FR-048)
public interface IPrincipalDirectory   // implemented in Hcif.Hosting from configuration; libraries never read HcifOptions
{
    Principal? FindByKey(string apiKey);
    Principal? Get(string principalId);
    IReadOnlySet<string> GetPermissions(string principalId);   // roles User | Admin | Integration + action grants/scopes
}
public interface IActionAuthorizationService
{
    ValueTask<AuthorizationDecision> AuthorizeAsync(Action action, Principal principal, CancellationToken ct);
    ConfirmationToken IssueConfirmation(Action action, Principal principal);       // bound to canonical action hash
    bool ValidateConfirmation(Action action, Principal principal, string token);
}
```

## Contract test kits

Each interface family ships an abstract xUnit contract test class (`*ContractTests<TImpl>`).
Registration (US7-1, SC-013) runs the kit for the component kind and accepts only on 100% pass.
The kits for `IAutomaton` cover the P45 checklist. The `IInferenceRule` kit requires positive,
negative, contradiction, missing-evidence and provenance cases (P47). The `IActionHandler` kit
uses mock integrations only (P48).
