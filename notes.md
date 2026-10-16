Build a **Feature Parity Coverage Dashboard** for the CDE → DDH broad dissemination migration to Neo.

**Objective:** Track implementation progress, test execution and behavioural parity between CDE (legacy) and DDH (replacement), using historical documents published within the last six months.

### 1. Feature Coverage Matrix

Create a dashboard table with the following columns:

| Feature | Implementation Status | Tests Identified | Tests Executed | Passed | Failed | Parity % |
|---|---|---:|---:|---:|---:|---:|
| Standard research publication | Complete | 15,000 | 15,000 | 15,000 | 0 | 100% |
| Restricted research | In Progress | 2,000 | 1,500 | 1,450 | 50 | 96.7% |
| Amendments | Pending | 300 | 0 | 0 | 0 | — |
| Embargoed reports | Complete | 150 | 150 | 150 | 0 | 100% |
| Withdrawals | Pending | 40 | 0 | 0 | 0 | — |
| Neo-first sequencing | In Progress | 17,490 | 10,000 | 9,990 | 10 | 99.9% |

All values above are illustrative. Actual values must come from the test harness.

### 2. Summary Metrics

Display KPI cards above the table:

- **Features Implemented:** X / Total
- **Features With Verified Parity:** X / Total
- **Tests Identified:** Total historical test cases
- **Tests Executed:** X / Total (%)
- **Tests Passed:** X (% of executed)
- **Tests Failed:** X
- **Critical Parity Failures:** X

### 3. Feature Tracking

Each feature should support:

- Implementation status: Pending / In Progress / Complete / Blocked
- Parity status: Not Tested / Testing / Passed / Failed
- Priority: Critical / High / Medium
- Assigned owner
- Link to associated test cases
- Known differences and outstanding issues

### 4. Test Execution Tracking

Each test should record:

- Document ID
- Feature(s) covered
- CDE expected result
- DDH actual result
- Pass/fail outcome
- Execution timestamp
- Failure reason or discrepancy category

A single document may cover multiple features. Ensure test counts are calculated per feature without incorrectly double-counting unique documents in overall totals.

### 5. Dashboard Requirements

- Progress bars for implementation and test execution.
- Colour-coded statuses.
- Filters for feature, priority, status and failures.
- Ability to drill down from a feature into its associated documents and test results.
- Clear distinction between **not implemented**, **not tested**, and **tested but failing**.
- Parity % = Passed / Executed × 100. Show N/A when no tests have executed.
- Use the latest valid test execution per document/feature when calculating current parity, while retaining historical runs.
- Support exporting the results to CSV.

### 6. Technical Requirements

Inspect the existing codebase and test harness before implementing anything. Reuse existing data models, APIs, UI components and project conventions.

Do not invent test results or hardcode metrics. If actual execution data is unavailable, display empty states or clearly labelled sample data.

Design the dashboard so that additional CDE features and test scenarios can be added without modifying dashboard logic.

**Expected outcome:** A single dashboard that shows what DDH functionality has been implemented, what has been tested, what matches CDE, what is failing and what remains outstanding.

The dashboard should make it easy to answer:

**"How confident are we that DDH has achieved feature parity with CDE for broad dissemination to Neo?"**