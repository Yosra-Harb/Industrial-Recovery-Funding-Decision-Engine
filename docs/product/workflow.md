# End-to-End Workflow

## Primary Data Flow

```text
Factory Master Profile
        +
Recovery Application
        +
Funding Provider
        +
Funding Programme
        +
Portfolio Strategy
        +
Programme Constraints
        ↓
1. Data Validation
        ↓
2. Eligibility Screening
        ↓
3. Recovery Needs Diagnosis
        ↓
4. Intervention Matching
        ↓
5. Feature / Indicator Generation
        ↓
6. Impact & Priority Assessment
        ↓
7. Candidate Factory–Intervention Set
        ↓
8. Portfolio Optimization
        ↓
9. Explainable Recommendation
        ↓
10. Human Review / Approval
        ↓
11. Funding Decision Record
        ↓
12. Post-Funding Monitoring
        ↓
13. Expected vs Actual Analysis
        ↓
14. Learning Dataset
        ↓
15. Future Predictive Models
        ↓
16. Future Re-optimization
```

## Automation Principle

The intended operating model is **automation-first, exception-driven human review**.

Routine cases should proceed automatically when policy permits.

Human attention is prioritized for:

- incomplete evidence
- conflicting data
- policy exceptions
- high-risk cases
- high-value cases where required
- final governance approval
- authorized overrides

## State Model — Initial Draft

```text
DRAFT
→ SUBMITTED
→ VALIDATING
→ INCOMPLETE | READY
→ ELIGIBILITY_CHECK
→ INELIGIBLE | NEEDS_REVIEW | ELIGIBLE
→ ASSESSMENT
→ OPTIMIZATION_READY
→ RECOMMENDED
→ COMMITTEE_REVIEW
→ APPROVED | REJECTED | DEFERRED
→ FUNDED
→ MONITORING
→ COMPLETED
```

Every state transition that affects a funding decision must be auditable.
