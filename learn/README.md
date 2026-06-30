# Learn

`learn/` 是 QE 学习路径。它不按时间组织，而按能力目标、完成判据和可验证输出组织。

| 页面 | 能力目标 | 完成判据 |
|---|---|---|
| `00-start-here.md` | 知道如何使用本仓库 | 能说清为什么先学 QE 原生 workflow |
| `01-qe-workflow-map.md` | 建立 QE 主计算图 | 能画出 SCF、收敛、relax、电子结构、声子和后处理的依赖关系 |
| `02-first-scf-loop.md` | 建立第一个可信 SCF 闭环 | 能读 input/output 并给出 PASS/WARN/BLOCK |
| `03-convergence-loop.md` | 建立数值收敛判断 | 能记录 cutoff、k 点、smearing 的最小测试 |
| `04-relaxation-loop.md` | 理解结构优化在 QE 中的角色 | 能读 force/stress 并判断是否允许下游 |
| `05-electronic-structure-loop.md` | 掌握 bands/DOS/PDOS 主线 | 能区分 k-path 与 uniform k mesh |
| `06-phonon-dfpt-loop.md` | 掌握 DFPT phonon 基本路线 | 能说明 `ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x` 分工 |
| `07-postprocessing-loop.md` | 理解 QE 后处理 | 能说明 `pp.x`、charge density、potential、ELF、work function 的数据来源 |
| `08-hpc-and-reproducibility.md` | 建立运行环境与记录意识 | 能记录 QE version、pseudopotential source、command、output review |
| `09-feature-expansion-map.md` | 了解 QE 功能边界 | 能区分核心学习和高级扩展 |
