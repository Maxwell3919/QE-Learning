# Molecular dynamics workflow

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、收敛性、结构、赝势、力和应力审阅
- 下游用途：有限温度结构采样、轨迹审阅、动力学稳定性检查或专题 workflow 的前置数据
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)
- 页面等级：B 级高级边界页。本页说明 QE 中 molecular dynamics 相关 workflow 的进入条件、程序边界、关键 input/output 审阅点和常见风险；不提供完整 MD 教程、统计力学教材、具体材料案例或可照抄参数模板。

## 适用问题

本页用于判断什么时候可以从静态 ground-state workflow 进入 QE 的分子动力学相关计算，并规定最小 input/output review 边界。适用问题包括：

- 需要在有限温度下观察结构、键长、配位、扩散、相变前兆或局域构型随时间的变化。
- 需要用短轨迹做计算设置、结构稳定性或 workflow 可行性的探索，而不是直接给出统计结论。
- 需要区分固定晶胞 Born-Oppenheimer MD、variable-cell MD 和 Car-Parrinello MD 的程序边界。
- 需要审阅 MD output 中每步电子收敛、温度控制、能量漂移、压力/应力、轨迹和 restart 信息。

不适合直接进入本页的问题：

- 基础 SCF 尚未稳定，力、应力、赝势、cutoff、k 点或 occupation/smearing 还没有通过审阅。
- 只想用 MD 代替结构优化，或把未平衡短轨迹当作热力学平均。
- 需要完整 ensemble 理论、扩散系数统计、自由能方法、增强采样或具体体系 protocol。
- 尚未明确使用 `pw.x` 的 `md`/`vc-md`，还是 `cp.x` 的 Car-Parrinello 路径。

## 进入该 workflow 前必须已经掌握什么

- [SCF workflow](../ground-state/scf.md)：能判断每一步电子自洽是否收敛，而不是只看 `JOB DONE`。
- [Relax workflow](../ground-state/relax.md)：能审阅最终力、最终坐标、结构是否物理合理，以及 relax 后为什么通常要重跑 static SCF。
- cutoff/k 点/occupation 收敛：MD 每一步都依赖力；静态总能收敛不自动等于力、应力和有限温度轨迹可靠。
- 赝势和质量记录：`ATOMIC_SPECIES` 中质量会进入动力学；赝势来源、元素质量和单位必须可追溯。
- 时间步长和轨迹概念：知道 `dt`、`nstep`、`iprint`、初始速度、温控、平衡段和生产段是不同层级的问题。
- ensemble 边界：能说明本次目标是近似 NVE、带 thermostat 的温度控制、还是 variable-cell/压力相关采样。
- restart/provenance 记录：能保存 input、output、轨迹、restart 路径、QE version、运行命令和中断/续跑方式。

## 计算图

```text
reviewed structure + pseudo + SCF numerical policy
  -> choose MD program boundary: pw.x md / pw.x vc-md / cp.x
  -> set trajectory length, timestep, dynamics and temperature/cell controls
  -> run MD with per-step electronic and ionic review
  -> inspect energy / temperature / pressure / structure / restart
  -> decide PASS / WARN / BLOCK before downstream analysis
```

MD 不应覆盖基础 SCF/relax 的 scratch。学习记录中建议使用独立 `prefix/outdir`，并把短测试轨迹、平衡轨迹和用于分析的轨迹分开命名。

## 需要的 QE 程序

- `pw.x`：执行 `calculation='md'` 或 `calculation='vc-md'`。这是本仓库基础 `pw.x` workflow 的自然延伸，电子结构在每个离子步重新求解或更新，属于 Born-Oppenheimer MD 边界。
- `cp.x`：执行 Car-Parrinello 相关任务，官方 `INPUT_CP` 有独立的 `calculation`、`electron_dynamics`、`ion_dynamics`、`dt` 和 thermostat 字段。它不是 `pw.x md` 的同义词。

本页只说明边界和审阅项。是否采用 `pw.x` 还是 `cp.x`，取决于目标问题、体系、资源和使用者是否理解对应动力学假设；不能只因为二者都叫 MD 就混用输入字段。

## 关键 QE 程序或输入字段

| 字段或程序 | 所属位置 | 用途 | 边界与审阅点 |
|---|---|---|---|
| `pw.x` | 程序 | 固定晶胞 `md` 或 variable-cell `vc-md` | 审阅 QE version、`calculation`、每步 SCF、力、温度、能量、轨迹和 restart |
| `calculation='md'` | `pw.x` `&CONTROL` | 固定晶胞分子动力学 | `tprnfor` 会自动为 `.TRUE.`；仍需检查 output 是否打印期望的力和轨迹信息 |
| `calculation='vc-md'` | `pw.x` `&CONTROL` | variable-cell molecular dynamics | `tstress` 和 `tprnfor` 会自动为 `.TRUE.`；必须额外审阅应力、压力、晶胞变化和 `&CELL` 设置 |
| `nstep` | `pw.x` / `cp.x` `&CONTROL` | 本次运行的 MD 或 CP steps 数 | 官方 `pw.x` 文档说明 MD restart 时仍会执行输入中的 `nstep` 步；续跑记录不能只写“继续上次” |
| `iprint` | `pw.x` / `cp.x` `&CONTROL` | 控制输出/轨迹写出频率 | 记录频率决定能否复查轨迹和温度/能量变化；过稀会丢失异常步 |
| `dt` | `pw.x` / `cp.x` `&CONTROL` | 时间步长 | `pw.x` 使用 Rydberg atomic units；`cp.x` 使用 Hartree atomic units。两者单位不同，不能直接照搬数值 |
| `ion_dynamics` | `pw.x` `&IONS` | `md` 下可选 Verlet、velocity-Verlet、Langevin 等官方列出的离子动力学路径 | 选项依赖 `calculation`；`relax`、`md`、`vc-md` 的允许值不同，必须按所用 QE 版本核对 |
| `ion_dynamics` | `cp.x` `&IONS` | CP 中控制离子如何移动，如 fixed、minimization、damped 或 Verlet 类路径 | 属于 `cp.x` 语义；不要把 `pw.x` 的 `md` 选项直接搬入 `cp.x` |
| `electron_dynamics` | `cp.x` `&ELECTRONS` | CP 中控制电子自由度传播或最小化 | Car-Parrinello 的核心边界之一；output review 必须检查电子绝热性/电子动能相关信息，而不是只看离子温度 |
| `ion_temperature` | `pw.x` / `cp.x` `&IONS` | 温度控制方式 | 官方选项随程序不同；`pw.x` 包含 `not_controlled`、`initial`、多种 rescaling、Nose-Hoover、Berendsen、Andersen、SVR 等路径；`cp.x` 选项更少，按 `INPUT_CP` 核对 |
| `tempw` | `&IONS` | 初始温度或多数 thermostat 的目标温度，单位 Kelvin | 只有在对应 `ion_temperature` 或随机初始速度语境下才有意义；不能单独代表 ensemble 已正确建立 |
| `fnosep` / `nhpcl` / `nhptyp` / `nhgrp` | `pw.x` / `cp.x` `&IONS` | Nose-Hoover 或 Nose-Hoover chain 相关字段 | 属于 thermostat 细节；需要记录取值和物理意图，不提供通用默认推荐 |
| `tolp` / `delta_t` / `nraise` | `pw.x` `&IONS` | rescaling、升降温、Berendsen、Andersen、SVR 等温控路径中的辅助参数 | 只有和具体 `ion_temperature` 选项一起才有定义；output review 应写明实际温控规则 |
| `cell_dynamics` | `pw.x` `&CELL` | `vc-md` 时控制晶胞动力学路径，如 Parrinello-Rahman 或 Wentzcovitch 相关选项 | 只在 variable-cell 边界审阅；固定晶胞 MD 不应误读为压力已被采样 |
| `press` / `wmass` | `pw.x` `&CELL` | variable-cell MD 或 relaxation 的目标压力与虚拟晶胞质量相关设置 | 单位和物理含义按官方文档；不在通用页给材料无关推荐值 |
| `ATOMIC_SPECIES` mass | input card | 原子质量进入动力学 | 质量、赝势文件和 species 标签必须可追溯；同一元素拆 species 时仍需合理质量 |
| `ATOMIC_VELOCITIES` | input card | 从输入给定初始速度或 restart 速度 | 是否读取取决于程序和 `ion_velocities`/restart 设置；必须在记录中写清初始速度来源 |
| `CONSTRAINTS` | input card | 约束动力学或约束优化 | 约束改变自由度和温度解释；不能把 constrained trajectory 与 unconstrained trajectory 混比 |

### `pw.x md` / `vc-md` 与 `cp.x` 的边界

`pw.x` 的 `calculation='md'` 和 `calculation='vc-md'` 属于 `INPUT_PW` 页面；`dt` 在 Rydberg atomic units，`ion_dynamics` 的允许值按 `calculation` 分支变化。`vc-md` 还需要进入 `&CELL` 审阅 `cell_dynamics`、`press`、`wmass` 等 variable-cell 字段。

`cp.x` 属于独立 `INPUT_CP` 页面；它的默认 `calculation` 边界是 Car-Parrinello 相关任务，并且有 `electron_dynamics`、电子虚拟质量、电子温控、CP restart 和 CP 专用输出。`cp.x dt` 使用 Hartree atomic units，与 `pw.x dt` 相差单位定义。阅读或迁移输入时必须先确认程序名，再解释字段。

### 温控与 ensemble 的审阅边界

`ion_temperature` 不等于“已经得到正确 ensemble”。它只说明代码如何处理离子温度或速度：

- `not_controlled` 或 `initial` 类设置更接近无持续温控的轨迹，但仍需检查能量漂移、初始速度和平衡状态。
- rescaling、Berendsen、Andersen、SVR、Nose-Hoover 等温控路径改变速度或引入 thermostat 变量；应记录 `tempw`、相关时间尺度/频率/步数参数和适用阶段。
- 升温、降温或速度重标定轨迹通常只能作为准备或平衡过程，不能直接与生产段统计混在一起。
- 对 constrained dynamics，温度和自由度定义受约束影响，应记录约束和自由度处理字段。

## input review 最小清单

```markdown
## MD Input Review

- QE program: pw.x / cp.x
- QE version:
- MD boundary: md / vc-md / cp / vc-cp:
- Structure source and pre-MD review:
- Pseudopotentials and atomic masses:
- `prefix/outdir` isolation:
- `calculation`:
- `nstep`:
- `iprint` / trajectory output frequency:
- `dt` and unit:
- `ion_dynamics`:
- Initial positions source:
- Initial velocities source:
- Temperature control: `ion_temperature`:
- `tempw` and thermostat-related fields:
- For `vc-md`: `cell_dynamics`, pressure/cell fields:
- For `cp.x`: `electron_dynamics` and CP-specific electronic settings:
- Constraints, if any:
- SCF/electronic convergence policy per step:
- Restart policy:
- PASS / WARN / BLOCK before running:
- Reason:
```

如果某个 thermostat 或 cell field 留给 QE 默认值，应在记录中写成“left to QE default for this QE version”，并保留所用官方页面版本或 QE version，而不是空白省略。

## output review 要点

```markdown
## MD Output Review

- QE program / version:
- Calculation type reported:
- Program boundary confirmed: pw.x / cp.x:
- Structure and pseudopotentials loaded:
- Atomic masses reported or traceable:
- Cutoff / k-points / occupation reported:
- `dt`, `nstep`, `iprint` reported:
- Ionic dynamics reported:
- Temperature control reported:
- For `vc-md`: cell dynamics / pressure / stress reported:
- For `cp.x`: electron dynamics / electronic kinetic behavior:
- Per-step SCF or electronic convergence:
- Total energy / conserved quantity trend:
- Temperature trend:
- Pressure / stress trend, if relevant:
- Force anomalies:
- Trajectory completeness:
- Geometry anomalies:
- Warnings / restarts / stopped-by-time messages:
- Files needed for restart:
- Equilibration segment separated from analysis segment:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

审阅时至少检查：

- output 是否确认了预期程序、`calculation`、时间步长单位语境、离子动力学和温控设置。
- `pw.x md` 每个离子步的电子 SCF 是否正常收敛；某一步不收敛会污染后续轨迹解释。
- 能量或 conserved quantity 是否出现系统性漂移；不能只看最后一步是否结束。
- 温度是否围绕目标或预期范围变化；升温、降温、平衡和生产段应分开标注。
- `vc-md` 是否同时审阅 stress、pressure、cell parameters 和体积变化；固定晶胞 MD 不提供同一层级的压力采样结论。
- `cp.x` 是否检查电子自由度行为；Car-Parrinello 轨迹不能只按 `pw.x` BOMD 的 output 习惯审阅。
- 轨迹是否完整写出，`iprint` 是否足以定位异常结构，restart 文件是否能支持可追溯续跑。
- `JOB DONE` 只说明程序结束；MD 还必须检查每步电子状态、轨迹质量、能量/温度/压力趋势和 warning。

## 常见风险

- 把 `pw.x md`、`pw.x vc-md` 和 `cp.x` 的输入字段混用，尤其是 `dt` 单位和 `electron_dynamics` 边界。
- 时间步长过大导致能量漂移、温度异常或结构崩坏，却只根据程序结束判断通过。
- 每步 SCF 不稳定或偶发不收敛，仍把整段轨迹用于后续统计。
- `nstep` 太短、未区分平衡段和生产段，却输出扩散、相变或热力学平均结论。
- 只记录最后结构，不保存轨迹、温度/能量历史和 restart 信息。
- `ion_temperature` 设置存在，但没有记录 thermostat 类型、目标温度、升降温路径或相关辅助字段。
- 对 `vc-md` 只看温度，不审阅压力、应力、cell dynamics 和晶胞变化。
- 初始速度来源不清，或 restart 时重复随机化速度而未记录。
- 约束、固定原子、拆分 species、质量设置或自由度处理没有进入温度解释。
- 复用 relax/scf scratch，导致轨迹来源、restart 和下游读取路径不可追溯。
- 把短测试轨迹、升温轨迹、平衡轨迹和生产轨迹混在同一个 PASS 结论下。

## 与基础 workflow 的关系

MD 位于基础 workflow 之上，而不是替代基础 workflow：

```text
SCF convergence + force/stress convergence + reviewed structure
  -> optional relax / static SCF baseline
  -> MD input review
  -> MD output review
  -> trajectory analysis or downstream workflow
```

- 对 [SCF workflow](../ground-state/scf.md)：MD 每步都依赖电子结构求解；SCF 稳定性、cutoff、k 点、occupation/smearing 和赝势审阅仍然保留。
- 对 [Relax workflow](../ground-state/relax.md)：relax 给出 0 K 附近的优化结构，MD 给出有限温度轨迹；二者回答的问题不同。MD 不能用来掩盖初始结构明显不合理或力未收敛的问题。
- 对 convergence workflow：静态总能收敛不足以证明 MD 可用；MD 还要关注力、应力、时间步长、温控、轨迹长度和能量漂移。
- 对 bands/DOS/phonon：通常不直接把某个 MD 中间步当成最终 ground-state 输入；若需要从轨迹抽样结构，应为每个抽样结构建立独立 static SCF/output review。
- 对 reproducibility：MD 必须记录初始结构、初始速度或随机化策略、restart 文件、`nstep` 分段、温控设置和轨迹文件。只有最终结构不足以复现动力学路径。

## PASS / WARN / BLOCK 边界

| 状态 | 判据 |
|---|---|
| PASS | 程序边界清楚；SCF/电子状态逐步可审；`dt`、`nstep`、温控、轨迹、restart 和 warning 已记录；能量/温度/压力趋势与任务目标相容；下游只使用被允许的数据 |
| WARN | 程序完成但轨迹较短、平衡不足、温控或能量漂移需要继续检查；可用于探索和调参，不用于强统计结论 |
| BLOCK | 程序/字段混用、单位不明、每步 SCF 或 CP 电子状态异常、结构崩坏、轨迹缺失、restart 不可追溯，或把短测试轨迹当最终物性结论 |

## 记录模板

```text
md.<program>.<system>.in
md.<program>.<system>.out
trajectory/
restart/
record.md
output-review.md
```

`record.md` 应写清楚程序边界、QE version、结构/赝势/质量来源、运行命令、`dt` 单位、`nstep` 分段、温控策略、restart 策略、output review 和 PASS / WARN / BLOCK 判断。

## 资料来源

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE `cp.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_CP.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- 本仓库：[SCF workflow](../ground-state/scf.md)
- 本仓库：[Relax workflow](../ground-state/relax.md)
