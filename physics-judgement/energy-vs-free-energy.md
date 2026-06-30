# Energy vs Free Energy

## 1. 核心判断结论

- **Strong:** 0 K DFT total energy difference 不能直接代表所有实验温度下稳定性。
- **Strong:** NEB 默认给 minimum energy path，不是 free-energy path。
- **Boundary:** finite-temperature stability 可能需要 ZPE、phonon free energy、QHA、anharmonicity、AIMD 或其他 free-energy 方法。
- **Version-sensitive:** MD/NEB/thermostat 和 package 输入需查当前 QE 文档。

## 2. 这个问题为什么会出现在 DFT/QE 中？

SCF/relax 给出的是选定模型下的 0 K energy surface 上的信息。实验温度下的稳定性和动力学涉及 entropy、phonon、anharmonicity、sampling 和可能的 metastability。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `calculation='md'` | `pw.x` | Born-Oppenheimer MD 入口 | 把短 MD 当充分采样 |
| `cp.x` 设置 | CP | Car-Parrinello MD 入口 | 混淆 BOMD/CPMD |
| `neb.x` 输入 | NEB | minimum energy path | 当 free-energy path |
| phonon/QHA 设置 | phonon tools | harmonic free energy | 虚频未处理仍做 free energy |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| 能量差与数值误差同量级 | 结论不稳 | 写 uncertainty |
| phonon 虚频 | harmonic free energy 边界失败 | 先做 imaginary triage |
| NEB barrier 未含 entropy | 势能路径不是自由能路径 | 限制解释 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 目标是 0 K energy/MEP，或已补充 free-energy 方法 | 允许相应结论 |
| WARN | 用 0 K 能量讨论有限温趋势 | 必须写温度边界 |
| BLOCK | 把未处理虚频或短采样结果当热力学稳定性定量 | 停止结论 |

## 6. 常见误区

- 0 K 能量就是实验稳定性
- NEB barrier 就是自由能势垒
- 短 AIMD 代表充分采样
- 虚频结构直接做 harmonic free energy
- 忽略 ZPE/entropy

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/molecular-dynamics.md](../workflows/advanced/molecular-dynamics.md)
- [workflows/advanced/neb-reaction-path.md](../workflows/advanced/neb-reaction-path.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)


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
