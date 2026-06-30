# Electron-phonon overview

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：严格 SCF、cutoff/k 点/smearing 收敛、结构和赝势记录、q-grid phonon、Wannier 基础
- 下游用途：electron-phonon coupling (EPC)、Wannier interpolation、EPW 相关高级物性分析或专题 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

本页是 electron-phonon coupling 的高级边界页，只说明进入 EPC/EPW 前后要审阅的数据链、程序接口和输出风险；不提供完整 EPW 教程，不写具体材料案例，也不提供可直接运行的真实输入。

## 适用问题

适合用本页判断的问题：

- 一个体系是否已经具备进入 EPC/EPW 学习或计算的前置数据质量。
- SCF、NSCF、DFPT phonon、Wannier interpolation 和 EPW 之间的数据依赖是否记录完整。
- k/q mesh、electron convergence、phonon convergence、`prefix`/`outdir` 是否能被追溯。
- EPC 结果是否只能作为探索线索，还是可以进入更严格的专题 workflow。

不适合用本页直接解决的问题：

- 从零搭建完整 EPW 输入。
- 解释某个具体材料的 EPC、超导、输运或谱函数结果。
- 用 EPW 修复未收敛 phonon、粗糙 Wannierization 或不合格 SCF。
- 把 AiiDA/AiiDAlab 自动化流程作为主线学习路径。

## 进入该 workflow 前必须已经掌握什么

- 基础 `pw.x` SCF/NSCF workflow，能读出 QE version、`prefix`、`outdir`、pseudopotential、cutoff、k 点、smearing 和电子收敛状态。
- cutoff、k mesh、smearing、SCF `conv_thr` 的收敛性记录方法。
- [phonon dispersion workflow](../phonon/phonon-dispersion-dfpt.md) 中的 `ph.x`、`q2r.x`、`matdyn.x` 数据链和 negative frequency 判断。
- q-grid phonon 的完整性检查：每个 q 点和不可约扰动是否收敛、dyn 文件是否完整、ASR 是否只是后处理选择而不是收敛替代品。
- Wannier90 基础概念：projections、disentanglement、frozen window、Wannier-interpolated bands 与 QE bands 的对照。
- 对高级结果保持 PASS/WARN/BLOCK 判断：流程跑完不等于结果已经物理可信。

## 关键 QE 程序或输入字段

| 字段或程序 | 所属环节 | 用途 | output review 关注点 |
|---|---|---|---|
| `pw.x` SCF | QE ground state | 为 DFPT、NSCF/Wannier 和 EPW 提供一致的电子基态 | QE version、pseudo、cutoff、k mesh、smearing、`conv_thr`、Fermi level/occupations 是否记录 |
| `prefix` | `pw.x`/`ph.x`/EPW | 连接同一体系的 saved data 和中间文件 | 各程序是否使用同一意图下的 `prefix`，是否误读旧数据 |
| `outdir` | `pw.x`/`ph.x`/EPW | scratch 与 restart 数据位置 | 路径是否存在、是否与生产/测试数据混用、是否可追溯 |
| electron k mesh | `pw.x`/NSCF/EPW | 控制电子态采样和 EPC 插值前置质量 | 是否做过目标性质相关收敛，金属体系是否特别检查 smearing/k mesh |
| `ph.x` | QE DFPT | 生成 phonon dynamical matrices 和电子-声子前置响应数据 | q 点、不可约扰动、`tr2_ph`、warning、dyn 文件完整性 |
| phonon q mesh | `ph.x` | 控制 phonon/EPC 采样基础 | `nq1/nq2/nq3` 是否记录，是否能支持目标 EPC 分辨率 |
| `tr2_ph` | `ph.x` | phonon 自洽响应阈值 | 每个 perturbation 是否收敛，是否有未完成或 recover 分支 |
| `q2r.x`/`matdyn.x` | QE phonon 后处理 | 审阅 IFC、ASR、phonon dispersion/DOS 的基础质量 | Gamma acoustic branches、negative frequencies、ASR 设置和 q-path 是否记录 |
| `pw2wannier90.x` | QE-Wannier90 interface | 把 QE 电子态转为 Wannier90 可用接口数据 | interface 文件是否来自匹配的 NSCF/SCF 数据，是否记录 projections/window |
| Wannier90 | 外部程序 | 构造 Wannier functions 和电子结构插值 | interpolated bands 是否与 QE bands 对照，spread/disentanglement 是否异常 |
| EPW | 外部程序 | 进行 Wannier-based electron-phonon interpolation 和高级 EPC 分析 | 是否读取匹配的 electron/phonon/Wannier 数据，coarse/fine mesh、单位、定义和 warning 是否明确 |

## Wannier90 / EPW interface 边界

Wannier90 和 EPW 不应被当作基础 QE 后处理命令。它们位于更严格的数据链后端：

```text
relaxed structure + pseudo
  -> strict pw.x scf
  -> pw.x nscf / electronic states for Wannier
  -> ph.x q-grid DFPT
  -> QE-Wannier90 interface
  -> Wannier90 quality check
  -> EPW interpolation / EPC analysis
```

边界判断：

- `ph.x` 提供的是 DFPT phonon 和响应数据质量边界；如果 phonon 未收敛，后续 EPC 结果应标记为 BLOCK。
- Wannier90 提供的是电子结构插值质量边界；如果 Wannier bands 不能复现目标能窗内 QE bands，EPW 结果至少应标记为 WARN，通常不能进入定量结论。
- EPW 是高级分析入口，不替代 `ph.x`、`q2r.x`、`matdyn.x` 的基础 phonon 审阅。
- AiiDA/AiiDAlab 可作为 provenance 或自动化参考，但不进入本页主线。

## k/q mesh 与收敛性边界

- electron convergence：SCF `conv_thr`、cutoff、k mesh、smearing 需要比普通 bands/DOS 后处理更严格，并记录收敛依据。
- phonon convergence：`ph.x` 的 `tr2_ph`、q-grid 完整性、每个 perturbation 收敛状态必须审阅。
- k/q mesh：EPC 对电子和声子采样都敏感，不能只记录一个默认网格；需要说明 coarse mesh 与后续 fine/interpolated mesh 的角色。
- 金属体系：smearing、Fermi surface sampling、k mesh 对 EPC 和 phonon 都可能非常敏感，应单独记录。
- 低维或极性体系：长程相互作用、LO-TO/non-analytic 分支、真空和边界条件会影响 phonon 前置质量；本页只提示风险，具体处理回到 phonon workflow 和官方文档。

## output review 要点

```markdown
## Output Review

- Program chain:
- QE / EPW / Wannier90 version:
- Structure relaxation status:
- Pseudopotential source:
- SCF dependency:
- `prefix` / `outdir`:
- Electron cutoff and k mesh:
- Smearing / occupations:
- Electron convergence threshold and final status:
- `ph.x` q mesh:
- `ph.x` `tr2_ph`:
- `ph.x` perturbation convergence:
- Dynamical matrix completeness:
- `q2r.x` / `matdyn.x` review:
- Gamma acoustic branches:
- Negative frequency review:
- Wannier interface files:
- Wannier projections / window:
- Wannier band comparison:
- EPW coarse electron/phonon mesh:
- EPW fine/interpolated mesh, if used:
- EPC quantity name, unit, and definition:
- Warnings:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

最低审阅要点：

- `ph.x` 是否对所有目标 q 点和 perturbations 收敛。
- dyn/IFC/frequency 文件是否完整，是否能回溯到同一个 `prefix`/`outdir` 数据链。
- k mesh、q mesh、smearing、electron/phonon thresholds 是否记录为可复查字段。
- Wannier interpolation 是否通过 QE bands 对照，而不只是程序正常结束。
- EPW 输出中的 EPC 量、单位、定义、mesh 角色是否明确。
- warning、recover、中断续算、旧 scratch 复用是否被纳入 PASS/WARN/BLOCK 判断。

## 常见风险

- 用未收敛或未完整审阅的 phonon 做 EPC。
- q-grid 太粗却解释精细 EPC 结构。
- k mesh、smearing、Fermi surface sampling 未收敛，尤其是金属体系。
- `prefix`/`outdir` 指向旧 scratch，导致 EPW 或 `ph.x` 读取了不匹配的数据。
- Wannierization 只看完成状态，不看 bands 对照、能窗和 projection 合理性。
- 把 ASR 或插值后处理当成 phonon 收敛性的替代。
- 混淆 coarse mesh、fine mesh、interpolated mesh 的角色。
- 不记录 EPC 输出量的定义和单位，导致不同文献或程序输出不可比较。
- 把 EPC/EPW 高级结果当基础 QE 输出，跳过 SCF/phonon/Wannier 的前置审阅。
- 将自动化 workflow 的成功状态等同于物理结果通过；本仓库主线仍以原生命令行 output review 为准。

## 与基础 workflow 的关系

- 依赖 [phonon dispersion workflow](../phonon/phonon-dispersion-dfpt.md) 的 SCF、q-grid phonon、ASR、negative frequency 和 output review 能力。
- 依赖基础 electronic workflow 的 bands/DOS 读图和收敛性记录能力。
- 依赖 [Wannier90](../../references/tools/wannier90.md) 相关边界：Wannier interpolation 质量必须通过与 QE bands 的对照判断。
- [EPW](../../references/tools/epw.md) 只作为高级入口；学习顺序应先确认 SCF/phonon/Wannier 质量，再阅读 EPW 数据依赖图。
- 高级开关改变 Hamiltonian、采样或插值数据链时，应重新审阅下游 bands、DOS、phonon 或 EPC 结果的可比性。
- 本页只说明边界和审阅字段；完整执行细节应在掌握基础 workflow 后按官方文档展开。

## 资料来源

- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- QE `ph.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE `pw2wannier90.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_pw2wannier90.html>
- EPW documentation: <https://docs.epw-code.org/>
- Wannier90 documentation: <https://wannier90.readthedocs.io/>
