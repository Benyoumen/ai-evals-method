# Monitoring an agent in production

What to check, step by step, once an agent runs for real users. The loop has nine steps in three phases: learn what fails, measure it, then watch it in production.

<p align="center"><img src="https://benyoumen.github.io/Benyoumen/assets/production-loop.svg" alt="Production eval loop in nine steps. Learn: define the outcome, log traces, human error analysis, failure taxonomy. Measure: deterministic checks, calibrated judges, versioned eval suite. Watch: ship and monitor, add failures to the eval set." width="100%"></p>

## The loop, step by step

| # | Step | What to check | Done when |
|---|---|---|---|
| 1 | Define the product outcome | What the user must get, in one sentence. The metric that proves it. The failure that would cost the most. | Every check traces back to this outcome |
| 2 | Log representative traces | Inputs cover real users, segments and edge cases. Each trace holds the input, output, tool calls, prompt version, model, cost and latency. Personal data is masked. | You can open any run and see why it did what it did |
| 3 | Human error analysis | A person who knows the domain reads 30 to 50 traces. Each gets Pass or Fail and one note on the first thing that went wrong. | Every trace in the sample has a note |
| 4 | Build a failure taxonomy | Group the notes into 4 to 6 named failure modes. Count each one. Rank by frequency and by cost to the user. | Each mode has a definition, an example and a count |
| 5 | Deterministic checks first | Schema, required fields, forbidden values, valid tool arguments, quotes that exist in the source. | Every objective rule runs in code, on every run |
| 6 | Calibrate judges where needed | One Pass/Fail judge per mode that code cannot check. Measure its agreement with human labels on cases it never saw. | It catches real failures and rarely flags good runs. [How](llm-as-a-judge.md) |
| 7 | Run a versioned eval suite | Dataset, checks, judges and prompt are all versioned. The same suite runs on every change, several times per case. | You see, case by case, what a change fixed or broke |
| 8 | Ship carefully and monitor | The release gate first. Then a staged rollout, judges on a sample of live runs, dashboards and alerts. | Alerts fire before users complain |
| 9 | Add new failures to the eval set | Each confirmed failure becomes a test case with its expected result. A new failure mode gets its own check. | The suite grows every week |

## What to watch in production

Slice every signal by prompt version, model and user segment. An average hides the segment that is failing.

| Signal | What it tells you | Alert when |
|---|---|---|
| Code check failures | The output broke a hard rule | Any failure on a path that writes data |
| Judge pass rate, per failure mode | Quality is drifting | It drops against last week or the last release |
| User feedback: thumbs-down, edits, retries | What users feel | The rate rises |
| Human override rate | Trust in the agent | People correct or reject more outputs |
| Cost per run and latency (p95) | Budget | Above the budget set in the PRD |
| Errors and timeouts | Reliability | Above the agreed threshold |
| New kinds of input | Gaps in the dataset | More inputs fall outside the known cases |
| Judge agreement with humans | The judge itself is drifting | Agreement falls after a model or prompt change |

## Cadence

| When | What |
|---|---|
| Every run | Code checks. Traces with prompt and model version. |
| Daily | Dashboards: pass rate, feedback, cost, latency, errors. |
| Weekly, 30 minutes | Read the worst runs and update the dataset (checklist below). |
| Each release | The full suite, several runs, then the [release gate](../README.md#the-release-gate). |
| Each model change | Re-run the suite and re-validate every judge before comparing numbers. |

## The weekly review

- [ ] Read the 10 worst runs: lowest scores, thumbs-down and human overrides.
- [ ] Label each one Pass or Fail and note the first thing that went wrong.
- [ ] Decide if each failure fits a known mode or is a new one. Update the counts.
- [ ] Add each confirmed failure to the dataset with its expected result.
- [ ] For a new mode, write a code check. Write a judge only if code cannot check it, and validate it.
- [ ] Read 10 judge verdicts yourself. They should still match your reading.
- [ ] Record one decision: fix the prompt, fix retrieval, add a guardrail, or accept the risk.

## Warning signs

- **Pass rate stays at 100% for weeks.** The dataset or the judge stopped finding failures. Read fresh traces.
- **The dashboard looks fine but users complain.** The checks measure the wrong thing. Go back to step 1.
- **The numbers jump after a model or judge change.** Re-validate the judge before you compare.
- **Only one overall score.** Split it by failure mode. Each mode needs its own fix.
