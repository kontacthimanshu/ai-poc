# Quickstart & Validation Guide: HCIF English-First Cognition

This guide holds runnable scenarios that prove the feature end-to-end. Commands follow
[contracts/cli.md](./contracts/cli.md), and payloads follow
[contracts/http-api.openapi.yaml](./contracts/http-api.openapi.yaml). Expected values map to the spec's
acceptance scenarios (US*n*-*m*) and success criteria (SC-*nnn*).

## Prerequisites

- .NET SDK 10.0.x (`dotnet --list-sdks`)
- About 500 MB of free disk for the originals, the release DB and the knowledge DB
- Network access **once** for acquisition. Alternatively, supply a pre-fetched
  `english-wordnet-2025-json.zip` (SHA-256 `7d749f6e2c39e6970e4997839dcf6e42fd281f3c2fae0171d2192bae8cfa4b51`)
  plus `LICENSE.md` and `WNDB_License.txt` from the `globalwordnet/english-wordnet` tag `2025-edition`

## 1. Build and run the fast test suites (no full dictionary needed)

```powershell
dotnet build Hcif.sln -warnaserror
dotnet test Hcif.sln --filter "Category!=FullDictionary"
```

Expected: all automata, contract, architecture, rule, handler and acceptance tests pass against the
checked-in OEWN mini-lexicon fixture (SC-001, SC-007–SC-009, SC-011, SC-013).

## 2. Initialize the dictionary (US2, SC-002, SC-012)

```powershell
dotnet run --project src/Hcif.Cli -- dictionary init
dotnet run --project src/Hcif.Cli -- dictionary status --json
```

Expected:
- `integrityResult = Verified`, `archiveSha256 = 7d749f6e…cfa4b51`
- counts ≈ lemmas 128,009 · senses 185,129 · synsets 107,519, with derived inverse relations reported separately
- unresolved internal references = 0 (external `wikidata` IDs are reported as external, not as unresolved)
- license files are present and hashed; `dictionary attribution` prints the Princeton WordNet and OEWN CC BY 4.0 notices

**Idempotency and rollback**:

```powershell
dotnet run --project src/Hcif.Cli -- dictionary import            # → AlreadyImported, exit 0
dotnet run --project src/Hcif.Cli -- dictionary import --force    # → identical contentHash
$env:HCIF_FAULT_INJECT="Indexing"; dotnet run --project src/Hcif.Cli -- dictionary import --force  # → Failed, exit 2
Remove-Item Env:HCIF_FAULT_INJECT
dotnet run --project src/Hcif.Cli -- dictionary lookup guitar     # previous release still answers (US2-4)
```

## 3. Offline lookup (US2-2, SC-003)

Disable the network adapter, or run the guarded suite:

```powershell
dotnet run --project src/Hcif.Cli -- dictionary lookup plays --relations hypernym --depth 3
dotnet test tests/Hcif.Integration.Tests --filter "Category=FullDictionary&Category=Offline"
```

Expected: `plays → play (+3SG +PRES)`. The verb senses include `play%2:36:01::`. Looking up `guitar`
gives hypernym chain `stringed instrument → musical instrument (03806455-n)`. There are zero network
calls.

## 4. Baseline sentence end-to-end (US1, US3, US4; FR-060)

```powershell
dotnet run --project src/Hcif.Cli -- chat --trace
> Seeta plays guitar.
> What instrument does she play?
> :trace
```

Expected for turn 1:
- **Data**: the exact text is preserved, with the source, timestamp and conversation ID.
- **Information**:
  - Declarative, affirmative, active voice, simple present.
  - `Seeta` is a ProperName subject with ReferentGender `Unknown`.
  - `plays → play` (3SG PRES), with sense `play%2:36:01::`.
  - `guitar` is the Theme, with isA musical instrument.
- **Knowledge**:
  - `PlaysInstrument(Seeta, Guitar)` is `UserAsserted`, attributed to the speaker.
  - `PlaysMusicalInstrument(Seeta)` is `LogicallyDerived` with `dependsOnUnverified=true`. It has a
    rule ID (`InstrumentGeneralization@1.0.0`) and a justification that lists the two hypernym edges.
  - An optional `PossiblyHasSkill` is `Hypothesized`.
  - There is **no** gender, personality, profession or contact-preference knowledge.
- **Intelligence**: the outcome is Acknowledge or Ask (e.g. "Does Seeta play guitar professionally,
  or is it a hobby?"), with a factor table.
- **Action**: a ConversationalReply only. The mock messaging outbox count is **0** (US5-1).

Expected for turn 2: `she` resolves to Seeta because she is the only compatible salient entity. The
answer is "guitar", grounded on the turn-1 assertion ID (US4-2).

API equivalent:

```powershell
$h = @{ "X-Hcif-Principal-Key" = "dev-local-key" }
$c = Invoke-RestMethod -Method Post http://localhost:5080/conversations -Headers $h
Invoke-RestMethod -Method Post "http://localhost:5080/conversations/$($c.conversationId)/turns" `
  -Headers $h -ContentType application/json -Body '{"text":"Seeta plays guitar.","includeTrace":true}'
```

## 5. Baseline corpus acceptance (SC-001)

```powershell
dotnet test tests/Hcif.Acceptance.Tests --filter "Category=BaselineCorpus"
```

This covers all 11 rows of the spec's corpus table: passive compatibility; negation as an opposing
assertion that is Disputed against the earlier one; a question that is not stored as a fact;
"might" → `Possible`; `The actor plays Hamlet` → role sense with no instrument; `He plays football`
→ sport sense; `blorptar` → `UNKNOWN_LEXEME` partial parse; draft-only; and a send that requires
resolution plus authorization.

## 6. Controlled action flow (US5, SC-008)

```text
> Draft a message inviting Seeta to a jam session     → MessageDraft returned, outbox 0
> Send that invitation to Seeta                       → Clarify (recipient not resolved) or ProposeAction (ConfirmationRequested)
> :confirm <actionId>                                 → MockMessagingSend executes once; Completed; audit recorded
> :confirm <actionId>                                 → 409 / no second send (idempotency)
```

Async, resumption and retry (FR-049, P18/P44):

```powershell
dotnet test tests/Hcif.Actions.Tests --filter "FullyQualifiedName~AsyncAndRetryTests"
```

Expected:
- The async mock moves `AwaitingExternalCompletion` to `Completed` (once, by poll or notification).
- A restart while awaiting resumes the run from its snapshot.
- Two transient failures then success give `attempts=3` and still only one send.

Also verify:
- `:cancel` produces `Cancelled`.
- A mock timeout produces `OutcomeUnknown`, which reconciles with no duplicate send.
- A handler error produces `Failed` and then `CompensationOrRecovery`. None of these report success.

## 7. Context, correction and erasure (US6)

```text
> Seeta plays guitar.   > Ravi plays football.   > What does she play?   → clarification "Who do you mean?" (two candidates, genders Unknown)
> I meant sitar, not guitar                       → new UserAsserted; old Retracted (supersededBy); derived knowledge re-justified for sitar
> :memory   then   :forget <memoryId>             → knowledge query no longer returns it; derived items with no other support retracted
> :feedback DecisionRelevance - "not what I asked" → feedback recorded (P37); knowledge status unchanged
```

## 7a. Turn outcomes and per-turn language (FR-005, P31)

```text
> (empty line / whitespace)                       → rootOutcome Rejected, Data retained
> ?!… 🎸                                          → rootOutcome Rejected (no letters/digits), Data retained
> सीता गिटार बजाती है।                              → rootOutcome Completed, decision Clarify ("I can only understand English for now."), LANGUAGE_UNSUPPORTED — not Rejected
> Seeta plays guitar. सीता गिटार बजाती है।          → sentence 1 processed; sentence 2 LANGUAGE_UNSUPPORTED; context language follows each turn
> Seeta plays guitar.                              → fully processed (session not bound to an earlier turn's language)
```

## 8. Extensibility and safety (US7, SC-009, SC-013)

```powershell
dotnet run --project src/Hcif.Cli -- components register --kind ActionHandler --path tests/fixtures/handlers/EchoHandler.dll  # Accepted
dotnet run --project src/Hcif.Cli -- components register --kind InferenceRule --path tests/fixtures/rules/EvilFileAccess.cs  # Rejected: System.IO
dotnet run --project src/Hcif.Cli -- rules check tests/fixtures/rules/cyclic.hrl                                            # BUDGET_EXCEEDED diagnostic, exits 4 within the limit
git diff --exit-code -- src/Hcif.Cognition/Definitions/CognitiveLifecycleAutomaton.json                                     # root definition unchanged
```

> **Deployment note (R12 residual risk)**: the rule-host process is resource-limited and
> analyser-restricted. For production, also deny network egress for `Hcif.RuleHost`, using a
> firewall rule or a container with `--network none`.

## 9. Evaluation and benchmark (SC-004–SC-006, SC-010)

```powershell
dotnet run --project src/Hcif.Cli -- eval run --dataset data/eval/semantic-core-v1 --out artifacts/eval-semantic.json
dotnet run --project src/Hcif.Cli -- eval run --dataset data/eval/morphology-v1     --out artifacts/eval-morph.json
dotnet run --project src/Hcif.Cli -- eval run --dataset data/eval/wsd-v1           --out artifacts/eval-wsd.json
dotnet run --project src/Hcif.Cli -- bench --utterances data/eval/bench-100.txt --warmup 10 --out artifacts/bench.json
```

Gates: semantic core ≥ 90% (n ≥ 300, per category); morphology ≥ 95% (n ≥ 100); WSD ≥ 85% selection
or correct abstention (n ≥ 100, abstention rate reported); p95 ≤ 2 s on the documented reference
machine. Reports include the dataset version and hash. These are release gates, not claims (FR-057).
