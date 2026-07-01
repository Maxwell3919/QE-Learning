# QE-Learning Handbook Index

本索引用于把 QE-Learning 当作参考手册查阅。它不替代各目录 README；它提供从读者目标到仓库页面的快速路径。

## 1. 新学习者路径

目标：建立 QE 原生命令行 workflow 的最小闭环。

1. [Start here](learn/00-start-here.md)
2. [Learn path](learn/README.md)
3. [Workflow map](learn/01-qe-workflow-map.md)
4. [SCF workflow](workflows/ground-state/scf.md)
5. [Convergence loop](learn/03-convergence-loop.md)
6. [Calculation record template](standards/calculation-record-template.md)

完成判据：能说明 input 来源、运行命令、关键 output 证据、`PASS / WARN / BLOCK` 和下游准入。

## 2. Workflow 审阅路径

目标：检查某一步 QE 计算是否可进入下游。

- Workflow overview：[workflows/README.md](workflows/README.md)
- Ground state：[workflows/ground-state/scf.md](workflows/ground-state/scf.md)
- Electronic structure：[workflows/electronic/bands.md](workflows/electronic/bands.md)
- Phonon / DFPT：[workflows/phonon/phonon-dispersion-dfpt.md](workflows/phonon/phonon-dispersion-dfpt.md)
- Advanced boundary：[workflows/advanced/dft-plus-u.md](workflows/advanced/dft-plus-u.md)
- Output review standard：[standards/output-review-checklist.md](standards/output-review-checklist.md)
- PASS/WARN/BLOCK standard：[standards/pass-warn-block.md](standards/pass-warn-block.md)

使用规则：先确认上游依赖和文件链，再读 input 字段，最后用 output evidence 给出准入判断。

## 3. Theory 回查路径

目标：解释 QE input/output 背后的最低理论。

- DFT / SCF：[theory-minimum/dft-ks-scf.md](theory-minimum/dft-ks-scf.md)
- Plane waves and cutoff：[theory-minimum/plane-wave-cutoff.md](theory-minimum/plane-wave-cutoff.md)
- Pseudopotentials：[theory-minimum/pseudopotentials.md](theory-minimum/pseudopotentials.md)
- Reciprocal space / BZ：[theory-minimum/reciprocal-space-and-brillouin-zone.md](theory-minimum/reciprocal-space-and-brillouin-zone.md)
- K-points / symmetry / k-path：[theory-minimum/kpoints-symmetry-kpath.md](theory-minimum/kpoints-symmetry-kpath.md)
- Band structure / DOS：[theory-minimum/band-structure-and-dos.md](theory-minimum/band-structure-and-dos.md)
- DFPT phonons：[theory-minimum/dfpt-phonons.md](theory-minimum/dfpt-phonons.md)

使用规则：理论页用于回查概念，不替代 workflow 页面和 output review。

## 4. Physics Judgement 路径

目标：判断结果是否可信，避免把可运行 output 写成越界物理结论。

- DFT approximation map：[physics-judgement/01-dft-approximation-map.md](physics-judgement/01-dft-approximation-map.md)
- Band gap boundary：[physics-judgement/04-band-gap-problem-and-derivative-discontinuity.md](physics-judgement/04-band-gap-problem-and-derivative-discontinuity.md)
- Functional / PP boundary：[physics-judgement/functional-choice-and-sensitivity.md](physics-judgement/functional-choice-and-sensitivity.md)
- U provenance：[physics-judgement/u-value-provenance-and-boundary.md](physics-judgement/u-value-provenance-and-boundary.md)
- SOC and symmetry：[physics-judgement/soc-and-symmetry-boundary.md](physics-judgement/soc-and-symmetry-boundary.md)
- Soft modes：[physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md](physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md)
- Work function：[physics-judgement/work-function-and-electrostatic-boundary.md](physics-judgement/work-function-and-electrostatic-boundary.md)
- Uncertainty statements：[physics-judgement/uncertainty-statement-template.md](physics-judgement/uncertainty-statement-template.md)

使用规则：先定位异常 output，再区分 numerical、model、pseudopotential、boundary、post-processing 和 workflow propagation error。

## 5. Source 与记录路径

目标：把判断写成可追踪的科研记录。

- Source index：[references/source-index.md](references/source-index.md)
- Canonical literature：[references/canonical-literature.md](references/canonical-literature.md)
- Citation policy：[standards/citation-and-source-policy.md](standards/citation-and-source-policy.md)
- Calculation record：[standards/calculation-record-template.md](standards/calculation-record-template.md)
- Project layout：[standards/project-layout.md](standards/project-layout.md)

使用规则：参数定义优先官方 INPUT reference；程序链优先官方文档；物理边界优先 canonical literature；教程只作为辅助。

## 6. 图像与 release 路径

目标：用原创示意图快速定位 workflow、理论对象和判断 gate，并了解 v0.2 当前完成边界。

- Diagram assets：[assets/diagrams/README.md](assets/diagrams/README.md)
- Ground-state chain：[assets/diagrams/workflows/qe-ground-state-chain.svg](assets/diagrams/workflows/qe-ground-state-chain.svg)
- Electronic chain：[assets/diagrams/workflows/qe-electronic-chain.svg](assets/diagrams/workflows/qe-electronic-chain.svg)
- Phonon / DFPT chain：[assets/diagrams/workflows/qe-phonon-dfpt-chain.svg](assets/diagrams/workflows/qe-phonon-dfpt-chain.svg)
- PASS/WARN/BLOCK gate：[assets/diagrams/physics-judgement/pass-warn-block-gate.svg](assets/diagrams/physics-judgement/pass-warn-block-gate.svg)
- v0.2 release notes：[RELEASE_NOTES_v0.2.md](RELEASE_NOTES_v0.2.md)
- v0.2 completion matrix：[V0_2_CONTENT_COMPLETION_MATRIX.md](V0_2_CONTENT_COMPLETION_MATRIX.md)

使用规则：图像是学习辅助和审阅地图，不替代 workflow 页面、output review、calculation record 或来源边界。
