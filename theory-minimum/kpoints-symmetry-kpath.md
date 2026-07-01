# K-points, symmetry and k-path

## QE 中对应的问题

k 点、symmetry 和 k-path 在 QE 中回答的是三类不同问题：

- uniform k mesh 是否足够支撑 Brillouin zone 积分，以及 total energy、force、stress、Fermi energy、DOS、phonon 等目标量是否对 k 点采样收敛；
- symmetry 是否被 QE 正确识别，并用于约化 irreducible k-points、检查结构等价性和减少计算量；
- high-symmetry k-path 是否对应当前结构的 reciprocal basis 和 cell convention，能否用于 bands 图中的路径色散解释。

最低判断应关注当前 workflow 使用的 k 点语义是否正确，并且 output 中是否有足够证据说明它可进入下游，而不能只看 k 点数是否“看起来够多”。SCF、NSCF dense mesh 和 bands path 不能互相替代：

- SCF / relax / phonon 前置计算使用 uniform k mesh 做积分，通常写作 `K_POINTS automatic`。
- DOS / PDOS / Fermi surface 通常需要比 SCF 更密的 uniform NSCF mesh。
- bands 使用 high-symmetry path，通常写作 `K_POINTS crystal_b` 或等价路径格式，只用于沿路径读取 eigenvalues，不用于 Brillouin zone 积分收敛。

### uniform k mesh 与 high-symmetry path

uniform k mesh 是积分网格。它覆盖整个 Brillouin zone，目标是让积分型物理量随网格加密而稳定。即使 QE 利用 symmetry 只实际计算 irreducible k-points，input 中的 mesh 仍然代表完整均匀采样。

high-symmetry path 是绘图路径。它只沿若干高对称点之间的线段取样，目标是展示特定方向上的能带色散。path 上的点不构成均匀积分网格，不能用来判断 total energy、DOS、Fermi surface 或 phonon 前置 SCF 是否收敛。

因此，bands workflow 的基本顺序是：先用已收敛的 uniform k mesh 得到 ground state，再沿记录清楚的 high-symmetry path 运行 `calculation='bands'`，最后用 `bands.x` 整理绘图数据。

### irreducible k-points 与 symmetry

QE 会根据结构、晶胞、原子位置、数值容差和 input 中的 symmetry 设置识别空间群相关操作，并把完整 k mesh 约化为 irreducible k-points。output 中看到的 irreducible k-points 少于 input mesh 总点数时，通常说明 symmetry 约化已经生效；是否合理需要结合结构和 symmetry 设置审阅。

需要审阅的是：

- 约化后的 irreducible k-points 数量是否与 symmetry 设置和结构维度大致一致；
- 修改 `nosym`、`noinv`、磁性、SOC、非共线设置或结构畸变后，symmetry operations 和 irreducible k-points 是否按预期变化；
- k mesh convergence 记录是否同时保存完整 mesh、shift flags、symmetry 设置和 output 报告的 irreducible k-points。

如果为了诊断、非标准路径、低维 slab、磁性或 SOC 等原因关闭某些 symmetry，记录中必须说明原因。不能只因为 bands 图或 DOS 看起来更顺就随意开关 symmetry。

### shifted / unshifted mesh

`K_POINTS automatic` 的第二行通常写作：

```text
<nk1> <nk2> <nk3> <sk1> <sk2> <sk3>
```

前三个数字是沿 reciprocal lattice 三个方向的 mesh 大小，后三个数字是 shift flags。`0 0 0` 表示 unshifted mesh，包含 Gamma 点；`1 1 1` 或部分方向为 `1` 表示沿对应方向平移半个网格间距。shift 会改变实际采样点、irreducible k-points 和某些对称点是否被直接采到。

shifted 和 unshifted 没有脱离体系和目标 workflow 的绝对优劣。最低要求是：k convergence 中不要同时改变 mesh 和 shift；一旦选择某个 shift policy，应在 SCF、NSCF、DOS、phonon 前置记录中保持可复查。

### 2D、金属和小带隙敏感性

k 点敏感性取决于电子结构、维度和目标量：

- 2D/slab：真空方向通常不需要密 k 点，但面内方向可能需要更密 mesh；如果结构实际是低维体系，应检查非周期方向采样是否符合模型假设。
- 金属：Fermi surface 附近占据变化快，SCF、DOS、Fermi energy、force 和 phonon 可能同时对 k mesh 与 smearing 敏感。
- 小带隙体系：粗 k mesh 可能误判带隙、带边位置或金属性；bands path 也可能错过真正 VBM/CBM，因此需要结合 dense mesh 或 DOS/NSCF 判断。
- phonon/DFPT：电子响应和 Kohn anomaly 等问题可能比普通 total energy 更依赖 k mesh 和 smearing，不能只沿用粗略 SCF 经验。

### k-path 与结构标准化

k-path 的标签和坐标只相对于某个标准化后的 cell convention 有意义。若 path 来自 SeeK-path、spglib 或人工整理，记录中至少要写明：

- 输入结构是否被标准化；
- 使用 primitive cell 还是 conventional cell；
- k 点坐标基底是 crystal reciprocal coordinates 还是其他约定；
- high-symmetry labels、路径段顺序和是否人工修改；
- bands input 中的 `CELL_PARAMETERS` / `ibrav` 是否与生成 k-path 的 cell convention 一致。

本仓库不展开结构标准化操作细节；这里只要求 bands workflow 能复查 k-path 与当前 QE 结构输入是否同一套 convention。

## 输入字段表

| 字段 | 所属位置 | 作用 | 需要成组固定/记录的对象 | output 中如何复核 | 常见误用 |
|---|---|---|---|---|---|
| `K_POINTS automatic` | card / `pw.x` | 定义 SCF、relax、NSCF、DOS 前置等 uniform k mesh | mesh 大小、shift flags、结构、cutoff、smearing、symmetry 设置 | output 中 k 点数量、irreducible k-points、k 点列表或 summary | 用 high-symmetry path 代替积分网格；只记录总点数不记录 shift |
| `<nk1> <nk2> <nk3>` | `K_POINTS automatic` 第二行 | 控制 reciprocal basis 三个方向的采样密度 | cell shape、维度、目标量、是否做 convergence scan | output 中 reported k-points；目标量随 mesh 的变化表 | 2D/slab 真空方向和面内方向同等加密；不同方向密度与晶胞长度不匹配 |
| `<sk1> <sk2> <sk3>` | `K_POINTS automatic` 第二行 | 控制 shifted/unshifted mesh | mesh scan、symmetry、smearing、下游 NSCF/DOS policy | output k 点列表、irreducible k-points 数量、input record | k convergence 中同时改 mesh 和 shift；重跑时忘记记录 shift |
| `K_POINTS crystal_b` | card / `pw.x` bands | 定义 high-symmetry k-path 和每段插值点数 | k-path 来源、labels、cell convention、`calculation='bands'` | bands output 的 k 点顺序；绘图横轴和标签 | 用 path 做 SCF/DOS/收敛；path 坐标与当前 cell 不一致 |
| `K_POINTS crystal` / explicit list | card / `pw.x` | 显式给出 k 点和权重，可能用于特殊 NSCF 或路径采样 | 每个 k 点、权重、坐标基底、是否用于积分 | output 打印的 k 点和权重 | 权重设置与用途不匹配；把普通显式列表误当作标准 bands path |
| `nosym` | `&SYSTEM` / `pw.x` | 关闭晶体 symmetry 对计算的使用 | 关闭原因、磁性/SOC/低维/非标准路径需求、对比 run | output 中 symmetry operations 和 irreducible k-points 变化 | 为了避免报错或修图随意打开；不记录原因 |
| `noinv` | `&SYSTEM` / `pw.x` | 控制是否使用反演约化 | 是否有反演、SOC/非共线/磁性设置、时间反演相关假设 | output 中 k 点约化变化 | 不理解反演和时间反演关系时随意设置 |
| `ibrav` / `CELL_PARAMETERS` | `&SYSTEM` 和 card / `pw.x` | 定义 direct cell，从而定义 reciprocal basis | 结构来源、单位、primitive/conventional convention、k-path 来源 | output 中 cell parameters、reciprocal axes、symmetry 信息 | QE input cell 与 k-path 生成 cell 不一致 |
| `ATOMIC_POSITIONS` | card / `pw.x` | 原子位置决定 symmetry 识别 | 坐标类型、结构是否已优化/标准化、容差影响 | output 中 atomic positions、symmetry operations | 微小畸变或坐标截断导致 symmetry 与预期不符 |
| `occupations` / `smearing` / `degauss` | `&SYSTEM` / `pw.x` | 与金属和小带隙体系的 k 点收敛耦合 | k mesh scan 全程固定；smearing scan 单独记录 | occupation scheme、Fermi energy、SCF 稳定性 | 把 k mesh 不足误解释为 smearing 选择问题，或反过来 |
| `calculation='bands'` | `&CONTROL` / `pw.x` | 沿给定 k-path 读取 ground state 后计算 eigenvalues | SCF 的 `prefix/outdir`、`nbnd`、k-path、结构 convention | output 是否读取已有 charge density；number of bands；k 点序列 | 以为 bands run 会重新完成 ground-state 积分 |

## output review

### 基本一致性

- output 中的 cell parameters、reciprocal axes、原子位置和 symmetry operations 是否与输入结构记录一致。
- `K_POINTS automatic` 的 mesh 和 shift 是否可从 input/output 复查；output 中 reported k-points 和 irreducible k-points 是否已记录。
- 使用 `nosym` / `noinv` 时，output 中 symmetry operations、irreducible k-points 和记录中的关闭理由是否一致。
- bands 计算是否读取同一 `prefix/outdir` 下的 SCF ground state，而不是读取旧 scratch 或另一个结构的 charge density。

### uniform mesh 判断

- k mesh convergence 是否只改变 mesh 或按计划改变 shift，其他 cutoff、structure、pseudo、smearing 和 SCF 阈值保持固定。
- total energy、force、stress、Fermi energy、DOS 近 Fermi 区域、phonon frequency 等目标量是否随 mesh 加密趋于稳定。
- 对金属、小带隙和 2D/slab 体系，是否额外审阅 Fermi 附近量、面内/非周期方向采样和 smearing 耦合。
- DOS/PDOS/Fermi surface 是否使用了合适的 dense NSCF uniform mesh，而不是只继承粗 SCF mesh。

### bands path 判断

- `<k_path>` 的来源、labels、路径段顺序、坐标基底和 cell convention 是否写入记录。
- `K_POINTS crystal_b` 中的点数、插值段和注释标签是否与 `kpath-source.md` 或等价记录一致。
- bands output 的 k 点顺序是否对应记录的 path；`bands.x` 输出的数据文件是否来自本次 bands run。
- 图中的高对称点标签、横轴断点和能量零点是否与 output review 一致。

### PASS / WARN / BLOCK

- PASS：uniform mesh 的目标量已有收敛记录，symmetry/shift 设置可复查，bands path 与当前 cell convention 一致，下游 workflow 依赖关系清楚。
- WARN：SCF mesh 足够用于探索但 DOS/phonon/Fermi surface 仍缺 dense mesh；或 k-path 来源可用但标准化记录不完整；或 symmetry 设置合理但关闭原因写得不充分。
- BLOCK：用 high-symmetry path 做积分；k mesh scan 同时改变多个变量且无法归因；irreducible k-points/symmetry 与预期严重不符但未解释；primitive/conventional cell 混用导致 bands 标签不可复查。

## 常见错误

| 错误 | 为什么危险 | 应如何修正 |
|---|---|---|
| 用 high-symmetry path 做 SCF、DOS 或 k 点收敛 | path 不是 Brillouin zone 积分网格，不能代表全区采样 | SCF/DOS 使用 uniform mesh；bands path 只用于 bands workflow |
| 用太粗 k mesh 判断金属性、小带隙或 Fermi surface | 粗采样可能漏掉带边、交叉或 Fermi 面细节 | 做 k mesh convergence；必要时增加 dense NSCF 或 DOS 检查 |
| 只记录 irreducible k-points，不记录原始 mesh 和 shift | 无法复现实完整采样，也无法判断 symmetry 约化来源 | 同时记录 `<nk1> <nk2> <nk3> <sk1> <sk2> <sk3>` 和 output 的 irreducible k-points |
| k convergence 中同时改变 mesh、shift、smearing 或 cutoff | 目标量变化无法归因 | 每次 scan 只改变一个策略维度，其他输入固定 |
| primitive cell 与 conventional cell 混用 | high-symmetry labels 和 k 点坐标会对应错误路径 | 记录 k-path 生成时的 cell convention，并让 QE bands input 使用同一 convention |
| 关闭 `nosym` / `noinv` 后不记录原因 | 计算量、约化规则和可比性改变，下游无法判断是否合理 | 在 output review 中写明关闭原因、预期影响和对比证据 |
| 2D/slab 沿真空方向设置过密或面内过稀 | 浪费计算或导致面内电子结构未收敛 | 按周期方向和目标量设置 mesh，并在记录中说明低维假设 |
| bands 与 DOS 对金属性判断冲突时只相信 bands 图 | path 可能没穿过关键 k 区域，DOS 也可能 mesh 不足 | 分别复查 bands path、DOS dense mesh、Fermi reference 和 smearing |
| 改结构后沿用旧 k-path | symmetry 和 reciprocal basis 可能改变，标签不再对应 | 结构优化、标准化或 cell convention 改变后重新生成/确认 k-path |

## 最低掌握深度

读完本页后，最低应能完成以下判断：

- 能在 QE input 中区分 `K_POINTS automatic`、显式 k 点列表和 `K_POINTS crystal_b` 的用途。
- 能解释 uniform k mesh 是积分采样，high-symmetry path 是 bands 绘图路径，二者不能互相替代。
- 能从 output 中找到 symmetry operations、reported k-points、irreducible k-points、Fermi energy 和 bands k 点序列等审阅对象。
- 能说明 shifted/unshifted mesh 的记录方式，并在 k convergence 中保持 shift policy 可复查。
- 能判断 2D/slab、金属、小带隙、DOS、Fermi surface 和 phonon 前置计算为什么通常比普通 SCF 更依赖 k 点策略。
- 能写出 k mesh convergence 的 PASS / WARN / BLOCK，而不是只写“k 点够密”。
- 能为 bands workflow 记录 k-path 来源、labels、cell convention、坐标基底和是否人工修改。
- 能识别 primitive/conventional cell 混用、关闭 symmetry 未记录、path 与当前结构不一致等会阻断下游解释的问题。

## 对应 workflow

- [K-point convergence workflow](../workflows/ground-state/kpoint-convergence.md)
- [Bands workflow](../workflows/electronic/bands.md)
- [DOS workflow](../workflows/electronic/dos.md)
- [Smearing and occupations workflow](../workflows/ground-state/smearing-and-occupations.md)
- [Phonon dispersion DFPT workflow](../workflows/phonon/phonon-dispersion-dfpt.md)
- [SeeK-path reference](../references/tools/seekpath.md)
- [physics-judgement/kmesh-smearing-sensitivity.md](../physics-judgement/kmesh-smearing-sensitivity.md)

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_BANDS reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
