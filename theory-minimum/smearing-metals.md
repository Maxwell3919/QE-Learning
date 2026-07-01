# Smearing and metals

## 本页解决什么问题

本页解释 `occupations`、`smearing` 和 `degauss` 在 QE 中如何影响金属、小带隙体系、DOS、Fermi surface、force、phonon 和 EPC 前置判断。它帮助用户判断 output 中的 Fermi energy、occupation、DOS broadening 和 smearing 设置是否能支撑下游，而不是把更平滑的曲线当成更可信的物理结论。

## 最低掌握深度

最低需要知道：

- 金属在 Fermi level 附近存在部分占据态，离散 k mesh 下 BZ integration 对 occupation 处理敏感。
- Smearing 是数值积分和占据平滑策略，不应直接当作材料真实热状态。
- `degauss` 会影响 energy、force、DOS、Fermi surface、phonon 和 EPC 相关量。
- SCF smearing 与 DOS 后处理 broadening 是不同层面的设置，必须分开记录。

## 物理图像

绝缘体在 Fermi level 附近有清楚能隙，occupied 和 unoccupied states 的分界相对稳定。金属的 Fermi level 穿过能带，极少量 k 点附近的能量误差就会改变占据、总能和力。有限 k mesh 无法精确表示真实 Fermi surface，SCF 迭代也可能因为占据突然跳变而振荡。

Smearing 把零温阶跃占据替换成一个平滑函数，使 Fermi level 附近的积分更稳定。它可以改善数值积分和 SCF 稳定性，但也会改变能量、力、DOS 形状和 phonon response 的数值表现。因此 smearing 要和 k mesh 一起收敛；一条平滑的 DOS 曲线只是后处理图像更平滑，不代表 Fermi surface 已被充分采样。

## QE 中的对应对象

| 对象 | QE 位置 | 判断意义 | output 证据 |
|---|---|---|---|
| `occupations='fixed'` | `&SYSTEM` / `pw.x` | 整数占据，常用于有清楚 gap 的体系 | occupation scheme、band occupation |
| `occupations='smearing'` | `&SYSTEM` / `pw.x` | 使用 smearing 处理部分占据 | smearing type、Fermi energy |
| `smearing` | `&SYSTEM` / `pw.x` | 展宽函数 | output 中 occupation summary |
| `degauss` | `&SYSTEM` / `pw.x` | 展宽宽度，单位 `Ry` | smearing width、energy/force sensitivity |
| `K_POINTS` | card / `pw.x` | 金属 BZ integration 与 smearing 强耦合 | k-point summary、irreducible k-points |
| `bz_sum` / `degauss` / `DeltaE` | `dos.x` | DOS integration 和 broadening | `dos.x` output、DOS 文件 |

相关 workflow：[smearing and occupations](../workflows/ground-state/smearing-and-occupations.md)、[DOS](../workflows/electronic/dos.md)、[Fermi surface](../workflows/electronic/fermi-surface.md)、[phonon dispersion](../workflows/phonon/phonon-dispersion-dfpt.md)。

## 核心概念

Smearing 用连续函数替代 Fermi level 附近的阶跃占据，使有限 k mesh 下的积分和 SCF 迭代更稳定。它是数值策略，不是材料真实热状态的默认解释。金属通常需要同时审阅 k mesh 和 smearing；绝缘体通常不需要无理由引入分数占据；小带隙或半金属问题需要边界化记录。

## 对 input 的影响

- `occupations` 的选择应与金属/绝缘体/小带隙判断一致。
- 使用 `smearing` 时必须记录函数和 `degauss`，并在 convergence 中和 k mesh 一起审阅。
- DOS 的 `dos.x` broadening 不应回写成 SCF occupation 结论。
- Phonon/EPC 前置 SCF 改变 smearing 后，需要重新生成一致的 `prefix/outdir` 数据链。
- `tetrahedra` 等积分方式适用边界应回到官方文档和具体 workflow 审阅，不能机械替代 smearing。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| occupation scheme | QE 按预期处理占据 | 体系金属性已最终判定 |
| Fermi energy | 本次设置下的电子化学势参考 | 可跨体系直接比较的绝对能级 |
| band occupation | 是否存在部分占据 | 部分占据一定是真实金属性 |
| total energy vs smearing | 数值敏感性 | 物理温度效应 |
| force/stress vs smearing | relax/vc-relax 是否受占据影响 | 结构模型已无误差 |
| DOS shape vs broadening | 图像处理和 Fermi-level 附近特征 | 更平滑即更可信 |
| phonon frequency vs smearing | 金属 response 敏感性 | 虚频物理来源已确定 |

## 常见误区

- smearing 越大图越平滑，所以越可信。
- `degauss` 不记录，导致 DOS、force 或 phonon 不可复查。
- 金属 DOS 不做 k mesh / smearing 敏感性。
- 把 smearing 当作材料真实热状态。
- 绝缘体无理由使用 smearing 后解释分数占据。
- 用 DOS broadening 掩盖 k mesh 不足。
- phonon 虚频随 smearing 变化时直接写物理不稳定。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | occupation policy 与体系和 workflow 目标一致；k mesh/smearing 敏感性已记录；Fermi energy 和 occupation 可解释 | 可进入 DOS、Fermi surface、relax、phonon 或对应后处理 |
| WARN | SCF 可运行但 smearing/k mesh 对目标量仍敏感，或小带隙边界未充分说明 | 可进入探索，不应给定量 DOS/Fermi surface/phonon 结论 |
| BLOCK | 金属使用不合适占据导致 SCF/force/phonon 失败；`degauss` 未记录；DOS/bands 冲突未解释 | 不允许进入相关下游 |

## 下游影响

- `SCF/relax/vc-relax`：occupation 影响总能、力和应力。
- `DOS/Fermi surface`：Fermi level、DOS near Fermi 和图像形状依赖 k mesh/smearing。
- `phonon/DFPT`：金属 phonon 对 Fermi-surface sampling 和 smearing 敏感。
- `EPC`：更依赖 dense k/q 和 smearing 数据链，本页只给最低边界。

## 与 physics-judgement 的边界

本页只解释 smearing 的最低使用。以下问题转到：

- [kmesh and smearing sensitivity](../physics-judgement/kmesh-smearing-sensitivity.md)
- [metals, Fermi surface and smearing](../physics-judgement/08-metals-fermi-surface-and-smearing.md)
- [EPC data chain and convergence](../physics-judgement/epc-data-chain-and-convergence.md)

## 来源与边界

- Stable: `occupations`、`smearing`、`degauss` 以 QE `INPUT_PW` 为准；DOS broadening 字段以 QE `INPUT_DOS` 为准。
- Boundary: 本页不提供材料无关的 `degauss` 数值；宽度由 k mesh、目标 observable 和 convergence evidence 决定。
- Internal standard: smearing policy 应写入 [output review checklist](../standards/output-review-checklist.md)。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- Methfessel and Paxton, High-precision sampling for Brillouin-zone integration.
- Blochl, Jepsen and Andersen, improved tetrahedron method.
- 本仓库：[smearing workflow](../workflows/ground-state/smearing-and-occupations.md)
