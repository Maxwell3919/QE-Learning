# K-Mesh and Smearing Sensitivity

## 1. 核心判断结论

- **Strong:** SCF uniform k mesh、DOS dense mesh、bands path 是不同对象。
- **Strong:** band path 不能替代 DOS mesh，也不能证明 k-point convergence。
- **Strong:** 金属体系的 total energy、force、phonon、EPC 对 k mesh 和 smearing 联合敏感。
- **Version-sensitive:** `occupations`、`smearing`、tetrahedron 选项需按当前 `INPUT_PW`/`INPUT_DOS` 核验。

## 2. 这个问题为什么会出现在 DFT/QE 中？

BZ integration 是数值积分问题；high-symmetry path 是可视化路径。金属的 Fermi surface 使积分对 k mesh、smearing 和 broadening 更敏感，且这种敏感性会传递到 phonon 和 EPC。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `K_POINTS automatic` | `pw.x` | uniform mesh for integration | 用 path 代替 mesh |
| `occupations` | `pw.x` | fixed/smearing/tetrahedra 等占据方案 | 金属 fixed occupations |
| `smearing/degauss` | `pw.x` | metallic integration broadening | degauss 过大解释物理 gap |
| `bz_sum/degauss` | `dos.x` | DOS integration / broadening | 能量峰由 broadening 造成仍当物理峰 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| irreducible k-points 太少或随 symmetry 异常变化 | mesh/symmetry 不适合目标 | 重审 mesh 和 shift |
| Fermi energy、DOS at EF 对 mesh/degauss 敏感 | metal sampling 未收敛 | 做联合 sensitivity |
| phonon/EPC 对 smearing 变化敏感 | Fermi surface sampling 影响 response | 回查 DFPT/EPC 数据链 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | mesh、shift、smearing 与目标量收敛记录完整 | 允许进入下游 |
| WARN | 只完成探索性 mesh 或 broadening，峰位/力/phonon 仍敏感 | 只写探索结论 |
| BLOCK | 用 band path 算 DOS，或金属 occupation 设置不满足目标 | 停止后处理 |

## 6. 常见误区

- 把 k-path 当 convergence mesh
- 只看 total energy 不看 force/stress/phonon
- 不记录 mesh shift
- smearing 太大仍解释细节峰
- DOS 和 bands 使用不同 energy zero 却不说明

## 7. 应回写到哪些 workflow 页面？

- [workflows/ground-state/kpoint-convergence.md](../workflows/ground-state/kpoint-convergence.md)
- [workflows/ground-state/smearing-and-occupations.md](../workflows/ground-state/smearing-and-occupations.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/electronic/fermi-surface.md](../workflows/electronic/fermi-surface.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)


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
