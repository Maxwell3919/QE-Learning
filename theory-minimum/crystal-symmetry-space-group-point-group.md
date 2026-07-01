# Crystal symmetry, space group and point group

## 本页解决什么问题

QE 会根据输入结构、晶胞、原子坐标、磁性、SOC、约束和数值容差识别或使用对称性。对称性影响 irreducible k-points、band degeneracy、phonon irreducible representations、selection rules、Berry/Wannier 分析和许多 output 的可解释性。本页给出空间群、点群、小群和倒空间对称性的最低背景，帮助读者知道什么时候应信任 symmetry reduction，什么时候需要重新审阅结构和路径。

## 物理图像

晶体对称性描述哪些操作会把晶体映射回自身。平移对称性给出晶格；点群操作描述旋转、镜面、反演等不改变原点的操作；空间群把点群操作和平移、螺旋轴、滑移面等组合起来。电子和声子的本征态会按这些对称性分类，因此对称性不仅节省计算量，也决定简并、选择定则和 band/path 标记。

倒空间继承实空间对称性。某个 `k` 点在对称操作下可能被映射到另一个等价 `k` 点；QE 可以只计算 irreducible wedge 中的代表点，再用 weights 完成 BZ integration。沿 high-symmetry path 画 bands 时，路径上的端点和线段通常对应高对称点、轴或面；这些标签只有在结构 convention 和 symmetry 与 path 来源一致时才有意义。

小群是保持某个 `k` 点不变的对称操作集合。Band degeneracy、irreducible representation labels 和 `bands.x` 的 symmetry classification 都与小群有关。SOC、noncollinear magnetism、磁性构型或结构畸变会改变可用小群，因此同一条 path 在不同模型下不能机械复用解释。

## 最低数学结构

一个空间群操作可以概念性写成：

```text
r -> R r + tau
```

其中 `R` 是旋转、镜面或反演等点操作，`tau` 是平移部分。倒空间中 `k` 的等价性来自 `R k` 与 `k + G` 的关系。若某个操作把 `k` 映射回等价点，它属于该 `k` 点的小群。

## QE 中的对应对象

| 对象 | QE 位置 | 含义 | output 证据 |
|---|---|---|---|
| `ibrav` / `CELL_PARAMETERS` | `pw.x` input | 定义晶胞与对称性识别基础 | cell、symmetry operations |
| `ATOMIC_POSITIONS` | `pw.x` input | 原子坐标决定实际 symmetry | equivalent atoms、symmetry summary |
| `nosym` / `noinv` | `&SYSTEM` | 控制 symmetry 使用 | irreducible k-points、symmetry section |
| spin / SOC settings | `nspin`、`noncolin`、`lspinorb` | 改变 time-reversal、spin rotation 和可用 symmetry | spin/SOC output、symmetry reduction |
| `bands.x lsym` | `bands.x` | band irreps / small-group classification | `filband.rap` |
| phonon irreps | `ph.x` / `dynmat.x` | Gamma 或 q-point 模式分类 | irreducible representations、mode labels |

## Output 中的可见证据

- `pw.x` setup 中的 symmetry operations 和 equivalent atoms 显示 QE 识别到的结构对称性。
- irreducible k-points 与 full mesh 的比例显示 symmetry reduction 是否生效。
- `bands.x` 的 `.rap` 文件或 symmetry labels 支持 band representation 审阅，但不自动给出拓扑或实验解释。
- `ph.x` 的 irreps 和 mode symmetry 支持 Gamma modes、IR/Raman activity 或 mode assignment 的最低审阅。
- SOC/noncollinear 输出和 warning 必须与 symmetry 使用一起审阅。

## 对 workflow 的影响

- k-point convergence 需要记录 full mesh、shift 和 irreducible points。
- bands path 需要与结构标准化、cell convention 和 symmetry 保持一致。
- DOS/PDOS 通常更依赖 uniform mesh；symmetry 影响 weights 和投影通道。
- phonon irreps、acoustic modes、Raman/IR activity 都依赖 symmetry。
- SOC 和磁性会改变 degeneracy、path labels、Fermi surface 和 band crossing 解释。

## 常见误解

- 看到 symmetry reduction 就认为结构正确。
- cell 或原子坐标轻微变化后仍沿用旧 high-symmetry path。
- 开启 SOC 后仍用无 SOC 的简并关系解释 bands。
- 关闭 symmetry 后不记录原因，导致 k-point、DOS 或 phonon 结果不可复查。
- 把 `bands.x` symmetry label 当作完整物理结论。

## 判断边界

对称性审阅的目标是确认计算对象和解释语言一致。若结构来源、cell convention、磁性、SOC 或 symmetry flags 改变，应重新检查 k-path、irreducible k-points、band degeneracy 和 phonon mode labels。若 output symmetry 与预期不符，应先进入 `WARN` 或 `BLOCK`，回到结构、坐标容差、磁性和 SOC 设置排查。

## 对应 workflow

- [k-point convergence](../workflows/ground-state/kpoint-convergence.md)
- [bands](../workflows/electronic/bands.md)
- [PDOS](../workflows/electronic/pdos.md)
- [Gamma phonon](../workflows/phonon/gamma-phonon.md)
- [IR/Raman](../workflows/phonon/ir-raman.md)
- [SOC and symmetry boundary](../physics-judgement/soc-and-symmetry-boundary.md)

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_BANDS reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
