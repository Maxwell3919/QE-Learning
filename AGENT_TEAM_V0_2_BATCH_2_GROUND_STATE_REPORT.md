# Agent Team v0.2 Batch 2 Ground-State Report

## Scope

Batch 2 completes the ground-state workflow handbook slice without changing electronic, phonon, advanced, theory-minimum or physics-judgement正文 pages. The batch focuses on ground-state workflow readability, evidence-table consistency and the first workflow diagram.

## Subagents used

| Subagent | Role | Target pages | Output |
|---|---|---|---|
| R2-1 SCF / NSCF research | Ground-state data-edge review | `scf.md`, `nscf.md` | Existing pages already cover `prefix/outdir`, output evidence and downstream gates |
| R2-2 relax / vc-relax research | Structure optimization boundary | `relax.md`, `vc-relax.md` | Existing pages already stress final static SCF after optimization |
| R2-3 convergence research | Numerical convergence boundary | cutoff, k-point, smearing pages | Existing pages already bind convergence to target observables |
| W2 integration | Page-level review | `workflows/ground-state/*.md` | Confirmed output-review templates are fenced examples, not duplicate page headings |
| D2-1 diagram subagent | Original workflow diagram | `assets/diagrams/workflows/qe-ground-state-chain.svg` | Added ground-state data-chain SVG |
| Main agent QA | Final integration | reports and links | Linked diagram from workflow overview and SCF page |

## Research summary

The ground-state pages were already close to v0.2 definition-of-done. They include calculation target, upstream dependency, input objects, output review, convergence/reliability, common errors, downstream impact and source boundary. Batch 2 therefore avoided large rewrites and focused on consistency and visual navigation.

## Writing changes

- Added [assets/diagrams/README.md](assets/diagrams/README.md) with diagram source-boundary policy.
- Added [assets/diagrams/workflows/qe-ground-state-chain.svg](assets/diagrams/workflows/qe-ground-state-chain.svg).
- Linked the diagram from [workflows/README.md](workflows/README.md) and [workflows/ground-state/scf.md](workflows/ground-state/scf.md).
- Reviewed apparent duplicate `output review` headings in ground-state pages and kept them as fenced record templates; page-level `## Output review` sections remain the operative headings.

## QE workflow QA findings

- SCF remains defined as a ground-state charge-density/data-edge step, not a final physical-result claim.
- Convergence pages continue to bind cutoff, k-point and smearing scans to target observables.
- Relax/vc-relax pages continue to require final static SCF before bands/DOS/phonon downstream claims.
- NSCF remains separated from SCF and from bands path calculations.

## Diagram QA findings

- The SVG is original and stored locally.
- It uses only generic workflow objects and no material-specific case.
- It does not include cutoff/kmesh/smearing values or any universal parameter.
- It is referenced by two Markdown pages and covered by local link checks.

## Source QA findings

No external source or new parameter definition was added. The diagram is an internal schematic of existing workflow relationships; official parameter definitions remain in QE `INPUT_*` references already cited by the workflow pages.

## Validation results

- `git status --short --branch`: batch branch with expected modified workflow files and new diagram/report files before staging.
- `git diff --check`: PASS.
- Markdown local links: PASS, `OK local markdown links checked: 152 files`.
- Diagram file existence/reference scan: PASS, `OK image links checked: 3 refs`.
- Private path / document residue scan: PASS for touched files.
- Project-scope drift scan: PASS; scope terms appear only in exclusion/gate wording.
- Fixed material and universal parameter scans: PASS for touched files.
- High-risk overclaim scan: PASS; `JOB DONE` hit appears only in the existing "does not prove SCF trustworthy" boundary row.
- Ground-state P/W/B and output-review preservation scan: PASS for all 7 ground-state pages.
- Forbidden directory scan: PASS.

## Remaining gaps

- Batch 8 will add broader diagram coverage for electronic, phonon, theory and judgement pages.
- Batch 7 will revisit standards pages for exact naming/recording consequences.

## Final QA Gate

| Gate | Result | Evidence |
|---|---|---|
| Physics accuracy | PASS | Diagram and heading cleanup do not add new physics claims; ground-state boundaries remain unchanged. |
| QE workflow correctness | PASS | SCF, convergence, relax/vc-relax, final static SCF and NSCF data edges are preserved. |
| Output-review framing | PASS | Page-level `## Output review` sections are preserved; fenced record templates remain examples rather than active headings. |
| PASS/WARN/BLOCK preservation | PASS | All ground-state workflow pages retain `PASS / WARN / BLOCK` sections. |
| Style consistency | PASS | Diagram links and captions were added without changing page scope; fenced templates remain visually distinct from page headings. |
| Source boundary | PASS | No new official parameter claims or literature metadata added. |
| Diagram validity, if applicable | PASS | Original SVG, generic labels, referenced by Markdown, no fixed parameters. |
| Local links | PASS | `OK local markdown links checked: 152 files`; image scan found 3 valid local image refs. |
| Private path / document residue | PASS | Touched-file scan found no private path or document residue. |
| Project-scope drift | PASS | No runtime/MCP/VibeDFT/workflow-engine content added. |
| Fixed material case scan | PASS | No concrete material cases added. |
| Universal parameter scan | PASS | No universal parameter recommendations added. |
| High-risk overclaim scan | PASS | Only hit is existing boundary text saying `JOB DONE` does not prove SCF trustworthiness. |
| Merge recommendation | READY | Batch 2 ground-state workflow PR is ready to merge. |
