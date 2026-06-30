# Ground-State vs Excited-State Boundary

## 1. 核心判断结论

- **Strong:** ordinary ground-state DFT bands/DOS 不能替代 quasiparticle 或 optical spectra。
- **Strong:** GW、BSE、TDDFT、XSpectra、TurboEELS/Yambo 等属于升级路径，不是普通后处理等价物。
- **Boundary:** DFT bands 可以作为初始态和定性图像，但不能自动承担 excited-state 定量。
- **Version-sensitive:** Yambo/GWL/TDDFPT/XSpectra 接口和输入需查各自官方文档。

## 2. 这个问题为什么会出现在 DFT/QE 中？

Ground-state DFT 优化密度和能量；激发态问题需要响应理论、多体格林函数或专用谱学程序。QE 生态提供入口，但每个入口有独立收敛参数和物理前提。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `nbnd` | `pw.x` | 空带覆盖激发态前置 | 空带不足仍解释高能谱 |
| GW/BSE/TDDFT 输入 | GWL/Yambo/TurboTDDFT | 准粒子/激发态方法入口 | 把 DFT 参数简单继承 |
| `input_dft` | `pw.x` | ground-state 起点模型 | 把起点误差当最终谱学 |
| broadening/window | spectroscopy tools | 谱线和能量窗口 | 未做独立 convergence |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| optical peak 与 KS gap 混用 | excited-state boundary 越界 | 转 GW/BSE/TDDFT |
| spectra 对空带/window 敏感 | 高级方法收敛未完成 | 做独立 convergence |
| XAS/EELS 解释缺少核心态/矩阵元边界 | 方法不匹配目标 | 回查专用程序文档 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 方法与目标匹配，参数和上游 ground state 记录完整 | 允许谱学结论 |
| WARN | 只用 DFT bands/DOS 做前置图像 | 只能写定性提示 |
| BLOCK | 用普通 DFT bands 定量解释 quasiparticle/optical/exciton/XAS/EELS | 停止结论 |

## 6. 常见误区

- 把普通 DFT bands 直接当作实验光谱
- GW 参数沿用 DFT convergence 即可
- 忽略 exciton
- 光谱 broadening 当物理寿命
- 没有上游质量审查就跑高级工具

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [learn/09-feature-expansion-map.md](../learn/09-feature-expansion-map.md)

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
