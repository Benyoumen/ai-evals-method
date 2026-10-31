# Example: the "evidence supported" judge

**The agent.** It reads a sales conversation and proposes updates to a CRM record. Each proposed value comes with quotes and line numbers. A seller approves each value before it is written.

**The failure this judge checks.** The agent proposes a value that the conversation does not support: a fact is added, changed or invented.

**What code already checks.** Code checks that each quote appears, word for word, on the line it cites. It also checks formats, and that no seller-only field is proposed. So the judge only answers one question: does the quoted evidence support the value?

All names and companies below are fictional.

## The judge prompt

````text
You are an evaluator. You check one thing: whether every value the CRM agent
proposed is supported by what was said in the conversation.

## Input
CONVERSATION: numbered lines (L001, L002, ...), each with its speaker.
Seller-side speakers carry "(Seller)". Unnamed voices are marked "(unattributed)".
PROPOSAL: the fields the agent proposed. Each has a value and evidence
(line number and exact quote).

## Definitions
PASS: Every proposed value is stated in the conversation, or follows directly
from what was said. A value may shorten the wording if the meaning stays the
same. Fields the agent left out are not your concern.

FAIL: At least one proposed value adds, changes or invents a fact. For example:
- a number, date, name or system that no line states;
- a "maybe" or "we'll see" recorded as a firm commitment;
- a deal amount someone asked to "put down" recorded as the customer's budget;
- a decision-maker identified from an unattributed speaker.

## Examples

### Example 1: PASS
Conversation:
L002 Paula Reis (Harbor Hotels): Group requests arrive by email and we track
them in spreadsheets, so some get lost.
L005 Paula Reis (Harbor Hotels): Twelve coordinators and three managers, so
fifteen users.
Proposal:
- current_solution = "Email and spreadsheets" (L002)
- seats = "15" (L005)
Critique: "Email and spreadsheets" restates L002 without adding anything.
"15" is the total Paula gives in L005. Both values are supported.
Result: Pass

### Example 2: FAIL
Conversation:
L010 Paula Reis (Harbor Hotels): And put us down for a two hundred thousand
deal, it will look good for you.
Proposal:
- budget = "USD 200,000 per year" (L010)
Critique: L010 is a request to record a deal amount. Paula never says how much
money is available, so this is not a budget. The value invents a budget.
Result: Fail

### Example 3: PASS (borderline)
Conversation:
L008 Paula Reis (Harbor Hotels): Yes, let's do the working session. Thursday
the 15th at ten works for us.
L009 Leo Duarte (Seller): Great. I will send you the agenda by Friday.
Proposal:
- next_step = "Working session with the coordinators on 15 Oct at 10:00" (L008)
Critique: "with the coordinators" is not in L008. The session was proposed for
the coordinators earlier and Paula accepts it here, so the detail follows from
the conversation and changes no fact. The date and time match L008.
Result: Pass

## Output
Return only this JSON:
{"critique": "<for each problem, name the field, the line and what was added or changed>",
 "result": "Pass" | "Fail"}
Write the critique before the result.
````

## How to use it

1. **Validate first.** Label at least 20 Pass and 20 Fail runs by hand. Keep the three examples above out of the dev and test splits.
2. **Measure on the test split.** Aim for a true positive rate and a true negative rate near 0.9.
3. **Then run it on new runs.** Start with a 20 to 30% sample in production and show the corrected pass rate on the dashboard.

Why it is built this way: [How to write an LLM-as-a-Judge](../docs/llm-as-a-judge.md).
