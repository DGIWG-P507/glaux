# Phase 1 implementation review — findings

[Review start page](README.md) · [Evidence](evidence/)

**No findings yet.** No review step has run. The [scoping observations](evidence/00-scoping-observations.md) are starting points for the steps to check, not findings.

## Format

Number findings `P1-01`, `P1-02`, … and never reuse a number. If a finding is withdrawn, keep it and mark it withdrawn with the reason.

```markdown
### P1-NN — Short title

**In plain English:** One or two sentences a non-developer can act on.

- **Step:** which review step found it, with a link to its evidence file.
- **Examined:** server commit (and planning commit if relevant).
- **What was seen:** the specific evidence, with file/line or run links.
- **Why it matters:** the consequence if left alone.
- **Severity:** Blocking (unsafe or will stop delivery) / Important (fix within the phase) / Minor (fix when convenient) / Note (information only).
- **Suggested owner:** the existing issue that should carry it, or "project-lead decision".
- **Status:** Open / Adopted (link to the action-list entry) / Declined (reason) / Withdrawn (reason).
```

A standards obligation, a project choice and a recommendation are different things. Say which one each finding is.
