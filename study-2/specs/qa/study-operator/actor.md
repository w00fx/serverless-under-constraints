---
schema_version: 1
actor_code: OPS2
actor: study-operator
status: draft            # draft → ratified when a human merges the PR
owner: Raphael Moura
---

# QA actor: study operator

**[PROPOSED]** The writer made this file from the HKP spec. The operator must accept, change or remove each expectation and each charter before the merge.

## Test as

The person who operates Study 2 from a clean checkout of committed source. This person does the load probe, validates each variant, does the canonical run, reads the results and closes each execution.

This person knows the command line and AWS. This person does not read the source of the study to understand a result.

## Interfaces

- The operator CLI of CTR-HKP-012.
- The files that the CLI writes into a package: CTR-HKP-007, CTR-HKP-008, CTR-HKP-009 and CTR-HKP-010.
- The study results file of CTR-HKP-011.
- Read-only access to the AWS console of the account of the environment input.
- The rehearsal tool of BR-HKP-066, only for development. Its packages are never evidence.

## Environment

The account and the Region that the HKP spec admits (BR-HKP-051 and OR-HKP-006), with the service quotas of OR-HKP-004. No other owner has the lease for the study, the account and the Region (BR-HKP-055).

## Human-only steps

- Make the coordination resource one time. An execution does not own it (BR-HKP-059).
- Give the AWS credentials of the account. An agent must not hold them.
- Activate the cost allocation tags and the billing data export in the account (BR-HKP-049).
- Push the witness record to the public repository (BR-HKP-048).
- Record the service-team review, or `none`, in the environment input (BR-HKP-051).
