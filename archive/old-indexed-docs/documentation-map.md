# QE 文档体系地图

本页用于 Phase 1：把已有调研转化为可长期维护的 GitHub 文档骨架。

## 顶层目录

| 路径 | 文档目的 | 学习阶段 | 内容范围 | 资料状态 | 最终能力 |
|---|---|---|---|---|---|
| `README.md` | QE 学习笔记入口 | 全阶段 | 定位、阅读顺序、维护原则 | 已写第一版 | 知道如何使用本仓库 |
| `00-learning-roadmap.md` | 旧版学习路线图 | 已废弃 | 旧版按时间组织，当前不再使用 | 已归档 | 当前以 `learn/` 的能力阶段为准 |
| `01-qe-ecosystem.md` | QE 工作流生态地图 | 入门后 | 原生模块、外部工具、学习优先级 | 已写第一版 | 知道每个工具插入 workflow 的位置 |
| `02-theory-minimum/` | QE 使用所需最低理论 | 入门到进阶 | DFT、cutoff、赝势、k 点、smearing、SCF、symmetry、DFPT | 占位，需官方/教材补充 | 能把理论映射到 input/output |
| `03-core-input-structure/` | 系统理解 QE input | 入门核心 | `pw.x` namelist 和 cards | 占位，需官方 input reference | 能解释关键参数而非照抄 |
| `04-workflows/` | 严谨 workflow 文档 | 核心 | SCF、收敛、relax、bands、DOS、phonon、后处理 | 3 篇已写第一版 | 能按 DAG 组织计算 |
| `05-postprocessing/` | 后处理与可视化 | 进阶 | charge density、ELF、potential、plotting | 占位 | 能从 QE 输出生成可解释图像 |
| `06-convergence-and-debugging/` | 可靠性与排错 | 核心到进阶 | SCF 失败、cutoff/k 点、赝势、phonon 虚频 | 占位 | 能区分数值问题和物理结论 |
| `07-hpc-and-reproducibility/` | HPC 与可复现 | 进阶 | SLURM、restart、scratch、provenance | 占位 | 能在集群上稳定运行并记录证据 |
| `08-tools-and-ecosystem/` | 工具深挖 | 进阶 | ASE、AiiDA、seekpath、phonopy、Wannier90 | 占位 | 能选择合适外部工具 |
| `09-templates-and-checklists/` | 使用规范 | 全阶段 | 命名、记录、checklist、报告模板 | 已写第一版 | 能把计算过程标准化 |

## 后续迭代顺序

1. 补全 `03-core-input-structure/01-pw-input-overview.md`，让初学者能读懂 SCF input。
2. 补全 `04-workflows/02-cutoff-convergence.md` 与 `03-kpoint-convergence.md`，避免默认参数陷阱。
3. 扩展 `04-workflows/09-phonon-gamma.md`，与 `10-phonon-dispersion.md` 明确分工。
4. 写 `06-convergence-and-debugging/07-phonon-instability-vs-numerical-error.md`。
5. 用 Si、Al、GaAs 各补一个最小可运行案例。
