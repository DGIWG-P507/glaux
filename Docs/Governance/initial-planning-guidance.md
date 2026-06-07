# Initial Planning Guidance

## Purpose

This document defines the planning-document workflow for Glaux efforts. It standardizes **what to write**, **in what order**, and **why** so planning stays consistent, traceable, and execution-ready.

## Planning Document Set

Every major effort should produce three core planning documents:

1. Contribution and Goal Definition
2. Implementation Guide
3. Roadmap

## Document 1: Contribution and Goal Definition

### Objective

Establish the mission and boundaries of the effort before design or scheduling begins.

### What it should contain

- Problem statement and operational need
- Contribution goal (the intended outcome)
- Contribution definition (what is included)
- Explicit scope boundaries (what is out of scope)
- Success criteria and quality expectations
- Primary assumptions and constraints
- Major stakeholders and intended users

### Why it comes first

Without a settled goal and scope baseline, architecture and planning drift quickly. This document is the contract for the rest of planning.

## Document 2: Implementation Guide

### Objective

Translate the approved goal into a technical execution design with clear implementation direction.

### What it should contain

- Architectural baseline and component boundaries
- Data model and API behavior decisions
- Integration points and dependency strategy
- Security, validation, and error-handling approach
- Conformance and verification strategy
- Testing approach (unit, integration, interoperability)
- Risks, mitigations, and design tradeoffs

### Why it comes second

The implementation guide depends on a fixed contribution definition. It should not be authored against moving goals.

## Document 3: Roadmap

### Objective

Convert the implementation design into an executable plan of work.

### What it should contain

- Ordered phases or work packages
- Task breakdown by capability or component
- Dependencies and sequencing rules
- Entry/exit criteria per phase
- Milestones and expected deliverables
- Validation checkpoints and quality gates
- Tracking model for progress and evidence

### Why it comes third

A roadmap without settled scope and design produces inaccurate estimates and unstable sequencing. Scheduling is reliable only after the first two artifacts are baselined.

## Required Creation Order

1. Contribution and Goal Definition
2. Implementation Guide
3. Roadmap

This order is mandatory for predictable planning quality.

## Governance Rules for This Workflow

- Do not start the Implementation Guide until Contribution and Goal Definition is approved.
- Do not start the Roadmap until the Implementation Guide is baselined.
- Record major planning changes as version updates, not silent rewrites.
- Keep traceability from goal -> design -> roadmap tasks.
- Keep each document independently readable and reviewable.

## Deliverable Quality Standard

A planning set is complete only when:

- Scope is explicit and testable
- Design choices are documented and defensible
- Roadmap sequencing is dependency-aware
- Verification expectations are present from the start

This guidance governs creation of the next three planning templates and all future planning packages in this repository.
