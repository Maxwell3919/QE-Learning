# Hybrid functional overview

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、cutoff/k 点收敛、bands/DOS 基础、结构和赝势记录
- 下游用途：高级电子结构审阅、带隙或能级相对变化分析、专题 workflow 的方法边界判断
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

本页是 hybrid functional 的高级边界页，不是完整教材。它说明什么时候可以考虑进入 hybrid 计算、进入前必须先掌握哪些基础 workflow、QE input/output 中哪些字段必须审阅，以及哪些结果不能只因为计算结束就视为可靠。

## 适用问题

Hybrid functional 在 QE 中主要用于把一部分 exact exchange 引入交换-相关泛函，从而改变 semilocal DFT 下的 Kohn-Sham 能级、带隙、局域态位置或相对能量判断。它适合被当作“更高成本、更强约束的电子结构模型”来审阅，而不是普通 SCF 上额外打开的单个开关。

适合进入本页边界判断的问题：

- semilocal DFT 给出的带隙、能级排序或局域态位置需要方法学交叉检查。
- 目标性质对交换-相关泛函非常敏感，且已有 PBE/PBEsol/LDA 等 semilocal 基线结果。
- 需要比较同一结构、同一赝势族、同一数值收敛策略下 semilocal 与 hybrid 的差异。
- 计算资源、时间预算和 checkpoint/restart 记录已经足够支撑昂贵的 SCF。

不适合作为本页直接处理的问题：

- 结构、赝势、cutoff、k 点、occupation 还没有通过基础 SCF 审阅。
- 只是想把某个默认带隙“调到实验值”。
- 尚未区分数值误差、结构误差、赝势误差和泛函误差。
- 需要完整 hybrid 理论、具体材料案例或可照抄参数模板。

## 进入该 workflow 前必须已经掌握什么

- 能独立完成并审阅 [SCF workflow](../ground-state/scf.md)，包括 `prefix/outdir`、赝势、cutoff、k 点、occupation/smearing、SCF 收敛和 warning 记录。
- 能完成 cutoff/k 点收敛性检查，并知道“SCF 收敛”不等于“参数收敛”。
- 能阅读 [Bands workflow](../electronic/bands.md)，知道 uniform k mesh、k-path bands、DOS/PDOS 的输入和解释边界不同。
- 能记录 QE version、运行命令、并行方式、wall time、内存或作业失败信息。
- 能把 semilocal 基线结果保存为可复查记录，而不是只保留最后一个数值。

进入 hybrid 前，至少应有一份普通 SCF/bands 的 `record.md` 或等价记录，写清楚结构来源、赝势来源、`ecutwfc/ecutrho`、`K_POINTS`、occupation、收敛状态和 PASS / WARN / BLOCK 判断。

## 计算图

```text
<reviewed semilocal baseline>
  -> choose hybrid functional boundary and EXX-related settings
  -> pw.x scf with hybrid functional
  -> output review: functional / EXX setup / convergence / cost
  -> optional bands or DOS-style follow-up with matching model boundary
  -> compare only against compatible, well-recorded baseline
```

Hybrid 计算不应覆盖原有 semilocal scratch。建议在学习记录中使用独立的 `prefix/outdir`，以免旧 charge density、wavefunction 或中间数据混入审阅。

## 需要的 QE 程序

- `pw.x`：执行 hybrid functional 的 SCF、NSCF 或 bands 类任务。hybrid 相关设置属于 `pw.x` 的输入边界。
- `bands.x`：如果后续整理 band data，仍按 bands workflow 审阅；但能带数据的物理模型已经由上游 `pw.x` hybrid 设置决定。

本页不展开 phonon、GW、Wannier 或 electron-phonon workflow。若 hybrid 结果要进入这些下游步骤，必须先查对应程序对 exact exchange / hybrid 数据链的支持边界。

## 关键 QE 程序或输入字段

| 字段或程序 | 所属位置 | 用途 | 审阅边界 |
|---|---|---|---|
| `pw.x` | 程序 | 执行包含 hybrid functional 设置的电子结构计算 | 确认 QE version、并行方式、scratch 路径和是否正常结束 |
| `input_dft` | `&SYSTEM` | 指定交换-相关泛函；会覆盖赝势文件中读入的泛函信息 | 必须确认取值、赝势兼容性和 output 回显；不要把它当无风险开关 |
| `exx_fraction` | `&SYSTEM` | 控制 hybrid 计算中的 exact-exchange fraction | 官方默认依赖所选 functional；除非有方法依据，不在通用模板中给材料无关默认值 |
| `screening_parameter` | `&SYSTEM` | HSE-like hybrid functional 的 screening 参数 | 只在相关 functional 边界内审阅；不要跨文献或跨 QE 版本盲目比较 |
| `ace` | `&SYSTEM` | 使用 adaptively compressed exchange operator | 影响性能和算法路径；output 和记录中应写清是否启用 |
| `ecutfock` | `&SYSTEM` | EXX operator 的 kinetic energy cutoff | 影响精度、速度和稳定性；官方说明含实现限制，不能未经测试随意降低 |
| `exxdiv_treatment` | `&SYSTEM` | 处理 small-q Coulomb divergence 的 EXX 专用方法 | 与晶胞形状和低维/各向异性边界相关；需要在记录中写清选择理由 |
| `x_gamma_extrapolation` | `&SYSTEM` | EXX 中 G=0 项 extrapolation | 是方法边界，不是收敛补丁；更改后需要重新比较结果 |
| `ecutvcut` | `&SYSTEM` | Coulomb divergence correction 的 reciprocal-space cutoff | 与 `exxdiv_treatment` 相关；只有在明确使用场景中审阅 |
| `nqx1/nqx2/nqx3` | `&SYSTEM` | Fock operator 的 q mesh | 可小于 k mesh，但直接影响成本和数值结果；必须记录与 k mesh 的关系 |
| `exx_maxstep` | `&ELECTRONS` | exact-exchange SCF 外层迭代上限 | 不收敛时要看是电子迭代、EXX 外层迭代还是资源失败 |
| `ecutwfc/ecutrho` | `&SYSTEM` | 基础波函数和电荷密度 cutoff | 仍需基线收敛证据；hybrid 不会自动修复 cutoff 不足 |
| `K_POINTS` | card | SCF uniform mesh 或 bands path | hybrid 成本对 k 点敏感；粗 k mesh 得到的带隙或能带不能直接当结论 |
| `nbnd` | `&SYSTEM` | 计算的 electronic states 数量 | bands/导带审阅时必须覆盖目标能量窗口；不要只沿用 SCF 最小需求 |

## 通用输入审阅清单

```markdown
## Hybrid Input Review

- QE 程序:
- QE version:
- 计算类型:
- Semilocal baseline record:
- Structure / pseudo consistency:
- `prefix/outdir` isolation:
- `input_dft`:
- EXX-related fields explicitly set:
- EXX-related fields left to QE defaults:
- `ecutwfc/ecutrho` convergence evidence:
- `ecutfock` policy:
- SCF `K_POINTS`:
- Fock q mesh (`nqx1/nqx2/nqx3`):
- occupation / smearing:
- `nbnd` and target energy window:
- Estimated cost / wall time / memory:
- Restart strategy:
- PASS / WARN / BLOCK before running:
- Reason:
```

如果某个 EXX 字段没有显式设置，应在记录中写成“left to QE default for this QE version”，而不是省略。这样后续比较不同 QE 版本、不同输入文件或不同人的计算时，才能知道差异来自哪里。

## output review 要点

```markdown
## Output Review

- QE 程序:
- 计算类型:
- QE version:
- `input_dft` / functional reported:
- Pseudopotentials loaded:
- Structure summary:
- Cutoff reported (`ecutwfc/ecutrho/ecutfock` if present):
- K-points reported:
- Fock q mesh / EXX setup reported:
- `nbnd` / number of Kohn-Sham states:
- Occupation / smearing:
- SCF convergence:
- EXX outer iterations:
- Total energy:
- Estimated SCF accuracy:
- Band gap / eigenvalue feature being inspected:
- Wall time / memory / parallel layout:
- Warnings:
- Restart / scratch status:
- Comparison target:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

输出判断时至少检查：

- `JOB DONE` 只说明程序结束；必须同时检查电子 SCF 和 EXX 相关迭代是否达到收敛。
- output 中回显的 functional、赝势、cutoff、k 点、occupation、band 数量应与 input review 一致。
- 若出现未收敛、达到 `electron_maxstep` / `exx_maxstep`、内存不足、wall-time 终止或并行 I/O 错误，先按运行问题归类，不要直接解释为物理现象。
- hybrid 与 semilocal 比较必须使用同一结构、同一赝势边界和可解释的数值设置；否则只能写成定性参考。
- band gap 或能级变化需要同时记录能量零点、`nbnd` 覆盖范围、k 点采样方式和是否来自 bands path 或 uniform mesh。

## 计算成本审阅

Hybrid 计算通常比 semilocal SCF 显著更贵，成本对体系大小、k 点、band 数、EXX/Fock q mesh、并行策略和 scratch I/O 都敏感。进入计算前应先做成本审阅：

- 估计 `nat`、`nbnd`、k 点数量和 `nqx1/nqx2/nqx3` 对任务规模的影响。
- 记录是否需要更多 wall time、内存、磁盘和 checkpoint/restart。
- 小规模测试只能证明输入链可运行，不能证明最终 k 点、cutoff 或 EXX 设置已经收敛。
- 若为 bands 或 DOS 后续任务，提前确认 `nbnd` 是否覆盖目标导带/价带窗口。
- 资源失败、队列超时、并行布局错误和文件系统问题应与物理不收敛分开记录。

## 常见风险

| 风险 | 表现 | 处理方式 |
|---|---|---|
| 把 `input_dft` 当作单点开关 | 只改 functional，不重审赝势、cutoff、k 点和成本 | 先建立 semilocal baseline，再写 hybrid input review |
| 未核对赝势和泛函边界 | output 回显的 functional 与预期不一致，或来源记录缺失 | 保存赝势文件名、来源、推荐 cutoff 和 output 回显 |
| k 点过粗 | 带隙、能级排序或金属性判断不稳定 | 明确写成 WARN/BLOCK，不能当最终结论 |
| `nbnd` 不足 | 导带窗口被截断，bands 图误读 | 在 bands 审阅中检查 number of Kohn-Sham states |
| EXX 设置未记录 | 不同 run 无法比较 | 显式记录设置值或“使用 QE 默认” |
| 资源失败误判为物理问题 | wall time、内存、I/O 或库问题导致异常结束 | 先按运行环境和作业日志诊断 |
| 用 hybrid 修复基础误差 | 结构未优化、cutoff/k 点未收敛、occupation 错误 | 回到基础 workflow 修正，不进入方法比较 |
| 下游 workflow 边界混乱 | 把 hybrid SCF 直接接到未核对支持的后处理 | 逐个查下游程序文档和数据依赖 |

## 与基础 workflow 的关系

- 与 SCF：hybrid 首先仍是 `pw.x` 电子自洽问题。SCF 的结构、赝势、cutoff、k 点、occupation、收敛和 warning 审阅规则全部保留。
- 与 convergence：hybrid 的 functional 改变不会替代 cutoff/k 点收敛测试；反而会放大成本和设置记录的重要性。
- 与 bands：hybrid bands 仍需要清楚区分 uniform SCF mesh 与 k-path bands。`nbnd`、能量零点、路径来源和 band data 文件仍按 bands workflow 审阅。
- 与 DOS/PDOS：DOS/PDOS 的 dense mesh、能量展宽和投影解释不能由一条 hybrid bands path 替代。
- 与 phonon/DFPT：本页不默认 hybrid 结果可直接进入 phonon 或 response workflow；必须单独确认对应程序和方法支持。
- 与 PASS/WARN/BLOCK：基础输入未审阅、hybrid output 未收敛、资源失败未分离、EXX 设置缺失或比较基线不可复查时，应给 WARN 或 BLOCK，而不是 PASS。

## 记录模板

```text
pw.scf.<system>.hybrid.in
pw.scf.<system>.hybrid.out
pw.bands.<system>.hybrid.in
pw.bands.<system>.hybrid.out
bands.<system>.hybrid.in
bands.<system>.hybrid.out
hybrid-input-review.md
record.md
```

`record.md` 至少写清：semilocal baseline 路径、hybrid input 边界、QE version、运行命令、资源消耗、output review、与 baseline 的可比性、PASS / WARN / BLOCK 判断和允许进入的下游 workflow。

## 资料来源

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE `bands.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
