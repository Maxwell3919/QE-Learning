# Imaginary Frequency Triage

## 1. 核心判断结论

- **Strong:** phonon 虚频至少分为数值误差、结构未充分优化、真实动力学不稳定三类。
- **Strong:** ASR 不能替代结构、SCF、cutoff、k/q mesh 收敛。
- **Strong:** phonon 是 DFPT response calculation，不是普通后处理。
- **Boundary:** 低维、极性、金属和软模体系需要额外边界审阅。

## 2. 这个问题为什么会出现在 DFT/QE 中？

DFPT phonon 求的是能量面对原子位移的二阶响应。小的负频率可能来自声学支数值漂移，也可能来自结构/SCF 不足；显著且收敛稳定的软模才可能指向真实动力学不稳定。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `tr2_ph` | `ph.x` | response self-consistency threshold | 沿用过松 SCF 判断 |
| `ldisp/nq1/nq2/nq3` | `ph.x` | q-grid phonon | q-grid 太粗仍解释 dispersion |
| `asr` | `matdyn.x/dynmat.x` | 声学和约束修正 | 用 ASR 掩盖未收敛 |
| `zasr/epsil` | `q2r.x/ph.x` | Born charge / long-range correction 边界 | metal/polar 条件未审阅 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| Gamma acoustic mode 不接近零 | translational invariance 残差或数值问题 | 检查 ASR、force、SCF、cutoff |
| 局部小虚频随参数消失 | 数值噪声 | 收敛扫描 |
| 稳定大虚频且 eigenvector 有物理模式 | 可能真实软模 | 写 WARN/BLOCK 并分析结构相 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 虚频不存在或已通过收敛/eigenvector/ASR 边界解释 | 允许进入 phonon 结论 |
| WARN | 小虚频或边界敏感，仍可做诊断性讨论 | 不得写稳定性定量结论 |
| BLOCK | 未区分三类来源就解释软模或热力学 | 停止下游 |

## 6. 常见误区

- 把任何虚频都直接判成真实不稳定
- ASR 后无虚频就算收敛
- 结构 force 未收敛却跑 phonon
- q-grid 太粗仍解释 DOS
- 忽略 metal smearing 对 phonon 的影响

## 7. 应回写到哪些 workflow 页面？

- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/phonon/gamma-phonon.md](../workflows/phonon/gamma-phonon.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/dielectric-born-charge.md](../theory-minimum/dielectric-born-charge.md)


## 结论强度

- Strong：本页中由经典理论、方法论文或官方文档直接支持的判断。
- Moderate：本页中由摘要、综述或多个方法来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 结论来源

| 结论 | 强度 | 支撑文献/文档 | 是否需要全文核验 |
|---|---|---|---|
| 本页核心判断需要写入 output review，而不是只作为理论背景 | Strong | 官方文档 / 方法论文 | 否 |
| 具体字段、默认值和版本行为必须回查当前官方文档 | Version-sensitive | QE INPUT_* / 工具官方文档 | 是 |

## 9. 资料来源

- Hohenberg and Kohn, *Inhomogeneous Electron Gas*：ground-state DFT 边界。
- Kohn and Sham, *Self-Consistent Equations Including Exchange and Correlation Effects*：KS 辅助体系。
- Quantum ESPRESSO official documentation and `INPUT_*` references：QE 字段和程序行为核验。
- Giannozzi et al., Quantum ESPRESSO code papers：QE 方法和模块边界。
- [references/canonical-literature.md](../references/canonical-literature.md)：本仓库 canonical literature 分级。
- [references/source-index.md](../references/source-index.md)：公开来源入口。
