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
- **Severity:** High (unsafe or likely to stop delivery) / Medium (worth addressing within the phase) / Low (address when convenient) / Note (information only).
- **Suggested owner:** the existing issue that could carry it, or "project-lead decision".
- **Status:** Open / Adopted (link to the action-list entry) / Not adopted (link to where the project lead's decision is recorded) / Withdrawn (reason).
```

Severity is the reviewer's assessment. Nothing in this file blocks work or becomes a requirement until the project lead adopts it through the existing change process.

A standards obligation, a project choice and a recommendation are different things. Say which one each finding is.
