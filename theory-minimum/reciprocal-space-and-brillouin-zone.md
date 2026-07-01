# Reciprocal space and Brillouin zone

## 本页解决什么问题

倒空间是周期晶体中描述电子态、声子态和响应函数的自然坐标系。QE 中的 `K_POINTS`、phonon q-point、band path、irreducible k-points、Fermi surface 和 Brillouin-zone integration 都建立在倒空间语言上。读懂本页后，学习者应能区分“用于积分的网格”和“用于可视化的路径”，并能解释为什么 cell、symmetry、SOC 或 magnetism 改变后，k/q 采样和路径也需要重新审阅。

## 物理图像

晶体的实空间由一组平移矢量重复生成。电子在这样的周期势中运动时，不需要用一个无限大的波函数直接描述整个晶体；Bloch theorem 说明电子态可以写成平面波相位因子乘以具有晶格周期的函数。这个相位因子的波矢就是 `k`，它标记电子态在周期平移下如何改变相位。

倒空间可以看成“相位变化的空间”。实空间晶格矢量描述原子如何周期排列，倒空间晶格矢量描述波函数相位和 Fourier 分量如何周期重复。Brillouin zone 是倒空间中的 primitive cell；由于相差一个倒格矢的 `k` 点描述等价的 Bloch 相位，电子结构只需在第一 Brillouin zone 中审阅。

声子也有类似结构。原子位移可以按波矢 `q` 分解成集体振动模式；`q=0` 表示所有原胞同相振动，有限 `q` 表示不同原胞之间存在相位差。因此 k-point 和 q-point 不是任意采样标签，而是周期体系中动量、相位和积分的基本语言。

## 最低数学结构

若实空间 primitive vectors 为 `a1, a2, a3`，倒空间 primitive vectors `b1, b2, b3` 满足：

```text
a_i · b_j = 2 pi delta_ij
```

Bloch state 的最低形式是：

```text
psi_nk(r) = exp(i k · r) u_nk(r)
```

其中 `u_nk(r)` 具有晶格周期。能带 `E_n(k)`、DOS、Fermi surface 和电子响应都来自不同 `k` 点上这些本征态的集合。实际计算中，连续 Brillouin zone 被有限 k mesh 近似；收敛测试审阅的是目标物理量随 mesh 变化是否稳定。

## QE 中的对应对象

| 对象 | QE 位置 | 含义 | output 证据 |
|---|---|---|---|
| `CELL_PARAMETERS` / `ibrav` | `pw.x` input | 定义实空间晶胞，也决定倒空间基矢 | cell、reciprocal axes、symmetry summary |
| `K_POINTS automatic` | `pw.x` card | 定义用于 BZ integration 的 uniform mesh | k-point count、weights、irreducible k-points |
| `K_POINTS crystal_b` / `tpiba_b` | `pw.x` bands input | 定义 high-symmetry path | bands k-point list |
| q-point / q-grid | `ph.x` input/output | 定义 phonon response 的倒空间采样 | q-point list、`nq1/nq2/nq3`、dyn files |
| `q_in_cryst_coord` | `matdyn.x` | 说明 phonon path 坐标基底 | `matdyn.x` input echo |
| irreducible k-points | `pw.x` output | symmetry 约化后的实际计算点 | k-point summary |

## Output 中的可见证据

- `number of k points` 与 k-point weights 显示 QE 实际读取的 mesh 或 path。
- `number of k points= ...` 后的 irreducible k-point 信息显示 symmetry 是否参与约化。
- cell 与 reciprocal axes 输出决定 `crystal`、`tpiba`、`crystal_b`、`tpiba_b` 坐标解释。
- `ph.x` q-point list 和 `dyn0`/dyn file sequence 显示 q-grid 是否完整。
- `bands.x`、`matdyn.x` 的 path 输出只能说明路径被读取，不能说明 BZ integration 已收敛。

## 对 workflow 的影响

- SCF、relax、NSCF、DOS、Fermi surface 依赖 uniform k mesh。
- bands 和 phonon dispersion 依赖 high-symmetry path，但 path 不替代 mesh convergence。
- phonon q-grid 决定 real-space IFC 的可恢复范围，影响 dispersion interpolation 和 phonon DOS。
- cell 改变后，原来“看起来相同”的 k mesh 可能对应不同 reciprocal spacing。
- symmetry 改变后，irreducible k-points、band degeneracy、phonon irreps 和 path labels 都可能变化。

## 常见误解

- 把 high-symmetry path 当作 BZ integration mesh。
- 只记录 irreducible k-points，不记录原始 full mesh 和 shift。
- 结构优化后继续沿用旧 k-path，而不确认 cell convention。
- 将 `q=0` phonon 当作完整 q-space 响应。
- 用更密 path 取代 DOS 或 Fermi surface 所需 uniform mesh。

## 判断边界

倒空间设置的 `PASS` 不是“k 点很多”，而是目标物理量对采样方式足够稳定且 output 可复查。若目标是 total energy、force、stress、DOS 或 phonon response，应检查对应 mesh；若目标是 band plot，应检查 path 来源、cell convention 和 energy reference。

## 对应 workflow

- [k-point convergence](../workflows/ground-state/kpoint-convergence.md)
- [bands](../workflows/electronic/bands.md)
- [DOS](../workflows/electronic/dos.md)
- [Fermi surface](../workflows/electronic/fermi-surface.md)
- [phonon dispersion DFPT](../workflows/phonon/phonon-dispersion-dfpt.md)

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- Monkhorst and Pack, Special points for Brillouin-zone integrations.
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
