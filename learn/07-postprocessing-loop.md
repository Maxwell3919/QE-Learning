# Postprocessing Loop

## 能力目标

理解 QE 后处理如何从已验证的 ground state 或 NSCF 状态生成可解释数据，并能判断这些数据是否允许进入可视化、物理解释和下游记录。

后处理不是“计算结束后画图”。它是从已有 `prefix/outdir` 中读取密度、势、波函数或本征值，再按特定物理量导出网格、profile 或可视化文件。这个阶段的核心能力是能回答三件事：数据从哪里来，图像能说明什么，哪些解释必须回到上游 output review。

本页只写通用依赖图、审阅逻辑和记录边界，不写具体材料案例，不提供真实图像或文件。具体输入字段、`plot_num` 和格式设置进入 `workflows/electronic/` 各页。

## 覆盖内容

- `pp.x`
- charge density
- electrostatic potential
- ELF
- work function
- Fermi surface
- visualization

## `pp.x` 的位置

`pp.x` 属于 Quantum ESPRESSO 的 PostProc 后处理程序。源码树中通常在 `PP/src/pp.x`；安装或编译完成后，它通常和 `pw.x`、`bands.x`、`dos.x` 等可执行程序一起出现在 QE 的可执行程序目录中，例如 `<qe_bin>/pp.x`。运行时不要只记录“用了 pp.x”，还要记录实际命令、QE version、`prefix`、`outdir`、输入文件和输出文件。

在学习记录里，`pp.x` 的定位应写成：

```text
pw.x scf / nscf 生成可读取的 prefix/outdir
  -> pp.x 从同一 prefix/outdir 读取已有数据
  -> filplot / fileout 生成网格、切片或可视化文件
```

如果 `pp.x` 读取的 `prefix/outdir` 不是已审阅的上游计算目录，后处理结果不能解释。

## 数据来源地图

| 后处理对象 | 主要程序链 | 数据来源 | 输出形态 | 最小审阅点 |
|---|---|---|---|---|
| charge density | `pw.x scf -> pp.x` | 已收敛 SCF ground-state charge density | 3D 网格、2D 切片或可视化格式 | `prefix/outdir`、SCF 收敛、FFT/grid、`plot_num`、输出维度 |
| electrostatic potential | `pw.x scf -> pp.x -> average.x` | 已收敛 SCF 的势相关数据 | 势网格、方向平均 profile | 势类型、单位、平均方向、profile 是否与目标几何一致 |
| ELF | `pw.x scf -> pp.x` | 已收敛 SCF density 及 ELF 所需的派生量 | ELF 网格或可视化格式 | `plot_num`、网格分辨率、等值面/切片设置、解释是否过度 |
| work function | `slab SCF -> pp.x potential -> average.x` | 同一 slab SCF 的 Fermi energy 与方向平均电势 | vacuum level、Fermi level、work-function review | 真空平台、能量零点、slab 法向、偶极/边界设置 |
| Fermi surface | `pw.x scf -> pw.x nscf dense mesh -> fs.x` | 金属体系 dense uniform k mesh 上的 eigenvalues 和 Fermi energy | BXSF 或其他 Fermi-surface 数据 | 金属性前提、NSCF k mesh、Fermi energy、bands/DOS 对照 |

charge density、potential 和 ELF 主要由 `pp.x` 从 SCF 数据中导出。work function 需要把 slab SCF 的 Fermi energy 与 `pp.x`/`average.x` 得到的 vacuum level 组合起来审查，不能只依赖 `pp.x` 单独输出。Fermi surface 不走 `pp.x` 主线，而是由 `fs.x` 读取 dense NSCF 数据生成 BXSF；它仍属于后处理学习阶段，因为它把上游电子结构数据转换成可视化对象。

## 上游状态依赖

后处理前必须先写清上游状态。依赖条件应达到“可以被复查”，不能只写“已经跑完”：

- SCF：input、output、QE version、赝势、cutoff、k mesh、occupation/smearing、收敛状态和 `prefix/outdir` 已记录。
- NSCF：只在 Fermi surface、DOS/PDOS 相关分支中需要；必须说明它读取哪个 SCF，使用 dense uniform k mesh，而不是 high-symmetry k-path。
- 结构：cell、原子位置、边界条件、slab 法向或平均方向能追溯到同一份上游记录。
- 能量参考：Fermi energy、VBM/CBM、自定义零点或 vacuum level 的来源已写清。
- scratch/restart：`outdir` 没有混用旧结果；如果 restart 或复制过 scratch，需要在记录中说明。

如果上游 SCF 是 `WARN`，后处理最多只能继承为 `WARN`，并必须写明哪些解释仍可用、哪些不能用。如果上游是 `BLOCK`，后处理不能用于物理结论，只能作为诊断材料。

## 可视化与物理解释边界

可视化文件只证明“某个数据被导出并被某个工具显示”，不自动证明物理解释成立。每张图或 profile 都应分成两层记录：

- 图像层：数据来源、程序链、文件名、网格/切片方向、色标或等值面、单位、能量零点和可视化工具。
- 解释层：从图中得到的物理判断、与哪些上游 output 或其他 workflow 互相支持、哪些判断仍是定性。

常见边界包括：

- charge density 可以展示空间电荷分布，但不能单独给出价态、键级或电荷转移定量结论。
- electrostatic potential 可以支持 profile、界面或 slab 前置分析，但势类型、平均方向和能量参考必须明确。
- ELF 可以辅助讨论电子局域化或成键特征，但不能直接当作定量键强度。
- work function 需要 vacuum plateau 与同一 SCF 的 Fermi energy；没有平台、方向错误或能量零点混乱时不能给数值结论。
- Fermi surface 只适合有费米面的体系；BXSF 图必须与 bands crossing 和 DOS at Fermi level 对照，不能由单张三维图单独判断金属性。

排查时先审上游，再审数据导出，最后才审绘图。不要用调色、插值、平滑或视角替代 output review。

## Output Review 怎么写

后处理 output review 要比普通“生成了文件”更严格。推荐模板：

```markdown
## output review

- QE 程序:
- 后处理对象:
- QE version:
- Upstream dependency:
- Source `prefix/outdir`:
- Upstream PASS / WARN / BLOCK:
- Structure / geometry dependency:
- Data source:
- Key input choices:
- Generated files:
- Grid / dimension / direction:
- Unit / energy reference:
- Visualization settings:
- Warnings:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream records:
- Blocked downstream records:
```

字段含义：

- `Upstream dependency` 写清来自 SCF、NSCF、slab SCF、potential profile 还是 dense NSCF。
- `Data source` 写清读取的是 charge density、potential、ELF 派生量、Fermi energy/eigenvalues，还是 direction-averaged profile。
- `Key input choices` 写 `plot_num`、`iflag`、`output_format`、`idir`、`filplot/fileout`、`filfermi` 等会改变数据语义的字段。
- `Grid / dimension / direction` 写 1D/2D/3D、切片方向、平均方向或 dense k mesh。
- `Unit / energy reference` 写清单位、Fermi energy、vacuum level、能量零点或 profile 参考。
- `Allowed downstream records` 只列允许进入的记录类型，例如 figure caption、potential profile review、work-function review、Fermi-surface consistency check。
- `Blocked downstream records` 写不能支持的结论，例如定量电荷转移、work-function 数值、金属性判断或跨体系比较。

## PASS / WARN / BLOCK

| 状态 | 判据 | 下游记录方式 |
|---|---|---|
| PASS | 上游 SCF/NSCF 已通过审阅；`prefix/outdir` 可追溯；后处理 input/output 匹配目标；文件、网格、单位和可视化设置已记录；解释没有超出数据边界 | 可进入对应 workflow 记录，并在 figure caption 或 review 中引用 output review |
| WARN | 上游可用但存在边界条件、收敛、网格、可视化或物理解释限制；结果可用于定性或诊断，但不宜支撑强结论 | 必须写明限制，只允许进入受限下游；下游标题或记录中保留 `WARN` |
| BLOCK | 上游未收敛或不可追溯；`prefix/outdir` 混用；`plot_num`/势类型/方向错误；关键文件缺失；物理前提不成立 | 不进入物理解释；只记录诊断、错误现象和需要重跑的上游步骤 |

PASS / WARN / BLOCK 要随下游传播。不能把一个 `WARN` 的 potential profile 在 work-function review 里写成无条件 `PASS`；也不能把一个 `BLOCK` 的 Fermi-surface BXSF 放进最终图像记录。

## 下游记录

后处理完成后，至少留下以下记录之一：

- charge-density review：数据来源、网格/维度、可视化设置、允许的定性解释。
- potential review：势类型、平均方向、profile 特征、是否允许进入 work-function。
- ELF review：等值面/切片设置、与 charge density/PDOS/结构的对照、解释边界。
- work-function review：vacuum level、Fermi energy、能量零点、平台判断、公式和限制。
- Fermi-surface review：dense NSCF 来源、Fermi energy、BXSF 文件、与 bands/DOS 的一致性。

推荐把图像文件、原始数据和解释分开列：图像记录负责可复现，物理解释负责可审查，PASS/WARN/BLOCK 负责下游准入。

## 完成判据

- 能说明后处理读取哪个上游 `prefix/outdir`。
- 能说明 charge density、potential、ELF、work function 和 Fermi surface 的数据来源。
- 能说明输出数据类型、单位、网格、方向、能量参考和可视化工具。
- 能说明图像是否可追溯到具体 input/output。
- 能写出后处理 output review，并给出 PASS / WARN / BLOCK。
- 能说明哪些结论允许进入下游记录，哪些必须标为受限或阻断。

## 可验证输出

- 后处理依赖图，至少包含 SCF、NSCF、`pp.x`、`average.x`、`fs.x` 与对应输出。
- 一份 postprocessing output review。
- 一份图像/物理解释边界说明。
- 一份 PASS / WARN / BLOCK 与下游准入记录。

## 推荐阅读

- [workflows/electronic/charge-density.md](../workflows/electronic/charge-density.md)
- [workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)
- [workflows/electronic/elf.md](../workflows/electronic/elf.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
- [workflows/electronic/fermi-surface.md](../workflows/electronic/fermi-surface.md)

## 后续进入哪个 workflow

完成本页后，按目标进入 `workflows/electronic/charge-density.md`、`electrostatic-potential.md`、`elf.md`、`work-function.md` 或 `fermi-surface.md`。进入前必须先写明：读取的上游 `prefix/outdir` 是什么，当前后处理对象的数据来源是什么，输出允许支持哪些下游结论。

## 资料来源

- QE INPUT_PP reference: <https://www.quantum-espresso.org/Doc/INPUT_PP.html>
- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE PostProc guide: <https://www.quantum-espresso.org/Doc/pp_user_guide/>
- XCrySDen: <http://www.xcrysden.org/>
