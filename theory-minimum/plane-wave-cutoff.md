# Plane-wave cutoff

## 本页解决什么问题

本页解释 `ecutwfc` 和 `ecutrho` 为什么是 QE 中最基本的数值收敛参数，以及它们如何影响 SCF、relax、vc-relax、bands、DOS、charge density、work function 和 phonon/DFPT。它帮助用户判断 cutoff scan 是否足以让当前 output 进入下游，而不是记住某个材料无关的 cutoff 数字。

## 最低掌握深度

最低需要知道：

- QE 使用 plane waves 展开 Kohn-Sham states；cutoff 控制基组大小。
- `ecutwfc` 控制 wavefunction basis，`ecutrho` 控制 charge density / potential cutoff 和相关 FFT grid。
- cutoff 是系统性数值收敛参数，不是物理模型参数；但它和 pseudopotential 类型强耦合。
- total energy、force、stress、phonon frequency、Born charge、work function 等对 cutoff 的敏感性不同。

## 物理图像

Plane-wave basis 把周期体系中的 Kohn-Sham 波函数写成许多不同波长的平面波叠加。波长越短，对应的 reciprocal vector 越大，也越能描述靠近原子核区域的快速变化和复杂节点结构。有限计算不能保留无限多平面波，因此 QE 用 kinetic-energy cutoff 只保留动能低于阈值的分量。

`ecutwfc` 控制波函数可以包含多少高频分量；`ecutrho` 控制电荷密度、势和相关 FFT 网格的表示能力。由于电荷密度由波函数乘积构成，它通常包含比波函数更高的 Fourier 分量。USPP 和 PAW 还引入 augmentation charge，使 density cutoff、smooth grid、force 和 stress 的收敛不能简单从 `ecutwfc` 推断。

Cutoff 的直觉是“分辨率”：cutoff 越高，能表示的最短波长越短，基组误差通常越小。但科研判断关注的不是 cutoff 数值本身，而是目标量是否稳定。Total energy 可能比 force、stress、phonon frequency、Born charge 或 work function 更早看起来平滑；这也是 cutoff convergence 必须按目标 observable 设计的原因。

## 最低数学结构

Plane-wave 展开可写成：

```text
psi_nk(r) = sum_G c_nk(G) exp[i(k+G)·r]
```

QE 中 `ecutwfc` 选择满足 `(1/2)|k+G|^2` 低于阈值的平面波集合，`ecutrho` 选择 density/potential 相关 Fourier 分量和 FFT grid。提高 cutoff 会扩大基组并改变 Hamiltonian 的数值表示；只有当目标 output 随 cutoff scan 稳定时，才可以把 cutoff 误差视为当前结论下可接受。

## QE 中的对应对象

| 对象 | QE 位置 | 判断意义 | output 证据 |
|---|---|---|---|
| `ecutwfc` | `&SYSTEM` / `pw.x` | wavefunction kinetic-energy cutoff，单位 `Ry` | kinetic-energy cutoff、basis/FFT summary |
| `ecutrho` | `&SYSTEM` / `pw.x` | charge density / potential cutoff，单位 `Ry` | charge density cutoff、FFT grid |
| `ATOMIC_SPECIES` / `.UPF` | card / file | PP 类型影响 cutoff 需求 | loaded pseudopotential、PP type、valence |
| `tprnfor` / `tstress` | `&CONTROL` | 单点 cutoff scan 中打印 force/stress | force table、stress tensor |
| `ph.x` frequencies | `ph.x` / `matdyn.x` | DFPT 下游对 cutoff 更敏感 | frequency shift、acoustic branch、imaginary modes |

相关 workflow：[cutoff convergence](../workflows/ground-state/cutoff-convergence.md)、[SCF](../workflows/ground-state/scf.md)、[phonon dispersion](../workflows/phonon/phonon-dispersion-dfpt.md)。相关标准：[output review checklist](../standards/output-review-checklist.md)。

## 核心概念

Plane-wave basis 用 reciprocal space 中的平面波表示周期体系的 Kohn-Sham states。cutoff 越高，保留的平面波越多，数值误差通常更小，但计算量增加。`ecutrho` 对 charge density 和 potential 的表示尤其重要；对 USPP/PAW 相关增强电荷、stress、phonon 和 response quantities，需要单独审阅。

Cutoff convergence 不是“参数越大越好”，而是目标 observable 在可接受成本内达到稳定。

## 对 input 的影响

- `ecutwfc` 和 `ecutrho` 应与 pseudopotential 来源、类型和推荐范围一起记录。
- cutoff scan 中应固定 structure、pseudopotential、XC、k mesh、occupation、smearing 和 SCF threshold。
- 若目标是 force/stress，需要确保 output 打印对应量；若目标是 phonon，需要在 phonon workflow 中复查频率和 acoustic branches。
- 换 PP、换 XC、换 valence configuration、换结构维度或进入更敏感下游时，旧 cutoff policy 不能自动继承。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| wavefunction cutoff | `ecutwfc` 实际生效 | 目标 observable 已收敛 |
| charge density cutoff | `ecutrho` 实际生效 | USPP/PAW、stress、phonon 已可靠 |
| loaded pseudopotential | PP 类型和 valence 可追踪 | PP transferability 已验证 |
| FFT grid / smooth grid | density/potential 网格可复查 | 网格无 aliasing 或 stress 噪声 |
| total energy vs cutoff | 第一层数值趋势 | force/stress/phonon 同样稳定 |
| force/stress vs cutoff | 结构优化和 cell 下游可审阅 | phonon 已稳定 |
| phonon frequency vs cutoff | response 下游稳定性线索 | 物理模型误差已消除 |

## 常见误区

- 把 cutoff 写成材料无关的万能推荐值。
- 只提高 `ecutwfc`，不检查 `ecutrho`。
- 只用 single-point total energy 判定所有下游可信。
- 换赝势后沿用旧 cutoff scan。
- cutoff scan 同时改变 k mesh 或 smearing，导致无法归因。
- 未收敛 SCF 点被放入 cutoff 表。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | 目标 observable 对 `ecutwfc/ecutrho` 已稳定；scan 变量可归因；SCF 全部收敛；PP 来源一致 | 可进入对应 workflow 的下一步 |
| WARN | total energy 稳定但 force/stress/phonon 或目标后处理未检查 | 可探索，不应进入高精度结构、phonon 或响应结论 |
| BLOCK | scan 点未收敛、PP/k mesh/smearing 混变、目标量仍系统漂移或换 PP 后未重测 | 不允许进入依赖 cutoff 质量的下游 |

## 下游影响

- `relax/vc-relax`：force/stress 对 cutoff 和 FFT grid 敏感。
- `bands/DOS/PDOS`：eigenvalues、projected states 和能量差继承 cutoff policy。
- `charge density / potential / work function`：density/potential 网格质量直接影响空间场。
- `phonon/DFPT`：频率、acoustic modes、Born charge、dielectric tensor 对 cutoff 更敏感。

## 与 physics-judgement 的边界

本页处理 numerical convergence。以下问题超出最低理论：

- cutoff 已收敛但 functional 不适合，见 [numerical vs model error](../physics-judgement/numerical-vs-model-error.md)。
- PP transferability，见 [pseudopotential transferability](../physics-judgement/pseudopotential-transferability-and-functional-consistency.md)。
- phonon 虚频是否物理，见 [imaginary frequency triage](../physics-judgement/imaginary-frequency-triage.md)。

## 来源与边界

- Stable: `ecutwfc`、`ecutrho` 和相关 `pw.x` 字段以 QE `INPUT_PW` 为准。
- Boundary: 本页不提供材料无关的 cutoff 数值；所有阈值由目标 observable 和 convergence evidence 决定。
- Internal standard: cutoff scan 的结果应写入 [output review checklist](../standards/output-review-checklist.md) 和 calculation record。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- Vanderbilt, Ultrasoft Pseudopotentials.
- Blochl, Projector Augmented-Wave Method.
- Lejaeghere et al., Reproducibility in density functional theory calculations of solids.
- 本仓库：[cutoff convergence workflow](../workflows/ground-state/cutoff-convergence.md)
