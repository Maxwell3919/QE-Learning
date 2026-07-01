# Smearing and metals

## QE 中对应的问题

`occupations`、`smearing` 和 `degauss` 共同决定 QE 如何给 Kohn-Sham 能级分配电子占据。它们是影响 SCF 收敛、Fermi energy、总能、力、DOS 和 phonon/DFPT 结果的数值策略，不能当作孤立的输入装饰。

在 QE workflow 中，这一页主要回答以下问题：

- 体系应按金属、绝缘体还是小带隙/边界体系处理？
- `occupations='fixed'`、`occupations='smearing'` 或 tetrahedron 类方法是否与计算目标一致？
- `smearing` 函数和 `degauss` 宽度是否只作为 Brillouin zone 积分和占据平滑的数值设置，而不是被误读为材料真实温度？
- Fermi energy、number of electrons、band occupation 和 DOS 近 Fermi 能级行为是否互相一致？
- SCF、relax、NSCF/DOS、phonon 中的 occupation 策略是否可追溯，是否允许进入下游？

最低理论判断是：金属在 Fermi 面附近有部分占据态，离散 k mesh 下占据会随能级微小移动突变，smearing 用平滑占据改善积分和 SCF 稳定性；绝缘体有明确能隙，通常不需要用 smearing 来制造分数占据；小带隙、半金属、掺杂、电荷态或带隙接近数值误差的体系必须作为边界情况记录，不能只靠一次 Fermi energy 输出下结论。

## 输入字段表

| 字段 | 所属程序/位置 | 作用 | input 判断 | output 中如何核对 | 常见风险 |
|---|---|---|---|---|---|
| `occupations` | `pw.x` / `&SYSTEM` | 选择电子占据策略，如 `fixed`、`smearing`、`tetrahedra`、`tetrahedra_opt`、`from_input` | 金属通常需要显式考虑 smearing 或适合目标的 Brillouin zone 积分；有清楚能隙的绝缘体通常应避免无理由 smearing | 输出中的 occupation scheme、band occupation、number of electrons、Fermi energy | 金属用 `fixed` 导致 SCF 振荡；绝缘体用 smearing 后把分数占据当作物理结论 |
| `smearing` | `pw.x` / `&SYSTEM` | 指定展宽函数，如 Gaussian、Methfessel-Paxton、cold smearing、Fermi-Dirac | 仅在 `occupations='smearing'` 等需要展宽策略时才有实际意义；不同 workflow 若改变函数需记录理由 | 输出中的 smearing type 或 occupation summary | 只记录 `degauss` 不记录函数；不同步骤混用函数导致能量、力或 DOS 不可比 |
| `degauss` | `pw.x` / `&SYSTEM` | 展宽宽度，单位 Ry，用于金属 Brillouin zone 积分 | 必须与 k mesh、体系金属性判断和下游目标一起收敛检查；本页不提供默认推荐值 | 输出中的 smearing width、SCF 稳定性、总能/力随宽度变化 | 宽度过大改变能量、力、Fermi 面和 DOS 形状；把 Ry 当 eV |
| `K_POINTS` | `pw.x` card | 决定 Brillouin zone 采样；金属和小带隙体系通常更敏感 | occupation/smearing scan 不能脱离 k mesh scan；tetrahedron 方法要求合适的 uniform k grid | 输出中的 k points、irreducible k points、weights | 用粗 k mesh 掩盖 smearing 问题；DOS/phonon 沿用不合适的 k mesh |
| `nbnd` | `pw.x` / `&SYSTEM` | 控制计算能带数量 | 金属、DOS、NSCF、带结构或 `from_input` 占据时需确认空带是否足够 | 输出中的 highest occupied / lowest unoccupied、band list、occupation | 空带不足导致 Fermi 附近态、DOS 或后处理范围不可靠 |
| `tot_charge` | `pw.x` / `&SYSTEM` | 改变总电子数 | 带电、掺杂或电子数非中性任务必须把 Fermi energy 解释为该输入模型下的结果 | 输出中的 number of electrons 和 charge summary | 把带电模型的 Fermi level 当作中性体相性质 |
| `bz_sum` | `dos.x` / `&DOS` | DOS 的 Brillouin zone 求和策略 | DOS 积分策略应与 SCF/NSCF 数据和解释目标一致 | `dos.x` output 中的求和方式；DOS 数据文件 | 把 DOS 后处理 broadening 当作 SCF smearing；tetrahedron 与 smearing 条件混淆 |
| `ngauss` | `dos.x` / `&DOS` | DOS Gaussian broadening 类型 | 只控制 DOS 后处理展宽，不自动证明 SCF occupation 合理 | `dos.x` output 和 DOS 曲线平滑程度 | 用图像平滑度代替 k mesh / occupation 收敛性 |
| `degauss` | `dos.x` / `&DOS` | DOS broadening，单位 Ry | 与 `pw.x` 的 `degauss` 分开记录；若覆盖默认读取行为需说明 | `dos.x` output、DOS 文件近 Fermi 能级形状 | 忽略单位；修改 DOS broadening 后仍声称与 SCF energy/force 可直接比较 |
| `DeltaE` | `dos.x` / `&DOS` | DOS 能量网格步长，单位 eV | 与目标能区分辨率匹配；不替代 broadening 或 k mesh 收敛 | DOS 文件能量间隔 | 过粗丢峰，过细放大离散 k mesh 噪声 |

## output review

阅读 `pw.x` output 时，先确认 input 是否真的被 QE 读成预期策略，再判断物理解释：

- `number of electrons` 是否与结构、电荷态、赝势价电子数和 `tot_charge` 一致。
- occupation scheme 是否与 input 中 `occupations`、`smearing`、`degauss` 一致。
- Fermi energy 是否出现；若出现，它是该计算设置下的电子化学势/占据参考，不自动等于可跨体系比较的绝对能级。
- band occupation 是否显示 Fermi 附近存在部分占据；若有分数占据，需要判断它来自真实金属性、边界小带隙、带电模型还是 smearing 过大。
- SCF iteration 是否出现由占据变化引起的振荡、缓慢收敛或重复 charge sloshing。
- 总能、力和应力是否对 `degauss` 或 k mesh 变化敏感；如果敏感，不能只用一次 SCF 收敛作为 PASS。

对 energy/force 的判断：

- 总能比较必须使用相同 occupation policy、smearing function、`degauss`、k mesh 和可比结构设置，除非记录中明确说明变更目的。
- relax 或 vc-relax 的力/应力可能受 smearing 影响；若 `degauss` scan 中力的方向或大小明显改变，应标记为 WARN 或 BLOCK，而不是只看 `convergence has been achieved`。
- tetrahedron 类方法适合某些 DOS/积分任务，但 QE 官方说明中普通 tetrahedron 不适合 force/optimization/dynamics；用于结构优化前必须回到 workflow 复查。

对 DOS 的判断：

- `dos.x` 必须读取目标 NSCF 数据，而不是旧 `prefix/outdir`。
- DOS 近 Fermi energy 的非零态密度、能隙、伪隙或尖峰要与 bands、Fermi energy 和 k mesh 一起判断。
- DOS 曲线更平滑不等于更物理；平滑可能来自更密 k mesh，也可能来自过大的后处理 broadening。
- 若 `dos.x` 中单独设置 `degauss` 或 `ngauss`，应把它记为 DOS broadening，不回写为 SCF occupation 结论。

对 phonon/DFPT 的判断：

- 金属 phonon 对 Fermi 面采样、k mesh、occupation 和 smearing 可能敏感。
- 若 phonon 频率、虚频或 electron-phonon 相关量随 `degauss` 明显变化，应先归类为数值敏感，不直接解释为稳定性结论。
- phonon workflow 使用的 SCF charge density 必须能追溯 occupation policy；改变 smearing 后应确认是否需要重新生成上游 SCF 数据。

## 常见错误

| 现象 | 可能原因 | 优先排查 | 判定倾向 |
|---|---|---|---|
| 金属 SCF 振荡或难收敛 | `occupations='fixed'`、k mesh 太粗、smearing 策略不合适 | output occupation scheme、Fermi energy、SCF residual、k mesh | 未解决前 BLOCK |
| 绝缘体出现分数占据 | 无理由使用 smearing、`degauss` 过大、带隙太小或 k mesh 不足 | band occupation、DOS/bands、`degauss` scan | 解释不清为 WARN/BLOCK |
| Fermi energy 被当作绝对能级比较 | 未说明能量零点、体系电荷态或计算设置 | record 中的 reference、DOS/bands 对齐方式 | 缺少零点说明为 WARN |
| DOS 显示金属性但 bands 显示有隙 | 能量零点不一致、DOS broadening 过大、NSCF k mesh/路径不同 | `dos.x` broadening、bands reference、NSCF mesh | 需复查，通常 WARN |
| DOS 很平滑但特征消失 | 后处理 `degauss` 或 `DeltaE` 设置不合适 | `bz_sum`、`ngauss`、`degauss`、`DeltaE`、k mesh | 不能作为最终图，WARN |
| relax 后结构或力随 smearing 明显变 | occupation 数值策略影响势能面 | 同结构下 energy/force 对比、k mesh 对比 | 下游 phonon 前 BLOCK |
| phonon 虚频随 smearing 消失或出现 | Fermi 面采样/occupation 未收敛 | 上游 SCF、k mesh、smearing policy、phonon output | 先标数值敏感，BLOCK |
| 不同 workflow 结果不可比 | SCF/relax/NSCF/DOS/phonon 中 occupation 或 broadening 未统一或未记录 | input 文件、record、output review | 缺少记录为 WARN |

## 最低掌握深度

完成本页后，学习者至少应能做到：

- 解释 `occupations` 是电子占据策略，`smearing` 是展宽函数，`degauss` 是展宽宽度，三者不是同一个概念。
- 说明金属为什么常需要 smearing：Fermi 面附近部分占据使离散 k mesh 积分和 SCF 迭代对能级微小变化敏感。
- 说明绝缘体为什么通常不应无理由 smearing：清楚能隙下 `fixed` 占据已经给出整数占据，额外展宽可能制造不必要的分数占据。
- 识别金属/绝缘体/小带隙边界不是只看一个标签，而要联合 Fermi energy、band occupation、DOS/bands、k mesh 和 smearing 敏感性。
- 区分 SCF smearing 与 DOS broadening：前者影响电子密度、总能、力和下游 charge density，后者影响 DOS 图像和积分展示。
- 对 energy、force、DOS、phonon 分别说明 smearing 可能影响什么，以及 output review 中应保留哪些证据。
- 在记录中把 occupation policy 写入 PASS / WARN / BLOCK：只有当 input、output、收敛性和下游用途一致时才允许 PASS。

## 对应 workflow

- [Smearing and occupations workflow](../workflows/ground-state/smearing-and-occupations.md)
- [DOS workflow](../workflows/electronic/dos.md)
- [Phonon dispersion DFPT workflow](../workflows/phonon/phonon-dispersion-dfpt.md)
- [Calculation record template](../standards/calculation-record-template.md)
- [PASS / WARN / BLOCK](../standards/pass-warn-block.md)
- [physics-judgement/kmesh-smearing-sensitivity.md](../physics-judgement/kmesh-smearing-sensitivity.md)

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- [Smearing and occupations workflow](../workflows/ground-state/smearing-and-occupations.md)
- [DOS workflow](../workflows/electronic/dos.md)
