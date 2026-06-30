# Wannier90 overview

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：已审阅的 SCF/NSCF、bands、结构、赝势、cutoff、k mesh 和 `nbnd` 记录
- 下游用途：Wannier interpolation、Berry 相关量、输运前处理和专题高级 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 适用问题

本页用于理解 QE 到 Wannier90 的接口边界：什么时候需要从 QE eigenvalues / wavefunctions 转到 maximally-localized Wannier functions，哪些上游数据必须准备好，以及 output review 应该怎样判断这条链是否足够进入后续高级分析。

适合本页处理的问题：

- 需要用 Wannier interpolation 得到更平滑或更密的 bands 表示。
- 需要为 Berry curvature、anomalous Hall、输运或模型 Hamiltonian 分析准备 Wannier 数据。
- 需要检查 `pw.x` SCF/NSCF、`pw2wannier90.x` 和 Wannier90 之间的 `prefix/outdir`、k mesh、bands 数量和投影是否一致。
- 需要建立 Wannier workflow 的记录模板和风险清单。

不适合本页处理的问题：

- 不从零讲完整 Wannier 教程。
- 不给具体材料案例。
- 不提供可直接运行的真实输入文件。
- 不替代基础 bands/DOS/PDOS workflow 的收敛性审阅。

## 进入该 workflow 前必须已经掌握什么

- `pw.x` SCF：能判断 ground-state density 是否完成，能追踪 `prefix/outdir`、赝势、cutoff、occupation/smearing 和 k mesh。
- `pw.x` NSCF：能解释 fixed-density eigenvalues / wavefunctions 的用途，知道 dense uniform k mesh 和 bands k-path 的区别。
- Bands workflow：能把 QE bands 作为后续 Wannier interpolation 的对照基准，而不是只看 Wannier90 是否正常结束。
- PDOS / orbital character：能根据目标能区和化学直觉选择候选轨道投影，不把 `projections` 当作任意调参项。
- `nbnd` 策略：能确保未占据态数量覆盖目标能量窗口和 disentanglement 需求。
- 记录规范：能把输入、输出、脚本、版本、warning、PASS/WARN/BLOCK 判断写入同一条计算记录。

## 计算边界图

```text
<structure> + <pseudo>
  -> pw.x scf on <scf_k_mesh>
  -> pw.x nscf on <wannier_uniform_k_mesh> with sufficient nbnd
  -> Wannier90 pre-processing / seedname.nnkp
  -> pw2wannier90.x interface files
  -> Wannier90 wannierization / interpolation
  -> comparison with QE bands and downstream advanced analysis
```

这里的 `<wannier_uniform_k_mesh>` 是 Wannier 数据链的核心边界之一。它通常不是 high-symmetry bands path；它必须与 Wannier90 设置和 `pw2wannier90.x` 读取的数据保持一致。

## 关键 QE 程序或输入字段

| 字段或程序 | 所属阶段 | 用途 | 常见风险 | output 中如何验证 |
|---|---|---|---|---|
| `pw.x calculation='scf'` | QE ground state | 生成上游电荷密度和基础波函数数据 | SCF 未审阅就进入 Wannier 链 | SCF output 有收敛状态、版本、k 点、cutoff 和 `JOB DONE`，且通过基础 output review |
| `pw.x calculation='nscf'` | QE fixed-density wavefunctions | 在 Wannier 用的 uniform k mesh 上生成 eigenvalues / wavefunctions | 用 bands path 代替 uniform mesh，或读取了错误 SCF | NSCF output 显示 non-self-consistent calculation，并能读到目标 `prefix/outdir` |
| `prefix/outdir` | `pw.x` / `pw2wannier90.x` | 连接 SCF、NSCF 和接口转换 | 不同阶段名称不一致，读取旧 scratch 或找不到 wavefunctions | 每个 output 中读取的 prefix、scratch 路径和文件时间线一致 |
| `nbnd` | `pw.x` | 控制进入 NSCF/Wannier 的 bands 数量 | 导带不足导致目标窗口缺态或 disentanglement 不稳定 | output 中 number of Kohn-Sham states 覆盖目标能量范围 |
| `K_POINTS automatic` / uniform k mesh | `pw.x` NSCF / Wannier90 | 定义 Wannier 化使用的 reciprocal-space 采样 | k mesh 太粗、与 Wannier90 设置不一致、误用 high-symmetry path | NSCF k 点数量、Wannier90 k 点设置和接口文件链互相一致 |
| `pw2wannier90.x` | QE post-processing | 把 QE wavefunctions/eigenvalues 转成 Wannier90 所需接口文件 | `seedname`、`prefix/outdir`、spin/SOC 选项或 projections 数据链不一致 | output 显示完成接口转换，并生成对应 `*.mmn`、`*.amn`、`*.eig` 等文件 |
| `Wannier90` | 外部程序 | 执行 projections、disentanglement、localization 和 interpolation | 只看程序结束，不看 spreads、窗口和 bands 对照 | `.wout` 中审阅迭代、spreads、warning、excluded/selected bands 和 interpolation 结果 |
| `projections` | Wannier90 input | 定义初始轨道投影或投影中心 | 投影无物理依据，和 PDOS / bands 轨道成分矛盾 | `.wout` 和投影相关输出能说明投影被识别，且结果与轨道预期相符 |
| `dis_win_*` / `dis_froz_*` | Wannier90 input | disentanglement 的外窗口和冻结窗口边界 | 窗口只为拟合图形而调，未记录选择理由 | `.wout` 中窗口、选带和最终 bands 对照可复核 |
| `*.mmn` / `*.amn` / `*.eig` | 接口文件 | 保存 overlap、projection 和 eigenvalue 数据 | 文件缺失、来自不同 run、被旧文件污染 | 文件名、时间、`seedname` 和 record 中的运行链一致 |

## 关键文件链

学习记录中至少应能追踪：

```text
pw.scf.<system>.in/out
pw.nscf.<system>.in/out
wannier90.<seed>.win
pw2wannier90.<seed>.in/out
<seed>.nnkp
<seed>.mmn
<seed>.amn
<seed>.eig
<seed>.wout
qe-bands-reference.*
wannier-bands-comparison.*
record.md
```

这些名称只是记录形态示意，不是要求照抄的真实输入命名。

## output review 要点

### 通用审阅模板

```markdown
## Output Review

- Workflow stage:
- QE version:
- Wannier90 version:
- SCF dependency:
- NSCF dependency:
- `prefix/outdir` consistency:
- Structure / cell convention:
- Pseudopotentials:
- Cutoff:
- SCF k mesh:
- Wannier k mesh:
- `nbnd`:
- Projections:
- Disentanglement / frozen windows:
- Interface files:
- Spread summary:
- QE bands reference:
- Wannier bands comparison:
- Warnings:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

### 判断标准

- SCF、NSCF 和 `pw2wannier90.x` 必须使用同一目标体系的 `prefix/outdir`，不能混入旧 scratch。
- NSCF 的 k mesh 必须是 Wannier 数据链所需的 uniform mesh；bands path 只能作为对照图来源。
- `nbnd` 必须覆盖目标能量窗口和候选未占据态。若调整 `nbnd`，应重跑相关 NSCF 和接口转换，而不是只改 Wannier90 设置。
- `projections` 应与目标轨道、PDOS 或已知 bands character 有可解释关系。
- `*.mmn`、`*.amn`、`*.eig`、`*.nnkp` 等接口文件应来自同一 `seedname` 和同一轮运行。
- `.wout` 中的 spread、disentanglement、warning 和最终收敛状态必须进入记录。
- Wannier interpolated bands 必须和 QE reference bands 在目标能区对照；若只在远离目标能区拟合良好，不能直接放行下游分析。
- 若后续要计算 Berry 或输运相关量，还要复查 spin、SOC、time-reversal、symmetry 选项是否和 SCF/NSCF 物理模型一致。

## 常见风险

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| `pw2wannier90.x` 找不到数据 | `prefix/outdir` 不一致，NSCF wavefunctions 不完整，scratch 被清理 | 对照 SCF/NSCF/pw2wannier90 三份输入输出 |
| 接口文件存在但结果异常 | 文件来自旧 run，`seedname` 混乱，k mesh 不匹配 | 检查文件时间、record、`*.nnkp` 与 NSCF k 点 |
| Wannier bands 和 QE bands 差异很大 | projections 不合适，能量窗口错误，`nbnd` 不足，disentanglement 不稳定 | 先查 `nbnd` 和窗口，再查投影 |
| spreads 很大或迭代不稳定 | 初始投影差、目标能区过宽、结构/对称性设置不一致 | 缩小问题边界，复查 orbital character |
| 图像看似拟合但不可复现 | 只记录最终窗口，没有记录尝试过程 | 把每次窗口、投影和 `nbnd` 变化写入 record |
| 下游 Berry/transport 结果不可解释 | 基础 bands 对照未通过，或 spin/SOC 物理模型前后不一致 | 回到 SCF/NSCF/bands review，不从后处理结果反推正确性 |

## 与基础 workflow 的关系

- 与 SCF：Wannier 链读取的是 SCF ground-state density 的后续数据；SCF 未通过 output review 时，Wannier 结果不能补救上游问题。
- 与 NSCF：Wannier 通常依赖 NSCF 生成的 uniform k mesh eigenvalues / wavefunctions；`nbnd` 和 k mesh 是最容易影响后续质量的边界字段。
- 与 Bands：QE bands 是 Wannier interpolation 的必要对照。Wannier bands 建立在基础 bands workflow 的审阅基准之上，不能替代基础 bands 审阅。
- 与 PDOS：PDOS / orbital character 是选择 `projections` 的重要依据；投影选择应有物理解释。
- 与 DOS/Fermi surface：若目标是 Fermi 附近性质，需额外审阅 smearing、Fermi energy、k mesh 密度和能量零点。
- 与高级专题：Berry curvature、输运、topological analysis 等应在 Wannier 基础质量通过后再展开；本页只定义进入这些专题前的边界检查。

## B 级完成判据

达到 B 级边界页时，应能做到：

- 解释 Wannier90 在 QE workflow 中解决的接口问题。
- 列出 `pw.x` SCF/NSCF、`pw2wannier90.x`、Wannier90 三段之间的文件链。
- 知道 `nbnd`、`projections`、`prefix/outdir` 和 k mesh 如何影响 Wannier 化质量。
- 能写出 output review，而不是只保存最终图。
- 能判断 Wannier 结果什么时候只能标记为 WARN 或 BLOCK。

仍不要求做到：

- 独立完成特定材料的最佳投影和窗口优化。
- 系统讲解 Wannier90 所有高级关键字。
- 给出可直接投产的自动化 workflow。

## 资料来源

- QE INPUT_pw2wannier90 reference: <https://www.quantum-espresso.org/Doc/INPUT_pw2wannier90.html>
- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- Wannier90 documentation: <https://wannier90.readthedocs.io/>
- Wannier90 project: <https://wannier.org/>
