# K-points, symmetry and k-path

## 本页解决什么问题

本页解释 k-point sampling、symmetry 和 high-symmetry k-path 在 QE 中分别解决什么问题，以及为什么 SCF/DOS 的 uniform mesh 不能被 bands path 替代。它帮助用户判断 k 点设置、irreducible k-points、symmetry 使用和 k-path 来源是否足以支撑 ground-state、electronic 和 phonon workflow。

## 最低掌握深度

最低需要知道：

- 周期体系的电子量需要在 Brillouin zone 中积分；uniform k mesh 服务积分。
- High-symmetry k-path 服务 band dispersion 可视化，不服务积分收敛。
- QE 会利用 symmetry 约化 k points；output 中 irreducible k-points 是重要审阅证据。
- 结构 relax、cell convention、SOC、magnetism、noncollinear 设置和 symmetry 开关都可能改变 k-path 和 k 点解释。

## 物理图像

周期晶体把实空间问题变成倒空间问题。Bloch theorem 说明，在周期势场中，电子态可以写成周期函数乘以相位因子；这个相位由波矢 `k` 决定。不同 `k` 点对应相邻原胞之间不同的相位关系，因此一个固体的电子结构不是少数离散能级，而是定义在 Brillouin zone 上的能量函数 `E_n(k)`。

Uniform k mesh 的作用是对 Brillouin zone 做积分。SCF total energy、charge density、force、stress、DOS 和 Fermi surface 都需要从整个 Brillouin zone 的采样中得到近似。High-symmetry k-path 的作用不同：它选取 Brillouin zone 中若干有代表性的线段，把 `E_n(k)` 画成能带图，方便观察色散、简并、带交叉和开隙。路径图能帮助解释，但它没有覆盖整个 Brillouin zone，不能替代积分网格。

Symmetry 是连接结构和倒空间采样的另一层信息。空间群描述晶体整体对称性，点群描述绕某一点的旋转、镜面和反演等操作；这些操作在倒空间中会把不同 k 点联系起来。QE 利用这些对称性减少实际计算的 irreducible k-points。若结构 relax 后晶胞变形，或 SOC、磁性、noncollinear 设置改变 Hamiltonian 的对称性，原来的 k-path、简并解释和 irreducible k-point 数量都需要重新审阅。

## 最低数学结构

实空间基矢 `a_i` 与倒空间基矢 `b_i` 满足 `a_i · b_j = 2 pi delta_ij`。Brillouin zone 是倒空间中的 primitive cell；`K_POINTS automatic` 近似的是其中的积分：

```text
integral over BZ f(k) dk  ->  sum over k_i w_i f(k_i)
```

Band path 则是若干特殊点之间的线段采样：

```text
Gamma -> X -> ...  gives E_n(k) along selected directions
```

Little group 是保持某个 k 点不变的对称操作集合。它影响该 k 点处能带简并和 mode 标记。QE 用户不需要完整群论推导，但需要知道：简并、band crossing、phonon irreps 和 SOC/magnetism 下的 symmetry reduction 都不是纯绘图问题，而会影响 output review 和下游解释。

## QE 中的对应对象

| 对象 | QE 位置 | 判断意义 | output 证据 |
|---|---|---|---|
| `K_POINTS automatic` | card / `pw.x` | SCF、relax、NSCF、DOS 前置的 uniform mesh | k-point summary、irreducible k-points |
| `<nk1> <nk2> <nk3> <sk1> <sk2> <sk3>` | `K_POINTS automatic` | mesh density 和 shift policy | reported k-points、weights |
| `K_POINTS crystal_b` / `tpiba_b` | card / bands run | high-symmetry path | bands k-point sequence |
| `nosym` / `noinv` | `&SYSTEM` | 控制 symmetry 约化 | symmetry operations、irreducible k-points |
| `ibrav` / `CELL_PARAMETERS` | `pw.x` input | direct/reciprocal cell convention | cell、reciprocal axes、symmetry |
| `bands.x` | post-processing | 整理 band data | `filband` / band data file |

相关 workflow：[k-point convergence](../workflows/ground-state/kpoint-convergence.md)、[bands](../workflows/electronic/bands.md)、[DOS](../workflows/electronic/dos.md)、[Fermi surface](../workflows/electronic/fermi-surface.md)、[phonon dispersion](../workflows/phonon/phonon-dispersion-dfpt.md)。相关理论回查：[reciprocal space and Brillouin zone](reciprocal-space-and-brillouin-zone.md)、[band structure and DOS](band-structure-and-dos.md)、[crystal symmetry, space group and point group](crystal-symmetry-space-group-point-group.md)。

## 核心概念

Uniform k mesh 是 Brillouin zone 积分网格。即使 QE 只实际计算 irreducible k-points，完整 mesh 与 symmetry weights 仍定义了积分。Bands path 是 reciprocal space 中若干线段的采样，用于显示 eigenvalue dispersion；它不覆盖整个 BZ，不能用于 total energy、DOS、Fermi surface 或 phonon 前置积分收敛。

Symmetry 决定 QE 能否约化 k points，也影响 band degeneracy、phonon irreps、SOC/magnetism 下的可比性。Symmetry 变化不是单纯输出格式变化，而可能改变计算对象。

## 对 input 的影响

- SCF、relax、NSCF、DOS 和 Fermi surface 应明确 uniform mesh 和 shift flags。
- Bands input 应记录 k-path 来源、坐标基底、labels、path 段和当前 cell convention。
- k-point convergence 中每次只改变 mesh 或按计划改变 shift，其他 cutoff、smearing、structure 和 symmetry 设置保持可归因。
- SOC、noncollinear 或磁性计算可能需要重新审阅 symmetry 和 k-path。
- 结构优化后 cell 改变时，应重新确认 k mesh density 和 bands path。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| reported k-points | QE 读取了目标 mesh 或 k-list | mesh 已收敛 |
| irreducible k-points | symmetry 约化实际生效 | symmetry 使用一定物理正确 |
| symmetry operations | 结构和设置下的 symmetry 状态 | k-path 与当前 cell 一致 |
| Fermi energy / occupations | k mesh 与 occupation 的耦合线索 | 金属性已充分收敛 |
| bands k-point sequence | path 被 bands run 读取 | path 可用于 DOS 或积分 |
| `bands.x` data | band data 来自本次 run | band gap 具备实验意义 |

## 常见误区

- 用 band path 做 DOS 或 k-point convergence。
- 把 k-path 当作 BZ integration mesh。
- 结构 relax 后不重新确认 k-path。
- 只记录 irreducible k-points，不记录完整 mesh 和 shift。
- 关闭 `nosym` / `noinv` 后不记录原因。
- SOC / magnetism 后仍沿用无 SOC/无磁性的 symmetry 解释。
- bands 与 DOS 对金属性判断冲突时只相信图像更平滑的一方。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | mesh、shift、symmetry、irreducible k-points、target observable convergence 和 k-path 来源可追踪 | 可进入对应 SCF/NSCF/DOS/bands/phonon 下游 |
| WARN | mesh 对部分目标尚未收敛；k-path 来源或 symmetry 关闭理由不完整 | 可探索，不应给定量 DOS/Fermi surface 或 band gap 结论 |
| BLOCK | 用 path 代替 uniform mesh；cell/k-path convention 错配；symmetry 输出异常未解释；k-point scan 无法归因 | 不允许进入相关下游结论 |

## 下游影响

- `SCF/relax/vc-relax`：total energy、force、stress 和 Fermi energy 依赖 k mesh。
- `NSCF/DOS/PDOS/Fermi surface`：通常需要 dense uniform mesh。
- `bands`：需要与当前 structure/cell convention 一致的 high-symmetry path。
- `phonon/DFPT`：电子响应和 Kohn anomaly 等问题可能对 k mesh 和 smearing 更敏感。
- `work function`：Fermi energy 和 slab 面内 k mesh 会影响结果。

## 与 physics-judgement 的边界

本页只解释 k 点与 symmetry 的最低使用边界。以下问题转到：

- [kmesh and smearing sensitivity](../physics-judgement/kmesh-smearing-sensitivity.md)
- [SOC and symmetry boundary](../physics-judgement/soc-and-symmetry-boundary.md)
- [band gap problem and delocalization](../physics-judgement/band-gap-problem-and-delocalization.md)

结构标准化和 k-path 生成细节不在本页展开。

## 来源与边界

- Stable: `K_POINTS` 和 symmetry 相关字段以 QE `INPUT_PW` 为准；`bands.x` 字段以 `INPUT_BANDS` 为准。
- Boundary: 本页不推荐材料无关的 k mesh；mesh 是否足够由目标 observable convergence 决定。
- Internal standard: k mesh 和 path 记录应进入 [output review checklist](../standards/output-review-checklist.md)。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_BANDS reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- Monkhorst and Pack, Special points for Brillouin-zone integrations.
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
- 本仓库：[bands workflow](../workflows/electronic/bands.md)、[DOS workflow](../workflows/electronic/dos.md)
