# How to write an LLM-as-a-Judge

An LLM judge is a second model that grades the agent on something code cannot check. Done well, it turns a vague worry ("does it make things up?") into a number you can put on a release gate.

## Why the PM should steer the judge

The judge defines what "good" means for the user. That is a product decision. Engineers make the judge run. The PM decides four things:

1. **Which failure to judge.** Pick the one that hurts users most, based on counts from real traces.
2. **Where Pass ends and Fail begins.** Write it in one sentence each, in the user's terms.
3. **When the number is trustworthy.** No judge score reaches a dashboard or a gate before validation.
4. **What happens on Fail.** Decide in advance whether a Fail blocks a release, alerts someone, or only feeds the weekly review.

## Before you write one

1. **Read traces first.** Write the judge for a failure you have seen, not one you imagine.
2. **Try code first.** Format, required fields, quotes and forbidden values belong in code. Code is free, instant and consistent.
3. **Fix the prompt first.** If the agent's prompt never asked for the behavior, add the instruction. The judge then guards against regressions.

## The four parts of a judge prompt

| Part | Rule |
|---|---|
| 1. One criterion | One failure mode per judge. Never "is this good?" |
| 2. Pass and Fail definitions | Binary only, taken from your failure notes. No 1 to 5 scales. If you need severity, write two binary judges. |
| 3. Two to four examples | A clear Pass, a clear Fail and a borderline case. Take them from the training labels only. |
| 4. Structured output | The critique comes first, then the verdict. Writing the reasons first makes the verdict more careful. |

## Give the judge only what it needs

| Failure mode | The judge needs |
|---|---|
| Invented facts | The source text and the agent's output |
| Tone mismatch | The user profile and the message |
| Instruction following | The rules and the response |
| Tool misuse | The conversation, the tool call and its result |

For long sources, pass the relevant excerpt, not the whole document.

## Validate before you trust it

1. Label at least 20 Pass and 20 Fail cases by hand.
2. Split the labels: about 15% train (examples for the prompt), 40% dev (to improve the prompt), 45% test (to report).
3. On the test split, measure the **true positive rate** (real failures the judge catches) and the **true negative rate** (good outputs it lets through). Aim for about 0.9 on both.
4. Report the corrected rate, not the raw one: `true pass rate ≈ (observed pass rate + TNR − 1) / (TPR + TNR − 1)`.
5. Validate again whenever the agent, the model or the judge prompt changes.

Start with the most capable model as the judge. Move to a cheaper one only after it shows the same agreement.

## Design the agent so it can be judged

The easiest agent to evaluate is one that shows its work. Five choices in the agent's own prompt make judging cheaper and more reliable:

- **Ask for evidence.** Each value comes with an exact quote and the line it came from. Code then verifies the quote, and the judge only decides if the quote supports the value.
- **Prefer no answer to a guess.** A missing value is cheap. A wrong value in the system of record is expensive.
- **Give every output a status**, such as proposed, no evidence or contradictory. A status can be checked by code.
- **Treat input as data, not instructions.** Add a test case where someone in the conversation asks the agent to record something it must not.
- **Keep a person in control.** The agent proposes, and a person approves. Evals reduce how often the person corrects the agent. They do not replace that person.

## Anti-patterns

- A vague criterion, like "is this helpful?"
- One judge that grades everything at once. Its verdicts tell you nothing to fix.
- No examples, so the judge guesses what counts as a failure.
- Dev or test cases used as examples. That leaks the answers.
- Scales from 1 to 5. Nobody agrees on the difference between a 3 and a 4.
- Trusting the score before validation.

## PM checklist

- [ ] The failure mode comes from real traces and has a count.
- [ ] Code cannot check it.
- [ ] Pass and Fail each fit in one sentence.
- [ ] There are at least 20 Pass and 20 Fail human labels.
- [ ] TPR and TNR are near 0.9 on the test split.
- [ ] The judge prompt is versioned and validated again after each change.
- [ ] The dashboard shows the corrected rate.
- [ ] Everyone knows what happens on a Fail.

See a full example: [the evidence-supported judge](../examples/evidence-supported-judge.md).
