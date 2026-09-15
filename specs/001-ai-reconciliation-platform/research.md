# Research: AI Reconciliation Platform

## Decision: Reconciliation outputs use only two legal statuses

**Decision**: Every transaction in scope must ultimately be classified as either RECONCILED or UNRECONCILED.

**Rationale**: The approved business requirement expressly removes any intermediate “requiring review” state as a financial classification. This simplifies operational and reporting logic and ensures a single authoritative status for downstream systems and financial reporting.

**Alternatives considered**: Creating a third status for “pending review” or “needs investigation” was considered but rejected because it would blur the financial truth model and create ambiguity in reporting and audit trails.

---

## Decision: Material mismatch rules are deterministic and explicit

**Decision**: Material mismatches are defined as transaction ID mismatch, MID mismatch, amount mismatch, or action type mismatch. Any material mismatch yields UNRECONCILED status and retains the transaction in final output with the specific mismatch reason(s).

**Rationale**: These fields directly affect financial correctness and source-of-truth alignment. They are the minimum set necessary to prevent silent loss and maintain operational trust.

**Alternatives considered**: Allowing mismatches to be dropped or masked behind a generic exception bucket was rejected because it would hide financially relevant discrepancies and compromise auditability.

---

## Decision: Matching score is used only when no material mismatch exists

**Decision**: When no material mismatch is detected, the platform computes a configurable matching score and compares it against a business-configured threshold. The transaction becomes RECONCILED only if the score meets the threshold; otherwise it becomes UNRECONCILED and is available for investigation.

**Rationale**: The business requirement explicitly prohibits inventing a hard-coded threshold and requires threshold configuration to be set by the business during configuration or pilot validation.

**Alternatives considered**: Hard-coded thresholds or binary matching without score-based fallback were rejected because they would force arbitrary policy decisions and reduce business flexibility.

---

## Decision: Investigations are transaction-centric and read-only for most roles

**Decision**: The analyst investigation view starts with a transaction ID and exposes reconciliation status, source/processor file origin, matching details, differing fields, mismatch reasons, and available lifecycle history.

**Rationale**: Financial investigation work is usually initiated from a single transaction and requires a single, complete evidence view to resolve a discrepancy without browsing disconnected data sources.

**Alternatives considered**: Batch-level investigation or list-first views were rejected because they raise the time to understand a specific issue and weaken traceability for affected transactions.

---

## Decision: Access is role-based and least-privilege by design

**Decision**: Merchant Support and merchant self-service users receive controlled read-only access appropriate to their permissions. Merchant Support can access limited high-level explanations; merchants can access their own transaction lifecycle and status within their scope.

**Rationale**: This balances business support needs with security and audit requirements and aligns with the constitution’s least-privilege rule.

**Alternatives considered**: Unrestricted access to all transaction history or a single all-access role was rejected as a security risk and inconsistent with the required authorization controls.

---

## Decision: Processor completeness monitoring is a required operational control

**Decision**: The platform compares expected transaction activity from the gateway source of truth to processor data received for the relevant period and processor and raises an alert when the completeness gap is detected.

**Rationale**: Reconciliation quality depends on data completeness, not just classifications. Without explicit completeness monitoring, the business may mistake a missing feed for a clean reconciliation result.

**Alternatives considered**: Silent fallback to “no data equals no issue” was rejected because it would create false confidence and masked operational gaps.

---

## Decision: Metrics are business-facing and operationally actionable

**Decision**: Finance and operations metrics include expected vs received transactions by processor, reconciled vs unreconciled counts and percentages, material mismatch counts by reason, average time to reconcile, and SLA breaches.

**Rationale**: These measures directly support business oversight and service quality decisions without exposing implementation details.

**Alternatives considered**: Purely technical metrics such as queue length or raw event counts were considered insufficient because they do not show financial business outcomes.

---

## Decision: AI supports investigation but never financial truth

**Decision**: AI may summarize evidence, prioritize cases, explain discrepancy patterns, and propose recommendations, but it cannot modify authoritative records or replace the deterministic reconciliation logic.

**Rationale**: The constitution and the approved requirements require AI to remain explainable, bounded, and audit-distance from hazardous financial actions.

**Alternatives considered**: Allowing AI to propose or execute financial state changes was rejected because it violates financial truth and human-controlled governance requirements.

---

## Decision: Controlled natural-language search must be translated into a constrained intent

**Decision**: Natural-language search is allowed only as a read-only, role-controlled capability. The AI converts user intent into a constrained structured search model and never generates unrestricted SQL or raw database queries.

**Rationale**: This preserves user convenience while preventing prompt injection, unauthorized data access, and database misuse.

**Alternatives considered**: Free-form query generation was rejected due to security and authorization concerns.

---

## Decision: Future-scope items remain explicitly out of MVP

**Decision**: Automated recovery of missing transactions and predictive reconciliation intelligence are future scope and not required for the initial MVP.

**Rationale**: The feature specification explicitly calls these items out as excluded so the MVP remains bounded and testable.

**Alternatives considered**: Including them in the MVP would expand risk and delay delivery of the required reconciliation foundation.
