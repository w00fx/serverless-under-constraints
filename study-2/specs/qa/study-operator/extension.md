---
schema_version: 1
actor: study-operator
area: extension
crosses: [CAP-HKP]
tags: [aws, release]
status: ratified            # draft → ratified when a human merges the PR
owner: Raphael Moura
---

# QA: extension — study operator

## Setup

1. Start from a clean checkout of a commit that has the extension revision of the HKP spec.
2. Make sure that the public repository has a witness record for the digests of this commit.
3. Select a load probe that passed and one verified variant validation for each variant of the extension run.

## Expectations

### QA-OPS2-011 — The operator can do each extension run from start to end

From a clean checkout and the selected qualifications, the operator can do each of the six extension runs and get its summary. The CLI tells the operator which runs come before `REPEAT_RUN`.

### QA-OPS2-012 — The summary tells at which arrival rate each variant stops to serve the sale

From the extension results file alone, the operator can read the break rate of each variant. The operator can read which service condition is false first, and the limits of the service conditions.

### QA-OPS2-013 — The extension does not change the results of the canonical run

After the derivation of the extension results file, the study results file has the same digest as before. The operator can read the result of the replay of the canonical run.

### QA-OPS2-014 — The operator can see what a fault stop did to each variant

From the summary of `STOP_RUN`, the operator can read the lost units, the three groups of fault buyers and each rule result of each variant.

### QA-OPS2-015 — The operator knows which part of the extension is complete

The operator can read from one document which extension runs are complete and which are not.

## Outcomes

- AC-HKP-065, AC-HKP-067, AC-HKP-068, AC-HKP-072, AC-HKP-073, AC-HKP-074 and AC-HKP-075.
- OR-HKP-012, OR-HKP-013, OR-HKP-014 and OR-HKP-017.

## Charters

- Examine a rate series with a result that is not monotonic to find an output that hides the label.
- Examine a `REPEAT_RUN` manifest that a person changed to find a trial that the rule does not give.
- Examine the extension results file to find a number of the canonical run that is different from the study results file.
- Examine a `STOP_RUN` summary to find a result that names a service as the cause of a lost unit.

## Out of scope

This area does not judge if a trial verdict or a measure is correct. The acceptance criteria of the HKP spec verify the oracle. This area does not judge articles or posts about the study.

## Report

For each expectation, the report gives the verdict (`pass`, `fail` or `blocked`), the steps that the tester did, and the result that the tester saw. For each charter, the report gives the tests that the tester tried. For each problem, the report gives the result, the correct result, and the steps that make the problem occur again.
