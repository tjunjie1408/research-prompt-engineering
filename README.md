# Research Sense in the Loop — Prompt Architecture & Decision Ledger

Copy, adapt, and keep your own judgment in the loop.

---

## Why This Exists: The Manager–Agent Split

When you ask an agent to *"summarize this paper"* or *"check if the fit worked,"* it blends **what the source says**, **what your code chose**, and **what it guessed** into one confident paragraph.

To keep ownership of the technical direction, treat the session like a **Manager–Researcher loop**:
- **You (The Manager)** own the hypothesis, set the information boundary, freeze the acceptance rule before seeing data, and make the final call.
- **Codex (The Researcher)** traces exact premises, challenges your intuition under strict constraints, verifies data integrity, and executes frozen checks mechanically.

```text
  [You: Frame the Question]                [Codex: Gather & Verify Evidence]
  ┌─────────────────────────┐              ┌───────────────────────────────────┐
  │ SCQA Delegation Prompt  │ ───────────► │ Odd Entries (Entry 1, 2, 4)       │
  │ • S: Current setup      │              │ • Source states (exact location)  │
  │ • C: Hidden doubt/risk  │              │ • Project choice (our defaults)   │
  │ • Q: 1 of 3 Questions   │              │ • Not established / NOT RUN       │
  │ • A: Output + Stop rule │              │ • Stops & waits for confirmation  │
  └─────────────────────────┘              └─────────────────┬─────────────────┘
               ▲                                             │
               │           [You: Log the Decision]           ▼
               │           ┌───────────────────────────────────┐
               └────────── │ "b" Entries (Entry 1b, 2b, 3, 4b) │
                           │ • PREP Judgment (Point/Reason/    │
                           │   Example/Point)                  │
                           │ • Pre-data expectation locked     │
                           └───────────────────────────────────┘
```

---

## 1. The Core Loop: Three Questions Instead of "Summarize"

Ask these **one round at a time**. Make your own decision between rounds before moving to the next question.

### Round 1 — Find the Assumption
> *"Which assumption does this claim depend on? Quote the source passage and its exact location, then show where my code (or design) makes the same assumption. List what the source did not decide for us."*

- **Why a summary fails:** A summary tells you the paper assumes $F_{\text{ext}} = 0$, but never checks whether your fitter hardcodes that assumption or which parameters your code invented.

### Round 2 — Test the Boundary
> *"If I did not know the true answer, what in the observed data alone could tell me this assumption is wrong? What else could produce the same signal? If my intuition is too broad, challenge it."*

- **Why a summary fails:** An agent will politely agree with your hunch unless you explicitly forbid ground-truth peeking and ask for competing explanations.

### Round 3 — Choose One Check (Freeze Before Running)
> *"Propose one feasible observation-only check. Fix the decision rule and thresholds before any result exists, and state which outcome would make me keep, revise, or abandon my hypothesis. Then stop and wait for my confirmation."*

- **Why a summary fails:** Defining success after seeing the output turns verification into curve-fitting.

---

## 2. How to Delegate to Codex: The SCQA Prompt Architecture

When delegating a task to Codex, use **SCQA** (*Situation – Complication – Question – Answer*) to make your thinking chain explicit.

> **Crucial Adaptation for AI Agents:**
> In a standard business presentation, **A (Answer)** is your conclusion.
> When prompting an agent, you do not know the conclusion yet—so **A (Answer Contract)** defines **the deliverable format, evidence labels, and the stop condition**.

### Copy-Paste SCQA Template

```text
[S — Situation: What exists right now]
We have [system / codebase / dataset] at [path] (read-only).
Currently, [what the model or code does, e.g., optimizer converges with 0 warnings].

[C — Complication: Why a naive check or summary fails]
I suspect [your intuition, e.g., high training residual hides a model error].
However, in a real setting we do not have [forbidden info: ground truth / future window / noise sigma],
and [competing risk: e.g., normal noise might also trigger a naive threshold].

[Q — Question: The exact analytical task for this round]
Using ONLY [allowed inputs], evaluate [specific question].
For each candidate signal, identify:
1. Required premise
2. Competing explanations
3. Whether it is assessable in our current setup

[A — Answer Contract: Deliverable format + Evidence labels + Stop boundary]
Write the result to [target file].
- Tag every claim as: Source states / Project choice / Not established.
- Mark any unexecuted check or unread file as: NOT RUN.
- Do NOT run new fits or choose the next step. Stop and wait for my judgment.
```

### Real Session Walkthrough (Prompt 2 — Testing the Residual Boundary)

| SCQA Block | What We Actually Sent / Enforced | Why It Kept Us Safe |
|---|---|---|
| **S — Situation** | Unforced spring-mass-damper model fit on 20 noisy points ($t \in [0,6]\text{ s}$). Run `E07` converged cleanly, but its training residual is $12.6\text{ cm}$ vs. $5\text{ cm}$ noise. | Anchors the agent to a concrete artifact (`E07`) instead of abstract theory. |
| **C — Complication** | My hunch is that residual size exposes the omitted force. But a real researcher doesn't know true $c, k$, force labels, future data, or true noise $\sigma$. | Blocks the agent from cheating with evaluator-only columns (`truth.csv`). |
| **Q — Question** | Within the observation window alone, which residual signals can test model adequacy, what premise does each require, and what competing causes exist? | Forces the agent to test **against** our hunch rather than rubber-stamp it. |
| **A — Answer Contract** | Output a comparison table in `residual-signals.md`, append evidence to `judgment-log.md` with strict labels, run **no** new code (`NOT RUN`), and stop. | Produced the exact finding (`Q75 = 5.30 cm > 5 cm` in unforced runs) that changed our mind. |

---

## 3. How to Record Your Own Call: The PREP Judgment Framework

Once Codex stops and returns evidence, **do not immediately prompt the next task.** Write down your own decision first using **PREP** (*Point – Reason – Example – Point*).

Forcing yourself to write a 4-line PREP entry prevents you from drifting along with whatever the agent suggested.

### Copy-Paste PREP Template (for Human Judgment Entries)

```markdown
## Entry [N]b — My Judgment

- **Point (Decision):** I am [keeping / revising / abandoning] my initial hypothesis and switching the next step to [chosen check].
- **Reason (Constraint):** [Why the previous idea failed or why this check fits the information boundary].
- **Example (Artifact Evidence):** In [file / metric returned by Codex], [concrete number, e.g., Q75 of the unforced group is 5.30 cm > 5.0 cm sigma], which means [consequence, e.g., a size threshold causes >25% false alarms].
- **Point (Committed Next Step & Doubt):** Next, we will [exact protocol to design/run]. My remaining uncertainty is [what could still fail, e.g., statistical power at n=20].
```

### Real Session Example (`judgment-log.md` — Entry 2b, First Change of Mind)

> - **Point:** Instead of checking residual magnitude, I am switching the next step to examine the temporal order of residual signs (**sign runs**).
> - **Reason:** Comparing residual size against $\sigma$ requires knowing the true noise level, which is unavailable in real data—and elevated residuals only show that joint assumptions failed, not uniquely that an external force exists.
> - **Example:** In `summary.json`, the 75th percentile (`Q75`) of training RMSE for the **correctly specified** unforced model is already **$5.30\text{ cm}$** (above $\sigma = 5\text{ cm}$), so a magnitude threshold would trigger false alarms in at least 1 of 4 valid runs.
> - **Point:** We will use a one-sided lower-tail sign-runs check ($\sigma$-free), measuring false alarms on the unforced group and detections on the forced group—while noting that $n=20$ over ~1.4 forcing periods may have limited power.

---

## 4. The Evidence Contract & `judgment-log.md`

### Required Epistemic Labels

Every statement Codex writes must carry one of four explicit status tags:

| Tag | Meaning | Verification Requirement |
|---|---|---|
| **`Source states`** | Explicitly written in the paper, lecture note, or saved artifact | Must cite exact page, equation, or file + line number |
| **`Project choice`** | A parameter, bound, or design choice our implementation made | Must cite the config or code line where we set it |
| **`Not established`** | Plausible, or claimed in an older note, but not verified in this round | Remains open; cannot be used as a proven premise |
| **`NOT RUN`** | Code, solver, test, or check that was not executed in this round | Prevents hallucinated execution or silent reuse |

### How `judgment-log.md` Differs from a Standard `.log` File

Audience question: *"How is `judgment-log.md` structured, and how does it differ from a normal `.log` file?"*

| Dimension | Standard Runtime `.log` | `judgment-log.md` (Human–Agent Decision Ledger) |
|---|---|---|
| **Author** | Application code / runtime logger | **Alternating turns:** Codex writes evidence (`Entry 1, 2, 4`); Human writes decisions (`Entry 1b, 2b, 3, 4b`) |
| **Content** | Timestamps, stack traces, stdout events | Premises, competing explanations, pre-data expectations, and claim boundaries |
| **Timing** | Emitted during or after execution | **Written before and between runs** (e.g., `Entry 3` locks the threshold and records human expectation `<60%` *before* scoring) |
| **Integrity check** | Usually append-only by convention | **Cryptographically audited:** Codex verifies SHA-256 hashes of prior entries (`preserved byte for byte`) and freezes protocol hashes (`pre-data-freeze.json`) before reading data |
| **Ownership line** | None | Explicit field: `Mechanical P6 output, not my judgment` vs. `Research judgment owner: User` |

---

## 5. Opening Brief Template (Start a New Session)

Paste this at the start of a fresh session to set the rules for all rounds:

```text
Goal: [One-sentence research or engineering question].
Context: [Project path] is read-only. Write all new session files to [session folder].

Rules for every round:
1. Do not assume my intuition is right, or that a diagnostic check will succeed or fail.
2. Tag every claim as Source states / Project choice / Not established, with exact file or page locations.
3. Anything not read, not verified, or not executed in this round must be marked NOT RUN. Never invent results.
4. Keep an append-only judgment-log.md: you write the evidence entry (Entry N), then stop so I can write my judgment entry (Entry Nb). Preserve all prior entries byte-for-byte.
5. If a round requires code, use a standalone environment, pin dependencies in uv.lock, and stop on any environment failure rather than changing the protocol.

Wait for Round 1 (SCQA prompt).
```

---

## 6. Six Guardrails for an Agent-Run Check

When you reach Round 3 and ask Codex to execute an evaluation script:

1. **Freeze the rule first:** Hash the protocol (`pre-data-freeze.json`) before reading any evaluation data.
2. **Write down your expectation:** Log what result you expect (`Entry 3`) before running the script.
3. **Test truth isolation:** Run automated preflight tests proving the scoring function cannot read ground-truth columns.
4. **Verify saved inputs before trusting them:** In our session, Codex rebuilt all 960 prediction grids (1,001 rows each) from saved parameters and checked that the max difference was `0.0 m` before scoring a single residual.
5. **Score blind, join labels later:** Freeze unlabeled per-run scores (`scoring-freeze.json`) before joining scenario labels (`unforced` vs. `forced`).
6. **Separate mechanical triggers from human claims:** Let the script trigger mechanical rules (`Priority 2`), then write your own bounded conclusion (`Entry 4b`).

---

## 7. Research Habit → Engineering Twin

You do not need to work on differential equations to use this workflow. Every habit maps directly to production engineering:

| Research Habit in This Demo | Production Engineering Twin |
|---|---|
| Ask for the assumption, not a summary | Ask what a design doc or RFC silently assumes about traffic or state |
| Tag `Source states` vs. `Project choice` | Separate vendor/spec guarantees from local config defaults |
| Forbid ground-truth peeking in Round 2 | Debug an incident using only production telemetry, not replay hindsight |
| Freeze the decision rule before scoring | Lock A/B test acceptance criteria and sample size before launch |
| Score blind, join labels later | Keep holdout evaluation isolated from prompt/model tuning |
| An alarm flags incompatibility, not the cause | A paging alert tells you an invariant broke, not the root cause |

---

## 8. Attribution & Origins

There are 20 critical questions into five sets:
1. *Is it worth reading?* (core problem, why it matters, real novelty without buzzwords)
2. *What did the author quietly assume?* (core premise, what collapses the claim, rival explanations)
3. *Does the design hold up the conclusion?* (bias, sample size, variable measurement)
4. *Don't be fooled by a pretty p-value* (practical significance, fragile cells, where it fails)
5. *Grow your next study from it* (other settings, unresolved issues, next three questions)

We compressed those 20 paper-reading questions into the **3-Question source-to-code loop**, paired them with **SCQA** (for agent delegation) and **PREP** (for human judgment logging), and applied them to an executable engineering workspace.

Two rules for the prompt: review critically rather than only summarizing, and cite the evidence for every judgment without presenting speculation as fact.
