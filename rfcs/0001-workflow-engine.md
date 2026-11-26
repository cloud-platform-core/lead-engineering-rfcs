# RFC 0001: Workflow engine for Example Lab analysis pipelines

> Fictional example. Example Lab, its situation and its targets are invented. The tool comparison fits this scenario only and is not a general ranking.

| Field | Value |
| --- | --- |
| Status | Accepted |
| Author | Platform engineer |
| Decider | Principal engineer |
| Required reviewers | Research Computing lead, Genomics Core scientist, data governance officer |
| Spec | [0001: Read QC pipeline MVP](../specs/0001-workflow-engine-mvp.md) |

## Summary

Use Nextflow for new production analysis pipelines. Prove it with one sequencing read quality control (QC) pipeline on our [Slurm](https://slurm.schedmd.com/overview.html) cluster and on [AWS Batch](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html). The main cost is that staff must learn a new workflow language.

## Problem

- Production analyses are shell scripts launched by hand.
- After a partial failure, someone works out what finished or reruns everything.
- Results do not record the code and software versions that produced them.
- Scripts assume the local cluster, so teams needing extra capacity copy and edit them.

Constraints: the Slurm cluster stays; an AWS account exists; staff mostly write Python and shell; two engineers will maintain the result; we do not run Kubernetes.

Non-goals: a web interface, notebooks, and rewriting stable scripts.

## Criteria

| ID | Criterion | Weight |
| --- | --- | --- |
| C1 | Same workflow logic runs on Slurm and AWS Batch | Must |
| C2 | Restart after failure without repeating completed steps | Must |
| C3 | Software environment can be pinned for each step | Must |
| C4 | Current staff can maintain it | Should |
| C5 | Existing domain pipelines can be reused | Should |
| C6 | No new platform to operate | Should |

## Options

### A. Keep scripts and add conventions

- For: no new tool or skills; nothing to migrate.
- Against: meeting C1 to C3 means building and maintaining our own workflow engine.

### B. Nextflow

- For: documented [executors](https://docs.seqera.io/nextflow/executor) include Slurm and AWS Batch; AWS support is a [core plugin](https://docs.seqera.io/nextflow/plugins), installed automatically when requested. [Resume](https://docs.seqera.io/nextflow/cache-and-resume) reuses completed tasks. Supports [several container runtimes](https://docs.seqera.io/nextflow/container), including Docker and Apptainer. [nf-core](https://nf-co.re/) is a community building Nextflow pipelines to shared guidelines. [Apache 2.0 license](https://github.com/nextflow-io/nextflow).
- Against: [built on Groovy](https://github.com/nextflow-io/nextflow), unfamiliar to our staff, and [needs Java](https://docs.seqera.io/nextflow/install). On AWS Batch, [every process needs a container and the work directory must be in object storage](https://docs.seqera.io/nextflow/executor). Shared file systems with inconsistent timestamps [can defeat resume](https://docs.seqera.io/nextflow/cache-and-resume) in the default mode.

### C. Snakemake

- For: a [Python-based workflow language](https://snakemake.readthedocs.io/en/stable/), the best fit for our skills. [Conda and Apptainer per rule](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html). [Reruns jobs based on changes](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html) to inputs and code. A public [workflow catalog](https://snakemake.github.io/snakemake-workflow-catalog/). [MIT license](https://github.com/snakemake/snakemake/blob/main/LICENSE.md).
- Against: cluster and cloud backends are [separate executor plugins](https://snakemake.readthedocs.io/en/stable/executing/executors.html), including ones for [Slurm](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) and [AWS Batch](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/aws-batch.html), to track and upgrade. The AWS Batch plugin requires job images that include Snakemake and a matching storage plugin.

### D. Argo Workflows

- For: [container-native engine](https://argo-workflows.readthedocs.io/en/latest/) and a graduated CNCF project; a natural candidate for a team already running Kubernetes.
- Against: it orchestrates jobs on Kubernetes. We would have to operate a new platform and solve Slurm integration separately.

### Assessment

| Criterion | A: Scripts | B: Nextflow | C: Snakemake | D: Argo |
| --- | --- | --- | --- | --- |
| C1 Slurm and Batch | No | Yes | Yes, with plugins | No |
| C2 Restart | No | Yes | Yes | Not assessed |
| C3 Pinned software | No | Yes | Yes | Not assessed |
| C4 Maintainable | Yes | Partly | Yes | Partly |
| C5 Reuse | No | Yes | Yes | Not assessed |
| C6 No new platform | Yes | Yes | Yes | No |

## Decision

Adopt **Nextflow (B)** for new production pipelines.

B and C both meet every "must", and C is stronger on maintainability. We choose B for two reasons:

1. Both target backends are covered in Nextflow's own executor documentation, and its AWS plugin is a core one. For two maintainers, we judge this less integration work than selecting and tracking Snakemake's executor and storage plugins.
2. Assumption for this scenario: Genomics Core has reviewed both catalogs and wants an nf-core pipeline for its next assay.

We accept a slower start. Existing scripts move only when another change justifies it.

## Dissent

The Research Computing lead preferred Snakemake: staff know Python, and demand for cloud capacity is unproven. The decider kept Nextflow but made staff maintainability a revisit trigger and a handover requirement.

## Risks and mitigations

| Risk | Mitigation | Owner |
| --- | --- | --- |
| Staff cannot maintain the pipeline | Small MVP, staff review of every change, tested runbook | Research Computing lead |
| Resume fails on the shared file system | Test on the real cluster; evaluate the documented `lenient` cache mode | Platform engineer |
| Cleanup removes files needed to resume | Set and document work directory retention | Research Computing lead |
| Unexpected cloud spend | Capacity limit and budget alert before the first cloud run | Platform engineer |
| Research data leaves approved systems | MVP uses synthetic data in the cloud; real data needs separate approval | Data governance officer |
| An upgrade changes behaviour | Pin Nextflow and plugin versions, container digests and release tags | Platform engineer |
| Knowledge leaves with the contractor | Handover is an acceptance criterion | Research Computing lead |

## MVP scope

**In:** one read QC pipeline reproducing the current script's metrics; Slurm and AWS Batch profiles; input validation; resume; a run manifest; runbook and handover.

**Out:** other pipelines, new scientific methods, real research data in the cloud, cost tuning.

## Success signals and revisit trigger

Checked 90 days after the pilot starts:

- At least 8 in 10 pilot read QC runs use the new pipeline.
- Each of 10 randomly chosen results traces to a commit, Nextflow version and container digests.
- Interrupted runs are resumed, with no full restarts caused by the pipeline.
- Two staff engineers have each released a change without contractor help.

If the last signal fails, the principal engineer opens a new RFC that reconsiders Snakemake. At six months, if nobody needs the cloud profile, review whether to keep supporting it.

## Open questions

Neither blocks the MVP.

- How long should work directories be kept? Owner: Research Computing lead.
- May real research data run in the cloud? Owner: data governance officer; needs its own RFC.

*Tool documentation checked 2026-11-26. Recheck before relying on it.*
