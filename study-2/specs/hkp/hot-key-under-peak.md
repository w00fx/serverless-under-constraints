---
schema_version: 1
capability_id: CAP-HKP
status: ratified     # draft → ratified when a human merges the spec PR
owner: Raphael Moura
approved_by: Raphael Moura
approved_at: 2026-11-15
provenance: Written on 2026-11-09 from the Study 2 entry of the private backlog (design of 2026-11-06, revision of 2026-11-09) and from the method of CAP-RUA. The owner accepted the proposals of the writer on 2026-11-14. The revision of 2026-11-15 adds the prime wave, the interval of the counters and the rule for rehearsals, from the results of rehearsal load probes. The owner accepted its proposals on 2026-11-15. The revision of 2026-11-16 adds the rule for the record of each unit. It also corrects the prime wave of the load probe, the documented limits and the reader capacity, from the results of the first full rehearsal. The extension revision of 2026-11-17 adds the extension: rate scenarios, long scenarios, the variant SHARDED_UPDATE and one fault of Result 2. Its questions come from the results of the canonical run of 2026-11-17. The owner accepted its proposals on 2026-11-17.
---

# Capability: Study 2 — Evaluate ticket-sale rules under a traffic peak on one hot key

Scope of Result 1: four variants, two scenarios and one lot for each sale. The apparatus has a controlled payment provider, an independent oracle, immutable evidence, cleanup and a leak audit.

The extension revision of 2026-11-17 adds an extension of Result 1 and one fault of Result 2. The section Extension gives its rules.

## Purpose

This capability measures the effect of a traffic peak on a ticket sale that has one hot key. It compares four serverless variants that sell the same lot to the same buyers.

The capability must get one thing right. The oracle must derive each verdict and each measure only from frozen evidence of independent sources.

A variant that breaks a business rule gives a correct result of the experiment. A variant that loses sales also gives a correct result. These results are not defects of the project.

## Research Question

When buyers arrive faster than one stock item accepts writes, how does each variant keep the sale rules? What does each variant cost the business in lost sales, fairness, delay and money?

This question is one case of the question of the laboratory: in which conditions is an architecture no longer the correct choice?

## Initial Hypotheses

The owner records these hypotheses before the first cloud execution. The oracle must not use a hypothesis as an input. A result that disagrees with a hypothesis is a correct result.

| ID | Hypothesis for the `PEAK` scenario | Documented behavior that the hypothesis uses |
|---|---|---|
| H1 | All four variants obey BR-HKP-002 through BR-HKP-007. No variant sells more tickets than the lot has. | Each reservation operation is atomic for one stock item or for one script call. |
| H2 | `DIRECT_TRANSACTION` sells fewer than 500 tickets for each second and has lost sales. | A partition accepts 1,000 write units for each second. A transactional write uses 2 write units for each item. Concurrent transactions on one item cancel. |
| H3 | `DIRECT_UPDATE` sells no more than 1,000 tickets for each second. It has fewer lost sales than `DIRECT_TRANSACTION`, and it has `ERROR` answers. | A partition accepts 1,000 write units for each second. A conditional write that does not succeed also uses write capacity. |
| H4 | `QUEUE_FIRST` has no `ERROR` answers and no lost sales. Its 99th percentile of the final outcome delay is more than 10 seconds. Its fairness count is less than the lot size. | A standard queue accepts almost unlimited `SendMessage` calls. Its sequence is best effort. The consumer limit controls the rate of the stock writes. |
| H5 | `HOT_TIER` has no lost sales, the shortest sale duration and the largest fairness count. The cache does not throttle commands. | An ElastiCache Serverless cache accepts 30,000 ECPUs for each second for one slot. |
| H6 | In the `CONTROL` scenario, all four variants obey all business rules and have no lost sales. | The control rate is below each documented limit in OR-HKP-007. |

### Hypotheses of the extension

The owner records these hypotheses before the first cloud execution of the extension. They use the results of the canonical run `05b090ae` of 2026-11-17. The oracle MUST NOT use a hypothesis as an input.

| ID | Hypothesis | Result or documented behavior that the hypothesis uses |
|---|---|---|
| H7 | In the rate runs, `DIRECT_TRANSACTION` serves the trials of `RATE_50` and `RATE_100`. Its break rate is 250, and `sold` is `true` at that rate. Its `sold` condition is `false` at `RATE_2500`. | The canonical run gave a peak ticket rate of 157 to 171 and 1,734 to 1,916 tickets for this variant. |
| H8 | In the rate runs, `DIRECT_UPDATE` serves each trial through `RATE_1000`. Its break rate is 2,500. | The canonical run gave a peak ticket rate of 1,815 to 1,828 for this variant. `DL-4` gives 1,000. |
| H9 | In the rate runs, `QUEUE_FIRST` has `answered` = `true` at each rate. Its `timely` condition is `false` at `RATE_500` and at each larger rate. Its break rate is not larger than 500. | The canonical run gave a 99th percentile of the final outcome delay of 248 to 254 seconds. Its peak ticket rate was 97 to 102. |
| H10 | In the rate runs, `HOT_TIER` serves each trial. Its break rate is `none`. | In the canonical run, the three `PEAK` trials of this variant had no lost sale and a fairness count of 9,990 to 9,995. |
| H11 | In `LONG_PEAK`, `DIRECT_TRANSACTION` sells 9,000 tickets or more. Its sale duration is more than 45 seconds. Its fairness ratio is less than `0.1000`. | A peak ticket rate of 157 to 171 for 60 seconds gives 9,420 to 10,260 tickets. The fairness count of the canonical run was 381 to 448. |
| H12 | In `LONG_KEY`, the `late_ticket_rate` of `DIRECT_UPDATE` is more than 1,000. | The peak ticket rate of the canonical run. `DL-10` gives burst capacity for up to 300 seconds. |
| H13 | One or more of the four new `PEAK` trials of `QUEUE_FIRST` has a fairness ratio of less than `0.8000`. Each of these trials has no lost sale and `answered` = `true`. | The canonical run gave the fairness counts 5,300, 9,000 and 9,224. A standard queue has a best-effort sequence. |
| H14 | In each `PEAK` trial of `SHARD_RUN`, `SHARDED_UPDATE` has `sold`, `fair` and `timely` = `true`. Its `no_outcome_fraction` is smaller than the fraction of `DIRECT_UPDATE` in that run. | Ten shards get approximately 500 purchase attempts for each second for each shard. `DL-4` gives 1,000 for one item. |
| H15 | In `PEAK_STOP`, no variant breaks BR-HKP-002 through BR-HKP-005. `DIRECT_TRANSACTION`, `QUEUE_FIRST` and `HOT_TIER` obey BR-HKP-006. `DIRECT_UPDATE` and `SHARDED_UPDATE` break BR-HKP-006 and have lost units. | The reservation operation of the first three variants also records the reservation. The two other variants record it with a different call (BR-HKP-067). |

## Study Boundary

Result 1 includes:

- four variants: `DIRECT_TRANSACTION`, `DIRECT_UPDATE`, `QUEUE_FIRST` and `HOT_TIER`.
- two scenarios: `CONTROL` and `PEAK`.
- one controlled payment provider with an authoritative payment ledger.
- one load generator with an authoritative load journal.
- a load probe that is a gate before all trials.
- a prime wave before each burst to a purchase function that answers synchronously, and before the second part of the load probe.
- one variant validation for each variant.
- one canonical run of 16 trials.
- an independent oracle.
- immutable evidence packages and amendments.
- normal cleanup, emergency cleanup and a leak audit.
- safety limits and later billing amendments.

The extension revision of 2026-11-17 adds the extension. The extension includes:

- six rate scenarios, two long scenarios and the fault scenario `PEAK_STOP`.
- a fifth variant, `SHARDED_UPDATE`, and its variant validation.
- six extension runs: `RATE_RUN_A`, `RATE_RUN_B`, `DEPTH_RUN`, `REPEAT_RUN`, `SHARD_RUN` and `STOP_RUN`.
- the service conditions of each trial and the break rate of each variant.
- an extension results file with a replay of the canonical run.

In Result 2, a component stops during the peak. The extension has one fault of Result 2: a purchase function that stops (BR-HKP-077). A cache that stops is not a part of this spec (OQ-HKP-024).

## Capability Language

**Sale**: The offer of one lot of tickets to buyers during one trial.
_Avoid_: Drop, event, campaign.

**Lot**: The fixed number of units that one sale offers.
_Avoid_: Stock, inventory, batch.

**Unit**: One sellable place in a lot. A unit has no seat number.
_Avoid_: Seat, item, token.

**Stock store**: The data store of a variant that holds the available units of a sale.
_Avoid_: Inventory, counter.

**Hot key**: The one key of the stock store that all reservation operations of a sale write.
_Avoid_: Hot partition, hot shard, hot item.

**Buyer**: One synthetic customer that makes one purchase attempt in one trial.
_Avoid_: User, client, customer.

**Purchase attempt**: The one request of a buyer for one unit of a sale.
_Avoid_: Order, checkout, transaction.

**Reservation**: The temporary hold of one unit for one buyer. A reservation is open until it becomes a ticket or a released unit.
_Avoid_: Hold, lock, admission.

**Reservation operation**: The one operation on the stock store that takes one available unit for one buyer, or refuses.
_Avoid_: Decrement, stock write.

**Release**: To put the unit of a reservation back into the available units.
_Avoid_: Roll back, undo, restock.

**Hold time**: The maximum time for which a reservation can stay open.
_Avoid_: TTL, timeout.

**Sold-out memo**: A value in the memory of one execution environment that tells that the last reservation operation found no available unit.
_Avoid_: Cache, flag, circuit breaker.

**Ticket**: The durable record that one buyer owns one unit of a sale.
_Avoid_: Order, booking, sale record.

**Charge**: One successful debit of the ticket price in the payment ledger.
_Avoid_: Payment, transaction, capture.

**Refund**: One successful credit in the payment ledger that cancels one charge.
_Avoid_: Reversal, chargeback.

**Net charge**: The sum of the charges of a buyer minus the sum of the refunds of that buyer.

**Payment provider**: The controlled module of the apparatus that approves or declines each charge request and writes the payment ledger.
_Avoid_: Gateway, processor, acquirer.

**Payment ledger**: The authoritative source of the charges and refunds of one trial.
_Avoid_: Variant data, log, trace.

**Ticket ledger**: The durable table to which a variant writes each ticket. A reader of the apparatus reads it directly.
_Avoid_: Orders table, variant summary.

**Stock snapshot**: A strongly consistent read of the stock store of a variant. A reader of the apparatus takes it.
_Avoid_: Metric, counter, dashboard value.

**Outcome record**: The durable record of the final outcome of one purchase attempt that got an `ACCEPTED` answer.
_Avoid_: Response, log line.

**Answer**: The response that the load generator receives for one purchase attempt, in one answer class of BR-HKP-021.
_Avoid_: Result, status, reply.

**Final outcome**: `TICKET`, `SOLD_OUT` or `PAYMENT_DECLINED` for one purchase attempt.
_Avoid_: Answer, verdict.

**Eligible buyer**: A buyer whose charge request the payment provider approves by the decline rule.
_Avoid_: Good buyer, paying buyer.

**Fair set**: The first eligible buyers in the arrival sequence, as many as the lot size.

**Lost sale**: A unit that stays without a ticket while an eligible buyer stays without a ticket.
_Avoid_: Missed sale, failed sale, abandoned sale.

**Variant**: A complete purchase strategy that the study puts through the declared protocol.
_Avoid_: An isolated service.

**Scenario**: `CONTROL` or `PEAK`.

**Burst**: The interval in which the load generator sends the purchase attempts of one trial.
_Avoid_: Spike, wave, ramp.

**Peak**: A burst whose arrival rate is more than the documented write limit of one stock item.

**Arrival sequence**: The position of a buyer in the send schedule of a trial. The first buyer has the number 1.
_Avoid_: Order, rank, queue position.

**Load generator**: The component of the apparatus that sends the purchase attempts on a fixed schedule and records each answer.
_Avoid_: Client, test tool, bot.

**Load journal**: The authoritative record of the schedule, the send times and the answers of one trial.
_Avoid_: Access log, client log.

**Load fidelity**: The agreement between the send times in the load journal and the declared schedule.

**Front door**: The HTTP API of a variant that receives the purchase attempts.
_Avoid_: Gateway, endpoint, edge.

**Purchase function**: The function of a variant that does the purchase protocol.
_Avoid_: Handler, worker, consumer.

**Preparation**: The documented capacity settings that a team makes before a planned peak.
_Avoid_: Warm-up, pre-warm, tuning.

**Prime wave**: A set of purchase attempts that the load generator sends before a burst. It makes each execution environment open its connections and run the purchase path one time.
_Avoid_: Warm-up, ramp.

**Prime sale**: A sale of 1 unit for one prime purchase attempt. It is not the sale of a trial.

**Prime proof**: The evidence that a prime wave reached the execution environments before the burst.

**Rehearsal**: An execution in the cloud for development. It is never evidence of the study.
_Avoid_: Dry run, trial run.

**Load probe**: A real-cloud qualification of the load generator, the front door type and the payment provider before all trials.
_Avoid_: Study trial, benchmark.

**Variant validation**: An immutable validation of one variant with two trials. It cannot support a comparison between variants.
_Avoid_: Canonical run, partial comparison.

**Apparatus**: The parts of the study that are not a variant. They are the load generator, the payment provider, the readers, the runner, the oracle and the verifier.
_Avoid_: Harness, test bench.

**Runner**: The program of the apparatus that does an execution from admission through cleanup.

**Verifier**: The program of the apparatus that examines a frozen package and derives results that a package cannot contain about itself.

**Operator**: The person who starts each execution from committed source.

**Owner**: The person who approves this spec.

**Execution**: One load probe, one variant validation or one run.
_Avoid_: Job, session.

**Admission**: The examination of an execution before its first mutation. Admission accepts or rejects the execution.
_Avoid_: Preflight, validation.

**Mutation**: A change to a cloud resource or to the lease.

**Manifest**: The frozen file that declares the inputs of an execution or of a trial.

**Capture**: One complete read of the five sources of a trial at one time.
_Avoid_: Snapshot (for all sources), dump.

**Package**: The immutable set of files that holds the evidence and the results of one execution.

**Amendment**: An immutable addition to a package. It never changes the files of the package.

**Equality projection**: The set of declared inputs that must be the same for all variants.

**Run attempt**: The admission cycle that starts with a proposed run identifier and ends with a rejection or with a run.

**Run**: One runner execution against one immutable run manifest of 16 trials.

**Trial**: One isolated execution of one variant in one scenario.

**Cell**: The set of trials that have the same variant and the same scenario.

**Oracle**: The independent evaluator that derives rule results, verdicts and measures from frozen evidence.
_Avoid_: Dashboard, alarm, variant summary.

**Rule result**: `pass`, `fail` or `indeterminate` for one business rule in one trial.

**Trial verdict**: `pass`, `fail` or `indeterminate` for one trial.

**Trial validity**: `valid`, `invalid` or `indeterminate`. It tells if the evidence of a trial can support a verdict.

**Measure**: A number that the oracle calculates for one trial and that is not a verdict.
_Avoid_: Metric, KPI, score.

**Settlement**: The proven condition in which the sale data of a trial no longer changes.
_Avoid_: An empty queue counter, a function return.

**Late evidence**: Related evidence that appears after the evidence freeze of a result. It never changes the frozen result.

**Package eligibility**: The judgment of a verifier that one package and its selected amendments are complete and unchanged.
_Avoid_: Verdict, comparison eligibility.

**Documented limit**: A capacity number that an AWS document gives and that OR-HKP-007 cites.
_Avoid_: Ceiling, quota (as a general word).

**Condition statement**: The set of declared conditions under which the study got a result.
_Avoid_: Disclaimer, caveat.

**Witness record**: A file in a public repository that holds the digests of this spec and of the oracle before the first cloud evidence. It shows no content of the study.
_Avoid_: Witness tag, pre-registration.

**Freeze**: To make a file permanent and to record its digest.

**Reserve**: To take one available unit for one buyer with one reservation operation.

**Throttle**: To refuse a request because the load is more than a capacity limit of a service.

**Derive**: To get a result from frozen evidence with the fixed rules of this spec.

**Evaluate**: To calculate the result of a rule or of a gate for one trial.

**Extension**: The set of executions that the extension revision of 2026-11-17 adds after the canonical run.
_Avoid_: Phase 2, follow-up.

**Extension run**: One of the six runs of the extension. It is not the canonical run.

**Rate scenario**: A scenario that has the form of `PEAK` at a smaller arrival rate.
_Avoid_: Step, load level.

**Rate series**: The six rate scenarios and `PEAK`, in the sequence of their arrival rates.
_Avoid_: Sweep, ramp.

**Long scenario**: A scenario with a burst of 60 seconds.

**Service condition**: One of four limits on the measures of a trial: `sold`, `fair`, `answered` and `timely`.
_Avoid_: SLO, threshold, pass mark.

**Serve**: To have all four service conditions true in one trial.
_Avoid_: Hold, pass, survive.

**Break rate**: The smallest arrival rate of the rate series at which a variant does not serve the trial.
_Avoid_: Knee, capacity, limit.

**Shard**: One of the stock items of a sale of `SHARDED_UPDATE`.
_Avoid_: Bucket, partition.

**Fault buyer**: A buyer of a `PEAK_STOP` trial that the fault rule of BR-HKP-077 selects.

**Fault stop**: The end of the process of a purchase function at the fault point of BR-HKP-077.
_Avoid_: Crash, kill, chaos.

**Lost unit**: A unit that is not an available unit, not a ticket and not in an open reservation at settlement (BR-HKP-006).

**Replay**: An evaluation of a frozen package with a later oracle. It changes no file of the package.

## Sale Domain

### BR-HKP-001 — Trial sale composition

Each trial MUST have only:

- one sale with one lot.
- one empty payment ledger for that trial.
- one empty ticket ledger partition for that trial.
- the buyers of its scenario, each with one purchase attempt for one unit.

The ticket price MUST be the same for all buyers. The lot size and the ticket price MUST agree with OR-HKP-001.

The payment ledger MUST contain only immutable charges and refunds. Declined charge requests and rejected calls MUST go to journals and not to the payment ledger.

Seat selection, more than one unit for each purchase attempt, price changes, waiting lists and transfers of tickets are not part of this study.

## Business Rules

The protocol of Study 2 gives these rules. They do not come from an external regulation. The oracle evaluates each rule at settlement.

In the formulas, `S` is the sale of the trial and `L` is its lot size. `P` is the ticket price, and `b` is one buyer of the trial.

### BR-HKP-002 — No oversell

WHEN a trial is settled, the ticket ledger MUST contain no more tickets for the sale than the lot size.

```text
count(tickets where sale_id = S) <= L
```

### BR-HKP-003 — A charged buyer has a ticket

WHEN a trial is settled, the net charge of each buyer MUST NOT be more than the price of the tickets of that buyer.

```text
for each b:  net_charge_minor(b) <= P * count(tickets of b)
```

A buyer with a net charge and no ticket breaks this rule.

### BR-HKP-004 — A ticket holder paid

WHEN a trial is settled, the net charge of each buyer MUST NOT be less than the price of the tickets of that buyer.

```text
for each b:  net_charge_minor(b) >= P * count(tickets of b)
```

A buyer with a ticket and no net charge breaks this rule.

### BR-HKP-005 — One ticket for each buyer

WHEN a trial is settled, each buyer MUST have no more than one ticket for the sale.

```text
for each b:  count(tickets of b) <= 1
```

### BR-HKP-006 — Reservation conservation

WHEN a trial is settled, each unit of the lot MUST be a ticket or an available unit. No reservation MUST stay open.

```text
available_units + open_reservations + count(tickets where sale_id = S) = L
open_reservations = 0
```

The oracle MUST read `available_units` and `open_reservations` from the last stock snapshot before the evidence freeze. The oracle MUST count the tickets in the ticket ledger.

A unit that is in no part of the first formula is a lost unit. A lost unit breaks this rule.

### BR-HKP-007 — Truthful final outcome

WHEN a trial is settled, the final outcome that each buyer received MUST agree with the ledgers.

| Final outcome that the buyer received | Tickets of the buyer | Net charge of the buyer |
|---|---:|---:|
| `TICKET` | 1 | `P` |
| `SOLD_OUT` | 0 | 0 |
| `PAYMENT_DECLINED` | 0 | 0 |

The final outcome of a buyer is the answer, if the answer is `TICKET`, `SOLD_OUT` or `PAYMENT_DECLINED`. For an `ACCEPTED` answer, the final outcome is the outcome record of the purchase attempt.

A buyer with an `ACCEPTED` answer and no outcome record at settlement breaks this rule.

A buyer with an `ERROR` or `NO_ANSWER` answer received no final outcome. This rule does not evaluate that buyer. BR-HKP-010 counts that buyer.

## Measures

A measure is not a verdict. The oracle MUST calculate each measure only for a trial whose trial validity is `valid`. For other trials, the oracle MUST give `null`.

### BR-HKP-008 — Lost sales

The oracle MUST calculate the lost sales of a trial with this formula.

```text
unsold_units           = max(0, L - count(tickets where sale_id = S))
denied_eligible_buyers = count(eligible buyers with no ticket)
lost_sales             = min(unsold_units, denied_eligible_buyers)
```

The oracle MUST find the eligible buyers from the load journal and the decline rule of BR-HKP-025. The oracle MUST NOT use the answer of a buyer in this formula.

### BR-HKP-009 — Fairness

The oracle MUST calculate the fairness count of a trial with this formula.

```text
fair_set        = the first min(L, count(eligible buyers)) eligible buyers by arrival_sequence
fairness_count  = count(buyers in fair_set that have a ticket)
fairness_ratio  = fairness_count / count(fair_set)
```

The oracle MUST give the ratio as a decimal string with four decimal places. The oracle MUST round the ratio half up.

### BR-HKP-010 — Answer exposure

The oracle MUST count the buyers of a trial for each answer class of BR-HKP-021.

The oracle MUST also count the buyers of the fair set for each answer class.

The oracle MUST count the tickets whose buyer has an `ERROR` answer or a `NO_ANSWER` answer. This count has the name `unannounced_tickets`.

### BR-HKP-011 — Delay

The oracle MUST calculate the answer latency of each buyer that received an answer.

```text
answer_latency_ms = answer_received_offset_ms - sent_offset_ms     # one clock: the load generator
```

The oracle MUST calculate the final outcome delay of each buyer that received a final outcome.

```text
answer TICKET, SOLD_OUT or PAYMENT_DECLINED:
  final_outcome_delay_ms = answer_latency_ms
answer ACCEPTED with an outcome record:
  final_outcome_delay_ms = outcome_record.decided_at - (burst_start_at + sent_offset_ms)    # two clocks
```

For each of the two values, the oracle MUST give the 50th percentile, the 99th percentile and the maximum. The oracle MUST use the nearest-rank method for a percentile.

A final outcome delay from an outcome record uses two clocks. Each result that contains it MUST have the label `cross_source_clock_assumption`.

### BR-HKP-012 — Sale duration and ticket rate

The oracle MUST calculate the sale duration and the maximum ticket rate of a trial.

```text
sale_duration_ms  = max(ticket.issued_at) - burst_start_at      # two different clocks
peak_ticket_rate  = the maximum count of tickets whose issued_at is in one interval of 1 second
```

The intervals of 1 second start at the smallest `issued_at` of the trial and do not overlap.

Each result that contains the sale duration MUST have the label `cross_source_clock_assumption`.

### BR-HKP-013 — Estimated usage cost for each ticket

The oracle MUST calculate an estimated usage cost for each trial from usage counters and from the frozen price inputs of the manifest. The price inputs MUST agree with OR-HKP-009. BR-HKP-065 gives the interval of the usage counters.

```text
request_cost_usd     = sum(usage_quantity(line) * unit_price_usd(line)) for the request-driven lines of the variant
cost_per_ticket_usd  = request_cost_usd / count(tickets where sale_id = S)
held_capacity_usd_per_hour = sum(unit_price_usd(line) * held_quantity(line)) for the held-capacity lines of the variant
```

A request-driven line is usage that changes with the number of purchase attempts. A held-capacity line is capacity that the variant pays for while it is idle.

The oracle MUST give `null` for `cost_per_ticket_usd` when the trial has no ticket. The oracle MUST give each amount as a decimal string with six decimal places.

Each result MUST show this value with the label `estimate`. The billing amendment of BR-HKP-057 is a different value, and the estimate MUST NOT replace it.

## Oracle Rules

### BR-HKP-014 — Independent sources

WHEN the oracle evaluates a business rule or a measure, it MUST use only these frozen sources of the trial:

| Source | Owner | What the oracle reads from it |
|---|---|---|
| Load journal | Load generator | Buyers, arrival sequence, send times, answers |
| Payment ledger | Payment provider | Charges and refunds |
| Ticket ledger | Variant, read by the apparatus | Tickets |
| Stock snapshot | Variant, read by the apparatus | Available units and open reservations |
| Outcome records | `QUEUE_FIRST`, read by the apparatus | Final outcomes and their times |

The apparatus MUST read the payment ledger, the ticket ledger and the outcome records with complete, strongly consistent reads.

For a variant that answers synchronously, the source of the outcome records is `not_applicable`.

Logs, metrics, traces and summaries of a variant can explain a result. They MUST NOT replace a source of this table.

The estimated usage cost of BR-HKP-013 is the one exception. It also uses the usage counters of OBS-HKP-001.

### BR-HKP-015 — Rule results and trial verdict

WHEN the oracle evaluates a trial, it MUST give one rule result for each of BR-HKP-002 through BR-HKP-007.

A rule result MUST be `indeterminate` when the evidence for that rule is not complete.

The oracle MUST derive the trial verdict with this sequence.

```text
if trial_validity != valid:               indeterminate
else if one rule result = fail:           fail
else if one rule result = indeterminate:  indeterminate
else:                                     pass
```

The runner MUST keep each `indeterminate` result. The runner MUST NOT do a trial again to replace a result.

### BR-HKP-016 — Trial validity

The oracle MUST derive the trial validity from these gates:

- independent sources (BR-HKP-014).
- traceability (BR-HKP-017).
- physical identity integrity (INV-HKP-001).
- load fidelity (BR-HKP-022).
- apparatus integrity (BR-HKP-023).
- settlement (BR-HKP-041).
- evidence integrity (BR-HKP-052).

Each gate MUST have one of these values.

```text
verified | invalid | unverified | not_applicable
```

The oracle MUST derive the trial validity with this sequence.

```text
if one gate = invalid:          invalid
else if one gate = unverified:  indeterminate
else:                           valid
```

| Situation | Trial validity | Trial verdict |
|---|---|---|
| All gates `verified`, all rules `pass` | `valid` | `pass` |
| All gates `verified`, 10,001 tickets for a lot of 10,000 | `valid` | `fail` |
| All gates `verified`, one reservation open at settlement | `valid` | `fail` |
| Load fidelity below the limit of OR-HKP-002 | `invalid` | `indeterminate` |
| DynamoDB throttled a request of the payment provider | `invalid` | `indeterminate` |
| The ticket ledger read is not complete | `indeterminate` | `indeterminate` |
| No settlement before the observation deadline | `indeterminate` | `indeterminate` |
| A proven oversell with optional telemetry missing | `valid` | `fail` |
| An incorrect manifest that admission rejects | no oracle result | no oracle result |

### BR-HKP-017 — Traceability

WHEN the oracle evaluates a trial, each record of the five sources MUST contain the `run_id`, the `trial_id` and the `sale_id` of that trial.

Each ticket, charge, refund and outcome record MUST contain the `buyer_id` of one buyer of the load journal.

Each reference to a manifest MUST agree with the frozen digest of that manifest.

A record that the oracle cannot relate to the trial makes the traceability gate `invalid`.

## Integrity Rule

### INV-HKP-001 — Physical identity integrity

In one trial, each of these identifiers MUST be unique:

- `buyer_id`.
- `ticket_id`.
- the identifier of each payment ledger record.
- the identifier of each outcome record.

A record of a different trial in a source of the trial makes this gate `invalid`.

Two tickets with different identifiers for one buyer do not break this rule. BR-HKP-005 evaluates them.

## Scenarios and Load

### BR-HKP-018 — Scenarios

The canonical run and each variant validation MUST have two scenarios and no more. BR-HKP-069, BR-HKP-070 and BR-HKP-077 give the scenarios of the extension.

| Scenario | Purpose | Arrival rate compared with the documented limits |
|---|---|---|
| `CONTROL` | To show that each variant and the apparatus operate correctly without a peak | Below each documented limit of OR-HKP-007 |
| `PEAK` | To put the hot key under a traffic peak | More than the documented write limit of one stock item |

The lot size, the number of buyers, the arrival rate and the burst duration of each scenario MUST agree with OR-HKP-001 and OR-HKP-002.

### BR-HKP-019 — Arrival schedule

The load generator MUST send the purchase attempts of a trial on an open-loop schedule. The send time of a purchase attempt MUST NOT depend on an earlier answer.

The schedule MUST have a constant interval between purchase attempts.

```text
scheduled_send_offset_ms(n) = (n - 1) * 1000 / arrival_rate      # n = arrival_sequence, from 1
```

The `buyer_id` of a buyer MUST contain its arrival sequence. The schedule MUST be the same for all variants in one scenario.

### BR-HKP-020 — Buyer model

Each buyer MUST make one purchase attempt and no more. The load generator MUST NOT send the purchase attempt of a buyer again.

Each buyer MUST wait for an answer until the buyer deadline of OR-HKP-002. After the buyer deadline, the load generator MUST record `NO_ANSWER` and MUST ignore a later response.

The condition statement MUST contain this buyer model.

### BR-HKP-021 — Answer classification

The load generator MUST put each purchase attempt into one answer class with this table. The first row that agrees gives the class.

| Row | Observation of the load generator | Answer class |
|---:|---|---|
| 1 | No complete response before the buyer deadline | `NO_ANSWER` |
| 2 | HTTP status 200 and a body that agrees with CTR-HKP-001 and has `answer` = `TICKET` | `TICKET` |
| 3 | HTTP status 200 and a body that agrees with CTR-HKP-001 and has `answer` = `SOLD_OUT` | `SOLD_OUT` |
| 4 | HTTP status 200 and a body that agrees with CTR-HKP-001 and has `answer` = `PAYMENT_DECLINED` | `PAYMENT_DECLINED` |
| 5 | The variant is `QUEUE_FIRST`, HTTP status 200, and the body has a queue message identifier | `ACCEPTED` |
| 6 | All other responses and all connection errors | `ERROR` |

The load journal MUST keep the HTTP status and the error kind of each `ERROR` answer.

### BR-HKP-022 — Load fidelity

WHEN the oracle evaluates a trial, the load fidelity gate MUST be `verified` only when all these conditions are true:

- the load journal has one record for each buyer of the scenario.
- the fraction of purchase attempts that the load generator sent on time is not less than the limit of OR-HKP-002.
- the load generator recorded no internal overload.

A purchase attempt is on time when its send time is not later than its scheduled time plus the lateness limit of OR-HKP-002.

An `ERROR` answer or a `NO_ANSWER` answer does not decrease the load fidelity. These answers are results of the variant.

IF the load journal shows that one condition is not true, THEN the gate MUST be `invalid`. IF the load journal is not complete, THEN the gate MUST be `unverified`.

### BR-HKP-023 — Apparatus integrity

WHEN the oracle evaluates a trial, the apparatus integrity gate MUST be `verified` only when all these conditions are true:

- DynamoDB throttled no request on the tables of the payment provider during the trial.
- the journal of the payment provider shows no internal error and no charge request longer than the maximum time of OR-HKP-003.
- each reader of the apparatus completed its reads.

IF the evidence shows a throttled request, an internal error or a charge request that was too long, THEN the gate MUST be `invalid`.

The evidence for this gate is the journal of the payment provider and the throttle counters of its tables. These counters are telemetry of the apparatus and not of a variant.

The apparatus MUST NOT be the cause of a difference between variants. This gate keeps that condition.

## Controlled Payment Provider

### BR-HKP-024 — Payment provider contract

The payment provider MUST be a module of the apparatus that operates in the purchase function. It MUST NOT be a different Lambda function, because Lambda limits a function to 10 requests for each second for each unit of concurrency.

The source of a variant MUST call the payment provider only through the interface of CTR-HKP-002. The source of a variant MUST NOT read or write the tables of the payment provider.

The payment provider MUST accept a charge request only when the request agrees with CTR-HKP-002.

WHEN the payment provider receives a correct charge request with a new `payment_request_id`, it MUST use the decline rule. For an approved request, it MUST write one charge, and only one, in one atomic operation.

WHEN the payment provider receives a charge request with a `payment_request_id` that it knows, it MUST give the first response again. It MUST NOT write a second charge.

IF a charge request has a known `payment_request_id` and different content, THEN the payment provider MUST reject it as a conflict.

WHEN the payment provider receives a correct refund request for a charge, it MUST write no more than one refund for that charge.

The payment provider MUST add no delay to its responses.

### BR-HKP-025 — Decline rule

The payment provider MUST decline a charge request when the arrival sequence of its buyer is a multiple of the decline modulus of OR-HKP-002. It MUST approve all other correct charge requests.

```text
declined(b) = (arrival_sequence(b) mod decline_modulus = 0)
eligible(b) = not declined(b)
```

The decline rule MUST be the same for all variants and all trials of one scenario.

## Variants

### BR-HKP-026 — Purchase protocol

Each variant MUST do each purchase attempt with this protocol.

1. The variant does one reservation operation for the buyer.
2. IF the reservation operation finds no available unit, THEN the final outcome is `SOLD_OUT`.
3. The variant sends one charge request for the reserved unit to the payment provider.
4. IF the payment provider approves, THEN the variant writes one ticket and closes the reservation. The final outcome is `TICKET`.
5. IF the payment provider declines, THEN the variant does a release. The final outcome is `PAYMENT_DECLINED`.
6. `QUEUE_FIRST` writes one outcome record with the final outcome. A variant that answers synchronously gives the final outcome in its response.

The `payment_request_id` MUST be a deterministic function of the `trial_id` and the `buyer_id`. The `ticket_id` MUST be a deterministic function of the `trial_id` and the `buyer_id`.

The reservation operation MUST be the first operation of a purchase attempt on the stock store. A variant MUST NOT read the stock store before it.

Each purchase function MUST keep a sold-out memo. WHEN a reservation operation finds no available unit, the execution environment MUST answer `SOLD_OUT` with no stock operation for the memo time of OR-HKP-003.

The sold-out memo MUST apply only to the sale of the reservation operation that set it.

After the memo time, the next purchase attempt in that execution environment MUST do a reservation operation. A variant MUST NOT use a different source for a `SOLD_OUT` final outcome.

### BR-HKP-027 — End of a reservation

WHEN a reservation is open at the end of its hold time, the variant MUST end it in one of two ways:

- IF the buyer has a charge and no refund, THEN the variant writes the ticket and closes the reservation.
- IF the buyer has no charge, THEN the variant does a release.

A variant can refund a charge and then do a release in place of the first way.

This rule also applies to each other record of BR-HKP-067 that names a unit.

The hold time MUST agree with OR-HKP-003 for all variants.

### BR-HKP-028 — Equal treatment

WHEN the study compares variants, all variants MUST have the same declared inputs, except for the differences that BR-HKP-031 through BR-HKP-034 and BR-HKP-076 declare.

The equality projection MUST contain at least:

- the sale fixture and the load schedule of the scenario.
- the decline rule and the payment provider.
- the buyer deadline, the hold time, the memo time and the retry policy.
- the type of the front door.
- the Region, the runtime, the memory size and the timeout of the purchase function.
- the concurrency limit of the purchase function, for the variants that answer synchronously.
- the preparation of BR-HKP-029.
- the prime wave of BR-HKP-064.

```text
comparison_eligibility = eligible
only if each equality projection passes and there is no difference that the spec does not declare
```

A rule result of `fail` does not make a comparison ineligible. A difference that the spec does not declare makes it ineligible.

### BR-HKP-029 — Preparation for a planned peak

A ticket sale is a planned peak. Each variant MUST have the documented preparation that a team makes before a planned peak.

The preparation MUST contain:

- provisioned concurrency for the purchase function, with the value of OR-HKP-004.
- warm throughput for the DynamoDB tables that OR-HKP-004 names, with the values of OR-HKP-004.
- a minimum ECPU limit for the cache of `HOT_TIER`, with the value of OR-HKP-004.
- a prime wave before each burst, with the values of OR-HKP-010.

A trial MUST NOT start before each preparation of its variant is complete. The trial manifest MUST record the proof that each preparation is complete.

Among the purchase functions of all variants, only the one of the current trial MUST have reserved concurrency and provisioned concurrency. The runner MUST set the reserved concurrency of each other purchase function to 0.

The preparation MUST NOT change the number of stock items of a sale.

### BR-HKP-030 — Retry policy

All variants MUST use the retry policy of OR-HKP-003.

The AWS SDK clients of a variant MUST use the declared retry mode and the declared maximum number of attempts.

The write that puts a unit back is an exception. The variant MUST NOT send it again after a result that it does not know (BR-HKP-067).

WHEN a reservation operation gets a transaction conflict, the variant MUST do it again with the declared conflict policy. WHEN the purchase function used all attempts of the conflict policy, the purchase attempt MUST end with an error response.

The payment provider MUST do the ledger write of a charge request one time and MUST NOT do it again automatically. A purchase function that sends a charge request again MUST use the same `payment_request_id`.

### BR-HKP-031 — Variant `DIRECT_TRANSACTION`

The front door of `DIRECT_TRANSACTION` MUST call the purchase function synchronously.

The stock store MUST be one DynamoDB item for each sale. The reservation operation MUST be one `TransactWriteItems` call that does these two writes:

- a conditional update of the stock item that decreases the available units by 1.
- a conditional put of the reservation record of the buyer.

The call MUST have a client request token that is a deterministic function of the `trial_id` and the `buyer_id`.

*Source for the pattern: AWS Database Blog, "Implement resource counters with Amazon DynamoDB", 2023-03-31, approach "Transaction with a client request token".*

### BR-HKP-032 — Variant `DIRECT_UPDATE`

The front door of `DIRECT_UPDATE` MUST call the purchase function synchronously.

The stock store MUST be one DynamoDB item for each sale. The reservation operation MUST be one conditional `UpdateItem` call that decreases the available units by 1.

The variant MUST write the reservation record of the buyer with a different call. The reservation operation and that call are not one atomic operation.

`DIRECT_UPDATE` also does a release with more than one call. BR-HKP-067 gives the rule for their sequence and for a refused write.

*Source for the pattern: Amazon DynamoDB Developer Guide, "Working with items", conditional writes. AWS Database Blog, "Implement resource counters with Amazon DynamoDB", approach "Atomic counters" with a condition.*

### BR-HKP-033 — Variant `QUEUE_FIRST`

The front door of `QUEUE_FIRST` MUST send each purchase attempt to one Amazon SQS standard queue with a service integration. No function MUST be in this path.

The front door MUST answer the buyer after the queue accepts the message. A consumer function MUST receive the messages through an event source mapping and MUST do the purchase protocol.

The stock store and the reservation operation MUST be the same as in `DIRECT_TRANSACTION`.

The event source mapping MUST have the batch size, the maximum concurrency and the partial batch response of OR-HKP-004. The queue MUST have a dead-letter queue with the receive limit of OR-HKP-004.

The consumer function MUST give the same final outcome for a message that it receives more than one time.

The consumer function MUST do the purchase attempts of one batch one after the other. One execution of the consumer MUST NOT have two reservation operations in progress at the same time. The event source mapping then controls how many reservation operations are in progress on the stock item.

The consumer function MUST NOT start a purchase attempt in the last 2 seconds before its timeout (OR-HKP-003). The consumer function MUST give the other messages of the batch back to the queue.

*The stock store has one item and no shards. The four variants of the canonical run have the same hot key.*

`QUEUE_FIRST` has no prime wave. This is a declared difference.

### BR-HKP-034 — Variant `HOT_TIER`

The front door of `HOT_TIER` MUST call the purchase function synchronously.

The stock store MUST be an Amazon ElastiCache Serverless cache with the Valkey engine. The reservation operation MUST be one call of one atomic script.

The script MUST decrease the available units by 1 and MUST record the reservation of the buyer in the same call. All keys of one sale MUST be in one hash slot.

The purchase function MUST connect to the cache through a VPC. The purchase function MUST reach DynamoDB through a gateway endpoint of that VPC.

The variant MUST write each ticket to DynamoDB before it answers `TICKET`.

The reservations of this variant are only in the cache. The condition statement MUST say that this variant has no durability for open reservations in Result 1.

## Staged Qualification

### BR-HKP-035 — Load probe before trials

The operator MUST do a load probe in the real cloud before the first variant validation.

The load probe MUST have two parts.

1. The load generator sends the `PEAK` schedule to a first probe route. That route has the front door type of the variants and a function that only answers.
2. The load generator sends purchase attempts to a second probe route at the provider rate of OR-HKP-004. The function of that route sends one charge request to the payment provider for each purchase attempt.

In the second part, the function MUST send the charge requests of OR-HKP-004 two times with the same `payment_request_id`.

The load generator MUST send a prime wave before the burst of the second part. The first part has no prime wave.

The load probe MUST do the two parts one after the other. In each part, only the function of that part MUST have the concurrency values of a purchase function that answers synchronously.

The load probe MUST NOT use a variant. A result of the load probe is not a result of the study.

### BR-HKP-036 — Probe verdict

The verifier MUST derive the probe verdict from the frozen evidence of the load probe.

```text
passed | failed | indeterminate
```

The probe verdict MUST be `passed` only when all these conditions are true:

- the load journal has one record for each scheduled purchase attempt.
- the on-time fraction is not less than the limit of OR-HKP-002.
- the fraction of HTTP status 200 answers is not less than the limit of OR-HKP-004.
- the payment provider answered each charge request correctly in the maximum time of OR-HKP-003.
- the payment ledger has one charge, and only one, for each approved `payment_request_id`.
- the payment ledger has no charge for a declined `payment_request_id`.
- the prime proof of the second part is complete.

IF the evidence shows that one condition is not true, THEN the probe verdict MUST be `failed`. IF the evidence is not complete, THEN the probe verdict MUST be `indeterminate`.

### BR-HKP-037 — Qualification binding

A variant validation or a run can select a load probe only when all these conditions are true:

- the probe verdict is `passed`.
- the package of the load probe is eligible.
- the cleanup of the load probe succeeded and its leak audit is clean.
- the account and the Region are the same.
- the load generator source, the payment provider source and the front door configuration have the digests that the load probe recorded.
- the service quotas that admission records are not lower than the quotas that the load probe recorded.

Admission MUST reject a selection that does not obey one of these conditions. A change of a source that these digests do not include MUST NOT make a new load probe necessary.

### BR-HKP-038 — Variant validation

The operator MUST do one variant validation for each variant before the canonical run.

A variant validation MUST have two trials of one variant: one `CONTROL` trial, then one `PEAK` trial. It MUST use the same protocol as a run.

The verifier MUST derive the validation result.

```text
verified | failed | indeterminate
```

The validation result MUST be `verified` only when both trials are `valid`, the `CONTROL` trial verdict is `pass`, and the operational closure is clean.

An operational closure is clean when the cleanup succeeded, the leak audit is `clean`, the lease is `released` and the runner knows no safety breach.

The verdict of the `PEAK` trial MUST NOT change the validation result.

A run can select a variant validation only when the variant source and the variant configuration have the digests that the validation recorded.

The evidence of a variant validation MUST NOT support a comparison between variants.

## Run Rules

### BR-HKP-039 — Run composition and sequence

A canonical run MUST have 16 trials in four blocks. The first block has one `CONTROL` trial for each variant. Each of the three other blocks has one `PEAK` trial for each variant.

The sequence of the trials MUST agree with OR-HKP-005. The runner MUST do the trials one at a time.

The runner MUST wait for the gap of OR-HKP-003 between the evidence freeze of a trial and the start of the next trial.

Admission MUST reject a run that has no selected load probe or that has no selected variant validation for one variant.

### BR-HKP-040 — Trial isolation

Each trial MUST have a new `trial_id`, a new `sale_id` and new buyers. Each trial MUST have its own stock item or its own stock keys.

Each record of a variant MUST contain its `trial_id`. A purchase attempt of one trial MUST NOT change the data of a different trial.

WHEN a trial starts, the queues of its variant MUST be empty and its function MUST have no active execution.

### BR-HKP-041 — Settlement

A trial is settled only when all these conditions are true:

- the load generator completed the schedule and the buyer deadline of the last buyer passed.
- the hold time passed after the buyer deadline of the last buyer.
- two complete captures of the five sources have the same digests.
- the stabilization interval of OR-HKP-003 is between the two captures.
- for `QUEUE_FIRST`, the source queue shows no visible, in-flight or delayed message in both captures.
- the apparatus recorded each related message of the dead-letter queue.

A queue counter alone MUST NOT show settlement. A change between two captures MUST start the stabilization interval again.

IF a trial is not settled at the observation deadline of OR-HKP-003, THEN the settlement gate MUST be `unverified`. Later evidence MUST NOT change that result.

An open reservation that does not change between the two captures does not prevent settlement. BR-HKP-006 evaluates it.

### BR-HKP-042 — Comparison eligibility

The comparison of a canonical run MUST be `eligible` only when all these conditions are true:

- all 16 trials have an oracle result.
- the trial validity of each trial is `valid`.
- the trial verdict of each `CONTROL` trial is `pass`.
- the equality projections of BR-HKP-028 pass.
- the evidence integrity gate is `verified`.
- the late evidence is `none` or `consistent`.
- no effective amendment contradicts a result.

A `PEAK` trial verdict of `pass` and a `PEAK` trial verdict of `fail` are both comparable results.

A cleanup problem, a leak or a safety breach MUST make the comparison ineligible only when it changed isolation, settlement or evidence.

## Result Statement

### BR-HKP-043 — Condition statement

Each run summary, each validation result and the study results file MUST contain a condition statement.

The condition statement MUST contain at least:

- the lot size, the number of buyers, the arrival rate and the burst duration of each scenario.
- the buyer model of BR-HKP-020, the buyer deadline and the memo time.
- the decline modulus.
- the concurrency limit and the provisioned concurrency of the purchase function.
- the service quotas of the account that admission recorded.
- the preparation values of each variant.
- the prime wave values of OR-HKP-010.
- the reservation operation of each variant.
- the Region.
- the service-team review that the environment input records, or `none`.

The values MUST come from the frozen manifests. A summary MUST NOT give a measure of a trial without the condition statement of its run.

### BR-HKP-044 — Comparison with documented behavior

For each variant, the run summary MUST contain one row for each documented limit of OR-HKP-007 that the variant uses.

Each row MUST contain the source, the documented value, the value that the study saw, and one of these relations.

```text
in_documented_limit | more_than_documented_limit | not_comparable
```

IF one row has `more_than_documented_limit`, THEN the run summary MUST set `documentation_difference` to `true`. Such a row is a question for the service team. It is not a result of the study.

The summary MUST name a business rule or a measure as the subject of each result. It MUST NOT name a service as the cause of a rule result without the row of this rule.

### BR-HKP-045 — Study results file

A derivation tool MUST write one study results file from the selected packages. This file is the only source of the numbers of the canonical run. BR-HKP-079 gives the file for the numbers of the extension.

Each value in the file MUST have the path of its source file in the package and the SHA-256 digest of that file.

The file MUST list each load probe and each variant validation that has a package. It MUST also list the ones that the operator did not select, with the reason.

The tool MUST refuse a package that is not eligible. The tool MUST refuse a set of packages with oracle results that do not have the same oracle digest.

BR-HKP-066 gives the number of rehearsals that this file contains.

### BR-HKP-046 — Public copy

A redaction tool MUST derive the public copy of a package. The raw package MUST NOT change.

The tool MUST replace each AWS account identifier with `111122223333`. It MUST record each changed file and the digest of each file in a redaction manifest.

IF the public copy contains an account identifier of the environment input after redaction, THEN the tool MUST stop with an error.

## Integrity of the Method

### BR-HKP-047 — Ratified spec before evidence

Admission of an execution that makes evidence MUST read this spec at the source commit. IF the `status` is not `ratified`, THEN admission MUST reject the execution.

The manifest MUST record the SHA-256 digest of the spec file.

### BR-HKP-048 — Final oracle and witness record before evidence

Each rule that can change a verdict or a measure MUST be in the oracle source before the first cloud execution that makes evidence.

One file in `study-2/` MUST list the oracle files. The oracle digest is the SHA-256 digest of that list with the SHA-256 digest of each file.

Before that execution, the operator MUST push a witness record to the public repository of the environment input. Each entry of the witness record MUST contain only these values:

- the SHA-256 digest of this spec file.
- the oracle digest.
- the date of the entry.

The witness record MUST NOT contain text of this spec or of the oracle. The operator MUST add an entry and MUST NOT change an earlier entry.

Admission MUST read the witness record from the public repository with no credential. Admission MUST calculate the two digests from the source commit.

IF the witness record has no entry with these two digests, THEN admission MUST reject the execution. IF admission cannot read the witness record, THEN admission MUST reject the execution.

The manifest MUST record the two digests and the commit of the public repository that has the entry.

### BR-HKP-049 — Billing readiness before deployment

Before the first deployment, admission MUST make sure that:

- each cost allocation tag of BR-HKP-059 is active in the account.
- a billing data export with resource identifiers is in the account.

IF admission cannot show one of these conditions, THEN admission MUST reject the execution.

### BR-HKP-050 — Source and deployment provenance

An execution that makes evidence MUST start from:

- a correct Git repository whose root for this study is `study-2/`.
- a committed `HEAD` that Git can resolve.
- a clean index and a clean worktree with no untracked file that Git does not ignore.
- no merge, rebase or cherry-pick that is not complete.
- a package lockfile that Git tracks and that has no change.

The manifest MUST record the commit, the tree, the lockfile digest, the tool versions, the oracle digest and the digest of the deployment assembly.

The runner MUST deploy only from the copy of the deployment assembly that is in the evidence package.

Tests, local emulation and synthesis can operate from a worktree that is not clean. They MUST NOT make evidence of the study. BR-HKP-066 gives the rule for a rehearsal in the cloud.

### BR-HKP-051 — Admission and manifest freeze

Admission MUST start when the runner makes a proposed identifier for the execution. Admission MUST do only read operations in the cloud.

The environment input MUST contain one 12-digit account identifier, the coordination resource, the public repository of the witness record and the service-team review. It MUST contain no credential.

Admission MUST make sure that the caller account is the account of the environment input. Admission MUST record the service quotas of OR-HKP-004.

The runner MUST freeze the execution manifest before the first mutation. The acquisition of the lease MUST be the first mutation.

IF admission rejects an execution, THEN the runner MUST record a structured rejection. It MUST make no manifest, no result and no change in the cloud.

A change of a declared field after the freeze MUST have a new execution identity.

### BR-HKP-052 — Evidence freeze, late evidence and amendments

The runner MUST freeze the evidence of a trial after settlement, the last capture and the oracle evaluation. The runner MUST freeze it before cleanup.

Late evidence MUST NOT change a frozen result or its digests. The runner MUST give late evidence one of these assessments.

```text
none | consistent | contradictory | unverified
```

The runner MUST monitor for late evidence for the interval of OR-HKP-003 after the last evidence freeze.

Evidence that arrives after the package is final MUST go into an immutable amendment. An amendment MUST refer to the first package and to the amendment before it.

A contradictory amendment MUST block the comparison until the operator selects a new immutable assessment.

The evidence integrity gate MUST be `verified` only when each indexed file has the digest that the evidence index records.

### BR-HKP-053 — Package eligibility

The verifier MUST calculate the package eligibility after the runner writes the package index. A summary inside the package MUST NOT contain it.

```text
package_eligibility: eligible | ineligible
```

The package index MUST contain the SHA-256 digest of each final file of the package, except itself. The runner MUST write it last.

The verifier MUST give the reasons for `ineligible`, the digest of the package index and the digest of the selected amendment.

### BR-HKP-054 — Serialization

Each record of a contract of this spec MUST be JSON in UTF-8 with these properties:

- each monetary amount is an integer of minor units.
- each count is an integer.
- each ratio and each cost is a decimal string.
- each time is an RFC 3339 string in UTC with milliseconds.
- each enumeration value is one of the values that this spec gives.
- each record has a `schema_version`.

A reader MUST reject a record that has one of these defects:

- a mandatory field is missing.
- a field is not in the schema.
- a value has the incorrect type.

## Operations

### BR-HKP-055 — Coordination lease

Only one owner at a time MUST have the lease for the study, the account and the Region.

```text
RUN | LOAD_PROBE | VARIANT_VALIDATION
```

The owner MUST send a heartbeat at the interval of OR-HKP-006. The lease MUST be stale after the stale limit of OR-HKP-006.

WHEN a heartbeat does not succeed, the runner MUST start no new trial. WHEN the lease is stale or has a different owner, the runner MUST stop the experiment and start emergency cleanup.

The final lease result MUST be one of these values.

```text
released | recovery_required | unverified
```

The end of the lease lifetime MUST NOT show a release.

### BR-HKP-056 — Safety limits

Admission MUST reject an execution when one of these conditions is true:

- the account is not the account of the environment input.
- the Region is not the Region of OR-HKP-006.
- a service quota is lower than its minimum in OR-HKP-004.
- the estimated cost is more than its limit in OR-HKP-006.
- a different owner has the lease.

Admission MUST calculate the estimated cost from the price inputs of OR-HKP-009 and the declared durations of the execution.

Each safety check MUST record its limit, the value that it saw, its result, its evidence and its time.

```text
within_limits | breached | unverified          # breached > unverified > within_limits
```

At the limit of the active time, the runner MUST start no new trial. The runner MUST keep the evidence that it has and MUST start emergency cleanup.

Cleanup MUST continue after the total time limit when it is not complete.

The cost limit is a limit for admission. It is not a guarantee of the bill.

### BR-HKP-057 — Attributable usage cost

A later billing import MUST accept only authoritative usage lines of the billing data export.

The import MUST relate a line to an execution only by these fields:

- the account and the resource identity.
- an active ownership tag.
- the service and the operation.
- the usage interval.

```text
billed_cost_check: within_limit | breached | unverified
```

Lines in a currency that is not USD, periods that are not complete, and shared charges MUST give `unverified`. The import MUST do no currency conversion.

The billing evidence MUST be an immutable amendment.

### BR-HKP-058 — Cleanup

Normal cleanup MUST do these steps in this sequence.

1. Complete the monitoring for late evidence and freeze its assessment.
2. Disable the load generator, the event source mappings and the front doors.
3. Record an operational snapshot.
4. Record the related messages of each dead-letter queue.
5. Erase the resources that the execution owns.
6. Audit each discovery surface.
7. Freeze the cleanup result, the summary and the package.

Emergency cleanup MUST stop the load generator and the consumers immediately. It MUST keep the available evidence before it erases a resource.

The cleanup result MUST be one of these values. A final package MUST NOT have the first two values.

```text
not_started | running | succeeded | partial | failed
```

### BR-HKP-059 — Cleanup ownership

Each resource of an execution that can have tags MUST have the tags `suc:project`, `suc:study_id`, `suc:run_id`, `suc:managed_by` and `suc:expires_at`.

Each resource of one variant MUST also have the tag `suc:variant_id`. A shared resource MUST NOT have it.

Cleanup MUST erase a resource only when the resource manifest contains it and its tags agree with the execution.

Cleanup MUST NOT erase a resource whose ownership is not clear. Cleanup MUST report that resource as inconclusive.

Cleanup MUST NOT erase the coordination resource or a bootstrap resource.

### BR-HKP-060 — Leak audit

The leak audit MUST examine these surfaces:

- the resource manifest and the stacks.
- the tag discovery surfaces.
- the list operations of each used service.
- the deterministic resource names.

```text
clean | leaks_detected | inconclusive
```

The audit result MUST be `clean` only when each query succeeds and each surface shows no resource for the interval of OR-HKP-006.

A query that does not succeed MUST make the audit result `inconclusive`.

### BR-HKP-061 — Operational and scientific independence

These dimensions are independent: the execution result, the trial verdicts, the comparison, the cleanup and the leak audit. The lease, the evidence integrity, the package eligibility and the safety are also independent dimensions.

A cleanup problem or a leak MUST NOT change a frozen trial verdict.

A run can complete its scientific cycle when a variant has a trial verdict of `fail`.

A leak that can cause a later purchase attempt, charge or ticket of a trial MUST make the settlement gate of that trial `invalid`.

### BR-HKP-062 — Study completion

Result 1 of the study is complete only when one canonical run has all these values in its first immutable package.

```text
comparison_eligibility = eligible
package_eligibility    = eligible       # from the verifier
cleanup_status         = succeeded
leak_audit_status      = clean
lease_status           = released
run_terminal_reason    = COMPLETED
```

The run MUST also have no known safety breach and no contradictory amendment.

A recovery after a run that was not clean can make a new run safe. It MUST NOT make that run complete the study.

### BR-HKP-063 — Platform baseline

*These choices are normative. The serverless strategies are the object of the study. A qualification is correct only for the source that it used.*

The implementation MUST be strict TypeScript on Node.js 24 in one npm project in `study-2/` with a committed lockfile. It MUST use AWS CDK v2 and the AWS SDK for JavaScript v3.

Study 2 MUST be an independent stack. It MUST share no deployed resource and no source directory with Study 1.

The front door of each variant MUST be an Amazon API Gateway HTTP API.

The payment ledger, the ticket ledgers and the outcome records MUST be in DynamoDB tables with on-demand capacity.

The stock store MUST be the only hot key of a sale. The partition key of each ticket, reservation record, outcome record and payment ledger record MUST contain the `buyer_id`. A table MUST NOT have a partition key that all purchase attempts of a sale write, except for the stock item.

The apparatus MUST do a complete read of such a table with a strongly consistent scan and a filter on the `trial_id`.

The load generator MUST operate in the Region of the variants. It MUST NOT use the concurrency of Lambda functions, because the variants and the payment provider use that quota.

## Prime Wave, Counters and Rehearsals

*The revision of 2026-11-15 added the rules of this section.*

### BR-HKP-064 — Prime wave before a burst

Before each burst to a purchase function that answers synchronously, the load generator MUST send one prime wave. This rule applies to the purchase functions of `DIRECT_TRANSACTION`, `DIRECT_UPDATE`, `HOT_TIER` and `SHARDED_UPDATE`, and to the function of the second probe route.

The first probe route MUST have no prime wave. Its function only answers and opens no connection. It answers fast, and Lambda then uses one execution environment for more than one request of a wave.

A prime wave MUST have the number of prime purchase attempts of OR-HKP-010. The load generator MUST send them at the same time.

Each prime purchase attempt MUST have its own prime sale. Each prime sale MUST have a lot of 1 unit and its own stock item or stock keys.

The purchase function MUST do a prime purchase attempt with the purchase protocol of BR-HKP-026. The purchase function MUST have no special path for a prime purchase attempt.

Each prime buyer MUST have an arrival sequence that the decline rule of BR-HKP-025 approves.

Each record of a prime purchase attempt MUST have a prime scope identifier in place of the `trial_id`. No trial MUST have that identifier. The oracle MUST NOT use a record that has a prime scope identifier.

The response to each purchase attempt MUST contain the identifier of the execution environment that answered.

The prime proof is complete only when all these conditions are true:

- each prime purchase attempt got an answer with HTTP status 200.
- the last answer arrived not later than the limit of OR-HKP-010.
- the answers contain no fewer different environment identifiers than the minimum of OR-HKP-010.

The runner MUST start the burst only when the prime proof is complete.

IF the prime proof is not complete, THEN the runner MUST make new prime sales. The runner MUST then send a new prime wave for the next start time. After the maximum number of prime waves of OR-HKP-010, the runner MUST stop the execution with a preparation timeout. That trial MUST have no oracle result.

`QUEUE_FIRST` MUST have no prime wave. Its front door has no function, and its queue must be empty at the start of a trial.

The trial manifest MUST record the prime proof.

### BR-HKP-065 — Interval of the counters of a trial

CloudWatch counters have no trial identifier, and their interval is 1 minute.

The burst of each trial MUST start at the start of a UTC minute plus the offset of OR-HKP-010. The prime wave MUST end before that minute starts.

The interval of the counters MUST be from the minute in which the burst starts through the minute of the evidence freeze. The runner MUST read the counters of OBS-HKP-001 and the usage counters of BR-HKP-013 only for this interval. The throttle counters of BR-HKP-023 MUST use the same interval.

Each reader of the apparatus MUST record the time and the read capacity that DynamoDB reports for each of its reads. The oracle MUST subtract that capacity from the read units of the table in the estimated usage cost.

The oracle MUST subtract only the reads whose time is in the interval of the counters. IF the result is less than 0, THEN the oracle MUST use 0 and MUST add the label `reader_capacity_more_than_counter`.

### BR-HKP-066 — Rehearsals

A rehearsal is an execution in the cloud that the operator starts with the rehearsal tool. Its purpose is development. A rehearsal is never evidence of the study.

Admission of a rehearsal can waive only these four checks:

- the source provenance of BR-HKP-050.
- the ratified spec of BR-HKP-047.
- the witness record of BR-HKP-048.
- the billing readiness of BR-HKP-049.

A rehearsal can change a reference value or an infrastructure option only with a recorded override and its reason. The package of a rehearsal MUST record each waived check and each override.

The runner MUST write the package of a rehearsal to `study-2/rehearsals/`. The runner MUST NOT write it to `study-2/evidence/`.

An execution that is not a rehearsal MUST NOT select a rehearsal package. The derivation tool MUST refuse a rehearsal package as the source of a value.

The study results file MUST give the number of rehearsal packages of each kind that are in `study-2/rehearsals/` at the time of the derivation. The file MUST say that a rehearsal is not evidence.

### BR-HKP-067 — Each unit has a record

*The revision of 2026-11-16 added this rule. In rehearsals, a variant closed a reservation and then lost its unit.*

A refused write is a write that the stock store answers with a throttle or with a false condition. The variant knows that a refused write made no change.

Each unit that is not an available unit and is not a ticket MUST have a durable record that names it.

A variant MUST NOT close a reservation as released before the unit of that reservation is an available unit again.

IF the stock store refuses the write that puts a unit back, THEN the record of that unit MUST stay open. The variant MUST do that write again until the unit is back.

A variant can take a unit and then be unable to write the reservation record. In that condition, the variant MUST put the unit back or MUST write a durable record that names the unit.

A variant MUST NOT put a unit back two times for one record. The record MUST show if its unit is back.

A write to the stock store can have a result that the variant does not know. After such a write, the variant MUST NOT do a write that can put a second unit back. The record MUST stay open, and BR-HKP-006 reports it.

`DIRECT_UPDATE` has no atomic operation for a unit and its record. In this variant, a write with an unknown result can cause a lost unit or an open reservation at settlement. This is a property of the variant, and the oracle reports it.

## Extension

*The extension revision of 2026-11-17 added the rules of this section. The questions come from the results of the canonical run of 2026-11-17.*

The canonical run measured one arrival rate above the documented limits. It did not show the arrival rate at which a variant stops to serve the sale. The extension measures that rate. It also adds the documented remedy for a hot key and one fault.

### BR-HKP-068 — Extension runs

The extension MUST have these six extension runs and no more.

| Extension run | Purpose |
|---|---|
| `RATE_RUN_A` | To measure the four variants of the canonical run in `RATE_50`, `RATE_100` and `RATE_250` |
| `RATE_RUN_B` | To measure the same four variants in `RATE_500`, `RATE_1000` and `RATE_2500` |
| `DEPTH_RUN` | To measure two long scenarios and four more `PEAK` trials of `QUEUE_FIRST` |
| `REPEAT_RUN` | To do the trials near each break rate again |
| `SHARD_RUN` | To measure `SHARDED_UPDATE` in the `PEAK` scenario |
| `STOP_RUN` | To measure each of the five variants when a purchase function stops |

Each rule of this spec for a canonical run MUST also apply to an extension run. The only differences are the differences that this section declares.

An extension run MUST select a load probe that obeys BR-HKP-037. It MUST select one verified variant validation for each variant that it has.

The operator MUST do the variant validation of `SHARDED_UPDATE` before the first extension run that has this variant.

In each trial of the extension, these inputs MUST be the same as in the canonical run:

- the preparation of BR-HKP-029 and the prime wave of BR-HKP-064.
- the buyer model of BR-HKP-020, the buyer deadline and the decline modulus.
- the hold time, the memo time and the retry policy.
- the ticket price and the currency.

An execution of the extension MUST NOT change a package, a result or a file of the canonical run.

The condition statement of an extension run MUST also contain:

- the values of OR-HKP-011 for each scenario of the run.
- the limits of the service conditions of OR-HKP-012.
- the shard values of OR-HKP-015, for a run that has `SHARDED_UPDATE`.
- the fault values of OR-HKP-016, for `STOP_RUN`.

### BR-HKP-069 — Rate scenarios

The extension MUST have six rate scenarios: `RATE_50`, `RATE_100`, `RATE_250`, `RATE_500`, `RATE_1000` and `RATE_2500`. The number in each name is the arrival rate in purchase attempts for each second.

Each rate scenario MUST have the burst duration of `PEAK`.

```text
buyers   = 10 * arrival_rate
lot_size =  2 * arrival_rate
```

`PEAK` obeys these two formulas. The rate series is the six rate scenarios and `PEAK`, in the sequence of their arrival rates.

The two rate runs have no `PEAK` trial. The three `PEAK` trials of the canonical run are the `PEAK` point of the rate series. A variant serves the `PEAK` point only when it serves each of these three trials. A service condition is `true` at the `PEAK` point only when it is `true` in each of these three trials.

The values of each rate scenario MUST agree with OR-HKP-011.

### BR-HKP-070 — Long scenarios

The extension MUST have two long scenarios: `LONG_PEAK` and `LONG_KEY`. Each MUST have a burst duration of 60 seconds.

`LONG_PEAK` MUST have the arrival rate and the lot size of `PEAK`. It shows a sale in which buyers continue to arrive after the first 10 seconds. Only `DIRECT_TRANSACTION` MUST have `LONG_PEAK` trials.

*In the canonical run, only `DIRECT_TRANSACTION` had lost sales of more than 1 % of the lot.*

`LONG_KEY` MUST have a lot that is not smaller than its number of buyers. The sale then cannot end during the burst. It shows the ticket rate of one stock item for 60 seconds. Only `DIRECT_UPDATE` MUST have `LONG_KEY` trials.

*In the canonical run, only `DIRECT_UPDATE` had a peak ticket rate of more than its documented limit.*

Each buyer of a long scenario MUST make one purchase attempt (BR-HKP-020). A long scenario does not show a buyer that sends a purchase attempt again.

The load probe sends no burst of 60 seconds. The load fidelity gate of BR-HKP-022 decides if a trial of a long scenario can be used.

The values of each long scenario MUST agree with OR-HKP-011.

### BR-HKP-071 — Service conditions

For each trial whose trial validity is `valid`, the oracle MUST calculate four service conditions.

```text
lost_sales_fraction = lost_sales / L
no_outcome_fraction = count(buyers with no final outcome) / count(buyers)

sold     = lost_sales_fraction        <= max_lost_sales_fraction
fair     = fairness_ratio             >= min_fairness_ratio
answered = no_outcome_fraction        <= max_no_outcome_fraction
timely   = final_outcome_delay_p99_ms <= buyer_deadline_ms
serves   = sold and fair and answered and timely
```

A buyer has a final outcome when BR-HKP-011 gives a final outcome delay for that buyer.

The limits MUST agree with OR-HKP-012. The oracle MUST compare each fraction before it rounds that fraction.

The oracle MUST give each fraction as a decimal string with four decimal places. The oracle MUST round each fraction half up.

IF no buyer of the trial has a final outcome, THEN `timely` MUST be `false`. IF the fair set is empty, THEN `fair` MUST be `false`.

A service condition is a measure. It MUST NOT change a rule result or a trial verdict.

### BR-HKP-072 — Break rate

The verifier MUST derive one break rate for each variant of the two rate runs. It MUST use the oracle results of the two rate runs and the replay of the canonical run (BR-HKP-079).

```text
break_rate = the smallest arrival rate of the rate series whose trial has serves = false
break_rate = none     when each trial of the variant in the rate series has serves = true
```

For each of the four service conditions, the verifier MUST also give the smallest arrival rate at which the condition is `false`.

IF a trial at an arrival rate larger than the break rate has `serves` = `true`, THEN the result MUST have the label `not_monotonic`.

After `REPEAT_RUN`, the verifier MUST derive `break_rate_confirmed` for each variant that has trials in that run.

```text
break_rate_confirmed = true
only if each repeat trial at the break rate has serves = false
and each repeat trial at the next smaller arrival rate has serves = true
```

The verifier MUST derive a break rate only when the comparison of each of the two rate runs is `eligible`.

IF the extension has more than one package of one rate run, THEN the verifier MUST derive no break rate and MUST give the reason.

### BR-HKP-073 — Ticket rate series

For each `valid` trial of a long scenario, the oracle MUST give the number of tickets in each interval of 1 second. The intervals are the intervals of BR-HKP-012.

```text
ticket_rate_series[k] = count(tickets whose issued_at is in interval k)        # k from 1
late_ticket_rate      = median(ticket_rate_series[31] through ticket_rate_series[60])
```

The oracle MUST give `late_ticket_rate` as a decimal string with one decimal place. IF the series has fewer than 60 intervals, THEN `late_ticket_rate` MUST be `null`.

For each `LONG_KEY` trial, the run summary MUST have a row that compares `late_ticket_rate` with `DL-4`. BR-HKP-044 gives the relations of that row.

### BR-HKP-074 — Composition and sequence of the extension runs

Each extension run MUST start with one `CONTROL` trial for each variant that it has. The other trials and their sequence MUST agree with OR-HKP-013.

| Extension run | Trials |
|---|---:|
| `RATE_RUN_A` | 16 |
| `RATE_RUN_B` | 16 |
| `DEPTH_RUN` | 11 |
| `SHARD_RUN` | 6 |
| `STOP_RUN` | 15 |
| `REPEAT_RUN` | BR-HKP-075 gives the number |

The runner MUST do the trials one at a time. The runner MUST wait for the gap of OR-HKP-003 between two trials (BR-HKP-039).

Each extension run MUST obey the safety limits of OR-HKP-014.

### BR-HKP-075 — Trials of the repeat run

`REPEAT_RUN` MUST come after the two rate runs. The comparison of each rate run MUST be `eligible`. The manifest of `REPEAT_RUN` MUST record the digest of the package index of each rate run and of the canonical run.

For each variant whose break rate is not `none`, `REPEAT_RUN` MUST have:

- one `CONTROL` trial.
- two trials at the break rate.
- two trials at the next smaller arrival rate of the rate series, when the series has one.

The `CONTROL` trials MUST be the first trials of the run. The other trials MUST be in two passes. In each pass, the arrival rates increase. For one arrival rate, the variants are in the sequence `DT`, `DU`, `QF`, `HT`.

A trial at the arrival rate 5,000 MUST be a `PEAK` trial.

Admission MUST derive this list from the packages of the two rate runs and of the canonical run. Admission MUST reject a manifest with a different list.

`REPEAT_RUN` MUST have the variant digests of the two rate runs. The two rate runs MUST have the same variant digests.

IF each variant has the break rate `none`, THEN the extension has no `REPEAT_RUN`.

### BR-HKP-076 — Variant `SHARDED_UPDATE`

*This variant is the documented remedy for a hot key. It has no hot key. It shows what the remedy gives under the same peak.*

The front door of `SHARDED_UPDATE` MUST call the purchase function synchronously.

The stock store MUST have the number of DynamoDB items of OR-HKP-015 for each sale. Each item is a shard. The partition key of a shard MUST contain the `sale_id` and the shard index.

The runner MUST divide the lot between the shards with the formula of OR-HKP-015.

The first shard of a buyer MUST be the shard that the formula of OR-HKP-015 gives for its arrival sequence.

The reservation operation MUST be one conditional `UpdateItem` call on one shard. That call decreases the available units of the shard by 1.

IF the shard has no available unit, THEN the purchase function MUST do the call on the next shard. The next shard has the next index, and the first index follows the last index.

The purchase function MUST go to the next shard only for a shard that has no available unit. For a throttle, an error or an unknown result of a shard call, the purchase function MUST do what `DIRECT_UPDATE` does for its reservation operation.

The purchase function MUST call each shard one time at most for one purchase attempt. A new attempt of the retry policy on the same shard is not a second call. The final outcome MUST be `SOLD_OUT` only when each shard has a memo or gave no available unit in this purchase attempt.

The execution environment MUST keep one sold-out memo for each shard. It MUST NOT call a shard during the memo time of that shard. WHEN each shard has a memo, the execution environment MUST answer `SOLD_OUT` with no stock operation.

The variant MUST write the reservation record of the buyer with a different call. The record MUST name the shard of its unit. A release MUST put the unit back to that shard.

BR-HKP-067 applies to this variant. `SHARDED_UPDATE` has no atomic operation for a unit and its record.

The stock snapshot MUST give the available units of each shard. The `available_units` of the snapshot MUST be their sum.

The stock table MUST have the default warm throughput of a new on-demand table (OR-HKP-004).

This variant has three declared differences: the number of stock items, more than one call in a reservation operation, and one memo for each shard.

*Source for the pattern: Amazon DynamoDB Developer Guide, "Using write sharding to distribute workloads evenly in your DynamoDB table", sharding with calculated suffixes.*

### BR-HKP-077 — Fault: a purchase function that stops

The scenario `PEAK_STOP` MUST have each value of `PEAK` and the fault of this rule. No other scenario MUST have a fault.

```text
fault_buyer(b) = (arrival_sequence(b) mod fault_modulus = fault_remainder)
```

The fault values MUST agree with OR-HKP-016. With these values, each fault buyer is an eligible buyer.

WHEN the reservation operation takes a unit for a fault buyer, the purchase function MUST end its process immediately. This is a fault stop.

After that reservation operation, the purchase function MUST do no call to a data store, to the payment provider or to the front door. It MUST write one line of telemetry before the fault stop.

The telemetry of each `PEAK_STOP` trial MUST give the number of these lines and the arrival sequence of each line. These values explain a result, and they decide nothing (BR-HKP-078).

The fault MUST apply only to the first execution of a purchase attempt. For `QUEUE_FIRST`, the fault MUST apply only when the receive count of the message is 1.

In `STOP_RUN`, the configuration of the purchase function MUST give the fault values and the `sale_id` of each `PEAK_STOP` trial of its variant. A purchase function with no fault values in its configuration MUST have no fault stop.

The fault MUST apply only to a purchase attempt for one of those `sale_id` values. The sale of a `CONTROL` trial and a prime sale are not in that list, and the fault does not apply to them.

The fault values and those `sale_id` values are not a part of the variant configuration of BR-HKP-038. The run manifest MUST record them.

The prime wave and the load generator of a `PEAK_STOP` trial MUST be the same as for a `PEAK` trial.

A variant MUST have no new mechanism that finds or repairs a fault stop. BR-HKP-027 and BR-HKP-067 give its only mechanisms.

### BR-HKP-078 — Lost units and fault buyers

For each `valid` trial, the oracle MUST give the number of lost units.

```text
lost_units = max(0, L - available_units - open_reservations - count(tickets where sale_id = S))
```

For each `valid` `PEAK_STOP` trial, the oracle MUST also count the fault buyers in each of these three groups:

- fault buyers with a ticket.
- fault buyers with a net charge and no ticket.
- fault buyers with no net charge and no ticket.

Each fault buyer is in one group and only one. The oracle MUST find the fault buyers from the load journal and the fault rule of BR-HKP-077. The oracle MUST NOT use the telemetry of a fault stop.

### BR-HKP-079 — Extension results file and replay

A derivation tool MUST write one extension results file from the selected packages of the extension. It MUST NOT change the study results file of BR-HKP-045.

Each value in the file MUST have the path of its source file in the package and the SHA-256 digest of that file.

The file MUST list each extension run, each load probe and each variant validation of the extension that has a package. It MUST give the number of rehearsal packages (BR-HKP-066).

The file MUST give the break rate of each variant and each `break_rate_confirmed` value.

The verifier MUST do a replay with the oracle of the extension. The replay MUST have the package of the canonical run and the packages of the four variant validations that it selected. A load probe has no replay.

In the replay, each rule result, each trial verdict and each measure of BR-HKP-008 through BR-HKP-013 MUST be equal to the frozen value.

The file MUST contain the record of that replay. The file MUST give the results of the `PEAK` trials of the canonical run with the label `canonical_run`.

The replay MUST compare only these values. It MUST NOT compare a version field of an oracle result.

IF a replay gives a different value, THEN the tool MUST stop with an error and MUST write no file.

IF the oracle changes after an extension run, THEN the operator MUST add an entry to the witness record (BR-HKP-048). The verifier MUST then do a replay of each earlier extension package with the same rule. In that replay, each measure of BR-HKP-071, BR-HKP-073 and BR-HKP-078 MUST also be equal to the frozen value.

The tool MUST read the witness record from the file `witness/study-2.jsonl` of the checkout. The tool MUST record the SHA-256 digest of that file. The tool MUST refuse a package whose oracle digest has no entry in the witness record.

### BR-HKP-080 — Eligibility and completion of the extension

The comparison of an extension run MUST be `eligible` only when each condition of BR-HKP-042 is true for all trials of that run.

An extension run is complete only when its first immutable package has each value of BR-HKP-062. It MUST also have no known safety breach and no contradictory amendment.

The extension is complete only when `RATE_RUN_A`, `RATE_RUN_B`, `DEPTH_RUN`, `SHARD_RUN` and `STOP_RUN` are complete. `REPEAT_RUN` MUST also be complete when BR-HKP-075 gives it one or more trials.

The extension results file MUST say which extension runs are complete. A public summary can give the results of a complete extension run before the extension is complete. That summary MUST name each extension run that is not complete. The extension results file gives this list.

The completion of Result 1 (BR-HKP-062) does not change.

## Reference Values

*The owner accepted these values on 2026-11-14. The merge of this spec makes them final.*

A canonical run MUST use the values of the ratified revision with no change. A change of a value after ratification MUST be a new revision of this spec.

### OR-HKP-001 — Sale fixture

| Field | `CONTROL` | `PEAK` |
|---|---:|---:|
| `currency` | `BRL` | `BRL` |
| `ticket_price_minor` | `15000` | `15000` |
| Ticket price for a person | BRL 150.00 | BRL 150.00 |
| `lot_size` | `600` | `10000` |
| Units for each purchase attempt | 1 | 1 |
| Stock items or hash slots for each sale | 1 | 1 |

`15000` minor units are BRL 150.00.

### OR-HKP-002 — Load schedule

| Parameter | `CONTROL` | `PEAK` |
|---|---:|---:|
| Arrival rate | 20 purchase attempts for each second | 5,000 purchase attempts for each second |
| Burst duration | 60 seconds | 10 seconds |
| Buyers | 1,200 | 50,000 |
| Interval between purchase attempts | 50 milliseconds | 0.2 milliseconds |
| Decline modulus | 20 | 20 |
| Declined buyers | 60 | 2,500 |
| Eligible buyers | 1,140 | 47,500 |
| Buyer deadline | 10 seconds | 10 seconds |
| Lateness limit of a send | 100 milliseconds | 100 milliseconds |
| Minimum on-time fraction | `0.9900` | `0.9900` |

| Reference row for BR-HKP-019 | `arrival_sequence` | `scheduled_send_offset_ms` |
|---|---:|---:|
| `PEAK`, first buyer | 1 | 0 |
| `PEAK`, second buyer | 2 | 0.2 |
| `PEAK`, last buyer | 50,000 | 9,999.8 |
| `CONTROL`, second buyer | 2 | 50 |
| `CONTROL`, last buyer | 1,200 | 59,950 |

### OR-HKP-003 — Timing and retry inputs

| Parameter | Value |
|---|---:|
| Timeout of a purchase function that answers synchronously | 9 seconds |
| Timeout of the consumer function of `QUEUE_FIRST` | 30 seconds |
| Time before its timeout in which the consumer function starts no purchase attempt | 2 seconds |
| Maximum time of one charge request in the payment provider | 3 seconds |
| Hold time | 60 seconds |
| Memo time of the sold-out memo | 1 second |
| Retry mode of each AWS SDK client | `standard` |
| Maximum attempts of each AWS SDK client | 3 |
| Conflict policy: total attempts | 3 |
| Conflict policy: wait | Full jitter, base 25 milliseconds, maximum 500 milliseconds |
| Visibility timeout of the source queue | 180 seconds |
| Observation deadline | 1,800 seconds after the start of the burst |
| Stabilization interval | 60 seconds |
| Interval between captures | 15 seconds |
| Gap between trials | 120 seconds |
| Monitoring for late evidence | 120 seconds |

### OR-HKP-004 — Capacity and preparation

| Parameter | Value |
|---|---:|
| Minimum Lambda quota for concurrent executions at admission | 1,000 |
| Minimum API Gateway account throttle at admission | 10,000 requests for each second, bucket of 5,000 |
| Reserved concurrency of a purchase function that answers synchronously | 600 |
| Provisioned concurrency of a purchase function that answers synchronously | 600 |
| Reserved and provisioned concurrency of the consumer function | 20 and 20 |
| Batch size of the event source mapping | 10 |
| Maximum concurrency of the event source mapping | 10 |
| Batching window of the event source mapping | 0 seconds |
| Partial batch response | On |
| Receive limit before the dead-letter queue | 5 |
| Provider rate in the load probe | 5,000 charge requests for each second, for 3 seconds |
| Charge requests that the load probe sends two times | `arrival_sequence` 1 through 1,000 |
| Minimum fraction of HTTP status 200 answers in the load probe | `0.9990` |
| Capacity mode of each DynamoDB table | On-demand |
| Warm write throughput of the payment ledger table and of each ticket ledger table | 6,000 write units for each second |
| Warm throughput of each other DynamoDB table | The default of a new on-demand table |
| Engine of the cache | Valkey, major version 8 or later |
| Minimum ECPU limit of the cache | 30,000 ECPUs for each second |

### OR-HKP-005 — Run sequence

`DT` is `DIRECT_TRANSACTION`, `DU` is `DIRECT_UPDATE`, `QF` is `QUEUE_FIRST`, and `HT` is `HOT_TIER`.

| Block | Scenario | Trial 1 | Trial 2 | Trial 3 | Trial 4 |
|---|---|---|---|---|---|
| 0 | `CONTROL` | `DT` | `DU` | `QF` | `HT` |
| 1 | `PEAK` | `DT` | `DU` | `QF` | `HT` |
| 2 | `PEAK` | `DU` | `HT` | `DT` | `QF` |
| 3 | `PEAK` | `QF` | `DT` | `HT` | `DU` |

In the three `PEAK` blocks, each variant has three different positions.

### OR-HKP-006 — Safety and coordination

| Input | Load probe | Variant validation | Canonical run |
|---|---:|---:|---:|
| Region | `us-east-1` | `us-east-1` | `us-east-1` |
| Maximum active time | 1,800 seconds | 5,400 seconds | 21,600 seconds |
| Time kept for cleanup | 900 seconds | 1,800 seconds | 1,800 seconds |
| Total time limit | 2,700 seconds | 7,200 seconds | 23,400 seconds |
| Limit of the estimated cost | USD 10.00 | USD 20.00 | USD 60.00 |
| Owners at the same time for the study, account and Region | 1 | 1 | 1 |

| Coordination input | Value |
|---|---:|
| Heartbeat interval | 30 seconds |
| Stale limit after the last heartbeat | 300 seconds |
| Interval of no resource for a clean leak audit | 120 seconds |

These values are maximums. They are not the durations that the owner predicts.

### OR-HKP-007 — Documented limits

Each row gives a value that an AWS document gives. External Sources gives the document and the date of the read.

| Row | Service | Documented limit | Value | Variants | Value of the study to compare |
|---|---|---|---:|---|---|
| `DL-1` | DynamoDB | Maximum write throughput of one partition | 1,000 write units for each second | `DT`, `DU`, `QF`, `SU` | Not comparable at 1 second. Telemetry explains. |
| `DL-2` | DynamoDB | Write units of one transactional write of an item of 1 KB or less | 2 | `DT`, `QF` | Not comparable. |
| `DL-3` | DynamoDB | Derived from `DL-1` and `DL-2`: transactional writes of one item | 500 for each second | `DT` | `peak_ticket_rate` |
| `DL-4` | DynamoDB | Derived from `DL-1`: standard writes of one item of 1 KB or less | 1,000 for each second | `DU` | `peak_ticket_rate` |
| `DL-5` | ElastiCache Serverless | Maximum ECPUs for one slot | 30,000 for each second | `HT` | Not comparable at 1 second. Telemetry explains. |
| `DL-6` | API Gateway | Account throttle for one Region | 10,000 requests for each second, bucket of 5,000 | All | Arrival rate of the scenario |
| `DL-7` | Lambda with a standard queue | Concurrent batches of one event source mapping | Starts at 5, adds 300 for each minute, maximum 1,250 | `QF` | Maximum concurrency of OR-HKP-004 |
| `DL-8` | Lambda | Default quota of concurrent executions | 1,000 | All | Not comparable. The quota of an account is not a limit of the service. The condition statement gives the quota that admission recorded. |
| `DL-9` | Lambda | Synchronous invocation requests of a function with provisioned concurrency | 10 for each second for each unit: 6,000 for 600 units | `DT`, `DU`, `HT`, `SU` | Arrival rate of the scenario |
| `DL-10` | DynamoDB | Burst capacity | Up to 300 seconds of unused read and write capacity | `DT`, `DU`, `QF`, `SU` | Not comparable. It can explain a `peak_ticket_rate` that is more than `DL-3` or `DL-4` for a short time. |

The rate of the `CONTROL` scenario is 20 purchase attempts for each second. It is below `DL-3`, `DL-4`, `DL-5` and `DL-6`.

`SU` is `SHARDED_UPDATE`. It has no row for the writes of one item, because the study does not measure the ticket rate of one shard.

### OR-HKP-008 — Worked example for the measures

This example uses a lot of 3 units, 8 buyers and a decline modulus of 4. The declined buyers are 4 and 8. The eligible buyers are 1, 2, 3, 5, 6 and 7. The fair set is 1, 2 and 3.

Case A:

| `arrival_sequence` | Answer | Tickets | Net charge |
|---:|---|---:|---:|
| 1 | `TICKET` | 1 | `15000` |
| 2 | `ERROR` | 0 | 0 |
| 3 | `TICKET` | 1 | `15000` |
| 4 | `PAYMENT_DECLINED` | 0 | 0 |
| 5 | `SOLD_OUT` | 0 | 0 |
| 6 | `TICKET` | 1 | `15000` |
| 7 | `SOLD_OUT` | 0 | 0 |
| 8 | `SOLD_OUT` | 0 | 0 |

Stock snapshot of case A: `available_units` = 0, `open_reservations` = 0.

Case B:

| `arrival_sequence` | Answer | Tickets | Net charge |
|---:|---|---:|---:|
| 1 | `TICKET` | 1 | `15000` |
| 2 | `ERROR` | 0 | 0 |
| 3 | `NO_ANSWER` | 1 | `15000` |
| 4 | `PAYMENT_DECLINED` | 0 | 0 |
| 5 | `ERROR` | 0 | 0 |
| 6 | `ERROR` | 0 | 0 |
| 7 | `SOLD_OUT` | 0 | 0 |
| 8 | `ERROR` | 0 | 0 |

Stock snapshot of case B: `available_units` = 1, `open_reservations` = 0.

Correct results:

| Value | Case A | Case B |
|---|---:|---:|
| Tickets | 3 | 2 |
| `unsold_units` | 0 | 1 |
| `denied_eligible_buyers` | 3 | 4 |
| `lost_sales` | 0 | 1 |
| `fairness_count` | 2 | 2 |
| `fairness_ratio` | `0.6667` | `0.6667` |
| `unannounced_tickets` | 0 | 1 |
| Buyers with `ERROR` | 1 | 4 |
| Buyers of the fair set with `ERROR` or `NO_ANSWER` | 1 | 2 |
| Rule results of BR-HKP-002 through BR-HKP-007 | All `pass` | All `pass` |
| Trial verdict | `pass` | `pass` |

### OR-HKP-009 — Price inputs

These are list prices for `us-east-1` in USD. The writer read them on 2026-11-14. The manifest MUST record these values. A change of an AWS price MUST be a new revision of this spec.

| Line | Class | Unit | Price in USD |
|---|---|---|---:|
| Lambda requests | Request-driven | 1,000,000 requests | `0.20` |
| Lambda duration with provisioned concurrency, x86 | Request-driven | GB-second | `0.0000097222` |
| Lambda duration with provisioned concurrency, Arm | Request-driven | GB-second | `0.0000077778` |
| Lambda duration on demand, x86 | Request-driven | GB-second | `0.0000166667` |
| Lambda duration on demand, Arm | Request-driven | GB-second | `0.0000133334` |
| Lambda provisioned concurrency, x86 | Held capacity | GB-second | `0.0000041667` |
| Lambda provisioned concurrency, Arm | Held capacity | GB-second | `0.0000033334` |
| API Gateway HTTP API | Request-driven | 1,000,000 requests | `1.00` |
| DynamoDB write request unit | Request-driven | 1,000,000 units | `0.625` |
| DynamoDB read request unit | Request-driven | 1,000,000 units | `0.125` |
| DynamoDB warm throughput increase, write | Held capacity, one time | Write unit of increase | `0.00065` |
| DynamoDB warm throughput increase, read | Held capacity, one time | Read unit of increase | `0.00013` |
| Amazon SQS standard queue | Request-driven | 1,000,000 requests | `0.40` |
| ElastiCache Serverless for Valkey, ECPUs that the trial used | Request-driven | 1,000,000 ECPUs | `0.0023` |
| ElastiCache Serverless for Valkey, ECPUs of the minimum limit that the trial did not use | Held capacity | 1,000,000 ECPUs | `0.0023` |
| ElastiCache Serverless for Valkey, data | Held capacity | GB-hour, minimum 0.1 GB | `0.084` |

Lambda bills provisioned concurrency for the time that it is set. Lambda rounds that time up to 5 minutes.

The purchase functions of all variants MUST have the same architecture and the same memory size. The manifest MUST record the two values.

The increase of the warm write throughput from 4,000 to 6,000 units costs USD 1.30 for each table.

### OR-HKP-010 — Prime wave values

| Parameter | Value |
|---|---:|
| Prime purchase attempts in one prime wave | 600 |
| Units in each prime sale | 1 |
| Start of the burst | The start of a UTC minute plus 500 milliseconds |
| Start of the prime wave | 3,000 milliseconds before the start of the burst |
| Limit for the last answer of the prime wave | 1,000 milliseconds before the start of the burst |
| Minimum number of different execution environments in the answers | 570 |
| Maximum number of prime waves for one trial | 3 |

| Reference row for BR-HKP-065 | Time in UTC |
|---|---|
| Start of the prime wave | `11:59:57.500` |
| Limit for the last answer | `11:59:59.500` |
| Start of the burst | `12:00:00.500` |
| First minute of the counters | `12:00` |

With these values, the wave starts 3.0 seconds before the burst. In probe 5, the wave also started 3 seconds before the burst, and the first requests ran 2.5 to 2.8 seconds before the burst.

### OR-HKP-011 — Scenarios of the extension

*The owner accepted the values of OR-HKP-011 through OR-HKP-017 on 2026-11-17.*

| Parameter | `RATE_50` | `RATE_100` | `RATE_250` | `RATE_500` | `RATE_1000` | `RATE_2500` | `LONG_PEAK` | `LONG_KEY` |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Arrival rate, purchase attempts for each second | 50 | 100 | 250 | 500 | 1,000 | 2,500 | 5,000 | 2,500 |
| Burst duration, seconds | 10 | 10 | 10 | 10 | 10 | 10 | 60 | 60 |
| Buyers | 500 | 1,000 | 2,500 | 5,000 | 10,000 | 25,000 | 300,000 | 150,000 |
| `lot_size` | 100 | 200 | 500 | 1,000 | 2,000 | 5,000 | 10,000 | 150,000 |
| Interval between purchase attempts, milliseconds | 20 | 10 | 4 | 2 | 1 | 0.4 | 0.2 | 0.4 |
| Declined buyers | 25 | 50 | 125 | 250 | 500 | 1,250 | 15,000 | 7,500 |
| Eligible buyers | 475 | 950 | 2,375 | 4,750 | 9,500 | 23,750 | 285,000 | 142,500 |
| Variants | `DT`, `DU`, `QF`, `HT` | `DT`, `DU`, `QF`, `HT` | `DT`, `DU`, `QF`, `HT` | `DT`, `DU`, `QF`, `HT` | `DT`, `DU`, `QF`, `HT` | `DT`, `DU`, `QF`, `HT` | `DT` | `DU` |

Each other value of these scenarios is the value of `PEAK` in OR-HKP-001 and OR-HKP-002. `PEAK_STOP` has each value of `PEAK`.

| Reference row for BR-HKP-019 | `arrival_sequence` | `scheduled_send_offset_ms` |
|---|---:|---:|
| `RATE_50`, second buyer | 2 | 20 |
| `RATE_50`, last buyer | 500 | 9,980 |
| `RATE_100`, last buyer | 1,000 | 9,990 |
| `RATE_250`, last buyer | 2,500 | 9,996 |
| `RATE_500`, last buyer | 5,000 | 9,998 |
| `RATE_1000`, last buyer | 10,000 | 9,999 |
| `RATE_2500`, last buyer | 25,000 | 9,999.6 |
| `LONG_PEAK`, last buyer | 300,000 | 59,999.8 |
| `LONG_KEY`, last buyer | 150,000 | 59,999.6 |

### OR-HKP-012 — Limits of the service conditions

| Limit | Value |
|---|---:|
| `max_lost_sales_fraction` | `0.0100` |
| `min_fairness_ratio` | `0.9000` |
| `max_no_outcome_fraction` | `0.0100` |
| `buyer_deadline_ms` | 10,000 |

These limits are a choice of the owner. The extension results file gives each measure, and a reader can apply different limits.

### OR-HKP-013 — Sequences of the extension runs

`SU` is `SHARDED_UPDATE`. The other short names are the names of OR-HKP-005.

`RATE_RUN_A`:

| Block | Scenario | Trial 1 | Trial 2 | Trial 3 | Trial 4 |
|---|---|---|---|---|---|
| 0 | `CONTROL` | `DT` | `DU` | `QF` | `HT` |
| 1 | `RATE_50` | `DT` | `DU` | `QF` | `HT` |
| 2 | `RATE_100` | `DU` | `HT` | `DT` | `QF` |
| 3 | `RATE_250` | `QF` | `DT` | `HT` | `DU` |

`RATE_RUN_B`:

| Block | Scenario | Trial 1 | Trial 2 | Trial 3 | Trial 4 |
|---|---|---|---|---|---|
| 0 | `CONTROL` | `DT` | `DU` | `QF` | `HT` |
| 1 | `RATE_500` | `DT` | `DU` | `QF` | `HT` |
| 2 | `RATE_1000` | `DU` | `HT` | `DT` | `QF` |
| 3 | `RATE_2500` | `QF` | `DT` | `HT` | `DU` |

In each rate run, each variant has three different positions in the blocks 1 through 3.

`DEPTH_RUN`:

| Trial | Scenario | Variant |
|---:|---|---|
| 1 | `CONTROL` | `DT` |
| 2 | `CONTROL` | `DU` |
| 3 | `CONTROL` | `QF` |
| 4 | `LONG_PEAK` | `DT` |
| 5 | `PEAK` | `QF` |
| 6 | `LONG_KEY` | `DU` |
| 7 | `PEAK` | `QF` |
| 8 | `LONG_PEAK` | `DT` |
| 9 | `PEAK` | `QF` |
| 10 | `LONG_KEY` | `DU` |
| 11 | `PEAK` | `QF` |

`SHARD_RUN`:

| Trial | Scenario | Variant |
|---:|---|---|
| 1 | `CONTROL` | `SU` |
| 2 | `CONTROL` | `DU` |
| 3 | `PEAK` | `SU` |
| 4 | `PEAK` | `DU` |
| 5 | `PEAK` | `SU` |
| 6 | `PEAK` | `SU` |

`STOP_RUN`:

| Block | Scenario | Trial 1 | Trial 2 | Trial 3 | Trial 4 | Trial 5 |
|---|---|---|---|---|---|---|
| 0 | `CONTROL` | `DT` | `DU` | `QF` | `HT` | `SU` |
| 1 | `PEAK_STOP` | `DT` | `DU` | `QF` | `HT` | `SU` |
| 2 | `PEAK_STOP` | `QF` | `HT` | `SU` | `DT` | `DU` |

In the two `PEAK_STOP` blocks, each variant has two different positions. BR-HKP-075 gives the sequence of `REPEAT_RUN`.

### OR-HKP-014 — Safety limits of the extension runs

| Input | `RATE_RUN_A` | `RATE_RUN_B` | `DEPTH_RUN` | `REPEAT_RUN` | `SHARD_RUN` | `STOP_RUN` |
|---|---:|---:|---:|---:|---:|---:|
| Maximum active time | 21,600 seconds | 21,600 seconds | 21,600 seconds | 25,200 seconds | 10,800 seconds | 21,600 seconds |
| Time kept for cleanup | 1,800 seconds | 1,800 seconds | 1,800 seconds | 1,800 seconds | 1,800 seconds | 1,800 seconds |
| Total time limit | 23,400 seconds | 23,400 seconds | 23,400 seconds | 27,000 seconds | 12,600 seconds | 23,400 seconds |
| Limit of the estimated cost | USD 60.00 | USD 60.00 | USD 60.00 | USD 75.00 | USD 30.00 | USD 60.00 |

The canonical run had 16 trials. It used 15,990 seconds, and its estimated cost was USD 31.16.

Each other safety input and each coordination input has the value of OR-HKP-006. The variant validation of `SHARDED_UPDATE` has the limits of a variant validation in OR-HKP-006.

These values are maximums. They are not the durations that the owner predicts.

### OR-HKP-015 — Shard values

| Parameter | Value |
|---|---:|
| Shards for each sale of `SHARDED_UPDATE` | 10 |

```text
units(i)       = floor(L / 10) + 1      when i < (L mod 10)            # i = shard index, from 0 through 9
units(i)       = floor(L / 10)          for each other shard
shard_index(b) = integer(first 8 hexadecimal digits of SHA-256(decimal text of arrival_sequence(b))) mod 10
```

The decimal text of an arrival sequence has only the characters `0` through `9` and has no separator.

| Reference row for the shard index | `arrival_sequence` | First 8 hexadecimal digits | `shard_index` |
|---|---:|---|---:|
| First buyer | 1 | `6b86b273` | 9 |
| Second buyer | 2 | `d4735e3a` | 4 |
| Third buyer | 3 | `4e074085` | 7 |
| First fault buyer | 125 | `0f8ef337` | 5 |
| Last buyer of `PEAK` | 50,000 | `60734f17` | 3 |

| Reference row for the units | `lot_size` | Units of each shard |
|---|---:|---|
| `PEAK` | 10,000 | 1,000 in each shard |
| `CONTROL` | 600 | 60 in each shard |
| Prime sale | 1 | 1 in shard 0, and 0 in each other shard |
| Example with a remainder | 1,003 | 101 in shards 0, 1 and 2, and 100 in each other shard |

### OR-HKP-016 — Fault values

| Parameter | Value |
|---|---:|
| `fault_modulus` | 250 |
| `fault_remainder` | 125 |
| Fault buyers in a `PEAK_STOP` trial | 200 |
| Fault buyers that are declined buyers | 0 |
| Fault buyers in the first 10,526 buyers | 42 |

| Reference row for BR-HKP-077 | `arrival_sequence` | Fault buyer |
|---|---:|---|
| First fault buyer | 125 | Yes |
| Second fault buyer | 375 | Yes |
| Last fault buyer | 49,875 | Yes |
| A buyer between two fault buyers | 250 | No |
| A declined buyer | 120 | No |

The first 10,526 buyers of `PEAK` contain the 10,000 eligible buyers of the fair set.

### OR-HKP-017 — Worked example for the service conditions and the break rate

Each row is one trial with a lot of 1,000 units and 5,000 buyers. The limits are the limits of OR-HKP-012.

| Row | Lost sales | Fairness ratio | Buyers with no final outcome | 99th percentile of the final outcome delay | `sold` | `fair` | `answered` | `timely` | `serves` |
|---|---:|---:|---:|---:|---|---|---|---|---|
| `SC-1` | 10 | `0.9000` | 50 | 10,000 milliseconds | `true` | `true` | `true` | `true` | `true` |
| `SC-2` | 11 | `0.9000` | 50 | 10,000 milliseconds | `false` | `true` | `true` | `true` | `false` |
| `SC-3` | 10 | `0.8990` | 50 | 10,000 milliseconds | `true` | `false` | `true` | `true` | `false` |
| `SC-4` | 10 | `0.9000` | 51 | 10,000 milliseconds | `true` | `true` | `false` | `true` | `false` |
| `SC-5` | 10 | `0.9000` | 50 | 10,001 milliseconds | `true` | `true` | `true` | `false` | `false` |
| `SC-6` | 1,000 | `0.0000` | 5,000 | No value | `false` | `false` | `false` | `false` | `false` |

In row `SC-6`, each buyer got an `ERROR` answer. No buyer has a final outcome, and the trial has no ticket.

Each row below is one variant in the rate series. `Y` is `serves` = `true`, and `N` is `serves` = `false`.

| Row | 50 | 100 | 250 | 500 | 1,000 | 2,500 | 5,000 | `break_rate` | Label |
|---|---|---|---|---|---|---|---|---|---|
| `BK-1` | `Y` | `Y` | `N` | `N` | `N` | `N` | `N` | 250 | None |
| `BK-2` | `Y` | `Y` | `Y` | `Y` | `Y` | `Y` | `Y` | `none` | None |
| `BK-3` | `N` | `N` | `N` | `N` | `N` | `N` | `N` | 50 | None |
| `BK-4` | `Y` | `Y` | `Y` | `Y` | `Y` | `Y` | `N` | 5,000 | None |
| `BK-5` | `Y` | `N` | `Y` | `N` | `N` | `N` | `N` | 100 | `not_monotonic` |

## Acceptance Criteria

Together, the acceptance criteria show that the capability operates correctly. An implementation run uses only this list to make the code and its tests.

Each business rule that the owner did not cancel has one or more criteria. Each criterion has a **Verification:** line that gives the method and the cases. `spec-anchored check-spec` finds each error against these two conditions.

A criterion with the method `golden` uses frozen synthetic evidence. The oracle or the verifier evaluates that evidence without the cloud.

### AC-HKP-001 — Control trial with all rules correct

Verifies BR-HKP-001, BR-HKP-002, BR-HKP-003, BR-HKP-004, BR-HKP-005, BR-HKP-006, BR-HKP-007 and BR-HKP-015.

- **Given** settled evidence of a `CONTROL` trial with 600 tickets, 600 charges and no refund
- **And** a stock snapshot with 0 available units and 0 open reservations
- **When** the oracle evaluates the trial
- **Then** each rule result of BR-HKP-002 through BR-HKP-007 is `pass`
- **And** the trial verdict is `pass`.

**Verification:** golden — frozen synthetic evidence of one settled `CONTROL` trial for each of the four variants. One more case has a ledger with a declined charge request, and the oracle rejects that ledger.

### AC-HKP-002 — Oversell is found

Verifies BR-HKP-002 and BR-HKP-015.

- **Given** settled evidence of a `PEAK` trial with 10,001 tickets for a lot of 10,000
- **When** the oracle evaluates the trial
- **Then** the rule result of BR-HKP-002 is `fail`
- **And** the trial verdict is `fail`.

**Verification:** golden — cases: 9,999 tickets (`pass`), 10,000 tickets (`pass`), 10,001 tickets (`fail`), a ticket ledger that is not complete (`indeterminate`).

### AC-HKP-003 — A charge without a ticket is found

Verifies BR-HKP-003.

- **Given** settled evidence in which one buyer has one charge, no refund and no ticket
- **When** the oracle evaluates the trial
- **Then** the rule result of BR-HKP-003 is `fail`.

**Verification:** golden — cases:

- One charge and no ticket (`fail`).
- One charge with its refund and no ticket (`pass`).
- Two charges and one ticket (`fail`).
- A currency that is not `BRL` (the oracle rejects the ledger).

### AC-HKP-004 — A ticket without a charge is found

Verifies BR-HKP-004.

- **Given** settled evidence in which one buyer has one ticket and no charge
- **When** the oracle evaluates the trial
- **Then** the rule result of BR-HKP-004 is `fail`.

**Verification:** golden — cases:

- One ticket and no charge (`fail`).
- One ticket with a charge and its refund (`fail`).
- One ticket with one charge of `15000` (`pass`).
- One ticket with one charge of `14999` (`fail`).

### AC-HKP-005 — A second ticket for one buyer is found

Verifies BR-HKP-005.

- **Given** settled evidence in which one buyer has two tickets with different `ticket_id` values
- **When** the oracle evaluates the trial
- **Then** the rule result of BR-HKP-005 is `fail`
- **And** the physical identity gate stays `verified`.

**Verification:** golden — cases: two tickets and two charges for one buyer (BR-HKP-005 is `fail`, BR-HKP-003 and BR-HKP-004 are `pass`), two tickets and one charge (BR-HKP-005 and BR-HKP-004 are `fail`).

### AC-HKP-006 — A reservation that stays open is found

Verifies BR-HKP-006.

- **Given** settled evidence of a `PEAK` trial with 9,990 tickets
- **And** a last stock snapshot with 9 available units and 1 open reservation
- **When** the oracle evaluates the trial
- **Then** the rule result of BR-HKP-006 is `fail`.

**Verification:** golden — cases:

- 9 available and 1 open (`fail`).
- 10 available and 0 open (`pass`).
- 9 available and 0 open as a lost unit (`fail`).
- 11 available and 0 open (`fail`).
- No stock snapshot (`indeterminate`).

### AC-HKP-007 — A false final outcome is found

Verifies BR-HKP-007.

- **Given** settled evidence in which one buyer received `TICKET` and has no ticket
- **When** the oracle evaluates the trial
- **Then** the rule result of BR-HKP-007 is `fail`.

**Verification:** golden — one case for each row of the table of BR-HKP-007 that agrees (`pass`). One case for each row that disagrees in the tickets (`fail`) and in the net charge (`fail`). `ACCEPTED` with a correct outcome record (`pass`). `ACCEPTED` with no outcome record (`fail`). `ERROR` with a ticket (the rule does not evaluate the buyer).

### AC-HKP-008 — Lost sales, fairness and answer exposure

Verifies BR-HKP-008, BR-HKP-009 and BR-HKP-010.

- **Given** the frozen evidence of case A and of case B of OR-HKP-008
- **When** the oracle calculates the measures
- **Then** each value agrees with the table of correct results in OR-HKP-008.

**Verification:** golden — each row of OR-HKP-008 for case A and case B. Other cases: more units than eligible buyers, a fairness ratio at the rounding tie `0.66665`, a trial that is not `valid` (each measure is `null`).

### AC-HKP-009 — Delay, sale duration and ticket rate

Verifies BR-HKP-011 and BR-HKP-012.

- **Given** frozen evidence with known send times, answer times, `decided_at` values and `issued_at` values
- **When** the oracle calculates the delay values, the sale duration and the maximum ticket rate
- **Then** each value agrees with the formulas of BR-HKP-011 and BR-HKP-012
- **And** each result with two clocks has the label `cross_source_clock_assumption`.

**Verification:** golden — cases:

- 100 latencies with a known 50th and 99th nearest-rank percentile.
- One buyer.
- No buyer with an answer (`null`).
- Tickets in two intervals of 1 second with 3 and 5 tickets (rate 5).

### AC-HKP-010 — Estimated usage cost for each ticket

Verifies BR-HKP-013.

- **Given** frozen usage counters and frozen price inputs for one trial with 10,000 tickets
- **When** the oracle calculates the estimated usage cost
- **Then** the request cost is the sum of quantity times unit price for each request-driven line
- **And** the result has the label `estimate`.

**Verification:** golden — cases:

- Two request-driven lines and one held-capacity line with known totals.
- A trial with no ticket (`null`).
- A usage line with no price input (the oracle rejects the calculation).
- An amount at the rounding tie of the sixth decimal place.

### AC-HKP-011 — The ledgers are the authority

Verifies BR-HKP-014.

- **Given** a ticket ledger with 10,000 tickets and a variant metric that shows 9,950 sales
- **When** the oracle evaluates the trial
- **Then** the oracle uses 10,000 tickets
- **And** the result does not use the metric as a source.

**Verification:** golden — cases: a metric that disagrees with a ledger, a missing optional metric (the verdict does not change), a ticket ledger capture that is not strongly consistent (the independent sources gate is `unverified`).

### AC-HKP-012 — Trial validity follows the table

Verifies BR-HKP-016.

- **Given** frozen evidence for one situation of the table of BR-HKP-016
- **When** the oracle evaluates the trial
- **Then** the trial validity and the trial verdict agree with the row of that situation.

**Verification:** golden — each row of the table of BR-HKP-016. One more case has one gate `invalid` and one gate `unverified` (the trial validity is `invalid`).

### AC-HKP-013 — A record that does not belong to the trial

Verifies BR-HKP-017 and INV-HKP-001.

- **Given** a ticket ledger capture that contains one ticket with the `trial_id` of a different trial
- **When** the oracle evaluates the trial
- **Then** the physical identity gate of INV-HKP-001 is `invalid`
- **And** the trial verdict is `indeterminate`.

**Verification:** golden — cases:

- A record of a different trial.
- A ticket whose `buyer_id` is not in the load journal (traceability `invalid`).
- Two outcome records with the same identifier (identity `invalid`).
- A manifest reference with an incorrect digest (traceability `invalid`).

### AC-HKP-014 — The schedule agrees with the reference rows

Verifies BR-HKP-018 and BR-HKP-019.

- **Given** the scenario values of OR-HKP-001 and OR-HKP-002
- **When** the load generator makes the schedule of a trial
- **Then** each scheduled offset agrees with the reference rows of OR-HKP-002
- **And** the schedule is the same for all four variants.

**Verification:** unit — each reference row of OR-HKP-002 and the count of buyers for each scenario. Property — the offset of a buyer does not depend on an answer. Contract — the reader rejects a manifest with a third scenario, with this criterion ID.

### AC-HKP-015 — One purchase attempt for each buyer

Verifies BR-HKP-020.

- **Given** a front door that gives no response to a purchase attempt for 12 seconds
- **When** the buyer deadline of 10 seconds passes
- **Then** the load journal records `NO_ANSWER` for that buyer
- **And** the load generator sends no second purchase attempt for that buyer
- **And** the load journal ignores the response that arrives later.

**Verification:** integration — a local front door with a controlled delay. Cases: a response at 9.9 seconds (answer kept), no response (`NO_ANSWER`), a response at 12 seconds (ignored), an `ERROR` answer (no second send).

### AC-HKP-016 — Each response gets one answer class

Verifies BR-HKP-021.

- **Given** one observation of the table of BR-HKP-021
- **When** the load generator classifies it
- **Then** the answer class is the class of the first row that agrees.

**Verification:** unit — each row of the table of BR-HKP-021. Other cases:

- Status 200 with an unknown `answer` (`ERROR`).
- Status 200 with a queue message identifier for a variant that is not `QUEUE_FIRST` (`ERROR`).
- Status 429 (`ERROR` with the status kept).
- A connection reset (`ERROR`).

### AC-HKP-017 — Load fidelity decides if a trial can be used

Verifies BR-HKP-022.

- **Given** a load journal of a `PEAK` trial in which 49,400 of 50,000 purchase attempts are on time
- **When** the oracle evaluates the load fidelity gate
- **Then** the gate is `invalid`
- **And** the trial verdict is `indeterminate`.

**Verification:** golden — cases:

- An on-time fraction of `0.9900` (`verified`).
- `0.9899` (`invalid`).
- 49,999 records (`unverified`).
- An internal overload record (`invalid`).
- 30,000 `ERROR` answers with all sends on time (`verified`).

### AC-HKP-018 — A problem of the apparatus makes a trial not usable

Verifies BR-HKP-023.

- **Given** throttle counters that show 12 throttled requests on the payment ledger table during the trial
- **When** the oracle evaluates the apparatus integrity gate
- **Then** the gate is `invalid`
- **And** the oracle gives `null` for each measure.

**Verification:** golden — cases:

- A throttled request on a table of the payment provider (`invalid`).
- A charge request that was longer than 3 seconds (`invalid`).
- An internal error in the journal of the payment provider (`invalid`).
- A reader that did not complete (`unverified`).
- A declined charge request (`verified`).
- A rejected charge request that was not correct (`verified`).

### AC-HKP-019 — The payment provider does each charge one time

Verifies BR-HKP-024.

- **Given** a payment ledger with one charge for a `payment_request_id`
- **When** the payment provider receives the same correct charge request again
- **Then** the payment provider gives the first response again
- **And** the payment ledger has one charge, and only one, for that `payment_request_id`.

**Verification:** contract — cases:

- A new identifier (one charge).
- The same request 100 times at the same time (one charge).
- The same identifier with a different amount (conflict).
- A request with a missing field (rejected, no charge).
- Two refund requests for one charge (one refund).
- A refund request for an unknown charge (rejected).

### AC-HKP-020 — The decline rule follows the arrival sequence

Verifies BR-HKP-025.

- **Given** a decline modulus of 20
- **When** the payment provider receives charge requests of the buyers 19, 20, 21 and 40
- **Then** it approves 19 and 21
- **And** it declines 20 and 40.

**Verification:** unit — cases: 19, 20, 21, 40 and 50,000. Property — the rule declines 2,500 buyers of the `PEAK` schedule and 60 buyers of the `CONTROL` schedule. Golden — the oracle and the provider give the same eligible buyers for case A of OR-HKP-008.

### AC-HKP-021 — Each path of the purchase protocol

Verifies BR-HKP-026.

- **Given** a sale with one available unit and a buyer that the provider approves
- **When** the purchase function does the purchase attempt
- **Then** the stock store gets one reservation operation, and that operation is its first operation
- **And** the payment provider gets one charge request with the deterministic `payment_request_id`
- **And** the ticket ledger gets one ticket with the deterministic `ticket_id`
- **And** the final outcome is `TICKET`.

**Verification:** integration — for each variant, with local emulation of the stock store. Cases: approved (`TICKET`), declined (`PAYMENT_DECLINED` and a release), no available unit (`SOLD_OUT` and no charge request), the same purchase attempt two times (one ticket, one charge). Memo cases: a second purchase attempt 0.5 seconds after a `SOLD_OUT` (no stock operation), and one 1.1 seconds after it (one reservation operation). Static check — no read of the stock store before the reservation operation.

### AC-HKP-022 — An open reservation ends at the hold time

Verifies BR-HKP-027.

- **Given** a reservation that is open 60 seconds after its start
- **And** a buyer with one charge and no refund
- **When** the variant ends the reservation
- **Then** the buyer has one ticket and the reservation is closed.

**Verification:** integration — for each variant. Cases: a charge and no ticket (ticket, or refund and release), no charge (release), a reservation at 59 seconds (no change), a reservation that is closed (no change). One more case has a release that the stock store refuses at the hold time, and the reservation stays open for the next pass.

### AC-HKP-023 — Equal inputs for all variants

Verifies BR-HKP-028.

- **Given** a canonical run in which one variant has a buyer deadline of 12 seconds
- **When** the verifier evaluates the equality projections
- **Then** the comparison is `ineligible`
- **And** the reason names the buyer deadline.

**Verification:** golden — cases:

- All projections equal (`eligible`).
- One case for each projection of BR-HKP-028 with a difference (`ineligible`).
- The declared differences of BR-HKP-031 through BR-HKP-034 (`eligible`).
- A `PEAK` trial verdict of `fail` with equal projections (`eligible`).

### AC-HKP-024 — No trial before the preparation is complete

Verifies BR-HKP-029.

- **Given** a variant whose provisioned concurrency is not ready
- **When** the runner is at the start of a trial of that variant
- **Then** the runner does not start the burst
- **And** the trial manifest gets the proof of each preparation only after it is complete.

**Verification:** integration — cases: provisioned concurrency not ready, warm throughput not complete, minimum ECPU limit not complete, all complete (the burst starts). One more case shows that the three other purchase functions have a reserved concurrency of 0 during the trial. Static check of the synthesis — each sale has one stock item.

### AC-HKP-025 — The retry policy is the declared policy

Verifies BR-HKP-030.

- **Given** a reservation operation that gets a transaction conflict three times
- **When** the purchase function does the purchase attempt
- **Then** the purchase function does the reservation operation three times and no more
- **And** the purchase attempt ends with an error response.

**Verification:** integration — cases:

- A conflict one time.
- Then success (`TICKET`).
- A conflict three times (error response).
- A throttled write (the SDK client makes no more than 3 attempts).
- A ledger write that does not complete (no automatic retry, and a second send has the same `payment_request_id`).

### AC-HKP-026 — Each variant has its declared structure

Verifies BR-HKP-031, BR-HKP-032, BR-HKP-033, BR-HKP-034 and BR-HKP-063.

- **Given** the deployment assembly of the four variants
- **When** a static check examines the assembly and the source of each purchase function
- **Then** each variant has the front door, the stock store and the reservation operation of its rule
- **And** each parameter agrees with OR-HKP-003 and OR-HKP-004.

**Verification:** integration — assertions on the synthesized templates. Cases: `DIRECT_TRANSACTION` uses one `TransactWriteItems` call with a client request token. `DIRECT_UPDATE` uses one conditional `UpdateItem` call. `QUEUE_FIRST` has a service integration with a standard queue, no function in the front path, and the event source mapping values. `HOT_TIER` has a serverless Valkey cache and one script call. Each front door is an HTTP API. The payment provider is a module and not a function. No variant source reads or writes the tables of the payment provider. The partition key of each table that is not the stock store contains the `buyer_id`. Study 2 shares no resource with Study 1.

### AC-HKP-027 — A message that arrives two times gives one final outcome

Verifies BR-HKP-033.

- **Given** a consumer function that completed the purchase attempt of a buyer with `TICKET`
- **When** the queue delivers the message of that buyer again
- **Then** the buyer has one ticket and one charge
- **And** the outcome record stays `TICKET`.

**Verification:** integration — cases: a second delivery after `TICKET`, after `PAYMENT_DECLINED` and after `SOLD_OUT`. A batch of 10 with one message that causes an error (only that message returns to the queue). A message that the consumer cannot complete in 5 receives goes to the dead-letter queue. A batch of 10 messages gives 10 reservation operations one after the other, and no two are in progress at the same time.

### AC-HKP-028 — The script reserves one unit in one call

Verifies BR-HKP-034.

- **Given** a cache with 1 available unit for a sale
- **When** 50 callers call the script for 50 different buyers at the same time
- **Then** one caller, and only one, gets a reservation
- **And** the cache shows 0 available units and 1 open reservation.

**Verification:** integration — against a local Valkey server. Cases: 50 callers for 1 unit, the same buyer two times (one reservation), a release (available units increase by 1), all keys of the sale in one hash slot. E2e — one call in the real cache of a variant validation.

### AC-HKP-029 — The load probe passes

Verifies BR-HKP-035 and BR-HKP-036.

- **Given** an admitted load probe in `us-east-1`
- **When** the load generator sends the `PEAK` schedule to the probe route
- **And** the probe sends 15,000 charge requests in 3 seconds, the first 1,000 two times
- **And** the load generator sends a prime wave before the second part
- **Then** the verifier derives the probe verdict from the frozen evidence
- **And** the probe verdict is `passed` only when each condition of BR-HKP-036 is true.

**Verification:** e2e — one load probe in the real cloud in `us-east-1`. Golden — the derivation of the probe verdict over the frozen evidence of that load probe.

### AC-HKP-030 — The load probe does not pass

Verifies BR-HKP-036.

- **Given** frozen evidence of a load probe in which the on-time fraction is `0.9700`
- **When** the verifier derives the probe verdict
- **Then** the probe verdict is `failed`
- **And** the result names the condition that is not true.

**Verification:** golden — one case for each condition of BR-HKP-036 that is not true (`failed`). Cases:

- Two charges for one `payment_request_id` (`failed`).
- A load journal that is not complete (`indeterminate`).
- A fraction of status 200 answers of `0.9990` (`passed`) and `0.9989` (`failed`).
- A prime proof that is not complete (`failed`).

### AC-HKP-031 — Only a usable load probe can be selected

Verifies BR-HKP-037.

- **Given** a load probe with the verdict `passed` and a leak audit that is `inconclusive`
- **When** admission of a variant validation selects that load probe
- **Then** admission rejects the execution with this criterion ID.

**Verification:** integration — one case for each condition of BR-HKP-037 that is not true (rejected). Cases:

- A change of the oracle source only (accepted).
- A change of the load generator source (rejected).
- A lower Lambda quota than the load probe recorded (rejected).

### AC-HKP-032 — The result of a variant validation

Verifies BR-HKP-038.

- **Given** a variant validation with two `valid` trials, a `CONTROL` verdict of `pass` and a `PEAK` verdict of `fail`
- **And** a clean operational closure
- **When** the verifier derives the validation result
- **Then** the validation result is `verified`.

**Verification:** golden — cases:

- `CONTROL` with `pass` and `PEAK` with `fail` (`verified`).
- `CONTROL` with `fail` (`failed`).
- One trial that is `indeterminate` (`indeterminate`).
- A cleanup that is `partial` (`indeterminate`).

Integration — a run rejects a validation whose variant digest is different. E2e — one variant validation for each variant in the real cloud.

### AC-HKP-033 — The run has 16 trials in the declared sequence

Verifies BR-HKP-039.

- **Given** an admitted canonical run
- **When** the runner makes the run manifest
- **Then** the manifest has 16 trials in the sequence of OR-HKP-005
- **And** the runner starts a trial only 120 seconds or more after the freeze of the trial before it.

**Verification:** unit — each cell of OR-HKP-005. Integration — cases:

- No selected load probe (rejected).
- A variant with no selected validation (rejected).
- A manifest with a different sequence (rejected).
- A gap of 119 seconds (the runner waits).

### AC-HKP-034 — Trials do not share data

Verifies BR-HKP-040.

- **Given** two trials of one variant in one run
- **When** the second trial starts
- **Then** it has a new `trial_id`, a new `sale_id`, new buyers and its own stock item or stock keys
- **And** no purchase attempt of the second trial changes a record of the first trial.

**Verification:** integration — cases:

- Two trials in sequence with local emulation.
- A queue that is not empty at the start (the runner does not start the trial).
- An active execution at the start (the runner does not start the trial).

### AC-HKP-035 — Settlement needs two equal captures

Verifies BR-HKP-041.

- **Given** a trial whose burst, buyer deadline and hold time are complete
- **And** two captures, 60 seconds apart, with the same digests
- **When** the runner evaluates settlement
- **Then** the trial is settled.

**Verification:** golden — cases:

- Two equal captures (settled).
- A new ticket between the captures (the interval starts again).
- A visible queue message in the second capture (not settled).
- No settlement at 1,800 seconds (the gate is `unverified`).
- One open reservation that does not change (settled, and BR-HKP-006 is `fail`).
- Captures 59 seconds apart (not settled).

### AC-HKP-036 — When a run can be compared

Verifies BR-HKP-042.

- **Given** a canonical run with 16 `valid` trials and four `CONTROL` verdicts of `pass`
- **And** equal projections, an evidence integrity gate of `verified` and late evidence of `none`
- **When** the verifier derives the comparison eligibility
- **Then** the comparison is `eligible`.

**Verification:** golden — cases:

- All conditions true (`eligible`).
- One case for each condition of BR-HKP-042 that is not true (`ineligible`).
- All `PEAK` verdicts `fail` (`eligible`).
- A leak that did not change isolation or evidence (`eligible`).

### AC-HKP-037 — The canonical run completes in the real cloud

Verifies BR-HKP-039, BR-HKP-058 and BR-HKP-062.

- **Given** a selected load probe and four selected variant validations
- **When** the operator does the canonical run in `us-east-1` from clean committed source
- **Then** the first package has 16 oracle results
- **And** the package has each value of BR-HKP-062.

**Verification:** e2e — one canonical run in the real cloud in `us-east-1`. Golden — the verifier derives the same values from the frozen package.

### AC-HKP-038 — Each summary has the condition statement

Verifies BR-HKP-043.

- **Given** a frozen run manifest and its trial manifests
- **When** the runner writes the run summary
- **Then** the summary has each field of the condition statement of BR-HKP-043
- **And** each value agrees with the manifests.

**Verification:** contract — the schema of CTR-HKP-008 makes each field of BR-HKP-043 mandatory. Golden — cases:

- A summary with no condition statement (the verifier rejects it).
- An environment input with no service-team review (the field is `none`).
- A validation result and the study results file (both have the statement).
- A summary with no prime wave values (the verifier rejects it).

### AC-HKP-039 — Each documented limit has a row

Verifies BR-HKP-044.

- **Given** a `DIRECT_TRANSACTION` trial with a `peak_ticket_rate` of 480
- **When** the runner writes the documented behavior rows
- **Then** the row of `DL-3` has the documented value 500, the value 480 and `in_documented_limit`
- **And** `documentation_difference` is `false`.

**Verification:** golden — cases:

- 480 against 500 (`in_documented_limit`).
- 500 against 500 (`in_documented_limit`).
- 501 against 500 (`more_than_documented_limit` and `documentation_difference` = `true`).
- A row of `DL-1` (`not_comparable`).
- A variant row with no source (the verifier rejects the summary).

### AC-HKP-040 — A result that disagrees with a hypothesis stays

Verifies BR-HKP-015.

- **Given** frozen evidence of a `PEAK` trial of `DIRECT_TRANSACTION` with 10,000 tickets and no lost sale
- **When** the oracle evaluates the trial
- **Then** the oracle gives the results that the evidence shows
- **And** the runner does not do the trial again.

**Verification:** golden — the case above. Static check — the oracle source has no reference to the hypotheses of this spec. It has no branch on a variant name in a rule or a measure.

### AC-HKP-041 — The study results file cites each value

Verifies BR-HKP-045.

- **Given** an eligible package of a canonical run, four selected variant validations and one validation that the operator did not select
- **When** the derivation tool writes the study results file
- **Then** each value has the path and the SHA-256 digest of its source file
- **And** the file lists all five validations, with a reason for the one that the operator did not select.

**Verification:** golden — cases:

- The set above.
- A package that is not eligible (the tool refuses).
- A source file whose digest changed (the tool refuses).
- Two packages with oracle results and different oracle digests (the tool refuses).
- A second derivation from the same packages (the same bytes).

### AC-HKP-042 — The public copy has no account identifier

Verifies BR-HKP-046.

- **Given** a raw package that contains the account identifier of the environment input in three files
- **When** the redaction tool derives the public copy
- **Then** each of the three files has `111122223333` in place of the account identifier
- **And** the redaction manifest lists the three files
- **And** the raw package has the same digests as before.

**Verification:** golden — cases: an identifier in an ARN, in a URL of a queue and in a JSON value. An identifier in a file of a type that the tool does not know (the tool stops with an error). A second account identifier that is not in the environment input stays, and the tool reports it.

### AC-HKP-043 — A draft spec cannot make evidence

Verifies BR-HKP-047.

- **Given** a source commit in which this spec has `status: draft`
- **When** the operator starts a load probe
- **Then** admission rejects the execution with this criterion ID
- **And** no change occurs in the cloud.

**Verification:** integration — cases: `draft` (rejected), `ratified` (accepted, and the manifest has the digest of the spec), a spec file that is missing (rejected).

### AC-HKP-044 — No witness record, no evidence

Verifies BR-HKP-048.

- **Given** a public repository whose witness record has no entry for the digests of the source commit
- **When** the operator starts a variant validation
- **Then** admission rejects the execution with this criterion ID.

**Verification:** integration — against a local Git remote. Cases:

- No witness record (rejected).
- An entry with a different oracle digest (rejected).
- An entry with a different spec digest (rejected).
- An entry with the two digests of the source commit (accepted, and the manifest has the two digests).
- A repository that admission cannot read with no credential (rejected).
- A changed oracle file that the list of oracle files contains (a different oracle digest).

### AC-HKP-045 — No billing readiness, no deployment

Verifies BR-HKP-049.

- **Given** an account in which the tag `suc:run_id` is not an active cost allocation tag
- **When** the operator starts a load probe
- **Then** admission rejects the execution with this criterion ID.

**Verification:** integration — with stubbed billing clients. Cases: one tag not active (rejected), no billing data export (rejected), an export with no resource identifiers (rejected), all conditions true (accepted).

### AC-HKP-046 — Evidence only from clean committed source

Verifies BR-HKP-050.

- **Given** a worktree with one changed tracked file
- **When** the operator starts a canonical run
- **Then** admission rejects the execution with this criterion ID.

**Verification:** integration — cases:

- A changed file (rejected).
- An untracked file that Git does not ignore (rejected).
- A changed lockfile (rejected).
- A merge that is not complete (rejected).
- A detached `HEAD` that is clean (accepted).
- An ignored evidence directory (accepted).

One more case shows that the runner deploys from the copy of the assembly in the package.

### AC-HKP-047 — The manifest is frozen before the first mutation

Verifies BR-HKP-051.

- **Given** an environment input with an account identifier that is not the caller account
- **When** the operator starts an execution
- **Then** the runner records a structured rejection
- **And** the runner makes no manifest, no result and no change in the cloud.

**Verification:** integration — cases: an incorrect account (rejected), an environment input with a credential field (rejected), a correct admission (the manifest freeze is before the lease acquisition, and the manifest has the quotas), a changed field after the freeze (a new execution identity is necessary).

### AC-HKP-048 — Late evidence does not change a frozen result

Verifies BR-HKP-052.

- **Given** a frozen trial result with 10,000 tickets
- **When** a ticket of that trial appears 30 seconds after the evidence freeze
- **Then** the frozen result and its digests do not change
- **And** the late evidence gets the assessment `contradictory`.

**Verification:** golden — cases:

- No late evidence (`none`).
- A late outcome record that agrees (`consistent`).
- A late ticket (`contradictory`, and it blocks the comparison).
- Emergency cleanup with no monitoring (`unverified`).
- An amendment chain with a missing parent (rejected).
- An indexed file with a different digest (the evidence integrity gate is `invalid`).

### AC-HKP-049 — Package eligibility comes from the verifier

Verifies BR-HKP-053.

- **Given** a final package in which one file changed after the runner wrote the package index
- **When** the verifier examines the package
- **Then** the package eligibility is `ineligible`
- **And** the verifier names the file.

**Verification:** golden — cases:

- A package with no change (`eligible`).
- A changed file (`ineligible`).
- A missing file (`ineligible`).
- A file that the index does not list (`ineligible`).
- A summary that contains a package eligibility field (rejected).

### AC-HKP-050 — Records agree with their contracts

Verifies BR-HKP-054.

- **Given** one record of each contract CTR-HKP-001 through CTR-HKP-011
- **When** a reader validates the record against its schema
- **Then** the reader accepts a correct record
- **And** the reader rejects a record with a missing mandatory field or an unknown field.

**Verification:** contract — for each schema:

- A correct record.
- A missing mandatory field.
- An unknown field.
- An amount as a decimal number.
- A time with no UTC marker.
- An enumeration value that this spec does not give.

Fuzz — random records never cause a crash that is not a clean rejection.

### AC-HKP-051 — One owner of the lease

Verifies BR-HKP-055.

- **Given** a run that has the lease and one heartbeat that did not succeed
- **When** the runner is at the start of the next trial
- **Then** the runner does not start the trial.

**Verification:** integration — cases:

- One heartbeat that did not succeed.
- Then recovery (the runner continues).
- A stale lease at 300 seconds (emergency cleanup).
- A different owner (emergency cleanup).
- A clean closure (`released`).
- The end of the lease lifetime with no release (`unverified`).
- A second owner at admission (rejected).

### AC-HKP-052 — The safety limits stop an execution

Verifies BR-HKP-056.

- **Given** a canonical run whose active time reaches 21,600 seconds during trial 12
- **When** the runner makes the safety check
- **Then** the runner starts no new trial
- **And** the runner keeps the evidence that it has and starts emergency cleanup.

**Verification:** integration — cases:

- An estimated cost of USD 60.01 (rejected).
- USD 60.00 (accepted).
- A Region that is not `us-east-1` (rejected).
- A Lambda quota of 999 (rejected).
- The limit of the active time (controlled stop).
- Cleanup after the total time limit (cleanup continues, and safety is `breached`).

### AC-HKP-053 — The billing amendment

Verifies BR-HKP-057.

- **Given** an eligible package and billing lines in USD that have the tags of its run
- **When** the operator imports the billing lines
- **Then** the import writes an immutable amendment with a `billed_cost_check`
- **And** the first package does not change.

**Verification:** golden — cases:

- All lines related and in the limit (`within_limit`).
- A total more than the limit (`breached`).
- A line in BRL (`unverified`).
- A line with no resource identity (`unverified`).
- A period that is not complete (`unverified`).

### AC-HKP-054 — Cleanup erases only what the execution owns

Verifies BR-HKP-058 and BR-HKP-059.

- **Given** an account that has a resource with only the tag `suc:project`
- **When** cleanup operates for a run
- **Then** cleanup does not erase that resource
- **And** the cleanup result reports it as inconclusive.

**Verification:** integration — cases:

- A resource in the manifest with the tags of the run (erased).
- A resource with tags of a different run (not erased).
- The coordination resource (not erased).
- A resource of one variant with no `suc:variant_id` tag (the static check of the assembly rejects it).
- Emergency cleanup (evidence kept before the first erase).
- The sequence of the steps of BR-HKP-058.

### AC-HKP-055 — The leak audit

Verifies BR-HKP-060.

- **Given** a completed cleanup and one list operation that does not succeed
- **When** the leak audit examines the account
- **Then** the audit result is `inconclusive`.

**Verification:** integration — cases:

- Each surface with no resource for 120 seconds (`clean`).
- One cache that stays (`leaks_detected`).
- A resource that appears at 100 seconds (`leaks_detected`).
- A query that does not succeed (`inconclusive`).

### AC-HKP-056 — A cleanup problem does not change a verdict

Verifies BR-HKP-061.

- **Given** a frozen trial verdict of `fail` and a cleanup that is `partial`
- **When** the runner writes the final summary
- **Then** the trial verdict stays `fail`
- **And** the summary shows the cleanup result as a different dimension.

**Verification:** golden — cases:

- `partial` cleanup with a frozen verdict.
- A function that stays and can write tickets of a trial (the settlement gate of that trial is `invalid`).
- A log group that stays (no effect on the comparison).

### AC-HKP-057 — A recovered run does not complete the study

Verifies BR-HKP-062.

- **Given** a canonical run whose first package has a leak audit of `leaks_detected`
- **When** the operator completes a recovery cleanup and the audit becomes `clean`
- **Then** the first package stays the same
- **And** that run does not complete Result 1.

**Verification:** golden — cases:

- The recovery above.
- A first package with each value of BR-HKP-062 (the study is complete).
- A first package with `comparison_eligibility` = `ineligible` (not complete).
- A known safety breach (not complete).

### AC-HKP-058 — Telemetry explains and does not decide

Verifies OBS-HKP-001 and OBS-HKP-002.

- **Given** a settled `PEAK` trial of `DIRECT_UPDATE`
- **When** the runner records the telemetry of OBS-HKP-001 after settlement
- **Then** the package has each available counter with its source and its interval
- **And** a counter that is not available has the value `unavailable` and does not change the trial verdict.

**Verification:** integration — with stubbed metric clients. Cases: all counters available, one counter not available (the verdict does not change), each journal event of OBS-HKP-002 has its rejection sibling. E2e — the telemetry files of one variant validation in the real cloud.

### AC-HKP-059 — No burst before the prime proof

Verifies BR-HKP-064.

- **Given** a prime wave of 600 prime purchase attempts whose answers contain 560 different environment identifiers
- **When** the runner evaluates the prime proof
- **Then** the runner does not start the burst
- **And** the runner sends a new prime wave with new prime sales for the next start time.

**Verification:** integration — with a local front door. Cases:

- 600 answers with status 200 and 595 environments (the burst starts).
- 570 environments (the burst starts).
- 569 environments (no burst).
- One answer with status 503 (no burst).
- A last answer 900 milliseconds before the start of the burst (no burst).
- Three prime waves with no complete proof (a preparation timeout and no oracle result).
- A trial of `QUEUE_FIRST` (no prime wave, and the burst starts).
- Two prime purchase attempts never have the same prime sale.

Golden — a trial whose tables contain 600 prime tickets and 600 prime charges gives the same oracle result as the trial with no prime record. E2e — one load probe in the real cloud with a prime wave before the second part and no prime wave before the first part.

### AC-HKP-060 — The counters of a trial contain no prime request

Verifies BR-HKP-065.

- **Given** a trial whose burst starts at `12:00:00.500` UTC and whose prime wave starts at `11:59:57.500` UTC
- **When** the runner reads the counters of the trial
- **Then** the first interval of the counters is the minute `12:00`
- **And** the counters contain no invocation of the prime wave.

**Verification:** unit — each reference row of OR-HKP-010, and a runner that is ready at `11:59:58.000` (the burst starts at `12:01:00.500`). Integration — with stubbed metric clients. Cases:

- A prime wave in minute `11:59` and a burst in minute `12:00` (the counters start at `12:00`).
- A prime answer that arrives at `12:00:00.100` (the prime proof is not complete).

Golden — an estimated usage cost in which the oracle subtracts 8,000 read units that the readers recorded. One more case has a read after the last minute of the interval, and the oracle does not subtract it. One more case has a result less than 0, and the oracle uses 0 with the label.

### AC-HKP-061 — A rehearsal is never evidence

Verifies BR-HKP-066.

- **Given** a rehearsal load probe with the verdict `passed`
- **When** admission of a variant validation that is not a rehearsal selects that load probe
- **Then** admission rejects the execution with this criterion ID.

**Verification:** integration — cases:

- A rehearsal package that an execution selects, and that execution is not a rehearsal (rejected).
- A rehearsal package that a rehearsal selects (accepted).
- A rehearsal that waives a fifth check (rejected).
- An override with no reason (rejected).
- A rehearsal package (the runner writes it to `study-2/rehearsals/`).
- A rehearsal package as the source of a value (the derivation tool refuses).

Golden — a rehearsal directory with 9 load probes, 7 variant validations and 1 run gives these three numbers in the study results file.

### AC-HKP-062 — A refused release keeps its unit

Verifies BR-HKP-067.

- **Given** a reservation of `DIRECT_UPDATE` whose buyer the payment provider declined
- **And** a stock store that refuses each attempt of the write that puts the unit back
- **When** the purchase attempt ends and the hold time passes
- **Then** the reservation stays open until the stock store accepts the write
- **And** the unit is an available unit again one time, and the variant then closes the reservation as released.

**Verification:** integration — for each variant, with a local stock store that refuses writes. Cases:

- A release that the stock store refuses on the purchase path and accepts for the closer (one unit back).
- A release that the stock store refuses two times for the closer and then accepts (one unit back).
- The write that closes the reservation does not succeed after the unit is back (no second unit).
- `DIRECT_UPDATE` takes a unit, cannot write the reservation record, and the stock store refuses the write that puts the unit back (a durable record names the unit, and the closer puts it back).
- A write to the stock store with an unknown result (no second write, and the record stays open).

Golden — a `DIRECT_UPDATE` trial with 10,495 units taken, 495 units back on the purchase path and 53 refused writes. After the closer, the stock snapshot shows 53 available units and 0 open reservations. BR-HKP-006 is `pass`, and the lost sales are 53.

### AC-HKP-063 — The scenarios of the extension agree with the reference rows

Verifies BR-HKP-069 and BR-HKP-070.

- **Given** the scenario `RATE_250`
- **When** the load generator makes the schedule of a trial
- **Then** the schedule has 2,500 buyers for a lot of 500 units
- **And** the last buyer has a scheduled send offset of 9,996 milliseconds.

**Verification:** unit — each column and each reference row of OR-HKP-011. Integration — cases:

- A rate scenario whose lot is not two times its arrival rate (rejected).
- A `LONG_KEY` trial with a lot that is smaller than its number of buyers (rejected).
- A `LONG_PEAK` trial of a variant that is not `DIRECT_TRANSACTION` (rejected).
- A `LONG_KEY` trial of a variant that is not `DIRECT_UPDATE` (rejected).
- The scenario `PEAK_STOP` (each value of `PEAK`).

### AC-HKP-064 — The service conditions follow the limits

Verifies BR-HKP-071.

- **Given** a `valid` trial with the values of row `SC-1` of OR-HKP-017
- **When** the oracle evaluates the trial
- **Then** `sold`, `fair`, `answered` and `timely` are `true`
- **And** `serves` is `true`, and the trial verdict does not change.

**Verification:** golden — each row of the first table of OR-HKP-017. Cases:

- A fraction that is more than its limit before the oracle rounds it and equal to the limit after (`false`).
- A trial that is not `valid` (each condition is `null`).
- A trial with the verdict `fail` and all four conditions true (`serves` is `true`, and the verdict stays `fail`).

Static check — the source of the service conditions has no branch on a variant name.

### AC-HKP-065 — The break rate comes from the rate series

Verifies BR-HKP-072.

- **Given** two eligible rate runs and a variant that has the results of row `BK-1` of OR-HKP-017
- **When** the verifier derives the break rate of that variant
- **Then** the break rate is 250
- **And** the result gives the smallest arrival rate at which each service condition is `false`.

**Verification:** golden — each row of the second table of OR-HKP-017. Cases:

- A rate run whose comparison is not `eligible` (no break rate).
- Two packages of `RATE_RUN_A` (no break rate, with the reason).
- A variant that serves two of the three `PEAK` trials of the canonical run (it does not serve the `PEAK` point).
- A variant with `fair` = `false` in one of the three `PEAK` trials (`fair` is `false` at the `PEAK` point).
- Two repeat trials at the break rate with `serves` = `false` and two at the next smaller rate with `serves` = `true` (`break_rate_confirmed` is `true`).
- One repeat trial at the break rate with `serves` = `true` (`break_rate_confirmed` is `false`).
- One repeat trial at the next smaller rate with `serves` = `false` (`break_rate_confirmed` is `false`).
- A break rate of 50, with no smaller rate (only the trials at the break rate decide).

### AC-HKP-066 — The ticket rate series of a long scenario

Verifies BR-HKP-073.

- **Given** a `valid` `LONG_KEY` trial with 1,800 tickets in each of the intervals 1 through 30
- **And** 1,000 tickets in each of the intervals 31 through 60
- **When** the oracle evaluates the trial
- **Then** the series has 60 values, and `late_ticket_rate` is `1000.0`
- **And** the row of `DL-4` has the value `1000.0` and `in_documented_limit`.

**Verification:** golden — the case above. Cases:

- Intervals 31 through 60 with 15 values of 1,000 and 15 values of 1,001 (`1000.5` and `more_than_documented_limit`).
- A series of 59 intervals (`late_ticket_rate` is `null`, and the row is `not_comparable`).
- An interval with no ticket (the value 0).
- A `LONG_PEAK` trial (a series, and no row of `DL-4`).

### AC-HKP-067 — Each extension run has its declared trials

Verifies BR-HKP-068 and BR-HKP-074.

- **Given** an admitted `RATE_RUN_A`
- **When** the runner makes the run manifest
- **Then** the manifest has 16 trials in the sequence of OR-HKP-013
- **And** the first four trials are `CONTROL` trials.

**Verification:** unit — each cell of each table of OR-HKP-013. Integration — cases:

- No selected load probe (rejected).
- A variant with no selected validation (rejected).
- A `SHARD_RUN` with no verified validation of `SHARDED_UPDATE` (rejected).
- A `STOP_RUN` variant whose source digest is different from the digest of its validation (rejected).
- A manifest with a different sequence (rejected).
- An estimated cost of more than the limit of OR-HKP-014 (rejected).
- An extension run that writes into a package of the canonical run (rejected).
- A condition statement with no limits of the service conditions (the verifier rejects the summary).
- A trial of the extension with a different hold time (the equality projection does not pass).

### AC-HKP-068 — The repeat run comes from the rate run

Verifies BR-HKP-075.

- **Given** two eligible rate runs with the break rates 250 for `DT`, 2,500 for `DU`, 50 for `QF` and `none` for `HT`
- **When** admission derives the trials of `REPEAT_RUN`
- **Then** the run has three `CONTROL` trials and ten other trials
- **And** each pass has the sequence `QF` at 50, `DT` at 100, `DT` at 250, `DU` at 1,000 and `DU` at 2,500.

**Verification:** golden — the case above. Integration — cases:

- Each break rate is `none` (no `REPEAT_RUN`).
- A manifest with a different list (rejected).
- A rate run whose comparison is not `eligible` (rejected).
- A variant digest that is different from the digest of the rate runs (rejected).
- Two rate runs with different variant digests (rejected).
- A break rate of 5,000 (two trials at `PEAK` and two trials at `RATE_2500`).
- A manifest with no digest of the package index of the canonical run (rejected).

### AC-HKP-069 — `SHARDED_UPDATE` goes to the next shard

Verifies BR-HKP-076.

- **Given** a sale of `SHARDED_UPDATE` with a lot of 10,000 units in ten shards
- **And** a buyer with the arrival sequence 1, whose first shard is shard 9
- **When** shard 9 has no available unit and shard 0 has one
- **Then** the purchase function takes the unit of shard 0
- **And** the reservation record names shard 0.

**Verification:** unit — each reference row of OR-HKP-015. Integration — with a local stock store. Cases:

- The first shard has a unit (one call).
- Nine shards have no unit and the tenth shard has one (ten calls and `TICKET`).
- A throttle of the first shard after each attempt of the retry policy (`ERROR`, and no call to the next shard).
- No shard has a unit (ten calls and `SOLD_OUT`).
- A shard with a memo (no call to that shard during the memo time).
- Each shard has a memo (`SOLD_OUT` with no stock operation).
- A shard after its memo time (the purchase function calls it again).
- A sale with a lot of 1,003 units (each shard has the units of OR-HKP-015).

### AC-HKP-070 — `SHARDED_UPDATE` puts a unit back to its shard

Verifies BR-HKP-076 and BR-HKP-067.

- **Given** a reservation of `SHARDED_UPDATE` in shard 4 whose buyer the payment provider declined
- **When** the variant does the release
- **Then** shard 4 has that unit again, and no other shard changes
- **And** the `available_units` of the stock snapshot is the sum of the ten shards.

**Verification:** integration — each case of AC-HKP-062 for `SHARDED_UPDATE`, with a local stock store. Golden — cases:

- A stock snapshot whose shards give a different sum than its `available_units` (the oracle rejects the snapshot).
- A settled trial with 10,000 tickets and no unit in each shard (BR-HKP-006 is `pass`).
- A settled trial with 9,990 tickets, 4 units in shard 2 and 6 units in shard 7 (BR-HKP-006 is `pass`).

### AC-HKP-071 — A fault stop ends the process after the unit

Verifies BR-HKP-077.

- **Given** a `PEAK_STOP` trial and a buyer with the arrival sequence 125
- **When** the reservation operation takes a unit for that buyer
- **Then** the process of the purchase function ends
- **And** the purchase function sent no charge request and no response for that buyer.

**Verification:** unit — each reference row of OR-HKP-016. Integration — for each of the five variants, with local stores. Cases:

- A fault buyer that gets a unit (a fault stop, and no later call).
- Three fault stops in one trial (the telemetry gives the number 3 and the three arrival sequences).
- A fault buyer for whom no unit is available (no fault stop and `SOLD_OUT`).
- A buyer that is not a fault buyer (no fault stop).
- A configuration with no fault values and the arrival sequence 125 (no fault stop).
- A `QUEUE_FIRST` message of a fault buyer with a receive count of 2 (no fault stop, and one final outcome).
- A prime purchase attempt with the arrival sequence 125 in a `PEAK_STOP` trial (no fault stop, and HTTP status 200).
- A purchase attempt with the arrival sequence 125 for a `sale_id` that is not in the configuration (no fault stop).
- A purchase attempt with the arrival sequence 125 in a `CONTROL` trial of `STOP_RUN` (no fault stop, because its `sale_id` is not in the configuration).
- A `STOP_RUN` variant whose validation had no fault values (admission selects the validation, because the variant digests are equal).
- A `STOP_RUN` manifest with no fault values or with no `sale_id` of a `PEAK_STOP` trial (rejected).
- A variant of the canonical run whose source has a change that is not the fault point (the static check rejects the variant).

### AC-HKP-072 — Lost units and the groups of fault buyers

Verifies BR-HKP-078.

- **Given** settled evidence of a `PEAK_STOP` trial with a lot of 10,000 units and 9,960 tickets
- **And** a stock snapshot with 0 available units and 0 open reservations
- **When** the oracle evaluates the trial
- **Then** `lost_units` is 40
- **And** the rule result of BR-HKP-006 is `fail`.

**Verification:** golden — the case above. Cases:

- 9,960 tickets and 40 available units (`lost_units` is 0, and BR-HKP-006 is `pass`).
- 10,001 tickets (`lost_units` is 0, and BR-HKP-002 is `fail`).
- A fault buyer with a ticket (the first group).
- A fault buyer with a net charge and no ticket (the second group, and BR-HKP-003 is `fail`).
- A fault buyer with no charge and no ticket (the third group).
- A fault buyer with a charge, a refund of the same amount and no ticket (the third group).
- A `PEAK_STOP` trial (the three groups contain 200 fault buyers).
- Telemetry that shows 41 fault stops (the oracle result does not change).
- A trial that is not `valid` (`null`).

### AC-HKP-073 — The extension results file does a replay of the canonical run

Verifies BR-HKP-079.

- **Given** the packages of the canonical run and the package of an eligible `RATE_RUN_A`
- **When** the derivation tool writes the extension results file
- **Then** the file has a replay record with equal values
- **And** the study results file of BR-HKP-045 has the same digest as before.

**Verification:** golden — cases:

- A replay with equal values (the tool writes the file).
- A replay with one measure that is different (an error, and no file).
- A package whose oracle digest has no entry in the witness record (refused).
- A rehearsal package (refused).
- A package of an extension run that is not eligible (refused).
- Two oracle digests in the extension with an equal replay (the tool writes the file).
- A replay of an extension package with a different `serves` value or a different `lost_units` value (an error, and no file).
- A `PEAK` trial of the canonical run (the file gives it with the label `canonical_run`).
- A replay with no package of a variant validation of the canonical run (refused).
- A checkout with no witness file (refused).

### AC-HKP-074 — When the extension is complete

Verifies BR-HKP-080.

- **Given** two complete rate runs, a complete `DEPTH_RUN` and a complete `REPEAT_RUN`
- **And** no package of `SHARD_RUN` or of `STOP_RUN`
- **When** the derivation tool writes the extension results file
- **Then** the file says that the extension is not complete
- **And** the file names `SHARD_RUN` and `STOP_RUN` as not complete.

**Verification:** golden — cases:

- All six extension runs are complete (the extension is complete).
- Each break rate is `none`, and the five other runs are complete (the extension is complete).
- An extension run that a recovery made safe (not complete).
- An extension run with one trial that is not `valid` (the comparison is `ineligible`).
- All `PEAK_STOP` verdicts are `fail` (the comparison is `eligible`).

### AC-HKP-075 — The extension runs complete in the real cloud

Verifies BR-HKP-068, BR-HKP-074 and BR-HKP-080.

- **Given** a selected load probe and the verified variant validations of each extension run
- **When** the operator does each extension run in `us-east-1` from clean committed source
- **Then** the first package of each run has each value of BR-HKP-062.

**Verification:** e2e — one execution of each extension run in the real cloud in `us-east-1`. Golden — the verifier derives the same values from each frozen package.

## Non-Goals

- This capability does NOT move real money and does NOT use a real payment provider.
- This capability does NOT stop a cache and does NOT remove the power of an Availability Zone. BR-HKP-077 gives the only fault of this spec.
- This capability does NOT compare ElastiCache node-based clusters or their durability modes. They are Result 2.
- This capability does NOT use more than one stock item or one hash slot for a sale in the four variants of the canonical run. Only `SHARDED_UPDATE` has shards (BR-HKP-076).
- This capability does NOT use a waiting room, a rate limit for each buyer, a CAPTCHA or bot detection.
- This capability does NOT let a buyer send a purchase attempt again.
- This capability does NOT sell seats with numbers, more than one unit for each purchase attempt, or more than one sale at the same time.
- This capability does NOT make statistical inference. It reports each trial of each cell.
- This capability does NOT say that one service is better than a different service. It reports rule results and measures under a condition statement.
- This capability does NOT use a different Region, a different runtime or a second account.
- This capability does NOT write articles, posts or images. The study results file is its last output.
- This capability does NOT change a frozen result or a package when late evidence, a recovery or billing data arrives.
- This capability does NOT make a library that Study 1 and Study 2 share.
- This capability does NOT infer exactly-once delivery from the behavior of a service.

## Contracts

Each contract has a JSON schema in `study-2/contracts/`. BR-HKP-054 gives the serialization. Each record has `schema_version`, `run_id`, `trial_id` and `sale_id`, unless the contract says differently.

### CTR-HKP-001 — Purchase request and response

- **Input:** from the load generator to the front door. `POST /sales/{sale_id}/purchases` with `buyer_id` and `arrival_sequence`.
- **Output for a variant that answers synchronously:** HTTP status 200 with `answer` (`TICKET`, `SOLD_OUT` or `PAYMENT_DECLINED`), `buyer_id`, `environment_id`, and `ticket_id` only for `TICKET`.
- **Output for `QUEUE_FIRST`:** HTTP status 200 with the queue message identifier.
- **Error sibling:** each other HTTP status. BR-HKP-021 classifies it as `ERROR`.

### CTR-HKP-002 — Payment provider and payment ledger

- **Charge request:** `payment_request_id`, `buyer_id`, `arrival_sequence`, `amount_minor`, `currency`.
- **Charge response:** `APPROVED` with `charge_id`, or `DECLINED`.
- **Refund request:** `refund_request_id` and `charge_id`. **Refund response:** `REFUNDED` with `refund_id`.
- **Error siblings:** `REJECTED` with a reason for a request that is not correct, and `CONFLICT` for a known identifier with different content.
- **Ledger record:** `record_id`, `kind` (`CHARGE` or `REFUND`), `payment_request_id`, `charge_id`, `buyer_id`, `amount_minor`, `currency`, `committed_at`.
- **Journal record:** each declined request, each rejected request, each conflict and each internal error of the provider, with its kind and its time.
- **Access:** only the payment provider writes its tables. Only the readers of the apparatus read them. A variant source calls only the charge interface and the refund interface.

### CTR-HKP-003 — Ticket

- **Record:** `ticket_id`, `buyer_id`, `payment_request_id`, `issued_at`.
- **Writer:** the purchase function of the variant. **Reader:** the apparatus, with a complete, strongly consistent read.

### CTR-HKP-004 — Stock snapshot

- **Record:** `variant_id`, `lot_size`, `available_units`, `open_reservations`, `read_consistency` (`strong`), `capture_sequence`, `captured_at`.
- **Record for `SHARDED_UPDATE`:** also `shards`, with the `shard_index` and the `available_units` of each shard.
- **Writer:** a reader of the apparatus that reads the stock store of the variant. A variant MUST NOT write this record.
- **Error sibling:** `SNAPSHOT_UNAVAILABLE` with a reason.

### CTR-HKP-005 — Outcome record

- **Record:** `outcome_id`, `buyer_id`, `final_outcome` (`TICKET`, `SOLD_OUT` or `PAYMENT_DECLINED`), `decided_at`.
- **Writer:** the consumer function of `QUEUE_FIRST`. **Reader:** the apparatus. The other variants write no outcome record.

### CTR-HKP-006 — Load journal

- **Header:** `scenario`, `variant_id`, `burst_start_at`, the schedule parameters, and each internal overload record of the load generator.
- **Record for each buyer:** `buyer_id`, `arrival_sequence`, `scheduled_send_offset_ms`, `sent_offset_ms`, `answer_class`, `http_status`, `error_kind`, `answer_received_offset_ms`.

### CTR-HKP-007 — Oracle result

- **Output, identity and validity:** the trial identity, each gate of BR-HKP-016 and the trial validity.
- **Output, results:** each rule result, the trial verdict, each measure of BR-HKP-008 through BR-HKP-013 and the labels.
- **Output, results of the extension:** also the measures of BR-HKP-071, BR-HKP-073 and BR-HKP-078.
- **Output, evidence:** a reference to each evidence file with its digest.

### CTR-HKP-008 — Run summary

- **Output, science:** the run identity, the condition statement, the 16 trial results and the comparison eligibility with its reasons.
- **Output, documentation:** the documented behavior rows and `documentation_difference`.
- **Output, operations:** the late evidence assessment, `cleanup_status`, `leak_audit_status`, `lease_status`, `safety_status`, `evidence_integrity_status` and `run_terminal_reason`.
- The summary MUST NOT contain the package eligibility.

### CTR-HKP-009 — Load-probe result

- **Output:** the probe identity, each condition of BR-HKP-036 with its value, the probe verdict, the bound digests, the recorded quotas and the operational closure.

### CTR-HKP-010 — Variant-validation result

- **Output:** the validation identity, the variant, the two trial results, the validation result, the bound digests, the condition statement and the operational closure.

### CTR-HKP-011 — Study results file

- **Output:** `results/study-2/results.json`. It has each selected package with the digest of its package index. It has each value with its source path and digest. It lists all load probes and variant validations. It gives the number of rehearsal packages of each kind.

### CTR-HKP-012 — Operator CLI

The operator CLI MUST have one operation for each of these actions:

- admission with no mutation.
- load probe.
- variant validation.
- canonical run.
- package verification.
- recovery cleanup.
- billing import.
- results derivation.
- redaction.
- extension run, with the name of the extension run.
- derivation of the extension results file.

Each operation MUST write one JSON document, and only one, to standard output. Each operation MUST end with exit code 0 only when it completed its action.

A rejection MUST contain the ID of the criterion that rejected the action. The rehearsal tool of BR-HKP-066 is a different tool, and it is not a part of the operator CLI. The names of the commands are a choice of the implementation, and the README of `study-2/` MUST give them.

### CTR-HKP-013 — Prime journal

- **Record for each prime purchase attempt:** the prime scope identifier, the prime sale, `sent_offset_ms`, `http_status`, `answer_class`, `environment_id` and `answer_received_offset_ms`.
- **Header:** the start of the prime wave, the start of the burst, the number of different environment identifiers and the result of the prime proof.
- **Writer:** the load generator. The prime journal is not a part of the load journal of the trial.

### CTR-HKP-014 — Extension run summary

- **Output, science:** the run identity, the name of the extension run, the condition statement, each trial result and the comparison eligibility with its reasons.
- **Output for `REPEAT_RUN`:** the digest of the package index of each rate run and of the canonical run, and each `break_rate_confirmed` value.
- **Output, documentation and operations:** the same values as CTR-HKP-008.
- The summary MUST NOT contain the package eligibility.

### CTR-HKP-015 — Extension results file

- **Output:** `results/study-2/results-extension.json`. It has each selected package of the extension with the digest of its package index. It has each value with its source path and digest.
- **Output, replay:** the oracle digest of the extension, the digest of each package of the replay and the result of the replay.
- **Output, witness:** the SHA-256 digest of the witness file.
- **Output, rate series:** the break rate of each variant, the smallest arrival rate at which each service condition is `false`, and the labels.
- **Output, completion:** each extension run with `complete` or `not_complete`, and the completion of the extension. `REPEAT_RUN` has `not_required` when BR-HKP-075 gives it no trial.
- The tool MUST NOT write `results/study-2/results.json`.

## Observability

### OBS-HKP-001 — Explanatory telemetry

After the settlement of each trial, the runner records these counters for the interval of the trial. They explain a result. They are never a source for a verdict or for a measure, except for the usage counters of BR-HKP-013.

- **DynamoDB:** consumed write units, write throttle events, key-range throttle events, the throttle reasons that the purchase function received, and the cancellation reasons of transactions.
- **Lambda:** invocations, duration, throttles, concurrent executions and spillover of provisioned concurrency.
- **API Gateway:** request count, 4xx count, 5xx count and integration latency.
- **Amazon SQS:** messages sent, messages received, the age of the oldest message and the messages of the dead-letter queue.
- **ElastiCache:** ECPUs, throttled commands, ECPUs of script commands and the latency of successful writes.

### OBS-HKP-002 — Journal of the execution

The runner records one event in the coordination journal for each of these steps. Each accepted step has a rejection sibling.

- **Events of admission:** admission accepted or rejected, manifest frozen, lease acquired or refused.
- **Events of a trial:** preparation complete or timed out, burst started, burst completed, capture taken or not complete, settlement reached or not reached, evidence frozen.
- **Events of closure:** cleanup step completed or not completed, package final.
- **Metrics that the operator monitors:** the on-time fraction of the load generator during a burst, and the number of trials with a frozen result.

## Dependencies

- **CAP-RUA (Study 1):** the source of the method. Study 2 restates the method rules that it uses and has no dependency on the deployed resources or the source directory of Study 1.
- **AWS services in `us-east-1`:** API Gateway, Lambda, DynamoDB, Amazon SQS, ElastiCache Serverless, CloudWatch, Cost Explorer and the billing data exports.
- **Coordination resource:** a person makes it one time for Study 2. An execution does not own it.
- **Public Git repository:** the holder of the witness record. It holds digests only.
- **QA contracts:** `study-2/specs/qa/study-operator/` crosses this capability.

## Threats to Validity and Limitations

1. Each buyer makes one purchase attempt. Real buyers send again after an error. This model makes the lost sales larger and the load smaller than in a real sale.
2. The schedule has a constant interval. Real arrivals come in groups.
3. The payment provider adds no delay and operates in the purchase function. A real payment is a network call that keeps a reservation open for a longer time.
4. The concurrency limit of 600 and the quotas of the account are conditions of each result. A different limit gives different numbers.
5. Three `PEAK` trials for each cell show the range of the results. They do not support statistical inference.
6. The final outcome delay and the sale duration use the clock of the load generator and the clocks of the functions. No AWS guarantee covers the difference.
7. The fairness measure uses the scheduled sequence. The sequence at the front door can be different by a few milliseconds.
8. Four variants in one Region cannot support a conclusion about all serverless architectures.
9. The configuration of each variant comes from variant validations before the canonical run. The study results file lists each validation to make this visible.
10. The sold-out memo lets an execution environment answer `SOLD_OUT` for 1 second while a released unit is available. It can cause lost sales in each variant.
11. `HOT_TIER` keeps open reservations only in memory. In Result 1, no component stops. Result 1 cannot show the cost of that choice.
12. CloudWatch gives most counters for intervals of 1 minute. The study cannot compare them with a limit for each second.
13. The estimated usage cost uses list prices and counters. The billing amendment can show a different amount.
14. SHA-256 digests show a change against a known digest. They do not show who made a file.
15. A raw package contains identifiers of the environment. Only the public copy of BR-HKP-046 is for publication.
16. The prime wave removes the cost of the first request from the burst. A sale that opens with no prime wave loses purchase attempts in its first half second at 512 MB. A rehearsal probe measured 679 throttled answers of 15,000.
17. `QUEUE_FIRST` has no prime wave. Its 20 consumer environments do their first request in the burst. This adds about 0.2 seconds one time for each environment.
18. One prime request for each environment does not make the code fully warm. In probe 5, the first requests of the burst were 35 milliseconds, and later requests were 12 milliseconds.

19. Rehearsals in the cloud came before the first evidence. They changed the implementation and showed that the prime wave is necessary. The study results file gives their number.
20. A unit that a variant puts back after the burst is an available unit that no buyer can get. The lost sales of a trial contain these units.
21. The rate runs have one trial for each variant at each arrival rate. `REPEAT_RUN` adds trials only near a break rate.
22. The lot of a rate scenario changes with its arrival rate. In `RATE_50`, one lost sale is 1 % of the lot.
23. The limits of the service conditions are a choice of the owner. A different limit gives a different break rate.
24. A long scenario has new buyers during the full burst. It does not show a buyer that sends a purchase attempt again, and its fairness ratio does not apply to such a buyer.
25. `LONG_KEY` cannot show the cause of a ticket rate above `DL-4`. DynamoDB keeps up to 300 seconds of burst capacity, and a burst of 60 seconds does not use all of it (OQ-HKP-025).
26. DynamoDB can put more than one shard of `SHARDED_UPDATE` in one partition. The study cannot see the partitions of a table.
27. The fault of BR-HKP-077 ends a process at one point of the purchase protocol. A real function can stop at each point, and a different point gives a different result.
28. After a fault stop, Lambda starts a new execution environment. That cost is a part of each `PEAK_STOP` result.
29. The fault of BR-HKP-077 does not stop the cache of `HOT_TIER`. Limitation 11 stays true after the extension (OQ-HKP-024).
30. The extension runs and the canonical run are on different days, and the `PEAK` point of the rate series comes from the canonical run. The `PEAK` trials of `DEPTH_RUN` and of `SHARD_RUN` show for two variants if the conditions are the same.
31. The hypotheses of the extension use the results of the canonical run. They are not independent of those results.

## Open Questions

The load generator host and its tool are a choice of the implementation. The load probe of BR-HKP-035 resolves by experiment if that choice can send the `PEAK` schedule.

The owner accepted the proposals of the writer on 2026-11-14. Each question that this decision closed shows RETIRED and names the IDs that answer it. The revision of 2026-11-15 closed OQ-HKP-006 and OQ-HKP-008. The owner accepted OQ-HKP-014 and OQ-HKP-015 on 2026-11-15. The revision of 2026-11-16 recorded the decisions OQ-HKP-016, OQ-HKP-017 and OQ-HKP-018. The revision of 2026-11-17 recorded the decision OQ-HKP-019. The extension revision of 2026-11-17 recorded the decisions OQ-HKP-020 through OQ-HKP-023 and OQ-HKP-026.

OQ-HKP-024 and OQ-HKP-025 stay open. They are questions for service teams, and they do not stop an execution of the extension.

### OQ-HKP-001 — Code and name of the capability (RETIRED)

- Decision: the code is `HKP`, and the name is "Hot Key Under Peak".
- Resolved by: the `capability_id` of this spec.

### OQ-HKP-002 — Four variants in place of the four architectures of the backlog (RETIRED)

- Decision: Result 1 has the four variants of this spec. `DIRECT_UPDATE` is a variant, because AWS documents that concurrent transactions on one item cancel. The node-based Valkey cluster is in Result 2.
- Resolved by: BR-HKP-031, BR-HKP-032, BR-HKP-033 and BR-HKP-034.

### OQ-HKP-003 — One stock item for `QUEUE_FIRST` (RETIRED)

- Decision: `QUEUE_FIRST` has one stock item and no shards.
- Resolved by: BR-HKP-033.

### OQ-HKP-004 — Load numbers and cost limits (RETIRED)

- Decision: a lot of 10,000, a rate of 5,000 for each second, a burst of 10 seconds and three `PEAK` trials for each cell. The sum of the cost limits for one load probe, four variant validations and one canonical run is USD 150.00.
- Resolved by: OR-HKP-002, OR-HKP-005 and OR-HKP-006.

With the price inputs of OR-HKP-009 and a memory size of 512 MB, the writer estimates USD 45 for all executions. This estimate has no measurement. Provisioned concurrency and warm throughput are its largest parts.

### OQ-HKP-005 — Two rules that the backlog does not have (RETIRED)

- Decision: the study has the two rules.
- Resolved by: BR-HKP-005 and BR-HKP-007.

### OQ-HKP-006 — Lambda quota of the sandbox account (RETIRED)

- Decision: the sandbox account has a quota of 2,000 concurrent executions. Admission of the rehearsals recorded it.
- Resolved by: BR-HKP-051 and BR-HKP-056.

### OQ-HKP-007 — DynamoDB warm throughput and price inputs (RETIRED)

- Decision: the price inputs are the list prices of 2026-11-14. Only the payment ledger table and the ticket ledger tables get a warm write throughput of 6,000 units. The first draft gave 20,000 units to each table, and that cost USD 10.40 for each table.
- Resolved by: OR-HKP-004 and OR-HKP-009.

### OQ-HKP-008 — Network path to the cache (RETIRED)

- Decision: the purchase function of `HOT_TIER` connects through a VPC with isolated subnets and a gateway endpoint for DynamoDB. The first variant validation of `HOT_TIER` measures the ECPU cost of the script.
- Resolved by: BR-HKP-034.

### OQ-HKP-009 — Tuning values of the variants (RETIRED)

- Decision: the values of this spec are the first values. If a variant validation shows that a value is not reasonable, the owner changes it in a new revision. That change makes a new validation of that variant necessary.
- Resolved by: OR-HKP-003 and OR-HKP-004.

### OQ-HKP-010 — Where the witness is public (RETIRED)

- Decision: the witness is a record in a public repository that holds digests only. It does not show Study 2 before its publication.
- Resolved by: BR-HKP-048.

### OQ-HKP-011 — Review by the service teams before the run (RETIRED)

- Decision: the owner invites reviewers of the Lambda, DynamoDB and ElastiCache teams while this spec is a draft. A review is not a gate.
- Resolved by: BR-HKP-051 and BR-HKP-043. The environment input and the condition statement record the review or `none`.

### OQ-HKP-012 — Ticket price, decline fraction and provider delay (RETIRED)

- Decision: a price of BRL 150.00, a decline fraction of 5% and no provider delay.
- Resolved by: OR-HKP-001, OR-HKP-002 and BR-HKP-024.

### OQ-HKP-013 — Name of the fictional marketplace (RETIRED)

- Decision: the spec uses general identifiers and no company name.
- Resolved by: OR-HKP-001.

### OQ-HKP-014 — Values of the prime wave (RETIRED)

- Decision: the values of OR-HKP-010 are the first values. They come from one rehearsal probe. If a variant validation shows that one prime wave is not sufficient for a variant, the owner changes the values in a new revision.
- Resolved by: BR-HKP-064 and OR-HKP-010.

### OQ-HKP-015 — Number of rehearsals in the study results file (RETIRED)

- Decision: the study results file gives the number of rehearsals of each kind. This number is a statement of the operator, because the rehearsal directory is local.
- Resolved by: BR-HKP-066.

### OQ-HKP-016 — Prime wave on the first probe route (RETIRED)

- Decision: the first part of the load probe has no prime wave. In five rehearsal probes with no prime wave, that route answered 50,000 of 50,000 requests. With a prime wave, 600 requests reached 340 to 430 execution environments, because the function answers fast.
- Resolved by: BR-HKP-035, BR-HKP-036 and BR-HKP-064.

### OQ-HKP-017 — Units that `DIRECT_UPDATE` lost in rehearsals (RETIRED)

- Decision: this was a defect of the implementation and not a result of the study. The variant closed a reservation as released, and then the stock store refused the write that puts the unit back. In four rehearsal trials, the lost units (42, 37, 53 and 46) were equal to the declined buyers with an `ERROR` answer. No write had an unknown result.
- Resolved by: BR-HKP-067 and AC-HKP-062.

### OQ-HKP-018 — Documented limits and reader capacity after the first full rehearsal (RETIRED)

- Decision: the row `DL-8` is not comparable. The row `DL-10` gives the burst capacity of DynamoDB. The oracle subtracts only the reads of the apparatus that are in the interval of the counters.
- Resolved by: OR-HKP-007 and BR-HKP-065.

### OQ-HKP-019 — Maximum concurrency of the consumer of `QUEUE_FIRST` (RETIRED)

- Decision: the maximum concurrency of the event source mapping is 10. Rehearsals of 2026-11-16 and 2026-11-17 with the rule of BR-HKP-033 gave the table below for the `PEAK` scenario. Each trial was valid and had the verdict `pass`, 10,000 tickets, no lost sale and no dead-letter message. The value of 10 gave the smallest 99th percentile of the final outcome delay. No smaller value was in 10 % of it.
- Resolved by: OR-HKP-004 and BR-HKP-033.

| Maximum concurrency | Transaction conflicts | Sale duration | Final outcome delay, 50th percentile | Final outcome delay, 99th percentile | Fairness count | Messages that the consumer received again |
|---:|---:|---:|---:|---:|---:|---:|
| 2 | 488 | 643.7 seconds | 695.7 seconds | 926.5 seconds | 4,856 | 38 |
| 5 | 2,447 | 423.6 seconds | 296.4 seconds | 380.7 seconds | 5,188 | 253 |
| 10 | 8,184 | 301.9 seconds | 141.5 seconds | 251.0 seconds | 5,260 | 1,251 |

In the rehearsals before BR-HKP-033 had this rule, the consumer did the 10 messages of a batch at the same time. Rehearsals with 2, 5, 10 and 20 then had 20,714 to 50,333 transaction conflicts for 10,000 tickets.

Lambda showed 3, 5 and 11 concurrent executions of the consumer for 2, 5 and 10. The cause of a number that is more than the maximum concurrency is not known. This is a question for the Lambda team.

### OQ-HKP-020 — Scenarios and number of trials of the extension (RETIRED)

- Decision: six rate scenarios with one trial for each variant, and more trials only near a break rate. Two long scenarios with two trials each. Four more `PEAK` trials of `QUEUE_FIRST`. Two `PEAK_STOP` trials for each variant.
- Resolved by: BR-HKP-069, BR-HKP-070, BR-HKP-074, BR-HKP-075, OR-HKP-011 and OR-HKP-013.

### OQ-HKP-021 — A buyer that sends a purchase attempt again (RETIRED)

- Decision: this spec does not add it. A second purchase attempt of one buyer is possible only with a variant that continues the first one. No variant of this study does that. `LONG_PEAK` shows a sale in which buyers continue to arrive.
- Resolved by: BR-HKP-070 and limitation 24.

### OQ-HKP-022 — Limits of the service conditions (RETIRED)

- Decision: a variant serves a trial when four conditions are true. The lost sales are 1 % of the lot or less. The fairness ratio is `0.9000` or more. The buyers with no final outcome are 1 % of the buyers or less. The 99th percentile of the final outcome delay is not more than the buyer deadline.
- Resolved by: BR-HKP-071 and OR-HKP-012.

### OQ-HKP-023 — Number of shards and shard function (RETIRED)

- Decision: ten shards for each sale. The first shard of a buyer comes from the SHA-256 digest of the arrival sequence.
- Resolved by: BR-HKP-076 and OR-HKP-015.

### OQ-HKP-024 — A cache that stops during the peak

- This question is open. It is a question for the ElastiCache team.
- The AWS FIS action `aws:elasticache:replicationgroup-interrupt-az-power` does not support a serverless cache. The writer found no documented operation that stops a serverless cache.
- Question: which writes can a serverless cache lose when a node stops, and how can a study cause that condition?
- Until the answer, no execution stops the cache, and the condition statement of `HOT_TIER` says this.

### OQ-HKP-025 — Ticket rate of one stock item above `DL-4`

- This question is open. It is a question for the DynamoDB team.
- In the canonical run, `DIRECT_UPDATE` had a peak ticket rate of 1,815 to 1,828 on one stock item. `DL-4` gives 1,000, and `DL-10` can explain a larger rate for a short time.
- Question: what limits the writes of one item of an on-demand table, and for how long?
- `LONG_KEY` gives the ticket rate for 60 seconds. It cannot show the cause.

### OQ-HKP-026 — Form of the fault (RETIRED)

- Decision: the process of the purchase function ends after the reservation operation takes a unit. The fault applies to 1 buyer of each 250, only on the first execution and only for the sales of the `PEAK_STOP` trials. One deployment of a function gives the values for all its trials of the run.
- Resolved by: BR-HKP-077 and OR-HKP-016.

## External Sources

These sources give the documented behavior that the hypotheses and OR-HKP-007 use. They do not give the business rules.

| Source | Behavior that this study uses | Retrieved |
|---|---|---|
| [DynamoDB: partition key design](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html) | Each partition gives a maximum of 3,000 read units and 1,000 write units for each second | 2026-11-09 |
| [DynamoDB: burst and adaptive capacity](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/burst-adaptive-capacity.html) | Adaptive capacity can isolate one item, and that item gets no more than the partition maximum. DynamoDB keeps up to 300 seconds of unused read and write capacity for bursts. A burst can use it faster than the capacity for each second | 2026-11-16 |
| [DynamoDB: transactions, how it works](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html) | A transactional write uses 2 write units for each item. Concurrent transactions on one item cancel with `TransactionCanceledException`. The SDKs do not retry that exception. A canceled transaction uses capacity | 2026-11-09 |
| [DynamoDB: working with items](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html) | A conditional write whose condition is false uses write capacity. A conditional update is the safe form of a counter | 2026-11-09 |
| [DynamoDB: key range throughput exceeded](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/throttling-key-range-limit-exceeded-mitigation.html) | The throttle reason `TableWriteKeyRangeThroughputExceeded` and the metric `WriteKeyRangeThroughputThrottleEvents` | 2026-11-09 |
| [DynamoDB: warm throughput](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/warm-throughput.html) | A team can increase warm throughput before a planned peak. AWS bills the increase | 2026-11-09 |
| [DynamoDB: warm throughput scenarios](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/warm-throughput-scenarios.html) | A new on-demand table starts with 12,000 read units and 4,000 write units for each second. The partition limit stays | 2026-11-09 |
| [AWS Database Blog: Implement resource counters with Amazon DynamoDB](https://aws.amazon.com/blogs/database/implement-resource-counters-with-amazon-dynamodb/) (2023-03-31) | The patterns of `DIRECT_TRANSACTION` and `DIRECT_UPDATE`. The post says that single-item approaches can reach limits | 2026-11-09 |
| [ElastiCache: scaling serverless caches](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Scaling-serverless.html) | A maximum of 30,000 ECPUs for each second for one slot. A minimum ECPU limit as preparation, which can take 60 minutes and which AWS bills | 2026-11-09 |
| [ElastiCache: how it works](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.corecomponents.html) | The definition of an ECPU. A command that uses more vCPU time uses more ECPUs | 2026-11-09 |
| [ElastiCache: serverless metrics](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/serverless-metrics-events-redis.html) | The metrics `ElastiCacheProcessingUnits`, `ThrottledCmds` and `EvalBasedCmdsECPUs` | 2026-11-09 |
| [Lambda: scaling behavior](https://docs.aws.amazon.com/lambda/latest/dg/scaling-behavior.html) | A function adds 1,000 execution environments for each 10 seconds. More requests get a 429 error | 2026-11-09 |
| [Lambda: quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) | The default quota of 1,000 concurrent executions. New accounts have a lower quota. Lambda limits synchronous invocation requests to 10 times the concurrency, and to 10 times the provisioned concurrency of a version or alias | 2026-11-09 |
| [Lambda: provisioned concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html) | Reserved concurrency is a maximum for a function. Provisioned concurrency keeps environments ready. 100 units stay unreserved | 2026-11-09 |
| [Lambda with Amazon SQS: scaling](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-scaling.html) | For a standard queue, Lambda starts with 5 concurrent batches, adds 300 for each minute and stops at 1,250. Maximum concurrency is from 2 through 1,000 | 2026-11-09 |
| [Amazon SQS: standard queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html) | Almost unlimited calls for each second, at-least-once delivery and best-effort sequence | 2026-11-09 |
| [API Gateway: quotas](https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html) | The account throttle of 10,000 requests for each second with a bucket of 5,000 | 2026-11-09 |
| [API Gateway: HTTP API quotas](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-quotas.html) | The maximum integration timeout of 30 seconds | 2026-11-09 |
| [API Gateway: service integrations for HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-aws-services.html) | The integration `SQS-SendMessage` without a function | 2026-11-09 |
| [AWS SDKs and Tools: retry behavior](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html) | The `standard` retry mode and the setting for maximum attempts | 2026-11-09 |
| [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/) | The prices of requests, duration and provisioned concurrency for x86. The step of 5 minutes for provisioned concurrency | 2026-11-14 |
| [Amazon API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/) | The price of HTTP API requests | 2026-11-14 |
| [Amazon DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/) | Warm throughput is a one-time charge for each unit of increase | 2026-11-14 |
| [AWS Database Blog: Pre-warming Amazon DynamoDB tables with warm throughput](https://aws.amazon.com/blogs/database/pre-warming-amazon-dynamodb-tables-with-warm-throughput/) (2024-11-13) | The price of a unit of increase in `us-east-1` | 2026-11-14 |
| [Amazon ElastiCache pricing](https://aws.amazon.com/elasticache/pricing/) | The prices of ECPUs and of data for ElastiCache Serverless for Valkey. The minimum of 100 MB | 2026-11-14 |
| [ElastiCache Serverless for Valkey: public endpoints](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-elasticache-serverless-public-endpoints/) (2026-09-29) | A cache with a public endpoint needs no VPC. It needs Valkey 9.0 or later, IAM authentication and TLS 1.3 | 2026-11-14 |
| [DynamoDB: write sharding](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-sharding.html) | A calculated suffix on the partition key distributes writes between partition key values. The pattern of `SHARDED_UPDATE` | 2026-11-17 |
| [AWS FIS: actions reference](https://docs.aws.amazon.com/fis/latest/userguide/fis-actions-reference.html) | The ElastiCache action interrupts the power of nodes of a replication group. It does not support a serverless cache | 2026-11-17 |

Retrieval notes:

- The transactions page says that a canceled transaction uses capacity, with a false condition as its example. It does not say this in the same words for a conflict.
- The retry page gives a 2026 behavior that starts only with an opt-in setting. This spec gives the retry values in OR-HKP-003 and does not use a default.
- The scaling page of ElastiCache does not say what a cache does at the limit for one slot. The study records `ThrottledCmds` to see it.
- The resource counters post gives no number for the limit of one item. The number comes from the partition pages.
- The AWS pricing pages for DynamoDB and Amazon SQS gave no price table on 2026-11-14. The prices of DynamoDB request units and of Amazon SQS requests in OR-HKP-009 come from pages that cite the AWS Price List.
- The Arm prices of Lambda in OR-HKP-009 come from pages that cite the AWS Price List. The x86 prices come from the AWS pricing page.
