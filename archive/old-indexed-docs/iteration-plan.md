# 后续迭代计划

## Phase 1: 已完成的第一版

- GitHub 文档目录结构。
- README、学习路线、生态地图。
- 理论最低目录占位。
- core input 目录占位。
- 3 篇 workflow 初稿：SCF、bands、phonon dispersion。
- 计算记录模板、文件命名规范、checklists。

## Phase 2: 补齐最小可运行学习闭环

目标：让学习者能用 Si 或 Al 完成 SCF -> convergence -> relax -> bands/DOS。

任务：

- 写 `03-core-input-structure/01-pw-input-overview.md` 详细版。
- 写 `04-workflows/02-cutoff-convergence.md` 详细版。
- 写 `04-workflows/03-kpoint-convergence.md` 详细版。
- 写 `04-workflows/04-relax.md` 详细版。
- 加入一个 Si 最小案例目录。

验收：

- 每个案例有 input、record、输出摘要和 PASS/WARN/BLOCK 判断。

## Phase 3: 补 phonon 分支

目标：把 phonon 从“会跑命令”写成 DFPT workflow。

任务：

- 写 `04-workflows/09-phonon-gamma.md` 详细版。
- 扩展 `04-workflows/10-phonon-dispersion.md`，加入真实 output 判断片段。
- 写 `06-convergence-and-debugging/07-phonon-instability-vs-numerical-error.md` 详细版。
- 加入 Si 或 GaAs phonon 最小案例。

验收：

- 文档明确区分 Gamma、q-grid、ASR、negative frequency、Born charge、dielectric constant。

## Phase 4: HPC 与自动化准备

目标：让笔记可扩展成 VibeDFT / agent-assisted QE skill 文档。

任务：

- 写 SLURM 模板和 restart 规范。
- 把 workflow 文档中的计算图转成机器可读任务字段。
- 设计 PASS/WARN/BLOCK 验证规则表。
- 对接 AiiDA/QE 自动化概念页。

验收：

- 每个 workflow 有输入前提、输出 artifact、验证规则和失败处理。

