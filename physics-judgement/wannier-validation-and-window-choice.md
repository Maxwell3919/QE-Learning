# Wannier Validation and Window Choice

## 1. 核心判断结论

- **Strong:** Wannierization 是子空间建模，不是纯粹拟合。
- **Strong:** Wannier-interpolated bands 必须在目标能区回对 QE bands。
- **Strong:** projection、frozen window、disentanglement window 必须记录。
- **Boundary:** spread 小不能单独证明 Wannier 模型成功。

## 2. 这个问题为什么会出现在 DFT/QE 中？

Wannier functions 将 Bloch 子空间转成局域有效模型。窗口和投影决定保留哪个低能子空间；如果 Wannier bands 不能复现目标 QE bands，则后续 Berry curvature、transport、topology、EPC interpolation 都缺少可信基础。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| projections | Wannier90 | 定义初始局域轨道 | 投影没有物理含义说明 |
| frozen window | Wannier90 | 必须精确保持的目标能区 | 窗口未覆盖目标 bands |
| disentanglement window | Wannier90 | 处理 entangled bands | 窗口过宽/过窄不审阅 |
| `pw2wannier90.x` 接口 | QE/Wannier90 | 把 QE 数据转入 Wannier | SOC/spin 数据链不一致 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| Wannier bands 与 QE bands 不匹配 | projection/window 失败 | 调整子空间并重审 |
| spread 异常或虽小但 bands 错 | 局域性指标不足以证明模型 | 必须回对 bands |
| 后续拓扑/EPC 无 band validation | 数据链断裂 | BLOCK 下游 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 目标能区 Wannier bands 回对 QE bands，window/projection 记录完整 | 允许进入插值/有效模型 |
| WARN | 只在部分能区匹配或 window 敏感 | 限制解释范围 |
| BLOCK | 未验证 Wannier bands 就进入 EPC/topology/transport | 停止下游 |

## 6. 常见误区

- spread 小就是成功
- 不记录 window
- projection 只按默认
- SOC 和 non-SOC 数据混用
- 用 Wannier 模型解释未覆盖能区

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
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
