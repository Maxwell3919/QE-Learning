# Workflows

`workflows/` 是 QE 原生命令行计算参考页集合，负责把学习路线落实到具体程序链、input 字段、output review、收敛要求和下游准入。每页都应说明计算目标、输入前提、计算图、关键输入参数、输出判断、收敛要求、常见错误和记录规范。

## 如何阅读 workflow 页面

阅读 workflow 页面时，不要把它当作“照抄即可运行”的具体材料教程，而应把它当作通用审阅框架。需要按计算目标查找页面时，可先用 [HANDBOOK_INDEX.md](../HANDBOOK_INDEX.md) 的 Workflow 审阅路径定位。每一页先看这一步计算想回答什么问题，再看它依赖哪一个上游状态，例如结构、赝势、SCF、relax 后 final SCF、NSCF 或 DFPT 中间文件。随后检查页面给出的通用输入模板、关键 output review 项、进入下游前的 `PASS / WARN / BLOCK` 判断，以及资料来源。

主线 workflow 页面说明 QE 基础计算链条中反复使用的数据依赖和审阅标准，例如 ground state、收敛、结构优化、电子结构、声子和常见后处理。高级页面主要用于划清能力边界、最低学习目标和官方资料入口，帮助判断某个主题是否已经超出基础路径；它们不替代官方手册、专门教程或针对具体体系的研究方案。

本目录不提供具体材料案例，也不把某一种材料的参数写成通用答案。页面中的模板、检查项和判断词只用于建立可复用的输入审阅、输出复查和下游准入框架；真实计算仍需根据材料、赝势、目标 observable 和收敛测试重新确定参数。

每页应重点看：

- 计算目标：本步骤产出的物理量或中间状态是什么。
- 上游依赖：读取哪一次结构、SCF、NSCF、relax 或 DFPT 数据。
- 通用输入模板：哪些 namelist、卡片和关键参数必须成组检查。
- output review：output 中哪些行决定结果是否可信。
- `PASS / WARN / BLOCK`：是否允许进入下游、只能探索，还是必须停止修复。
- 资料来源：判断依据来自官方文档、QE 输出、教程说明还是本仓库规范。

一个合格的 workflow 页面至少要回答：本步骤想解决什么计算问题，上游依赖是什么，使用哪些 QE 程序，哪些 namelist/card 决定物理模型或数值精度，output 中哪些证据能支持 `PASS / WARN / BLOCK`，哪些 warning 会阻止下游，以及相关字段是否需要回到当前 QE 官方文档核验。

Ground-state 主链可先参考下图，再进入具体页面审阅每一步的 input/output 证据。

![QE ground-state workflow chain](../assets/diagrams/workflows/qe-ground-state-chain.svg)

图：ground-state workflow 的数据边界示意。该图用于定位 SCF、收敛、relax/vc-relax、final static SCF 和 NSCF 的关系；正式准入仍以各页面的 output review 和 `PASS / WARN / BLOCK` 为准。

## 目录

- `ground-state/`：SCF、NSCF、收敛、relax、vc-relax。
- `electronic/`：bands、DOS、PDOS、charge density、potential、ELF、work function、Fermi surface。
- `phonon/`：Gamma phonon、DFPT dispersion、phonon DOS、Born effective charge、IR/Raman、debugging。
- `advanced/`：spin、SOC、DFT+U、vdW、hybrid、MD、NEB、EPC、Wannier90、pseudopotential generation overview。

高级功能页面用于建立边界和最低学习目标，不替代官方专门教程。

## 资料来源

- [references/source-index.md](../references/source-index.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
