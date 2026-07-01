# Kohn-Sham Eigenvalue Boundary

## 本页解决什么问题

本页用于判断 QE 输出的 Kohn-Sham eigenvalues 可以如何使用，以及不能如何解释。`pw.x`、`bands.x`、`dos.x` 和 `projwfc.x` 产生的电子结构图对 workflow 审阅、趋势分析和态成分判断很有用，但它们来自 Kohn-Sham auxiliary system，不应无条件写成 quasiparticle excitation、optical spectrum 或实验能级。

## 典型 output 现象

| output 现象 | 可能含义 | 回查动作 |
|---|---|---|
| bands 图给出 VBM/CBM 或 Fermi crossing | KS dispersion 可用于内部电子结构分析 | 检查 k-path、energy reference、functional boundary |
| DOS 与 bands 的 gap 或 metallicity 判断不一致 | k-path、uniform mesh、smearing 或能量零点不一致 | 交叉审阅 SCF、NSCF、DOS、bands |
| PDOS 显示某轨道贡献显著 | projector-defined orbital trend | 回查 `projwfc.x` 投影通道和 pseudopotential |
| SOC 或 DFT+U 后 band degeneracy 改变 | Hamiltonian/model 已改变 | 重新审阅 symmetry、k-path、DOS/PDOS 数据链 |
| Fermi level 或 VBM/CBM 零点未记录 | 图像不可复查 | 回到 output 和绘图脚本记录 energy zero |

## 可能原因

- 数值误差：k-path 不完整、NSCF uniform mesh 不足、`nbnd` 不覆盖目标能区、smearing 影响 Fermi 附近判断。
- 模型误差：semilocal functional、DFT+U、hybrid、SOC 或 vdW 改变 KS eigenvalues 的解释边界。
- 赝势误差：PP valence、relativistic treatment 或 projector 改变 bands/PDOS。
- 边界条件误差：surface/vacuum、charged system 或低维模型导致能量参考不可直接比较。
- 后处理误差：绘图时移动能量零点、插值、band sorting 或投影标签未记录。
- workflow 传播误差：bands、DOS、PDOS 读取了不同 `prefix/outdir` 或不同 Hamiltonian。
- 真实物理效应：spin splitting、SOC splitting、band inversion 或 symmetry breaking 可能真实存在，但需要额外证据支撑。

## 必查 QE input/output

| 对象 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `calculation='bands'` | `pw.x` | 沿 high-symmetry path 计算 KS dispersion | 把 path 当作 BZ integration |
| `K_POINTS crystal_b/tpiba_b` | `pw.x` | k-path 坐标和 cell convention | 结构 relax 后沿用旧 path |
| `K_POINTS automatic` | `pw.x nscf` | DOS/PDOS uniform mesh | 用 path 数据做 DOS |
| `nbnd` | `pw.x` | 空带和目标能区覆盖 | 空带不足仍解释高能特征 |
| `occupations/smearing/degauss` | `pw.x` | Fermi 附近 occupation | smearing artifact 被解释为 gap 或 DOS 峰 |
| `filband/fildos/filpdos` | 后处理程序 | 图件数据来源 | 图来自旧文件或旧模型 |
| `noncolin/lspinorb/HUBBARD` | `pw.x` | Hamiltonian 分支 | SOC/U 与非 SOC/无 U 数据混用 |

## 判断规则

- Kohn-Sham eigenvalue dispersion 可用于 DFT-level band structure、相对趋势、metallicity 初步判断和与 DOS/PDOS 交叉审阅。
- Kohn-Sham eigenvalue 不等同于一般 quasiparticle excitation energy。若目标是光谱、激子、准粒子修正或光学峰，应进入 GW/BSE/TDDFT/XSpectra/Yambo 等边界。
- `bands` 和 `DOS` 回答的问题不同：k-path 显示路径色散；uniform mesh 支撑 DOS 和积分型判断。
- energy zero 必须明确：Fermi level、VBM、CBM、vacuum level 或自定义零点不能混用。
- bands、DOS、PDOS 互相冲突时，先回查上游 SCF/NSCF、kmesh、smearing、`prefix/outdir` 和 energy reference，再讨论物理原因。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 上游 SCF/NSCF 数据链完整；k-path / uniform mesh 与目标匹配；energy reference、`nbnd`、occupation 和模型边界已记录 | 允许写 DFT-level dispersion、DOS/PDOS 交叉解释和趋势判断 |
| WARN | 数据链可追踪，但只支撑定性讨论；缺少 quasiparticle/optical 方法、functional sensitivity 或 SOC/U 对照 | 可用于内部分析和带边界的趋势描述 |
| BLOCK | 直接把 KS gap 或 eigenvalue 写成准粒子/光学/实测能量；bands/DOS/PDOS 数据链混用；能量零点不明 | 不允许进入定量电子激发、光谱或器件级结论 |

## 不能做出的过度结论

- 不能把 `filband.gnu` 曲线本身写成光谱。
- 不能把 semilocal DFT 的 gap 数值无边界地写成实测带隙。
- 不能用 DOS 曲线平滑程度证明 band gap 或 Fermi-level DOS 定量可信。
- 不能用 PDOS 投影强度单独证明价态、电荷转移或局域化机制。
- 不能在 SOC、DFT+U 或 hybrid 切换后沿用旧的简并、k-path 或 energy reference 解释。

## 下游影响

- `bands`：用于 KS dispersion 和 band crossing 的图像证据。
- `DOS/PDOS`：需要 uniform mesh、projection 边界和 energy zero 对齐。
- `Fermi surface`：依赖 dense NSCF、Fermi level、band crossing 和 smearing。
- `Wannier/GW/BSE/EPC`：KS bands 是质量检查和输入之一，不替代高级方法的验证。

## 与 theory-minimum / workflows / standards 的关系

- 理论回查：[DFT, Kohn-Sham and SCF](../theory-minimum/dft-ks-scf.md)、[K-points, symmetry and k-path](../theory-minimum/kpoints-symmetry-kpath.md)、[Smearing and metals](../theory-minimum/smearing-metals.md)
- Workflow 回查：[bands](../workflows/electronic/bands.md)、[DOS](../workflows/electronic/dos.md)、[PDOS](../workflows/electronic/pdos.md)、[Fermi surface](../workflows/electronic/fermi-surface.md)
- 相关判断：[Band gap problem and delocalization](band-gap-problem-and-delocalization.md)、[Ground-state vs excited-state boundary](ground-state-vs-excited-state.md)
- 记录规范：[output review checklist](../standards/output-review-checklist.md)

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| Kohn-Sham system 是 auxiliary system | Strong | Kohn-Sham canonical literature |
| KS eigenvalue 可用于 DFT-level bands/DOS，但不是通用激发能 | Strong | DFT foundation 与 excited-state review |
| `bands.x`、`dos.x`、`projwfc.x` 字段行为 | Version-sensitive | 当前 QE `INPUT_*` 文档 |
| gap 或光谱解释需要模型边界 | Boundary | band-gap / excited-state literature |

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_BANDS reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- QE INPUT_PROJWFC reference: <https://www.quantum-espresso.org/Doc/INPUT_PROJWFC.html>
- Kohn and Sham, Self-Consistent Equations Including Exchange and Correlation Effects.
- Onida, Reining and Rubio, Electronic excitations: density-functional versus many-body Green's-function approaches.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[source index](../references/source-index.md)
