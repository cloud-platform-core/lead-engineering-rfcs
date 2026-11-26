# Lightweight RFCs: a worked demo

**State:** worked example. Part of [how technical decisions flow](https://github.com/cloud-platform-core).

This repository shows one way to lead technical work: write down significant decisions as short RFCs (requests for comments), then hand the build to engineers or contractors as a spec with testable acceptance criteria.

The setting is cloud and platform engineering for life-science research. **Example Lab is fictional**, as are its people, systems, targets and events. The software tools named in the example are real, and claims about them link to their documentation.

## Why

- Problems surface while changes are still cheap.
- Later readers see what was decided, why, and what was turned down.
- People can build without having been in the original conversation.
- Anyone can propose, and the decider is named in advance.

## When to write one

Write an RFC if any of these is true:

- The choice is costly to reverse.
- It affects more than one team, or an interface others depend on.
- It adds a tool, service or data path that someone must operate.
- It changes how data is protected, or commits significant money or time.
- Capable people disagree, and the disagreement keeps coming back.

Do not write one for:

- Bug fixes, routine upgrades or refactoring that keeps behaviour.
- Choices one team can easily undo.
- Work already covered by an accepted RFC. Write a spec or open a pull request.
- Urgent incident response. Act first, then write an RFC if a lasting decision remains.

If unsure, write half a page on the problem and ask.

## Lifecycle

| Status | Meaning |
| --- | --- |
| Draft | Author is shaping it. Early comments welcome. |
| In review | Open pull request, reviewers named, end date set. |
| Accepted | Decision made. Work may start. |
| Rejected | Decided against. Merged anyway, with the reason. |
| Postponed | Not worth deciding now. States what would reopen it. |
| Withdrawn | Author stopped pursuing it. |
| Superseded | Replaced by a later RFC, which it links to. |

To propose, copy the [template](template/rfc-template.md) to `rfcs/NNNN-short-name.md` and open a pull request.

## Review and decisions

- The **author** states the problem, gathers input and answers comments.
- **Required reviewers** own affected systems, data or users.
- The **decider** is one named person who did not write the RFC, often a principal engineer. Budget or staffing questions go to the accountable manager.

1. Review runs for five working days by default. The decider may extend it.
2. Comments are marked **blocking** or **suggestion**. A blocking comment says what would resolve it.
3. The decider then posts the proposed outcome and allows two working days for new objections.
4. The group seeks agreement. Without it, the decider decides and records the dissent in the RFC.
5. Silence from a required reviewer is not approval. The decider chases it or names a substitute.

A decision is reopened only on new information or a stated revisit trigger, through a new RFC that supersedes the old one. Acceptance sets direction; it does not by itself assign people or dates.

## Index

| No. | RFC | Status | Spec |
| --- | --- | --- | --- |
| 0001 | [Workflow engine for analysis pipelines](rfcs/0001-workflow-engine.md) | Accepted (fictional) | [MVP spec](specs/0001-workflow-engine-mvp.md) |

## Proposals

A proposal is shorter than an RFC. It moves an existing decision forward using evidence.

| Proposal | Status |
| --- | --- |
| [Move Nextflow to Adopt](proposals/example-nextflow-to-adopt.md) | In review (fictional) |

Also: [How we build](https://github.com/cloud-platform-core/tl-guidelines/blob/main/how-we-build.md), from requirement to release, and the [technology agreement](https://github.com/cloud-platform-core/tl-guidelines/blob/main/agreement.md) that records where decisions land.

## Sources

This process adapts widely used public practice:

- [Rust RFCs](https://github.com/rust-lang/rfcs): proposals as pull requests, a threshold for "substantial" changes, a final comment period with a proposed outcome (shortened here), and acceptance that does not guarantee implementation.
- [PEP 1](https://peps.python.org/pep-0001/), the Python Enhancement Proposal process: explicit statuses, a named decision maker, and a record of rejected ideas.
- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) by Michael Nygard, and [adr.github.io](https://adr.github.io/): short decision records with context and consequences, superseded and not deleted.
