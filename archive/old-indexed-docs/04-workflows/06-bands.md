# Bands workflow

## 1. 计算目标

Bands workflow 的目标是沿 Brillouin zone 中高对称路径计算 Kohn-Sham eigenvalues，并生成可解释的能带图。它用于判断带隙、直接/间接跃迁、能带交叉、简并、SOC/磁性/DFT+U 等对电子结构的影响。

Bands 不是 DOS 的替代品。Bands 使用路径采样，DOS 通常需要均匀且更密的 k 网格。

## 2. 输入前提

- 结构已经 relax 或来自可信实验结构。
- 已完成并验证 SCF。
- cutoff、k 点和 smearing 策略已有基本收敛证据。
- 已确定使用 primitive cell 还是 conventional cell。
- 已生成或手工确认高对称 k-path。

## 3. 计算图

```text
structure + pseudopotential
  -> pw.x scf on uniform k-mesh
  -> k-path generation / validation
  -> pw.x bands calculation on high-symmetry path
  -> bands.x
  -> plot data / band figure / interpretation
```

## 4. 需要的 QE 程序

- `pw.x`：先跑 `calculation='scf'`，再跑 `calculation='bands'`。
- `bands.x`：整理 bands 计算结果，输出绘图文件。
- 可选：seekpath、AiiDA `PwBandsWorkChain`、Python/matplotlib。

## 5. 输入文件模板

SCF：

```fortran
&CONTROL
  calculation = 'scf',
  prefix = 'si',
  outdir = './out',
  pseudo_dir = './pseudo',
/
&SYSTEM
  ibrav = 0,
  nat = 2,
  ntyp = 1,
  ecutwfc = 40,
  ecutrho = 320,
  occupations = 'fixed',
/
&ELECTRONS
  conv_thr = 1.0d-8,
/
ATOMIC_SPECIES
  Si 28.0855 Si.upf
ATOMIC_POSITIONS crystal
  Si 0.000000 0.000000 0.000000
  Si 0.250000 0.250000 0.250000
K_POINTS automatic
  8 8 8 0 0 0
CELL_PARAMETERS angstrom
  ...
```

Bands：

```fortran
&CONTROL
  calculation = 'bands',
  prefix = 'si',
  outdir = './out',
  pseudo_dir = './pseudo',
/
&SYSTEM
  ibrav = 0,
  nat = 2,
  ntyp = 1,
  ecutwfc = 40,
  ecutrho = 320,
  nbnd = 12,
  occupations = 'fixed',
/
&ELECTRONS
  conv_thr = 1.0d-8,
/
ATOMIC_SPECIES
  Si 28.0855 Si.upf
ATOMIC_POSITIONS crystal
  ...
K_POINTS crystal_b
  4
  0.0000 0.0000 0.0000 30 ! Gamma
  0.5000 0.0000 0.5000 30 ! X
  0.5000 0.2500 0.7500 30 ! W
  0.3750 0.3750 0.7500 1  ! K
CELL_PARAMETERS angstrom
  ...
```

`bands.x`：

```fortran
&BANDS
  prefix = 'si',
  outdir = './out',
  filband = 'bands.si.dat',
/
```

## 6. 关键参数解释

| 参数 | 作用 | 错误用法 | 输出中如何验证 |
|---|---|---|---|
| `calculation='bands'` | 沿给定 k-path 计算 eigenvalues | 用它替代 SCF | bands run 会读取已有 SCF 数据 |
| `prefix/outdir` | 连接 SCF 与 bands | 与 SCF 不一致 | 报找不到 charge density 或使用旧数据 |
| `nbnd` | 计算能带数量 | 导带不够，图像截断 | output 中有 number of Kohn-Sham states |
| `K_POINTS crystal_b` | 以路径段形式给高对称点 | 用 automatic mesh 画 bands | output k 点序列应对应路径 |
| `nosym` | 某些非标准路径/低维/磁性场景可能需要 | 无脑关闭或无脑开启 | 检查对称性和 k 点是否被改写 |

## 7. 输出文件与判断标准

必须检查：

- SCF 是否已经可靠完成。
- bands 计算是否读取了同一 `prefix/outdir`。
- k-path 是否对应目标晶体的 primitive cell。
- `nbnd` 是否覆盖关注的价带和导带范围。
- Fermi energy 参考是否清楚；画图时是否统一能量零点。
- 若存在交叉或简并，`bands.x` 排序结果是否需要人工检查。

图像判断：

- 带隙大小、直接/间接类型。
- Fermi level 是否穿过能带。
- 高对称点标签是否与路径一致。
- 异常断裂、突跳或重复路径是否来自 k-path/坐标错误。

## 8. 收敛性要求

Bands 图的漂亮不代表结果可信。至少需要：

- 使用已收敛的 SCF cutoff/k 点/赝势。
- 对金属或小带隙体系检查 smearing 和 k 点对 Fermi 附近能带的影响。
- 对 SOC、磁性、DFT+U、应变等体系，保持与 SCF 设置完全一致。

## 9. 常见错误与诊断

| 现象 | 可能原因 | 排查顺序 |
|---|---|---|
| bands.x 找不到数据 | `prefix/outdir` 不一致或 scratch 被清理 | 先看 SCF 和 bands 输入 |
| 图上高对称点不对 | primitive/conventional cell 混用 | 用 seekpath 或 spglib 重新标准化 |
| 导带不够 | `nbnd` 太小 | 增加 `nbnd` 并重跑 bands |
| DOS 与 bands 金属性判断不一致 | DOS k mesh 不足或能量零点不一致 | 分别检查 DOS workflow 和 Fermi reference |
| 能带乱跳 | band crossing、排序、对称性或路径问题 | 检查 `bands.x` 输出和原始 eigenvalues |

## 10. 最小可运行案例

建议第一版使用 Si：

- SCF：`pw.scf.Si.in`
- Bands：`pw.bands.Si.in`
- Post：`bands.Si.in`
- Plot：`bands.Si.dat` -> Python/matplotlib

必须记录：

- k-path 来源：seekpath、教程、文献或手工。
- 使用 primitive cell 还是 conventional cell。
- 能量零点：Fermi energy、VBM 或其他。

## 11. 与其他 workflow 的关系

- 与 SCF：bands 依赖 SCF ground state。
- 与 DOS/PDOS：bands 解释沿路径的色散，DOS/PDOS 解释态密度和轨道贡献，二者互补。
- 与 Wannier90：若需要高精度插值、Berry curvature、输运等，应进入 Wannier workflow。

## 12. GitHub 记录规范

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

`kpath-source.md` 至少写明：标准化工具、晶胞选择、路径标签、坐标基底。

## 13. 资料来源

- 已有调研：`Quantum ESPRESSO as a Modern Research Stack.md`
- 已有调研：`Quantum ESPRESSO Tutorial Website Ecosystem.md`
- QE `bands.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
- AiiDA Quantum ESPRESSO docs: <https://aiida-quantumespresso.readthedocs.io/>

