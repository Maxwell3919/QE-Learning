# Start Here

## 如何使用本仓库

本仓库按 QE 实际使用流程组织。建议先读 `learn/`，再进入 `workflows/` 的具体页面，并用 `standards/` 固化命名、记录和 output review。

## 为什么先学 QE 原生 workflow？

QE 的核心能力来自原生命令行程序之间的数据传递：`pw.x` 产生 ground state，`bands.x`、`dos.x`、`projwfc.x`、`pp.x`、`ph.x`、`q2r.x`、`matdyn.x` 等读取或转换上游状态。先理解这些程序和文件边界，才能判断外部工具是否真的帮上忙。

## 为什么不能只复制 input？

input 能跑通只说明语法和环境没有立即失败。科研计算还必须判断赝势、cutoff、k 点、smearing、结构状态、SCF 收敛、output warning 和下游依赖。

## 为什么每一步都要读 output？

QE 的可靠性证据在 output 中：total energy、SCF iteration、estimated accuracy、Fermi energy、forces、stress、irreducible k-points、loaded pseudopotentials、phonon perturbation convergence 等。没有 output review，就不能判断结果是否可信。

## 为什么要保留计算记录？

个人学习笔记也需要可追溯。至少记录结构来源、赝势来源、input 文件、命令、QE 版本、运行环境、output 摘要、收敛状态和 PASS/WARN/BLOCK 判断。

## 结构学习边界

结构操作学习将作为独立项目展开；本仓库只说明 QE 对结构输入的要求。

## 自动化工具边界

AiiDA 等自动化工具暂时不作为主线。它们适合在理解 QE 原生命令行 workflow 后，作为高级 workflow/provenance 参考。
