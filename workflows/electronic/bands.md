# Bands workflow

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 结构输入边界：k-path 依赖结构标准化；本页只说明 QE 对 k-path 输入的要求。
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

Bands workflow 沿 Brillouin zone 高对称路径计算 Kohn-Sham eigenvalues，用于判断带隙、直接/间接跃迁、能带交叉、简并，以及 SOC、磁性、DFT+U 等设置对电子结构的影响。

Bands 使用路径采样；DOS/PDOS 通常使用均匀且更密的 k 网格。两类 workflow 的输入和输出判断不能互相替代。

## 输入前提

- `<structure>` 已经优化或来自可信结构来源。
- 已完成并审阅 SCF，`prefix/outdir` 可被 bands 计算读取。
- cutoff、SCF k 点和 occupation/smearing 策略已有基本收敛证据。
- 已明确 primitive cell / conventional cell 的选择。
- 已生成或人工确认 `<k_path>`，并记录路径来源和坐标基底。

## 计算图

```text
<structure> + <pseudo>
  -> pw.x scf on <uniform_k_mesh>
  -> k-path generation / validation
  -> pw.x bands on <k_path>
  -> bands.x
  -> band data / figure / interpretation
```

## 需要的 QE 程序

- `pw.x`：先跑 `calculation='scf'`，再跑 `calculation='bands'`。
- `bands.x`：整理 bands 计算结果，输出绘图数据。
- 可选辅助：SeeK-path、spglib、Python/matplotlib。

## 通用输入模板

SCF：

```fortran
&CONTROL
  calculation = 'scf',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
/
&SYSTEM
  ibrav = 0,
  nat = <number_of_atoms>,
  ntyp = <number_of_species>,
  ecutwfc = <wavefunction_cutoff>,
  ecutrho = <charge_density_cutoff>,
  occupations = '<occupation_scheme>',
/
&ELECTRONS
  conv_thr = <scf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>

ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>

K_POINTS automatic
  <nk1> <nk2> <nk3> <sk1> <sk2> <sk3>

CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

Bands：

```fortran
&CONTROL
  calculation = 'bands',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
/
&SYSTEM
  ibrav = 0,
  nat = <number_of_atoms>,
  ntyp = <number_of_species>,
  ecutwfc = <wavefunction_cutoff>,
  ecutrho = <charge_density_cutoff>,
  nbnd = <number_of_bands>,
  occupations = '<occupation_scheme>',
/
&ELECTRONS
  conv_thr = <scf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>

ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>

K_POINTS crystal_b
  <number_of_path_points>
  <kx_1> <ky_1> <kz_1> <npoints_to_next> ! <label_1>
  <kx_2> <ky_2> <kz_2> <npoints_to_next> ! <label_2>

CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

`bands.x`：

```fortran
&BANDS
  prefix = '<system>',
  outdir = '<scratch_dir>',
  filband = 'bands.<system>.dat',
/
```

## 输入字段说明

| 字段 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|
| `calculation='bands'` | 沿给定 k-path 计算 eigenvalues | 用它替代 SCF | bands run 会读取已有 SCF 数据 |
| `prefix/outdir` | 连接 SCF 与 bands | 与 SCF 不一致 | 报找不到 charge density 或读取旧数据 |
| `nbnd` | 控制输出能带数量 | 导带数量不够，图像截断 | output 中有 number of Kohn-Sham states |
| `K_POINTS crystal_b` | 描述高对称路径 | 用 uniform mesh 画 bands | output k 点序列应对应路径 |
| `nosym` | 特定非标准路径、低维或磁性场景可能需要 | 不分析对称性就开关 | 检查对称性和 k 点是否被改写 |

## 通用输出审阅模板

```markdown
## Output Review

- QE 程序:
- 计算类型:
- QE version:
- SCF dependency:
- `prefix/outdir` consistency:
- Structure / cell convention:
- K-path source:
- K-path labels:
- Number of bands:
- Fermi energy reference:
- Band data file:
- Warnings:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

## 输出判断标准

- bands 计算应读取同一 `prefix/outdir` 下的 SCF ground state。
- `<k_path>` 必须对应当前使用的 cell convention；primitive/conventional cell 混用会导致标签错误。
- bands 图审阅的是 Kohn-Sham eigenvalues。它可以支持能带趋势、交叉检查和态成分讨论，但不能单独作为实验 quasiparticle gap、optical gap、excited-state spectrum 或拓扑结论。
- 若研究目标超出普通 ground-state DFT，应回查 [physics-judgement/kohn-sham-eigenvalue-boundary.md](../../physics-judgement/kohn-sham-eigenvalue-boundary.md) 和 [physics-judgement/ground-state-vs-excited-state.md](../../physics-judgement/ground-state-vs-excited-state.md)。
- `PASS` 只表示 bands workflow 数据链可用；若要写 gap 或 spectroscopy 定量结论，仍需要额外 model boundary 和 uncertainty statement。
- `nbnd` 应覆盖关注的价带和导带范围。
- 画图前必须明确能量零点：Fermi energy、VBM、CBM 或自定义参考。
- 高对称点标签、路径段数量和绘图横轴应与 `<k_path>` 记录一致。
- 若存在 band crossing 或异常跳变，需要检查 `bands.x` 排序和原始 eigenvalues。

## 收敛性要求

- 使用已经通过审阅的 SCF cutoff、k 点、赝势和 occupation 设置。
- 对金属或小带隙体系，检查 smearing 和 SCF k 点对 Fermi 附近能带的影响。
- 对 SOC、磁性、DFT+U、应变等体系，bands 设置应与 SCF 的物理模型保持一致。

## 常见错误与诊断

| 现象 | 可能原因 | 排查顺序 |
|---|---|---|
| `bands.x` 找不到数据 | `prefix/outdir` 不一致或 scratch 被清理 | 先看 SCF 和 bands 输入 |
| 高对称点标签不对 | cell convention 或 k-path 坐标基底错误 | 重新检查结构标准化和路径来源 |
| 导带不够 | `nbnd` 太小 | 增加 `nbnd` 并重跑 bands |
| DOS 与 bands 金属性判断不一致 | DOS k mesh 不足或能量零点不一致 | 分别检查 DOS workflow 和 Fermi reference |
| 能带跳变 | band crossing、排序、对称性或路径问题 | 检查 `bands.x` 输出和原始 eigenvalues |

## 通用学习模板

学习时应选择结构简单、电子状态明确、k-path 来源可靠的体系。具体体系不写入本仓库主文档；个人学习记录中应保留 input、output、路径来源、绘图脚本和 output review。

## 记录模板

```text
pw.scf.<system>.in
pw.scf.<system>.out
pw.bands.<system>.in
pw.bands.<system>.out
bands.<system>.in
bands.<system>.out
bands.<system>.dat
kpath-source.md
record.md
```

`kpath-source.md` 至少写明：结构标准化工具、cell convention、路径标签、坐标基底和是否人工修改。

## 与其他 workflow 的关系

- 与 SCF：bands 依赖 SCF ground state。
- 与 DOS/PDOS：bands 解释路径色散，DOS/PDOS 解释态密度和轨道贡献。
- 与 Wannier90：若需要高精度插值、Berry curvature、输运等，应进入 Wannier workflow。

## 后续完善重点

- 补充 k-path 来源记录规范。
- 补充 band figure 能量零点和标签检查规范。
- 补充 `nbnd`、SOC、磁性和 DFT+U 场景下的 bands 审阅要点。

## 资料来源

- QE `bands.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
- Pranab Das QE tutorial: <https://pranabdas.github.io/espresso/>
