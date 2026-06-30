# Start Here

## 如何使用本仓库

本仓库按 QE 实际使用流程组织，不把学习目标定义为“找到一个 input 并跑起来”。建议先读 `learn/` 建立主线，再进入 `workflows/` 的具体页面，并用 `standards/` 固化命名、记录和 output review。

每一轮学习都应形成一个闭环：

1. 明确这一步计算想证明什么；
2. 写出或改写 input；
3. 用 QE 原生命令运行；
4. 阅读 output 中的关键证据；
5. 给出 PASS / WARN / BLOCK；
6. 决定是否允许进入下游。

## 为什么先学 QE 原生 workflow？

QE 的核心能力来自原生命令行程序之间的数据传递：`pw.x` 产生 ground state，`bands.x`、`dos.x`、`projwfc.x`、`pp.x`、`ph.x`、`q2r.x`、`matdyn.x` 等读取或转换上游状态。先理解这些程序和文件边界，才能判断每个 output 来自哪里、下游程序依赖什么、哪些文件不能混用。

原生 workflow 也是学习 QE 的最小可解释层。图形界面、脚本生成器和 workflow manager 可以提高效率，但它们最终仍在组织 QE input、命令、scratch 数据和 output。如果还不能直接读懂这些对象，自动化只会隐藏错误来源，而不会提高结果可信度。

## 为什么不能只复制 input？

input 能跑通只说明语法和运行环境没有立即失败，不说明结果已经可用。一个结束的作业可能仍然有不合适的赝势、过低的 cutoff、过粗的 k 点、未说明的 smearing、没有达到目标的 SCF 收敛、残余力或应力过大、warning 未解释，或者与下游计算所需的 `prefix`、`outdir`、结构状态不一致。

因此，本仓库不把“能运行”当作完成判据。完成判据必须来自 input、command、environment 和 output 的共同证据，尤其是 output review。

## 为什么每一步都要读 output？

QE 的可靠性证据在 output 中：total energy、SCF iteration、estimated accuracy、Fermi energy、forces、stress、irreducible k-points、loaded pseudopotentials、phonon perturbation convergence 等。没有 output review，就不能判断结果是否可信。

output review 至少要回答这些问题：

- 程序是否正常结束，还是只是生成了部分 output？
- output 中读取的结构、赝势、cutoff、k 点、smearing 是否与 input 和学习目标一致？
- SCF、relax、phonon 或后处理步骤的关键收敛证据是否达到当前目标要求？
- warning、restart、旧 scratch 数据、单位、对称性变化和下游依赖是否已经解释清楚？
- 当前结果能支持下一个 workflow 步骤，还是只能作为排错或探索记录？

读 output 是 QE 学习的核心训练。QE 用户真正需要建立的能力，是从 output 中判断“这个计算现在处于什么状态”，而不是只记住某个 input 模板。

## 如何使用 PASS / WARN / BLOCK？

每次计算记录都应给出 PASS / WARN / BLOCK，并写明理由和证据。这个判断把“程序是否结束”转换成“结果是否能支撑下游”。

| 状态 | 使用场景 | 下游决策 |
|---|---|---|
| PASS | 程序完成，关键输出达到目标阈值，参数、来源和环境可追踪，未解释风险已经排除 | 可以进入当前目标允许的下游 |
| WARN | 程序完成，但仍有明确数值或物理风险；证据足够用于学习、比较或排错，不足以作为最终结论 | 只在风险可接受时继续，并在记录中保留限制 |
| BLOCK | 当前结果不能支撑下游；关键收敛失败、来源不清、文件混用或 output 与 input/command 对不上 | 停止下游，先重算、修复输入或重新设计 workflow |

PASS / WARN / BLOCK 用来给下一步行动设置闸门。看到 `JOB DONE` 后仍然可能是 WARN 或 BLOCK；只有当 output review 支持当前目标，才可以写 PASS。

## 为什么要保留计算记录？

个人学习笔记也需要可追溯。至少记录结构来源、赝势来源、input 文件、命令、QE 版本、运行环境、output 摘要、收敛状态和 PASS / WARN / BLOCK 判断。

本仓库提供学习路径、workflow 说明、标准和模板，但不保存个人具体材料案例，也不要求提交实际 input/output。个人练习时应在自己的学习记录中保存 input、关键 output 摘要、错误信息、参数变化、判断证据和下一步动作；仓库正文只保留可复用的判断方法。

## 结构学习边界

结构构建、CIF 操作、对称性标准化、超胞、缺陷和 slab 等主题属于 Structure-Learning；本仓库只说明 QE workflow 对结构输入和结构状态的要求。

## 自动化工具边界

AiiDA 等自动化工具不作为基础学习路径。它们适合在理解 QE 原生命令行 workflow 后，作为 optional advanced reference，用来学习 workflow 编排和 provenance，而不是替代 input/output 阅读。

## 资料来源

- [README.md](../README.md)
- [learn/README.md](README.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [standards/calculation-record-template.md](../standards/calculation-record-template.md)
- [references/source-index.md](../references/source-index.md)
