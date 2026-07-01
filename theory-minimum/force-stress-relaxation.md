# Force, stress and relaxation

## 本页解决什么问题

本页解释 force、stress、relax 和 vc-relax 在 QE 中如何决定结构是否可以进入 final static SCF、bands、DOS、phonon 或 response workflow。它帮助用户读懂 output 中的 forces、stress tensor、pressure、final coordinates 和 final cell，而不是只看 `JOB DONE` 或最后能量。

## 最低掌握深度

最低需要知道：

- Force 是 total energy 对 atomic positions 的导数，用来判断 internal coordinates 是否接近平衡。
- Stress 是 total energy 对 cell deformation 的响应，用来判断 cell shape/volume 是否接近平衡。
- `relax` 优化原子坐标；`vc-relax` 同时优化原子坐标和 cell。
- relax/vc-relax output 是优化轨迹和最终结构来源；进入性质计算前通常需要基于 final structure 重做 static SCF。

## QE 中的对应对象

| 对象 | QE 位置 | 判断意义 | output 证据 |
|---|---|---|---|
| `calculation='relax'` | `&CONTROL` | 固定 cell 优化 internal coordinates | ionic step、final forces、updated positions |
| `calculation='vc-relax'` | `&CONTROL` | 优化 internal coordinates 和 cell | cell step、stress、pressure、final cell |
| `forc_conv_thr` | `&CONTROL` | force convergence threshold | final force、convergence message |
| `press_conv_thr` | `&CELL` | pressure convergence threshold | pressure、stress tensor |
| `ion_dynamics` | `&IONS` | ionic optimizer | optimizer history、step status |
| `cell_dynamics` / `cell_dofree` | `&CELL` | cell optimizer 和自由度 | cell updates、constraints、final cell |
| `ATOMIC_POSITIONS` / `CELL_PARAMETERS` | cards | input/final structure 数据 | final coordinates、final cell parameters |

相关 workflow：[relax](../workflows/ground-state/relax.md)、[vc-relax](../workflows/ground-state/vc-relax.md)、[SCF](../workflows/ground-state/scf.md)、[Gamma phonon](../workflows/phonon/gamma-phonon.md)。

## 核心概念

结构优化是在当前 DFT 模型和数值参数下寻找能量面上的局部平衡点。`relax` 只改变原子坐标，固定 cell；`vc-relax` 同时改变 cell 和原子坐标，因此对 stress、cutoff、k mesh 和 cell constraints 更敏感。优化完成并不自动给出最终电子结构结论；它给出的是下一次 static SCF 的结构输入。

## 对 input 的影响

- `forc_conv_thr` 应与下游目标匹配；phonon 和 response 通常需要更谨慎的结构审阅。
- `vc-relax` 需要明确 `&CELL`、`cell_dynamics`、`press`、`press_conv_thr` 和 `cell_dofree`。
- 低维体系或表面/界面模型不能无边界放开非物理 cell 自由度；本页只保留边界，结构构建细节不展开。
- `vc-relax` 后 cell 改变会改变 k-point density、k-path 和 stress/cutoff 审阅条件。
- 优化后的 final coordinates/cell 应被写入新的 static SCF 输入，而不是继续使用旧 input。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| ionic/cell step history | 优化路径可追踪 | 每个中间态都可用于性质计算 |
| final forces / total force | internal coordinates 达到当前力目标 | phonon 已可信 |
| stress tensor / pressure | cell relaxation 证据 | cell model 已物理正确 |
| final coordinates | 下游结构来源明确 | final electronic structure 已完成 |
| final cell parameters | `vc-relax` 后 cell 可复查 | k mesh/k-path 仍沿用旧设置 |
| convergence message | optimizer 停止原因 | 数值参数和模型误差已消除 |
| warnings / restart | 优化异常可追踪 | warning 可忽略 |

## 常见误区

- 直接用 relax 最后一帧做最终 bands。
- 只看 `JOB DONE` 不看 forces/stress。
- `vc-relax` 后只更新 atomic positions，不更新 cell。
- cell 改变后不重新确认 k mesh density 和 k-path。
- 残余应力未解释就进入 phonon。
- 优化失败后盲目续算，不审阅 optimizer history。
- 把受约束的 relax 结果当成完全无约束平衡态。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | final forces、stress/pressure、final coordinates/cell 和停止原因支持当前目标；final static SCF 计划明确 | 可进入 final static SCF，再进入 bands/DOS/phonon 等 |
| WARN | 结构可用于继续优化或探索，但 force/stress、constraints、restart 或 static SCF 边界仍需补充 | 可继续优化或诊断，不应进入最终 phonon |
| BLOCK | 未达力/应力收敛；final structure 不可追踪；cell/constraint 与目标冲突；无 final static SCF 却进入性质结论 | 不允许进入依赖平衡结构的下游 |

## 下游影响

- `SCF`：final structure 需要独立 static SCF 建立清晰数据边。
- `bands/DOS/PDOS`：结构和 cell 改变会影响 band path、k mesh 和 electronic states。
- `phonon/DFPT`：残余 force/stress 可造成 acoustic 异常或虚频。
- `work function`：slab geometry、vacuum 和 surface relaxation 影响 electrostatic potential。
- `advanced workflows`：NEB、MD、EPC、IR/Raman 均继承结构质量。

## 与 physics-judgement 的边界

本页只说明 force/stress 的最低 QE 审阅。以下问题转到：

- [relax is not final electronic structure](../physics-judgement/relax-is-not-final-electronic-structure.md)
- [energy vs free energy](../physics-judgement/energy-vs-free-energy.md)
- [surfaces, interfaces and electrostatics](../physics-judgement/11-surfaces-interfaces-and-electrostatics.md)
- [imaginary frequency triage](../physics-judgement/imaginary-frequency-triage.md)

## 来源与边界

- Stable: `relax/vc-relax`、`forc_conv_thr`、`press_conv_thr`、`&IONS`、`&CELL` 字段以 QE `INPUT_PW` 为准。
- Boundary: 本页不规定材料无关的力/应力阈值；阈值应由目标 workflow 和 convergence evidence 决定。
- Internal standard: structure status 应写入 [output review checklist](../standards/output-review-checklist.md) 和 calculation record。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- 本仓库：[relax workflow](../workflows/ground-state/relax.md)、[vc-relax workflow](../workflows/ground-state/vc-relax.md)
