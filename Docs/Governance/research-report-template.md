# Section [NN]: [Topic Title] - Research Report

**Topic ID:** [Must match overall research plan index, e.g., IDR-SRV-001]<br>
**Report Status:** [Draft | In Review | Final]<br>
**Research Plan:** [Relative or absolute link to topic research plan]<br>
**Overall Research Plan:** [Relative or absolute link to controlling overall research plan]<br>
**Research Questions Covered:** [Count and short scope statement]<br>
**Methodology Used:** [Short summary of how research was performed]<br>
**Research Time:** [Actual time spent and date range]<br>
**Primary Source(s):**
- [Authoritative source URL/path]
- [Authoritative source URL/path]
**Supporting Resources:**
- [Related report or plan]
- [Related report or plan]
**Document Purpose:** [What decision(s) this report enables]<br>
**Author(s):** [Name/Team]<br>
**Accepted By:** [TBD until controlling-plan owner acceptance]<br>
**Acceptance Date:** [TBD until accepted]<br>
**Date:** [Month Day, Year]<br>
**Last Updated:** [Month Day, Year]

---

## Usage Rules

This report template is for topic-level reports that must correlate to governance topic plans.

1. `Topic ID` must exactly match the ID in the overall research plan index.
2. `Research Plan` must link to the corresponding topic-specific research plan.
3. Every conclusion must be backed by explicit, reproducible evidence and references.
4. Normative or authoritative claims must be grounded in the controlling source. Implementation behavior, examples, tests, discussions, and community practice are informative evidence unless the controlling source gives them another status.
5. Mutable technical evidence must identify a release, tag, commit, version, or dated retrieval sufficient to reproduce the finding.
6. Clearly distinguish source-backed findings, analyst inference, and project recommendations. Never present inference or implementation precedent as a standards obligation.
7. If a source is controlled, inaccessible, missing, or ambiguous, identify the limitation, do not invent its contents, narrow affected conclusions, and record what remains unresolved.
8. Reconcile material conflicts with accepted prior topic reports, or identify the conflict and required project decision explicitly.
9. Recommendations must be bounded, decision-usable, and understandable to the project lead, implementers, and later AI agents.
10. Report content is final only when all core questions from the topic plan are addressed or explicitly marked unresolved.
11. A topic is accepted for downstream use only when the controlling-plan owner has reviewed the report for plan alignment and decision usability, and the `Accepted By` and `Acceptance Date` fields are completed.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against Plan Success Criteria
10. Next Steps and Handoff
11. References
12. Appendices (Optional)

---

## 1. Executive Summary

Provide a concise, plain-language, decision-oriented summary that can stand on its own for the project lead, implementers, and later AI agents.

- Problem or decision context
- Most important findings
- Final recommendation
- Notable unresolved risk (if any)

---

## 2. Scope and Plan Alignment

Show explicit correlation to governance planning artifacts.

- Topic ID and title confirmation
- Topic plan objective coverage statement
- In-scope items completed
- Out-of-scope items (if any)

### Research Question Coverage Matrix

| Plan Question ID | Question (Short Form) | Coverage Status | Evidence Location |
|---|---|---|---|
| [Q1] | [Question summary] | [Complete/Partial/Unresolved] | [Section/appendix/reference] |
| [Q2] | [Question summary] | [Complete/Partial/Unresolved] | [Section/appendix/reference] |

---

## 3. Evidence Base

Document the evidence used and how it was validated.

### 3.1 Primary Sources Reviewed

| Source | Type | Version / Release / Commit / Status | Authority Class | Stable Anchor | Access Date | Availability / Limitations |
|---|---|---|---|---|---|---|
| [URL/path] | [Spec/code/repo/etc.] | [Identifier] | [Normative/profile/project/etc.] | [Clause/requirement/path/line] | [YYYY-MM-DD] | [Available/gap/limitation] |

### 3.2 Supporting Sources Reviewed

| Source | Version / Release / Commit | Relevance / Evidence Class | Stable Anchor | Access Date | Limitations |
|---|---|---|---|---|---|
| [URL/path] | [Identifier] | [Observed implementation/secondary/test/etc.] | [Path/section/line] | [YYYY-MM-DD] | [How used and limited] |

### 3.3 Evidence Quality Notes

- [Authority hierarchy used for this topic]
- [Controlled, inaccessible, missing, ambiguous, or mutable-source limitations]
- [Material evidence conflicts and how they were resolved]
- [Confidence note and effect on conclusions]

---

## 4. Findings by Research Question

Report findings grouped by topic-plan questions, not by ad hoc narrative. Label source-backed findings, analysis/inference, and project recommendations so their status cannot be confused.

### 4.1 [Question ID/Theme]

**Finding:** [Clear statement]

**Evidence:**
- [Direct source citation with specific section/path]
- [Direct source citation with specific section/path]

**Interpretation:** [What the evidence means for this topic]

### 4.2 [Question ID/Theme]

**Finding:** [Clear statement]

**Evidence:**
- [Direct source citation]

**Interpretation:** [Decision-relevant meaning]

---

## 5. Decision Analysis

Evaluate viable options and trade-offs before final recommendation.

| Option | Benefits | Costs/Risks | Standards/Compatibility Impact | Recommendation |
|---|---|---|---|---|
| [Option A] | [Benefit] | [Cost/risk] | [Impact] | [Keep/Reject/Conditional] |
| [Option B] | [Benefit] | [Cost/risk] | [Impact] | [Keep/Reject/Conditional] |

---

## 6. Key Recommendations

Provide explicit, bounded, actionable recommendations.

1. [Recommendation statement]
   - Rationale: [Short rationale]
   - Preconditions: [If any]
   - Priority: [High/Medium/Low]
2. [Recommendation statement]
   - Rationale: [Short rationale]
   - Preconditions: [If any]
   - Priority: [High/Medium/Low]

---

## 7. Implementation Implications and Estimates

Capture practical downstream impact for planning artifacts.

### 7.1 Implications

- [Architecture implication]
- [Implementation implication]
- [Testing or validation implication]

### 7.2 Effort/Complexity Estimate

| Work Item | Relative Complexity | Estimate | Assumptions |
|---|---|---|---|
| [Item] | [Low/Medium/High] | [Time/points] | [Assumptions] |

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- [Risk/constraint]
- [Risk/constraint]

### 8.2 Open Questions

- [Open question and why unresolved]
- [Open question and next action]

---

## 9. Validation Against Plan Success Criteria

Map report outcomes back to the topic plan success criteria.

| Topic Plan Success Criterion | Validation Status | Evidence |
|---|---|---|
| [Criterion] | [Met/Partially Met/Not Met] | [Section/reference] |
| [Criterion] | [Met/Partially Met/Not Met] | [Section/reference] |

---

## 10. Next Steps and Handoff

Define immediate follow-on work and ownership.

1. [Next action] - Owner: [Name/Team] - Due: [YYYY-MM-DD]
2. [Next action] - Owner: [Name/Team] - Due: [YYYY-MM-DD]

---

## 11. References

List all cited sources with reproducible links.

- [Full URL or repository path]
- [Full URL or repository path]
- [Full URL or repository path]

---

## 12. Appendices (Optional)

Include supporting detail that would otherwise disrupt report readability.

- Raw extraction tables
- Command logs
- Comparison matrices
- Supplemental examples

---

## Report Completion Checklist

- [ ] Topic ID matches overall research plan index
- [ ] Topic research plan is linked and aligned
- [ ] Core research questions are covered or explicitly unresolved
- [ ] Findings are evidence-backed with reproducible references
- [ ] Normative and informative evidence are classified and not conflated
- [ ] Mutable sources identify a version, release, tag, commit, or dated retrieval
- [ ] Controlled, inaccessible, missing, or ambiguous evidence limitations are explicit
- [ ] Source-backed findings, analyst inference, and project recommendations are distinguishable
- [ ] Conflicts with accepted prior reports are reconciled or explicitly escalated
- [ ] Executive summary is independently readable by the project lead, implementers, and later AI agents
- [ ] Recommendations are explicit and actionable
- [ ] Risks and open questions are documented
- [ ] Success criteria validation is complete
- [ ] Plan-owner acceptance and acceptance date are recorded before the topic is treated as complete downstream
- [ ] Next steps are assigned
