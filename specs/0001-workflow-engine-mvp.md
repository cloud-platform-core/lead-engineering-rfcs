# Spec 0001: Read QC pipeline MVP

> Fictional example. Implements [RFC 0001](../rfcs/0001-workflow-engine.md).

| Role | Responsibility |
| --- | --- |
| Research Computing lead | Owns this spec and accepts the work |
| Contractor engineer | Builds, tests, documents and hands over |
| Platform engineer | Provides infrastructure and reviews code |
| Genomics Core scientist | Confirms outputs are correct |
| Principal engineer | Decides changes that affect the RFC |

## Outcome

A staff member can run read QC from a samplesheet on Slurm or AWS Batch, resume it after an interruption, and show which versions produced the results. The RFC's MVP scope applies: **only synthetic data goes to the cloud.**

## Example Lab provides

- Repository, CI runner and access.
- The current QC script and metric definitions.
- A test set of 8 synthetic samples with an expected `summary.csv`.
- A Slurm account and shared scratch space.
- An AWS Batch queue, job role, storage bucket, capacity limit and budget alert. Building these is not contractor work.

Report a missing item as soon as it blocks work.

## Interface

```
nextflow run <repository> -r <tag> -profile <slurm|awsbatch> \
  --input samples.csv --outdir <path>
```

- **Input:** CSV with `sample_id`, `reads_1` and optional `reads_2`.
- **Output:** a new directory per run, `<outdir>/<tag>/<run-id>/`, holding one report per sample, `summary.csv`, `manifest.json` and the Nextflow [trace file](https://docs.seqera.io/nextflow/reports).
- **Configuration:** site values live outside pipeline logic. A `test` [profile](https://docs.seqera.io/nextflow/config) supports CI.

## Work breakdown

Deliver in order, in small pull requests, with a short demonstration after each package. WP3 and WP4 may run in parallel. Sizes are relative.

| Package | Deliverable | Needs | Size | Criteria |
| --- | --- | --- | --- | --- |
| WP1 | Project skeleton, stubs, CI | Repository | S | AC-1 |
| WP2 | Validation and QC steps on Slurm | WP1, test set | M | AC-2, 5, 6 |
| WP3 | Resume, manifest, output layout | WP2 | S | AC-4, 7, 9 |
| WP4 | AWS Batch profile | WP2, cloud resources | M | AC-3, 8 |
| WP5 | Release, runbook, handover | WP3, WP4 | S | AC-10 |

First two working days: the contractor returns questions, assumptions and risks as comments on this spec.

## Acceptance criteria

Each is pass or fail. Attach evidence (CI link, log or file) to the pull request.

| ID | Criterion | Evidence |
| --- | --- | --- |
| AC-1 | Every process has a [stub](https://docs.seqera.io/nextflow/process). On a clean checkout, `nextflow run . -stub-run -profile test` exits 0 in CI. | CI log |
| AC-2 | The test set on `slurm` exits 0 and yields 8 sample reports and a `summary.csv` identical to the expected file. | Comparison output |
| AC-3 | The same commit and test set on `awsbatch` yield the same `summary.csv` as AC-2. | Comparison output |
| AC-4 | Stop a Slurm run after at least 3 samples finish, then rerun with `-resume`. No finished task is submitted again, and the output passes AC-2. | Scheduler records, both trace files |
| AC-5 | A missing column, duplicate `sample_id` or missing input file each causes a non-zero exit before any task is submitted. The message names the fault and, where relevant, the row. | Automated tests |
| AC-6 | Every container is referenced by digest. CI fails when a tag-only reference is introduced. | Passing and deliberately failing CI runs |
| AC-7 | `manifest.json` records commit, tag, Nextflow and plugin versions, container digests, parameters and input checksums. A reviewer reproduces `summary.csv` using it and the inputs. | Manifest, comparison output |
| AC-8 | No credentials or account identifiers are committed. Site values are supplied by configuration. A secret scan of the full history has no unresolved findings. | Scan report, review |
| AC-9 | Two runs with the same inputs, tag and `--outdir` leave both result sets intact. | Directory listing, checksums |
| AC-10 | A staff engineer who did not write the code launches, resumes and rolls back a run using only the runbook. | Reviewer's notes, resulting fixes |

## Release

Releases are immutable Git tags using [Semantic Versioning](https://semver.org/), starting in the 0.x series. Production runs select a tag with [`-r`](https://docs.seqera.io/nextflow/reference/cli/run), a Nextflow version with [`NXF_VER`](https://docs.seqera.io/nextflow/updating-nextflow) and pinned [plugin versions](https://docs.seqera.io/nextflow/plugins).

| Stage | Exit condition |
| --- | --- |
| 1. Candidate | AC-1 to AC-9 pass on the tagged commit |
| 2. Shadow | Three pilot batches run on Slurm beside the old script; the scientist accepts every difference |
| 3. Pilot | AC-10 passes; four weeks as the primary method with no rollback |
| 4. Default | Owner approves; old script is archived read-only |

## Rollback

**Triggers:** unexplained differences in results; a release fails work the previous one completed; suspected exposure of credentials or data (also stop runs and follow the security process).

1. Stop new runs on the affected release and tell users.
2. Run the previous tag. Before a second release exists, use the old script.
3. Keep affected outputs, mark them as suspect, and rerun those samples. Do not assume `-resume` will reuse work across releases.
4. Record the cause. Fix it in a new tag; never move an existing one.

The old script stays runnable until stage 4.

## Done and change control

Done means all criteria pass with evidence, the runbook is merged, a handover session is held, no blocking review comments remain, and the owner accepts.

If a criterion cannot be met or proves wrong, say so within one working day and propose an edit to this spec by pull request. The owner approves it. A change that affects the RFC decision goes to the principal engineer.
