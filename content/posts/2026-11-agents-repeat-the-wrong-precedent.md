---
title: "Coding agents repeat the wrong precedent. Here is what stops them"
date: 2026-11-18
tags: [ai-agents, software-architecture, experiments]
summary: "Four pre-registered studies, 588 agent runs: agents copy the wrong precedent near a change, the project tests do not stop it, a test for the rule does, and one sentence at the old wrong code prevents most violations."
---

*A short report on Studies A to D of the [agent-rule-compliance](https://github.com/mohaqeq/agent-rule-compliance) project. It continues [the first post](/posts/2026-11-agents-already-read-the-code/).*

A project has rules that its tests do not always examine. An architecture decision says "send webhooks through this
function". A security rule says "this record stays restricted after the customer is deleted". When does an AI coding
agent break such a rule, and what prevents it?

I did four studies with 588 agent runs on a real codebase. The short answer:

1. **If the code near the change breaks the rule, the agent breaks it again, in almost every run.**
2. **The tests that the project has do not prevent this. One test written for the rule prevents it each time.**
3. **A static check finds only one violation in four. But it finds the old wrong code before the change.**
4. **One sentence at that old wrong code ("this breaks a rule, do not copy it") prevents most violations.**

## The setup

- **Codebase:** saleor, a large open-source shop system with written architecture decisions.
- **Rules:** six rules from those decisions. Three name a code form (for example "use `pk`, not `id`"). Three are
  about behaviour (for example "a gift card balance change must be atomic").
- **Agents:** Claude Haiku and Claude Sonnet, with no project documentation. They could read all the code.
- **Grading:** hidden tests. Each grader was proven before the runs: it fails on the base, it passes on a correct
  solution, and it finds a solution that does the task but breaks the rule.
- The hypotheses, the tasks and the analysis of each study were published before its first run.

## Study A: what causes the violation

21 tickets. Each rule has a ticket at a site where the near code follows the rule, has no example, or breaks the
rule. 210 runs.

| Code near the change | Runs that broke the rule, Haiku | Runs that broke the rule, Sonnet |
|---|---|---|
| Follows the rule | 17% | 17% |
| No example | 23% | 13% |
| Breaks the rule | 87% | 97% |

- **A wrong precedent is copied.** The difference is 70 points for Haiku and 80 points for Sonnet.
- **A stronger model does not help.**
- **A missing precedent is not the problem.** The risk is a wrong one.
- Among runs that did the task correctly, 0 of 117 broke a rule at sites without a wrong precedent. At sites with
  a wrong precedent, 37 of 42 broke it.

## Study B: what prevents it

The six tickets with a wrong precedent. Each agent got nothing, the project tests to run, the rule text, or a test
written for the rule. 234 runs.

| What the agent gets | Haiku | Sonnet |
|---|---|---|
| Nothing | 87% | not tested |
| The project tests to run | 83% | 83% |
| The rule text from the project documentation | 20% | 22% |
| A test written for the rule | 0% | 0% |

- **The project tests prevent almost nothing.** They pass, because they do not examine the rule at the new place.
  The agent takes the pass as confirmation.
- **The rule text helps much, but not always.** It failed most where the near code was a strong opposite example.
- **A rule test prevented all violations, and it was not more expensive.** The agents read the failure and
  corrected the code.

## Study D: can a general static check find the violations?

A fresh agent wrote one static check for each rule. It saw only the rule texts and the codebase, not the tickets
or the results. I ran the checks one time on all 444 runs of Studies A and B.

- **On the changed lines, the checks found 26% of the 156 violations.** One rule had 100%. One had 0%.
- **39% of the violations come from old code in the touched file.** The agent did not write the forbidden form. It
  used a value from an old line that contains it. A check of the changed lines cannot see this.
- **35% have no static form.** "The card stays restricted" is not a code pattern. Only a test found these.
- **But the same checks find the wrong precedent before the change**, for four of the six rules.

## Study C: tell the agent about the wrong precedent

The six tickets again. Each agent got nothing, a two-line comment above each piece of old wrong code, or the same
text in the ticket with a list of the places. The comment says: "the code below breaks a project rule and stays
only until it is repaired. Do not copy it", and the rule in one sentence. 144 runs.

| What the agent gets | Haiku | Sonnet |
|---|---|---|
| Nothing | 87% | 94% |
| A comment at the old wrong code | 20% | 18% |
| The same text in the ticket, with the places | 7% | 33% |

- **The two measures are as effective as the full rule text**, and a tool can make them.
- **For the four rules where the static check found the places**, both models had 0 to 2 violations in each group
  with a measure. Without it, they broke the rule in every run.
- **With the text in the ticket, Haiku also repaired the old code in all 20 such runs.** The precedent is then gone
  for the next change too. The change was larger, and a team must expect that.
- **The measures fail when the agent uses the old code without a copy of its form.** The comment said "do not copy",
  and the agent did not copy. It used a variable from the marked line.

## What this means in practice

- An agent treats the near code as the specification. Each old violation is an instruction to the next agent.
- "The agent ran the tests and they passed" is not evidence that the change obeys the rules of the project.
- The order of measures that these studies support:
  1. **Make the important rules executable.** A test for the rule was the only measure with 0%.
  2. **Scan for old violations and mark or repair them.** This is cheap, a tool can do most of it, and it removes
     most of the risk.
  3. **Document the rule.** The text helps, but less than the two measures above.
- Do not expect a stronger model to solve this. In these studies it did not.

## Limits

- One codebase, one language and six rules. The intervals are over rules, and six is a small number.
- The models are from one supplier, and the author of the static checks also.
- Each rule test was written with knowledge of the ticket. A real project must write it before the change.
- In Study C the places of two rules were given by hand.
- The Sonnet parts are small: 3 runs for each ticket and group.
- An error in my test tooling made the first 36 Sonnet runs with a test command in Study B not valid. I did them
  again. This is not in the published plan. The report describes it, and the first attempts are public.

## Next

All four studies looked at one change at a time. The next question is about time: does one violation make the next
one more probable through a chain of changes, and do the measures above stop that?

The data, the graders, the checks, the analysis and the tool-call logs of all runs are public:
[github.com/mohaqeq/agent-rule-compliance](https://github.com/mohaqeq/agent-rule-compliance).
The full reports are in `research/docs`, documents 12 to 15.

*The task authors, the coding agents, the author of the static checks and most reviewers in these studies were
Claude models (Haiku, Sonnet, Opus, Fable).*
