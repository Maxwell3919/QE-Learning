# Work Function and Electrostatic Boundary

## 1. 核心判断结论

- **Strong:** work function 不是单行公式，而是 slab electrostatics workflow 的结果。
- **Strong:** 需要同时审阅 vacuum plateau、Fermi level、平均方向、dipole correction、charged state 和周期镜像。
- **Boundary:** semiconductor/insulator surface 的 Fermi-level convention 和 band alignment 需要额外说明。
- **Version-sensitive:** `tefield/dipfield/assume_isolated`、ESM 等字段需按 QE 文档核验。

## 2. 这个问题为什么会出现在 DFT/QE 中？

work function 的表达式依赖 vacuum level 和 electronic chemical potential，但这两个量必须来自稳定 slab 电势平台和一致的上游 SCF。周期边界会引入镜像和偶极问题，带电 slab/缺陷还需要有限尺寸边界判断。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `plot_num` | `pp.x` | 输出 electrostatic/local potential | 选错 potential 类型 |
| `average.x` 输入 | `average.x` | 沿表面法向平均电势 | 平均方向错误 |
| `tefield/dipfield` | `pw.x` | 偶极/电场边界入口 | 未记录 correction 却解释 work function |
| `assume_isolated` | `pw.x` | 低维/孤立边界处理入口 | 与 slab 模型不匹配 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| vacuum plateau 不平 | 真空不足或偶极问题 | 增加真空/检查 dipole correction |
| Fermi level 来源不清 | 能量零点不可复查 | 回查 SCF/DOS 和电势对齐 |
| charged slab warning | 周期镜像/背景电荷问题 | 停止无修正定量解释 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | plateau、Fermi level、平均方向、边界修正和结构记录完整 | 允许报告 work function |
| WARN | 平台可读但 vacuum/dipole/convention 敏感 | 只写趋势 |
| BLOCK | 无稳定平台或 charged/boundary 条件不满足 | 不得解释 work function |

## 6. 常见误区

- 只写 `Vvac - Ef` 不审阅电势
- 平均方向错
- 把 charged slab 当 neutral slab
- 真空不足仍报告数值
- 半导体表面不说明 Fermi convention

## 7. 应回写到哪些 workflow 页面？

- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
- [workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)
- [workflows/electronic/charge-density.md](../workflows/electronic/charge-density.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/functional-choice.md](../theory-minimum/functional-choice.md)
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

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)：`assume_isolated`、`tefield/dipfield` 等字段边界。
- [QE INPUT_PP](https://www.quantum-espresso.org/Doc/INPUT_PP.html)：电势后处理字段边界。
- [QE PostProc documentation](https://www.quantum-espresso.org/Doc/pp_user_guide/)：`pp.x` 与 `average.x` 后处理入口。
- [references/canonical-literature.md](../references/canonical-literature.md)：本仓库 canonical literature 分级。
- [references/source-index.md](../references/source-index.md)：公开来源入口。
