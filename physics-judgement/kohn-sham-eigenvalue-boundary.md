# Kohn-Sham Eigenvalue Boundary

## 1. 核心判断结论

- **Strong:** KS eigenvalues 是辅助体系本征值，不能无条件等同 quasiparticle energy。
- **Strong:** KS band gap、fundamental gap、optical gap 是不同对象。
- **Moderate:** bands/DOS/PDOS 对趋势、态成分和交叉审阅有价值，但需要写明 energy reference 和 functional boundary。
- **Boundary:** 研究目标若是光谱、激子或准粒子，应进入 GW/BSE/TDDFT/XSpectra/Yambo 边界。

## 2. 这个问题为什么会出现在 DFT/QE 中？

QE 的 `pw.x` bands/NSCF 和 `bands.x`/`dos.x` 输出来自 KS 辅助体系。许多电子结构图仍然有科研价值，但它们的解释层级低于 ground-state total energy，并且对 functional、U、SOC、smearing 和 k-path 选择敏感。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `calculation='bands'` | `pw.x` | 沿 high-symmetry path 计算 KS eigenvalues | 把 path 当 BZ integration |
| `nbnd` | `pw.x` | 控制空带覆盖范围 | 空带不足仍解释高能谱 |
| `K_POINTS crystal_b` | `pw.x` | 定义 band path | 未说明 cell convention 和 path 来源 |
| `filband/lsym/no_overlap` | `bands.x` | 控制 band 输出、对称标记和排序 | 忽略 crossing/sorting 风险 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| band gap 与 DOS gap 不一致 | energy zero、k mesh 或 smearing 不一致 | 交叉检查 bands/DOS/NSCF |
| Fermi level 未说明来源 | 能量参考不可复查 | 记录 SCF/NSCF/bands 零点 |
| SOC 或 noncollinear 后简并改变 | 模型改变了 band topology/splitting | 回查 SOC/symmetry 页面 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 只做 KS 图像、趋势或态成分分析，且边界写清 | 允许进入 bands/DOS/PDOS 对照 |
| WARN | 讨论 gap 或谱峰但未做 quasiparticle/optical 方法 | 只能写趋势或辅助判断 |
| BLOCK | 把普通 DFT eigenvalues 作为实验光谱定量结论 | 转 GW/BSE/TDDFT/XSpectra/Yambo |

## 6. 常见误区

- 把 `filband.gnu` 曲线当最终实验谱
- 不区分 VBM/CBM、Fermi level 和 vacuum reference
- 用 bands path 计算 DOS
- 忽略 `nbnd` 对高能窗口的限制
- SOC 前后混用 band 解释

## 7. 应回写到哪些 workflow 页面？

- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)


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
