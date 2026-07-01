# QE-Learning v0.2 Release Notes

## Scope

v0.2 将 QE-Learning 从 v0.1 reference-handbook foundation 推进为内容更充分、导航更清楚、图像辅助更完整的 QE / DFT 学习与查阅手册。它覆盖 QE 原生命令行 workflow、input/output 阅读、数值收敛、结果可信度判断、最低理论背景、source boundary、科研记录和 uncertainty statement。

本版本仍保持项目边界：它不是自动化计算平台、具体材料案例库、参数推荐表或论文综述集合。所有参数定义、程序行为和版本敏感字段仍应回到 QE 官方文档和相关工具官方文档核对。

## What changed since v0.1

- 建立 v0.2 content completion matrix 和 agent-team 执行报告。
- 补强 ground-state、electronic、phonon/DFPT 三条主线 workflow 的参考手册风格页面。
- 完成 theory-minimum 的最终深度回填，使核心概念能回到 QE input/output 与 workflow 准入。
- 完成 physics-judgement 的最终深度回填，强化 numerical/model/PP/boundary/post-processing/workflow error 分类。
- 完成 standards/source spine 的最终整理，统一 calculation record、output review、`PASS / WARN / BLOCK` 和 source policy。
- 增加原创 SVG 学习图，并在相关页面中引用。
- 更新 handbook index、目录 README、completion matrix、总报告和 release notes。

## Content areas completed

- `learn/`：能力路径、完成判据和可验证输出。
- `workflows/ground-state/`：SCF、NSCF、relax/vc-relax、cutoff/k-point/smearing convergence。
- `workflows/electronic/`：bands、DOS、PDOS、charge density、electrostatic potential、ELF、work function、Fermi surface。
- `workflows/phonon/`：Gamma phonon、DFPT dispersion、q2r/matdyn、phonon DOS、Born charge、dielectric tensor、IR/Raman、debugging。
- `theory-minimum/`：DFT/KS/SCF、plane waves、pseudopotentials、reciprocal space、symmetry、bands/DOS、smearing、force/stress、DFPT phonons、polar response、magnetism/SOC/DFT+U。
- `physics-judgement/`：DFT approximation map、band gap、functional/PP/U/SOC/vdW boundaries、soft modes、polar response、work function、Wannier、EPC、excited-state、free energy、uncertainty statement。
- `standards/`：record template、file naming、project layout、source policy、output-review checklist、`PASS / WARN / BLOCK`。
- `references/`：source index、canonical literature spine、BibTeX starting point、tool/tutorial boundary cards。

## Diagrams added

All diagrams are original SVG assets under `assets/diagrams/` and are referenced by Markdown pages.

- `assets/diagrams/workflows/qe-ground-state-chain.svg`
- `assets/diagrams/workflows/qe-electronic-chain.svg`
- `assets/diagrams/workflows/qe-phonon-dfpt-chain.svg`
- `assets/diagrams/theory-minimum/bands-dos-evidence-boundary.svg`
- `assets/diagrams/theory-minimum/phonon-dynamical-matrix-picture.svg`
- `assets/diagrams/physics-judgement/pass-warn-block-gate.svg`
- `assets/diagrams/physics-judgement/model-vs-numerical-error.svg`
- `assets/diagrams/physics-judgement/uncertainty-statement-chain.svg`

## Validation performed

Batch PRs ran local QA gates before merge. The final release-candidate pass used:

- `git status --short --branch`
- `git diff --check`
- full Markdown local-link check
- diagram reference and SVG XML check
- private-path and document-residue scans
- project-scope drift scan
- fixed material case scan
- universal-parameter scan
- high-risk overclaim scan
- forbidden-directory scan
- source-boundary / `PASS / WARN / BLOCK` / output-review framing scans
- `canonical.bib` duplicate-key scan

The exact final command results are recorded in [AGENT_TEAM_V0_2_BATCH_9_RELEASE_FINALIZATION_REPORT.md](AGENT_TEAM_V0_2_BATCH_9_RELEASE_FINALIZATION_REPORT.md).

## Known limitations

- v0.2 is content-complete for the current handbook scope, but it is not a substitute for expert review of every scientific statement.
- Some external links and BibTeX metadata may still benefit from periodic spot checks.
- Tool pages remain boundary and source-routing cards; they are not full tutorials for Wannier90, EPW, Yambo, AiiDA or other tools.
- Future improvements should be selected page expert reviews, not another broad rewrite.

## What remains future work

- Selected expert review of high-risk pages such as band-gap interpretation, phonon soft modes, symmetry/k-path and EPC data-chain boundaries.
- External source verification pass for canonical literature metadata and official-document links.
- Readability pass for dense pages after reader feedback.
- Optional additional diagrams only where they clarify a concrete workflow or judgement boundary.

## Release recommendation

If the final Batch 9 QA gate remains `READY`, tag this version as:

```text
v0.2-content-complete-handbook
```
