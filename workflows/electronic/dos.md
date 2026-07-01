# DOS workflow

## 本页解决什么问题

本页说明如何审阅 total DOS workflow：从已审阅 SCF 和 uniform dense NSCF 数据生成 density of states，并判断 DOS 曲线是否可以用于 Fermi level 附近态密度、带隙趋势和与 bands/PDOS 的交叉检查。DOS 依赖 Brillouin zone 积分，不能用 high-symmetry bands path 代替。

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 上游依赖：[workflows/ground-state/scf.md](../ground-state/scf.md)、[workflows/ground-state/nscf.md](../ground-state/nscf.md)
- 理论边界：[theory-minimum/kpoints-symmetry-kpath.md](../../theory-minimum/kpoints-symmetry-kpath.md)、[theory-minimum/smearing-metals.md](../../theory-minimum/smearing-metals.md)
- 规范入口：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 上游依赖

- SCF ground state 已通过 output review。
- NSCF 使用与目标 DOS 匹配的 dense uniform k mesh；路径型 k-list 不可用于 DOS。
- `prefix/outdir` 指向同一结构、赝势、泛函、spin/SOC/occupation 设置的数据链。
- `occupations`、`smearing`、`degauss` 或 tetrahedron 相关策略已记录。
- `Emin/Emax/DeltaE` 覆盖目标能区，且 energy zero/Fermi reference 明确。

## 计算图

```text
final static SCF
  -> pw.x nscf on <dense_uniform_k_mesh>
  -> dos.x
  -> fildos
  -> output review
```

![Bands, DOS and PDOS evidence boundary](../../assets/diagrams/theory-minimum/bands-dos-evidence-boundary.svg)

图：bands、DOS 和 PDOS 的证据边界。DOS 来自 uniform BZ sampling 和 broadening；它可以与 bands/PDOS 交叉审阅，但不能用平滑曲线替代 k-mesh、smearing 和 energy reference 检查。

## 关键 QE 输入对象

| 字段 / 设置 | 程序 | 控制什么 | 常见风险 | Output 中如何验证 |
|---|---|---|---|---|
| `K_POINTS automatic` | `pw.x nscf` | DOS 积分网格 | 用 bands path 做 DOS | NSCF output 的 k 点数量和 irreducible k-points |
| `occupations/smearing/degauss` | `pw.x`, `dos.x` | 金属占据和 DOS 展宽 | 未记录 broadening 却解释峰位 | output 中 occupation、Fermi energy 和 DOS 设置 |
| `bz_sum` | `dos.x` | DOS BZ 求和方法 | 与 NSCF 网格或目标不匹配 | `dos.x` output / input record |
| `fildos` | `dos.x` | DOS 输出文件 | 文件覆盖或来源不明 | 生成的 DOS 文件 |
| `Emin/Emax/DeltaE` | `dos.x` | 能量窗口和步长 | 未覆盖目标能区或分辨率误判 | DOS 文件首列范围和间隔 |
| `prefix/outdir` | `dos.x` | 读取上游数据 | 读取旧 NSCF 或错误结构 | `dos.x` output 的读取信息 |

## 命令与文件边界

```bash
pw.x -in pw.nscf.<system>.in > pw.nscf.<system>.out
dos.x -in dos.<system>.in > dos.<system>.out
```

`dos.x` 读取的是 `prefix/outdir` 中的 NSCF 数据，不读取 `pw.nscf.<system>.out` 文件本身。`fildos` 是可审阅的 DOS 数据产物；绘图脚本应记录能量零点是否移动到 Fermi level、VBM/CBM 或其他参考。

## 通用输入模板

```fortran
&DOS
  prefix = '<system>',
  outdir = '<scratch_dir>',
  fildos = 'dos.<system>.dat',
  Emin = <energy_min>,
  Emax = <energy_max>,
  DeltaE = <energy_step>,
  bz_sum = '<bz_integration_method>',
  degauss = <dos_broadening>,
/
```

## Output review

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 | WARN/BLOCK 触发 |
|---|---|---|---|---|
| NSCF 上游 | NSCF output、`dos.x` 读取信息、`prefix/outdir` | DOS 来自目标 dense uniform mesh | k mesh 已对目标特征收敛 | 读取路径型 k-list 或错误 scratch 为 `BLOCK` |
| k-points | NSCF k-point summary | DOS 积分网格可复查 | DOS 峰位已定量可信 | mesh 太稀且用于定量峰位为 `WARN/BLOCK` |
| occupation / broadening | NSCF 和 `dos.x` input/output | smearing、degauss、tetrahedron 选择可复查 | 平滑曲线代表物理精度 | broadening 未记录或影响结论为 `WARN` |
| energy grid | `fildos`、`Emin/Emax/DeltaE` | 能量窗口和分辨率可复查 | 目标能区之外也可靠 | 能区缺失或步长不适合目标为 `WARN/BLOCK` |
| Fermi reference | NSCF output、DOS 文件注释、plot script | 能量零点来源明确 | DFT Fermi level 或 gap 具实验定量意义 | Fermi/VBM/CBM 参考混乱为 `BLOCK` |
| 与 bands 对照 | bands 图、DOS near Fermi level | 金属性/带隙趋势可交叉检查 | 两者一致就消除模型误差 | DOS 与 bands 冲突未解释为 `WARN/BLOCK` |

## 收敛与可靠性

- DOS 的 k mesh 和 broadening 需要按目标 observable 审阅；曲线平滑不等于收敛。
- 金属体系的 DOS at Fermi level 对 k mesh、smearing 和 Fermi level 更敏感。
- band gap 或 metallicity 判断应与 bands、SCF occupation 和 Fermi energy 交叉审阅。
- total DOS 与 PDOS 的差异可能来自 projection 定义、能量网格、broadening 或上游波函数，不应直接解释为物理矛盾。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | SCF/NSCF 为 `PASS`；uniform dense mesh 可追踪；DOS 文件完整；energy grid、broadening 和 Fermi reference 已记录；与 bands 基本一致或差异已解释 | 允许进入 DOS 图、PDOS 对照和谨慎的态密度趋势判断 |
| `WARN` | mesh/broadening 只适合探索；能区略窄；DOS 与 bands 有可解释但未完全收敛的差异 | 只允许探索性图像和参数调试 |
| `BLOCK` | 使用 bands path；上游 NSCF 为 `BLOCK`；`fildos` 缺失；energy zero 不明；DOS 与 bands 冲突且无法解释 | 不允许进入 DOS 解释、图件归档或定量结论 |

## 常见误区

- 用 high-symmetry k-path 数据计算 DOS。
- 看到 DOS 曲线平滑就认为 k mesh 收敛。
- 不记录 `degauss`、`bz_sum`、energy zero，却解释 DOS peak 或 band gap。
- 把 semilocal DFT DOS gap 写成实验 gap。
- DOS 与 bands 冲突时只改绘图脚本，不回查 NSCF 和 Fermi reference。

## 下游影响

DOS 可用于 figures、DOS/PDOS interpretation、Fermi-level analysis、Fermi surface 前置判断和 publication record。进入 PDOS、Fermi surface、Wannier 或 EPC 前，DOS 只能作为交叉审阅证据，不替代各自的数据链收敛。

## 来源与边界

- QE `dos.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- 本仓库规范：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
- 物理边界：[physics-judgement/kmesh-smearing-sensitivity.md](../../physics-judgement/kmesh-smearing-sensitivity.md)、[physics-judgement/kohn-sham-eigenvalue-boundary.md](../../physics-judgement/kohn-sham-eigenvalue-boundary.md)、[physics-judgement/04-band-gap-problem-and-derivative-discontinuity.md](../../physics-judgement/04-band-gap-problem-and-derivative-discontinuity.md)
