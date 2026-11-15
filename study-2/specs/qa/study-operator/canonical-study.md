---
schema_version: 1
actor: study-operator
area: canonical-study
crosses: [CAP-HKP]
tags: [aws, release]
status: ratified            # draft → ratified when a human merges the PR
owner: Raphael Moura
---

# QA: canonical study — study operator

## Setup

1. Select a load probe that passed.
2. Select one verified variant validation for each of the four variants.

## Expectations

### QA-OPS2-005 — The operator can do the canonical run from start to end

From a clean checkout and the selected qualifications, the operator can do the canonical run and get its summary. The CLI gives each manual step that is necessary.

### QA-OPS2-006 — The summary tells what the study found and under which conditions

From the summary alone, the operator can read each trial verdict, each measure and the condition statement. The operator can tell if the comparison is eligible, and why.

### QA-OPS2-007 — The summary names rules and measures, not services, as the subject

Each result in the summary has a business rule or a measure as its subject. Each documented limit has a row with its source, its documented value and the value that the study saw.

### QA-OPS2-008 — The operator can see how the run closed

The operator can read the cleanup result, the leak audit result, the lease result and the safety result as different values.

### QA-OPS2-009 — A different person can make the same check

The operator can give the package to a different person. That person verifies it with the documented command and gets the same package eligibility.

### QA-OPS2-010 — The public copy does not show the account

The operator can make a public copy of a package. The copy does not contain the account identifier of the environment input.

## Outcomes

- AC-HKP-033, AC-HKP-036, AC-HKP-037, AC-HKP-038, AC-HKP-039, AC-HKP-041, AC-HKP-042, AC-HKP-049 and AC-HKP-057.
- OR-HKP-005, OR-HKP-007 and OR-HKP-008.

## Charters

- Examine a run whose results disagree with the hypotheses to find an output that hides them or makes them weaker.
- Examine a summary to find a measure that it shows without the condition statement.
- Examine a run that reaches the limit of the active time to find evidence that the runner loses.
- Examine the study results file to find a variant validation that it does not list.

## Out of scope

This area does not judge if a trial verdict or a measure is correct. The acceptance criteria of the HKP spec verify the oracle. This area does not judge articles or posts about the study.

## Report

For each expectation, the report gives the verdict (`pass`, `fail` or `blocked`), the steps that the tester did, and the result that the tester saw. For each charter, the report gives the tests that the tester tried. For each problem, the report gives the result, the correct result, and the steps that make the problem occur again.
