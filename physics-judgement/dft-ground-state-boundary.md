# DFT Ground-State Boundary

## 1. 核心判断结论

- **Strong:** DFT 的核心对象是 ground-state density、total energy 以及由能量导数得到的 force/stress；不是所有实验谱学量。
- **Strong:** SCF 的 `JOB DONE` 只能说明程序结束，不能说明目标 observable 属于 ground-state DFT 可定量解释范围。
- **Boundary:** bands、DOS、PDOS 是 KS 辅助体系的电子结构图像，适合趋势和成分分析，但需要写明解释边界。
- **Version-sensitive:** 具体 QE output 字段、XML 数据和后处理文件格式需按当前 QE 文档核验。

## 2. 这个问题为什么会出现在 DFT/QE 中？

HK/KS DFT 把多电子 ground state 问题转化为密度泛函和 KS 辅助单粒子方程。QE 的 SCF、force、stress、relax 和 vc-relax 都围绕这个 ground-state 闭环展开；而 quasiparticle energy、optical excitation、XAS/EELS 等目标需要额外理论层或专用程序。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `calculation` | `pw.x` | 区分 SCF、relax、NSCF、bands 等任务对象 | 把后处理任务当成 ground-state 重新求解 |
| `input_dft` | `pw.x` | 定义 XC 模型，影响 ground-state 能量面 | 未记录 functional 却比较能量或结构 |
| `conv_thr` | `pw.x` | 电子 ground state 收敛门槛 | 只看结束信息不看 estimated accuracy |
| `tprnfor/tstress` | `pw.x` | 输出 force/stress 以审阅 ground-state 导数 | 未输出应力却解释 vc-relax 或表面应力 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| total energy、density、force、stress 收敛 | ground-state 闭环可审阅 | 检查 SCF/relax/vc-relax 页面 |
| band gap、DOS peak、PDOS orbital feature 被当实验谱 | KS 辅助量越界解释 | 回查 KS eigenvalue boundary 和 excited-state 页面 |
| SCF warning 未解释 | ground-state 数据链可能不稳 | 检查 `prefix/outdir`、pseudo、occupation、symmetry |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 目标是 ground-state 能量、结构、力、应力，且数值和模型记录完整 | 允许进入对应下游 |
| WARN | 目标包含 bands/DOS/PDOS 趋势或态成分解释，但未声称实验谱学定量 | 允许进入带边界解释 |
| BLOCK | 目标是 optical/quasiparticle/spectroscopy 定量，却只提供普通 DFT SCF/bands | 停止定量结论，转 advanced/excited-state 边界 |

## 6. 常见误区

- 把 total energy 的可比较性外推到所有 observable
- 把 KS bands 直接称为实验能带
- 只要 SCF 收敛就进入光谱结论
- 未记录 functional/pseudo 却比较不同模型的能量
- 忽略 force/stress 是能量导数而非独立实验量

## 7. 应回写到哪些 workflow 页面？

- [workflows/ground-state/scf.md](../workflows/ground-state/scf.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)


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
