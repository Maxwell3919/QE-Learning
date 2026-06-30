# Relax Is Not Final Electronic Structure

## 1. 核心判断结论

- **Strong:** relax/vc-relax 是结构优化 workflow，不是最终 bands/DOS/phonon 结论。
- **Strong:** 进入 bands、DOS、PDOS、phonon 前应使用 final structure 重做 static SCF。
- **Strong:** vc-relax 改变 cell 后必须重审 k-point density、stress、symmetry 和 cutoff 敏感性。
- **Boundary:** final structure 可以作为下游输入，不能直接替代下游 output review。

## 2. 这个问题为什么会出现在 DFT/QE 中？

结构优化沿 Born-Oppenheimer energy surface 寻找力/应力满足条件的结构。优化过程中每步电子状态、晶胞、对称性和 k density 都可能改变；因此 final static SCF 是把最终结构重新变成可追踪上游态的必要闭环。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `calculation='relax'` | `pw.x` | 固定晶胞原子优化 | 把 final coordinates 当最终性质 |
| `calculation='vc-relax'` | `pw.x` | 原子和晶胞优化 | cell 变化后沿用旧 k density |
| `forc_conv_thr/press_conv_thr` | `pw.x` | 力/压力收敛门槛 | 只看结束信息不看残余力/应力 |
| `&IONS/&CELL` | `pw.x` | 优化算法和 cell 自由度 | 约束未记录却解释结构 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| final coordinates 已更新 | 结构可作为下游输入 | 用 final structure 重做 static SCF |
| final force/stress 接近阈值 | 结构质量边界 | 决定 PASS/WARN/BLOCK |
| vc-relax 后 symmetry/k mesh 改变 | 下游采样条件改变 | 重审 k density |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | final force/stress 和优化停止原因可信，并计划或完成 final static SCF | 允许进入下游 |
| WARN | 结构可继续优化或探索，但 force/stress/SCF 余量不足 | 不得做最终 phonon |
| BLOCK | 未收敛或无法追踪 final structure，却进入 bands/DOS/phonon | 停止下游 |

## 6. 常见误区

- 把 relax output 总能当最终电子结构结果
- 未复制 final coordinates 到 static SCF
- vc-relax 后不重审 k mesh
- 低维体系无边界地放开真空方向
- phonon 前不检查 force/stress

## 7. 应回写到哪些 workflow 页面？

- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)

## 8. 应回写到哪些 theory-minimum 页面？

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
