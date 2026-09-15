# Data Model: AI Reconciliation Platform

## Core Entities

### Transaction Record

Represents a single financial transaction from an authoritative source or received processor feed.

**Fields**
- transaction_id: unique transaction reference
- merchant_id: merchant or MID identifier
- amount: monetary value for the transaction
- action_type: financial action such as fee, funding, or settlement
- source_system: gateway, processor, or other originating source
- source_file_origin: file, feed, or batch reference for the source record
- reconciliation_status: RECONCILED or UNRECONCILED
- matching_score: optional numeric score used when no material mismatch exists
- lifecycle_events: related events such as funding, settlement, dispute, or chargeback records

**Relationships**
- A Transaction Record can have many Lifecycle Events
- A Transaction Record can belong to one Reconciliation Outcome
- A Transaction Record can have many Mismatch Reasons if it is UNRECONCILED

### Reconciliation Outcome

The final decision for a transaction after deterministic evaluation.

**Fields**
- transaction_id
- outcome_status: RECONCILED or UNRECONCILED
- evaluation_timestamp
- matching_score
- source_evidence_summary
- mismatch_reasons[]

**Validation Rules**
- outcome_status must be one of RECONCILED or UNRECONCILED
- a material mismatch must produce UNRECONCILED
- the transaction must remain visible even when UNRECONCILED
- reviewer actions, notes, dispositions, and review timestamps are tracked in Investigation Case and audit history rather than in the deterministic outcome itself

### Mismatch Reason

A structured explanation for why a transaction is UNRECONCILED.

**Fields**
- reason_code: transaction_id_mismatch, mid_mismatch, amount_mismatch, action_type_mismatch
- description
- field_name(s)
- severity

**Relationships**
- One Reconciliation Outcome can include multiple Mismatch Reasons
- Mismatch Reasons are associated with the specific field values that differ across sources

### Processor Data Snapshot

The expected transaction population from the payment gateway compared with the processor-received population for a defined period and processor context.

**Fields**
- processor_id
- reporting_period
- expected_transaction_population
- received_transaction_population
- expected_transaction_count
- received_transaction_count
- missing_records
- unexpected_records
- completeness_gap_detected
- completeness_alert_status
- source_file_reference

**Validation Rules**
- expected_transaction_population and received_transaction_population must be compared by period and processor
- missing_records identifies expected transactions not present in the processor-received population
- unexpected_records identifies processor-received records that are not present in the expected gateway population
- completeness_gap_detected is true when expected and received populations differ materially or when required records are missing or unexpected
- the snapshot supports completeness comparison without prescribing physical storage or implementation details

### Investigation Case

A human-managed record representing an unreconciled transaction or issue requiring review.

**Fields**
- case_id
- transaction_id(s)
- case_status
- opened_at
- assigned_user
- evidence_summary
- audit_history
- final_disposition

**Relationships**
- A Case is linked to one or more Transaction Records
- A Case records all reviewer actions and decisions

### Lifecycle Event

A historical event associated with the transaction, including settlement, funding, disputes, and chargebacks where available.

**Fields**
- event_id
- transaction_id
- event_type
- event_timestamp
- source_system
- description

**Relationships**
- One Transaction Record can have many Lifecycle Events
- Lifecycle Event data may be partial if source data is missing or incomplete

### Operational Metric

A business-facing or operational measure used for monitoring performance and completeness.

**Fields**
- metric_name
- processor_id
- reporting_period
- value
- unit
- threshold_status

**Examples**
- expected_vs_received_transactions
- reconciled_vs_unreconciled_ratio
- material_mismatch_count_by_reason
- average_reconciliation_time
- sla_breach_indicator

## State Transitions

### Transaction Reconciliation State

1. Initial transaction record is ingested and normalized.
2. Deterministic comparison is performed against expected and received data.
3. If a material mismatch is detected, state becomes UNRECONCILED and mismatch reasons are attached.
4. If no material mismatch is detected, the matching score is evaluated.
5. If score meets threshold, state becomes RECONCILED.
6. If score is below threshold, state becomes UNRECONCILED and the transaction enters investigation.

### Case Management State

1. Case created for an UNRECONCILED transaction.
2. Evidence and audit history are attached.
3. Reviewer adds disposition or notes.
4. Final outcome is recorded and retained in audit history.

## Data Quality Rules

- A transaction must never disappear from the reconciliation result because of a mismatch.
- Any material mismatch must be visible as a reason on the transaction record and in downstream reporting.
- AI-generated outputs must not be treated as source-of-truth values.
- All access to transaction detail, merchant lookup, and case data must be permission-scoped and auditable.
