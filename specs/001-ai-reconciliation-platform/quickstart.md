# Quickstart Validation Guide

## Purpose

This guide provides end-to-end validation scenarios for the AI Reconciliation Platform MVP. It is intentionally focused on proving business behavior and operational outcomes rather than implementation code.

## Prerequisites

- A valid reconciliation batch containing expected transaction activity and processor-received activity for a defined reporting period
- At least one merchant and one processor context with known transaction records
- User roles configured for Finance Operations, Merchant Support, merchant self-service, and investigation review
- Access controls and audit logging enabled

## Validation Scenarios

### 1. Deterministic reconciliation outcome

1. Load a sample reconciliation batch with a mix of matched and mismatched transactions.
2. Run the reconciliation process for the selected period.
3. Verify that every transaction appears in the final output.
4. Confirm that transactions with transaction ID, MID, amount, or action type mismatches are marked UNRECONCILED and include their specific mismatch reason(s).
5. Confirm that non-material mismatches are evaluated via the configured matching score, resulting in RECONCILED or UNRECONCILED as appropriate.

**Expected outcome**: Final output contains a reliable RECONCILED/UNRECONCILED result for each transaction and includes explicit mismatch reasons where applicable.

### 2. Transaction-centric investigation

1. Choose a transaction ID from an UNRECONCILED result.
2. Open the transaction investigation view.
3. Verify that the view shows the current status, source or processor file origin, matching details, differing fields, mismatch reasons, and relevant lifecycle information where available.

**Expected outcome**: An analyst can understand the reconciliation state and evidence without switching to unrelated screens or data sources.

### 3. Merchant support and self-service access

1. Sign in as a Merchant Support user and lookup a known transaction.
2. Verify access is limited to the authorized data and explanation level.
3. Sign in as a merchant user and access a self-service transaction lifecycle view.
4. Confirm the user can view the transaction lifecycle, including reconciliation, funding, settlement, dispute, and chargeback events where corresponding data exists.
5. Attempt unauthorized access for another merchant’s record and verify the request is denied.

**Expected outcome**: Access is role-based, least-privilege, and auditable.

### 4. Processor-data completeness monitoring

1. Configure expected gateway activity and compare it with received processor data for a processor and period.
2. Remove or withhold a portion of the processor file to simulate incompleteness.
3. Run the completeness comparison.
4. Verify that an alert is generated for the completeness gap.

**Expected outcome**: Missing or incomplete processor data is surfaced as an operational issue, not silently ignored.

### 5. Business metrics validation

1. Run a representative reconciliation batch and record the results.
2. Open the business metrics dashboard.
3. Verify the system displays expected vs received transactions by processor, reconciled vs unreconciled counts and percentages, material mismatch counts by reason, average reconciliation time, and SLA breaches.

**Expected outcome**: Business users can see operational performance and mismatch trends without implementation detail.

### 6. Audit history and AI governance

1. Open an unreconciled case and review the investigation history.
2. Add a reviewer action or disposition.
3. Verify the action is retained in audit history.
4. Invoke the governed read-only AI search capability using the case/transaction context and confirm the output clearly distinguishes facts, inference, and recommendation.
5. Confirm AI never changes authoritative financial records.

**Expected outcome**: The investigation remains traceable, explainable, and separate from financial authority.

## Exit Criteria

The feature is ready for the next phase when all scenarios above can be completed with evidence that the platform preserves deterministic financial truth, enforces least privilege, and keeps investigation and reconciliation flows auditable.
