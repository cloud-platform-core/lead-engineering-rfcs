# Example proposal: Move Nextflow to Adopt

> Fictional example. Example Lab, its people, results and numbers are invented. It shows how the [team technology agreement](https://github.com/cloud-platform-core/tl-guidelines/blob/main/agreement.md) works once something has been built.

| Field | Value |
| --- | --- |
| Status | In review |
| Change | Nextflow, Trial to Adopt, for production pipelines on Slurm |
| Level | Standards and defaults, technology register |
| Author | Platform engineer |
| Owner | Research Computing lead |
| Decider | Principal engineer |
| Based on | [RFC 0001](../rfcs/0001-workflow-engine.md) and [Spec 0001](../specs/0001-workflow-engine-mvp.md) |

## Summary

Make Nextflow the default for new production pipelines on Slurm. The pilot met the four success signals set in RFC 0001. AWS Batch stays in Trial because there has been little demand for it.

## Evidence

Checked 90 days after the pilot started. All results are invented.

| Success signal from RFC 0001 | Result | Met |
| --- | --- | --- |
| At least 8 in 10 pilot read QC runs use the new pipeline | 23 of 26 runs | Yes |
| 10 random results trace to a commit, version and container digests | 10 of 10 | Yes |
| Interrupted runs resume, with no full restarts caused by the pipeline | 4 interruptions, 4 resumed | Yes |
| Two staff engineers each release a change without contractor help | 2 engineers, 1 release each | Yes |

## Checks before Adopt

| Check | State |
| --- | --- |
| Done real work | Read QC pilot on Slurm |
| Two people can operate it | Both staff engineers have released a change |
| Runbook tried by a non-author | Done under acceptance criterion AC-10 |
| Security, data and cost | Reviewed for Slurm; real research data in the cloud is not approved |
| Way out | Containers can be reused; workflow logic would need rewriting |

## What changes

- New production pipelines on Slurm use Nextflow without further approval.
- Existing scripts move only when another change justifies it.
- The register entry names its owner and is reviewed in six months.

## What does not change

- AWS Batch stays in **Trial**, using synthetic data only. At the six-month review in RFC 0001, keep it, adopt it or drop it.
- Exploratory analysis and notebooks are outside this proposal.

## Dissent

None open. The earlier preference for Snakemake, recorded in RFC 0001, stays on file with its revisit trigger.

## Risks

| Risk | Mitigation | Owner |
| --- | --- | --- |
| Only two people know the tool well | A third engineer releases a change before the six-month review | Research Computing lead |
| An upgrade changes behaviour | Keep versions pinned and test upgrades against the expected results | Platform engineer |

## Revisit

Reopen if fewer than two staff engineers can maintain the pipelines, or if an upgrade cannot reproduce the expected results.

## Open question

How long should work directories be kept? Owner: Research Computing lead. This does not block the move.
