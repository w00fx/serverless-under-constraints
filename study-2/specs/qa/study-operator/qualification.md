---
schema_version: 1
actor: study-operator
area: qualification
crosses: [CAP-HKP]
tags: [aws, smoke]
status: ratified            # draft → ratified when a human merges the PR
owner: Raphael Moura
---

# QA: qualification — study operator

## Setup

1. Start from a clean checkout of a commit in which the HKP spec is ratified.
2. Make sure that the public repository has a witness record for the digests of this commit.
3. Give an environment input for the sandbox account.

## Expectations

### QA-OPS2-001 — Admission tells why it rejects

When admission rejects an execution, the operator sees the reason and the ID of the criterion. Nothing changes in the cloud.

### QA-OPS2-002 — The load probe gives one clear verdict

After a load probe, the operator can read the probe verdict and the value of each condition of the verdict from one document.

### QA-OPS2-003 — A variant validation shows if the variant is ready

After a variant validation, the operator can read the two trial results, the validation result and the condition statement. The operator can tell if the canonical run can select this validation.

### QA-OPS2-004 — The operator knows the cost before the execution starts

Before the first mutation, the operator sees the estimated cost of the execution and its limit.

## Outcomes

- AC-HKP-029, AC-HKP-030, AC-HKP-031, AC-HKP-032, AC-HKP-043, AC-HKP-044, AC-HKP-045 and AC-HKP-052.
- OR-HKP-002, OR-HKP-004 and OR-HKP-006.

## Charters

- Examine a load probe with a lower Lambda quota to find a verdict that hides the cause.
- Examine a variant validation that stops during the `PEAK` trial to find a resource that stays in the account.
- Examine admission with a draft spec and with no witness record to find a path that makes evidence.

## Out of scope

This area does not judge if a trial verdict or a measure is correct. The acceptance criteria of the HKP spec verify the oracle.

## Report

For each expectation, the report gives the verdict (`pass`, `fail` or `blocked`), the steps that the tester did, and the result that the tester saw. For each charter, the report gives the tests that the tester tried. For each problem, the report gives the result, the correct result, and the steps that make the problem occur again.
