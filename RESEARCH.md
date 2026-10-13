# Research note: LLD practice

> Written from my own working knowledge of these tools. **Re-verify the tool details below before submitting**; products change.

## The learner problem
LLD is easy to start and hard to judge. A learner can sketch a Parking Lot and still not know whether
responsibilities are well placed, whether an abstraction earns its keep, or what an interviewer would challenge.
Three things hurt repeated practice:
1. **No feedback loop.** Reading a "reference solution" is not the same as being reviewed.
2. **Many valid answers.** A single answer key punishes good alternative designs.
3. **No memory.** Each attempt stands alone, so learners cannot see whether they are improving.

## Approaches I looked at
| Approach | Good at | Gap for LLD practice |
|---|---|---|
| Course/book content (e.g. Educative-style LLD courses, GitHub "awesome LLD" repos, Refactoring.Guru for patterns) | Explaining patterns and worked solutions | Passive; no review of *your* design |
| Coding platforms (LeetCode, HackerRank style) | Deterministic tests, instant verdicts | Tests check output, not structure or responsibility placement |
| Diagram tools (draw.io, Excalidraw, PlantUML) | Free-form expression | No evaluation at all |
| System-design practice sites with AI feedback | Open-ended answers, conversational critique | Feedback is mostly unstructured LLM prose, not reproducible, hard to compare across attempts |
| Mock interviews with peers/mentors | Best quality of reasoning feedback | Slow, costly, not repeatable on demand |
| Pasting a design into a general chatbot | Instant | Inconsistent rubric, no history, no deterministic floor |

## Key gaps
- No tool combines **structured submission + consistent rubric + explainable feedback + attempt history**.
- Pure-LLM grading is inconsistent run-to-run and can be steered by the submission itself.
- Pure-rules grading is consistent but cannot discuss trade-offs.

## Product direction (MVP)
A focused **practice loop**: choose problem → write design (class outline + short rationale) → submit → get feedback → compare with last attempt → try again.

Decisions that follow from the gaps:
- **A learner provides** classes/interfaces with fields and method signatures **plus a short rationale**. That is the minimum from which responsibilities, relationships and extensibility can be judged. Method bodies are deliberately out of scope for the MVP.
- **Feedback must be valid for more than one design.** The rubric names *concepts and behaviours with aliases*, never class names. Subjective points are phrased as trade-off questions (SUGGESTION), reserved ISSUE for objective gaps (missing concept, undeclared type).
- **Deterministic vs LLM.** Rules produce the score and stable finding codes (reproducible, diffable between attempts). The LLM adds qualitative commentary only and never changes the score.
- **Improvement, not one-off solving.** Every attempt is stored; the next attempt shows what was resolved, still open and new.
- **Failure is a normal state.** AI review is optional: if it times out the learner still gets rule feedback (status PARTIAL) and can retry.

## Non-goals
Accounts/LMS, code execution, diagram image recognition, large-scale infrastructure.
