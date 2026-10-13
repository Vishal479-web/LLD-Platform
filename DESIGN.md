# Design note

## MVP and user flow
`Choose problem → Start attempt (DRAFT) → write design + rationale → Submit → (SUBMITTED → EVALUATING) → COMPLETED | PARTIAL | FAILED → review feedback & diff vs previous attempt → Try again`

Submission = text design (DSL or JSON) + rationale. A syntax error is reported **synchronously** (HTTP 422 with line numbers) and the attempt stays a DRAFT. Evaluation runs in a background worker; the UI polls.

## Class map
```
domain.py     Attempt (state machine), Finding, Feedback, compare_attempts()
model.py      DesignModel / ClassSpec / MethodSpec        <- the one thing evaluators understand
formats.py    DesignParser (ABC) -> DslParser, JsonParser; FormatRegistry
problems.py   Problem, Rubric(Concept, Behaviour, PatternHint), ProblemCatalog
evaluators.py Evaluator (ABC) -> RuleBasedEvaluator ; EvaluationPipeline
llm.py        LLMClient (Protocol) -> AnthropicClient ; LLMEvaluator
repository.py AttemptRepository (ABC) -> SqliteAttemptRepository
service.py    PracticeService (application layer)
bootstrap.py  composition root     app.py  thin HTTP adapter
```
Dependencies point inward: `app → service → {repository, formats, evaluators, problems} → domain/model`. Nothing in the domain knows about HTTP, SQL or an LLM vendor.

## Key decisions
- **Attempt owns its transitions** (`_ALLOWED` table). Illegal moves raise `InvalidTransition` (double submit, edit after submit, complete before evaluating). The service cannot corrupt state.
- **Normalise early.** Every format parses to `DesignModel`; evaluators are format-agnostic. Adding PlantUML or a diagram export = one new `DesignParser` + `registry.register(...)`.
- **Rubric as data.** Adding a problem = adding a `Problem` with aliases. No new evaluator code.
- **Strategy + pipeline for evaluation.** `Evaluator` is the extension point. `EvaluationPipeline` distinguishes *required* (failure → FAILED) from *optional* (failure → PARTIAL, retried with backoff). A future evaluator (e.g. a metrics-based or code-based one) is added in `bootstrap.py` only.
- **Deterministic floor, LLM ceiling.** Score and diffing use only rule findings (`source == "rules"`, stable `code`). AI findings are validated (severity whitelist, length caps, max count) and learner text is passed as delimited, untrusted data.

## Deterministic vs LLM
| Deterministic (rules) | LLM |
|---|---|
| Parse validity, undeclared types | Is the responsibility split *convincing* for this design? |
| Required concepts / behaviours present (alias match) | Naming, cohesion, coupling nuance |
| God class, data-only classes, isolated classes | Whether the rationale matches the structure |
| Abstraction exists and has ≥2 implementations | Alternative approaches and their trade-offs |
| Score, attempt-to-attempt diff | Narrative summary |

## Failure handling
- Parse error → sync 422, stays DRAFT.
- Optional AI error/timeout/garbage → bounded retry, then `PARTIAL` + note; manual `retry` endpoint.
- Required evaluator error → `FAILED` with a message (no stack trace), retryable.
- Crash mid-evaluation → `recover_stale()` at startup turns EVALUATING into FAILED (retryable).
- A worker exception can never leave an attempt stuck in EVALUATING.

## Scaling note (light HLD)
Single process + thread pool + SQLite is enough for the prototype. For more users: swap `SqliteAttemptRepository` for Postgres, swap the executor for a queue/worker (the service already hands off only an attempt id), and rate-limit/cached LLM calls. No domain changes needed.

## Trade-offs / limitations
- Alias matching is heuristic: a very unusual naming can be missed (learner sees a suggestion naming the expected concept, so it is explainable, but can feel harsh). Rationale text is not parsed by rules.
- Method *bodies* and sequence behaviour are not evaluated.
- Learner identity is a header (`X-Learner`), no auth.
- Drafts are saved only on submit in the UI (the API supports `PUT` anytime).
