# Learn

`learn/` 是 QE 学习路径入口，不是教程正文，也不是完成证明。阅读顺序不按天数或周数安排，而按能力目标、完成判据和可验证输出推进：先知道自己要掌握什么，再用 output 和记录判断是否可以进入下一阶段。

## 阅读方式

先读 `learn/`，用每一页建立问题框架、术语边界和判断标准；如果需要从具体问题反查页面，可先用 [HANDBOOK_INDEX.md](../HANDBOOK_INDEX.md) 定位入口。再读 `workflows/`，把同一阶段对应到 QE 原生命令、input/output 和可靠性检查。`theory-minimum/` 用于回查最低理论，`physics-judgement/` 用于解释结果可信度，`standards/` 用于把学习过程写成可复查记录。

如果学习某一步时需要图像辅助，先从 [assets/diagrams/README.md](../assets/diagrams/README.md) 查当前图像清单；图像只帮助定位 workflow 和判断关系，完成判据仍以 output review 与记录为准。

推荐每个阶段都按同一轮次阅读：

1. 在 `learn/` 中确认本阶段的能力目标。
2. 到 `workflows/` 中查看对应 workflow 的输入、输出和常见检查点。
3. 回到 `learn/` 表格，用完成判据判断是否已经能独立复述和审查。
4. 留下一份可验证输出：图、依赖关系、参数表、output review 或 PASS / WARN / BLOCK 记录。

不要把学习计划写成固定日期表。每次推进只看三件事：当前能力目标是否清楚，完成判据是否达到，是否留下能被复查的输出。

| 页面 | 能力目标 | 完成判据 |
|---|---|---|
| `00-start-here.md` | 知道如何使用本仓库 | 能说清为什么先学 QE 原生 workflow |
| `01-qe-workflow-map.md` | 建立 QE 主计算图 | 能画出 SCF、收敛、relax、电子结构、声子和后处理的依赖关系 |
| `02-first-scf-loop.md` | 建立第一个可信 SCF 闭环 | 能读 input/output 并给出 PASS / WARN / BLOCK |
| `03-convergence-loop.md` | 建立数值收敛判断 | 能记录 cutoff、k 点、smearing 的最小测试 |
| `04-relaxation-loop.md` | 理解结构优化在 QE 中的角色 | 能读 force/stress 并判断是否允许下游 |
| `05-electronic-structure-loop.md` | 掌握 bands/DOS/PDOS 主线 | 能区分 k-path 与 uniform k mesh |
| `06-phonon-dfpt-loop.md` | 掌握 DFPT phonon 基本路线 | 能说明 `ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x` 分工 |
| `07-postprocessing-loop.md` | 理解 QE 后处理 | 能说明 `pp.x`、charge density、potential、ELF、work function 的数据来源 |
| `08-hpc-and-reproducibility.md` | 建立运行环境与记录意识 | 能记录 QE version、pseudopotential source、command、output review |
| `09-feature-expansion-map.md` | 查询 QE 功能边界和扩展入口 | 能区分核心学习、高级扩展和只需知道入口的专题 |

## 阶段完成判断

每个阶段不以“读完页面”作为完成。只有当下面三类证据都能给出时，才算可以进入下一阶段：

- 能力复述：能不用复制原文，说明本阶段在 QE workflow 中解决什么问题、依赖哪些上游状态、会影响哪些下游步骤。
- Output 判断：能指出该阶段至少需要查看哪些 output 证据，并给出 PASS / WARN / BLOCK 或等价的准入判断。
- 可验证输出：能留下可复查的学习产物，例如 workflow 依赖图、参数扫描表、input/output 对照、收敛记录、结构优化准入说明或 provenance 摘要。

如果只能复述概念，但不能指出 output 中的证据，本阶段还没有完成。如果能跑出文件，但不能说明上游来源、参数选择和下游准入，也还没有完成。

## `learn/` 与 `workflows/` 的配合

`learn/` 负责学习路线和完成判据，`workflows/` 负责具体 QE 程序、输入输出和审查细节。两者配合时不要把 `workflows/` 当作孤立教程复制，也不要把 `learn/` 当作目录清单跳过。

- 学 SCF 时，先在 `learn/` 明确 ground state 的角色，再到 `workflows/ground-state/` 查 `pw.x scf`、k 点、cutoff 和 occupations 的判断。
- 学 relaxation 时，先在 `learn/` 明确 force/stress 与下游准入，再到 `workflows/ground-state/` 查 `relax`、`vc-relax` 和 final static SCF 的关系。
- 学电子结构与后处理时，先在 `learn/` 区分 ground state、NSCF、bands、DOS、PDOS 和空间场数据，再到 `workflows/electronic/` 查具体程序读写什么。
- 学声子与响应性质时，先在 `learn/` 明确 DFPT 的上游依赖和文件转换链，再到 `workflows/phonon/` 查 `ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x` 的分工。

每次跨到 `workflows/` 后，都要回到本页确认是否满足完成判据；没有 output review 的学习记录，只能作为草稿。

## 推进规则

- 先核心后扩展：先掌握 SCF、收敛、relax、电子结构、声子和后处理，再阅读高级功能边界。
- 先证据后结论：先读 input/output、参数记录和 warning，再写可靠性判断。
- 先最小闭环后复杂组合：每个阶段先完成一个能被复查的最小 workflow，再增加参数或功能。
- 先记录来源后沉淀经验：参数定义、模板和教程经验需要能追踪到公开来源；个人判断要标明依据和版本边界。

`09-feature-expansion-map.md` 是扩展功能地图，不是基础学习闭环的必修阶段。读者完成 SCF、收敛、relax、电子结构、phonon/DFPT、后处理和记录规范后，再用它判断某个高级主题应进入 `workflows/advanced/`、`physics-judgement/` 还是外部官方文档。

## 进一步理论学习

完成基础 workflow 后，如果需要解释 band gap、磁性、SOC、DFT+U、vdW、phonon 虚频、work function、EPC 或 excited-state 边界，请进入 [physics-judgement/](../physics-judgement/)。该目录不替代 workflow 页面，而是帮助把 output 中的异常、近似来源和物理失败模式转化为更可靠的 PASS / WARN / BLOCK 判断。
