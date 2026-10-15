# Language Pack Contract (P8, P29, P56)

A language pack is a versioned directory of declarative resources plus an assembly that implements
`ILanguagePack` (see [component-interfaces.md](./component-interfaces.md)). The pack exposes
**every** language-specific service the Information stage invokes: language identifier,
tokenizer, morphology, closed-class lexicon, grammar, parser, entity recognizer, coreference
resolver, semantic and pragmatic interpreters, and response realizer. Core automata never reference
a concrete pack (P56). An architecture test enforces this.

## Layout (`src/Hcif.Language.English/Pack/`)

```text
pack.json                       # manifest (below)
tokenizer/rules.json            # segmentation & tokenization rules, abbreviations list
lexicon/closed-class.json       # determiners, pronouns (+person/number/grammaticalGender), auxiliaries,
                                # modals, prepositions, conjunctions, wh-words, negators — absent from WordNet
morphology/irregular.json       # irregular forms WITH feature bundles (e.g. went→go +PAST)
morphology/rules.json           # suffix detachment rules (-s/-es/-ies/-ed/-ing/-er/-est, doubling, e-drop)
grammar/english.hgr             # declarative feature CFG (extends constitution P9 baseline)
semantics/role-mappings.json    # syntax→semantic role rules (active/passive → canonical args, FR-013)
semantics/predicates.json       # lexical-frame → canonical predicate (e.g. play%2:36:01:: + Theme → PlaysInstrument)
pragmatics/speech-acts.json     # sentence type/markers → speech act & intent class (P17)
responses/*.yaml                # response realization templates (+ personality variants, hedges)
corpus/                          # versioned grammatical test corpus (P46)
```

## Manifest `pack.json`

```json
{
  "packId": "en",
  "version": "1.0.0",
  "languageTag": "en",
  "script": "Latn",
  "contractVersion": "1.0.0",
  "dictionaryProvider": { "kind": "OEWN", "release": "oewn-2025" },
  "grammar": { "file": "grammar/english.hgr", "formalism": "feature-cfg/earley", "version": "1.0.0" },
  "corpus": { "dir": "corpus", "version": "1.0.0" },
  "ambiguityPolicy": { "retainParsesWithinMargin": 0.15, "maxRetainedParses": 5, "maxTokensPerSentence": 40 }
}
```

## Grammar file `.hgr` (format sketch)

Productions use the P9 EBNF style plus feature constraints in `[...]`, with unification variables
`?x`:

```text
S[type=decl]           -> NP[case=nom, per=?p, num=?n] VP[per=?p, num=?n, vform=fin]
VP[voice=passive]      -> AUX[lemma=be, per=?p, num=?n] V[vform=pastpart] (PP[prep=by] @agent)?
NP                     -> PROPN | PRON | DET? ADJP* N PP*
```

Semantic annotations (`@agent`, `@theme`) feed `semantics/role-mappings.json`. The production
grammar MUST support recursion, agreement, ambiguity and exceptions beyond the baseline (P9).

## Corpus item schema (`corpus/*.jsonl`)

```json
{"id":"en-base-001","text":"Seeta plays guitar.","category":"declarative-simple",
 "expect":{"sentenceType":"Declarative","polarity":"Affirmative","voice":"Active",
   "predicate":{"lemma":"play","sense":"play%2:36:01::"},
   "args":[{"role":"Agent","head":"Seeta","mention":"ProperName"},
           {"role":"Theme","head":"guitar","isA":"03806455-n"}],
   "tense":"Present","aspect":"Simple","referentGender":"Unknown"}}
```
