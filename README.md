# Research Sense in the Loop — prompt kit

From the DevDay Exchange Community talk "Codex as a Research Partner". Copy, adapt, and keep your own judgment in the loop.

## 1. Three questions instead of "summarize"

Ask them one at a time, and decide yourself before asking the next one.

1. **Find the assumption**
   > Which assumption does this claim depend on? Quote the source passage and its location,
   > then show where my code (or design) makes the same assumption. List what the source
   > did not decide for us.
2. **Test the boundary**
   > If I did not know the true answer, what in the observed data alone could tell me this
   > assumption is wrong? What else could produce the same signal? If my question is too
   > broad, say so.
3. **Choose one check**
   > Propose one feasible check. Fix the rule before any result, and state which outcome
   > would make me keep, revise, or abandon my hypothesis. Then stop and wait for me.

## 2. Evidence labels

Ask the agent to tag every claim:

| Label | Meaning |
|---|---|
| Source states | The paper, document or record says it (give the location) |
| Project choice | We decided it; the source did not prescribe it |
| Not established | No evidence yet; stays unknown |

## 3. Opening brief template

```text
Goal: [one sentence research or engineering question].
Context: [project path], read-only. Write all new files to [new folder].
Rules for every round:
- Do not assume my intuition is right, or that a check will succeed or fail.
- Tag claims as Source states / Project choice / Not established, with file or page locations.
- Anything not read, not verified or not executed is marked NOT RUN. Never invent results.
- Pin dependencies and keep the lock file. If the environment fails, stop and report.
- After each round, stop and wait for my decision.
Start with question 1 only.
```

## 4. Guardrails for an agent-run check

1. Freeze the decision rule (and its hash) before reading any data.
2. Run edge-case tests before the real run.
3. Re-check saved inputs before using them; stop on any mismatch.
4. Score without labels, freeze the scores, then join labels.
5. Write down your own expectation before the result, and compare afterwards.
6. Lock dependencies before handing execution to a sandboxed agent; then run frozen/offline.

## 5. Research habit → engineering twin

| Research habit | Engineering twin |
|---|---|
| Ask for the assumption, not a summary | Ask what a design or RFC silently assumes |
| Freeze the decision rule first | Set A/B acceptance criteria before launch |
| Score blind, join labels later | Keep test data apart from tuning data |
| An alarm is not the cause | An alert firing is not a root cause |
| Report weak cells separately | Break metrics down by segment, not just the mean |

## 6. Where the questions come from

The question-driven approach is adapted from a short video on reading papers with AI.
It groups 20 questions into five sets:

1. Is it worth reading? (core problem, why it matters, real novelty, what is left without buzzwords)
2. What did the author quietly assume? (core assumption, which one collapses the paper, correlation vs causation, rival explanations)
3. Does the design hold up the conclusion? (bias, sample size, how the core variable was measured, alternative measurement)
4. Don't be fooled by a pretty p-value (practical significance, robust vs fragile results, odd results, when it fails)
5. Grow your next study from it (other settings, unresolved issues, three next questions, link to your own work)

The video also gives two rules for the prompt itself: "Don't just summarize; review it like
a critical peer reviewer" and "For every judgment, cite where the evidence is; don't write
speculation as fact."
