# Output Review Checklist

本页用于把 QE output 从“程序跑完”审阅成可记录的 `PASS / WARN / BLOCK` 状态。它不要求仓库保存具体 output 文件，也不替代具体 workflow 页面；审阅者只需要在个人计算记录中写明 output 位置、关键证据、状态理由和允许进入的下游。

每个 workflow 至少执行四步：

1. 确认 output 对应正确的 input、command、`prefix/outdir` 和 QE 程序。
2. 从 output 中摘录决定状态的关键行或数值。
3. 按本页列出的 WARN/BLOCK 触发项给出状态。
4. 明确写出 `Downstream allowed`，不要只写 `JOB DONE`。
5. 写出 `Uncertainty statement`，区分 numerical uncertainty、model uncertainty、pseudopotential uncertainty、finite-size/boundary uncertainty 和 workflow propagation uncertainty；没有这一步，`PASS` 只能表示 workflow 通过，不能直接升级为科研定量结论。

状态定义以 [pass-warn-block.md](pass-warn-block.md) 为准：

- `PASS`：允许进入列出的正式下游。
- `WARN`：只允许进入低风险探索或诊断下游；不得作为最终结论。
- `BLOCK`：不得进入下游，先修复输入、参数、文件链或 workflow。

## SCF

### Output 中要看

- [ ] 程序、input 文件名、`prefix/outdir`、赝势、cutoff、k-points、smearing 与记录一致。
- [ ] SCF iteration 正常结束，电子收敛达到 input 中的阈值。
- [ ] total energy、estimated scf accuracy、Fermi energy 或 band edge 信息已记录。
- [ ] 电荷、磁矩、自旋、非共线或 SOC 相关设置与 input 目标一致。
- [ ] warning、error、FFT/grid、对称性、负电荷密度、占据数异常等信息已解释。

### WARN 触发

- 程序完成但电子收敛较慢、接近最大 iteration，或 convergence 余量很小。
- smearing、k-points、cutoff 或混合参数只适合探索，尚无收敛测试。
- output 出现可解释 warning，但不直接破坏当前目标。
- 金属/绝缘体占据特征与预期不完全一致，需要后续确认。
- 自旋或对称性结果可疑，但当前只用于参数摸底。

### BLOCK 触发

- SCF 未收敛、达到最大 iteration、异常中止，或没有完整结束信息。
- `prefix/outdir` 混用旧数据，output 与 input/command 对不上。
- 赝势来源不明，泛函、相对论类型或元素配置混用。
- cutoff、k-points、smearing、电子数、结构或晶胞单位明显错误。
- 未解释 warning 直接影响能量、电荷、占据、对称性或波函数可靠性。

### 允许进入的下游

- `PASS`：NSCF、bands、DOS、PDOS、pp.x、relax/vc-relax 前的基准检查、phonon 前置静态检查。
- `WARN`：参数调试、收敛测试、结构或赝势排错；不得作为最终 bands/DOS/phonon 依据。
- `BLOCK`：不允许进入任何依赖波函数、电荷密度或总能的下游。

## NSCF

### Output 中要看

- [ ] 读取的 `prefix/outdir` 来自已审阅的 SCF，且结构、赝势和电子设置一致。
- [ ] k-point mesh 或 k-list 符合目标：dense uniform mesh、DOS mesh、Fermi surface mesh 或其他明确用途。
- [ ] band 数量、占据、Fermi energy、能带范围满足后处理需求。
- [ ] output 没有读取旧电荷密度、波函数或重启文件失败的证据。
- [ ] 并行、diagonalization、memory warning 不影响当前数据完整性。

### WARN 触发

- NSCF 完成，但 k-point 密度只适合探索。
- band 数量刚好够用，对高能空带、投影或光学类后处理余量不足。
- Fermi level、占据或带隙信息与 SCF 有差异，需要确认是否由 k-mesh 或 smearing 引起。
- 读取文件链可追踪，但发生过 restart 或轻微 I/O warning。

### BLOCK 触发

- NSCF 没有读取正确 SCF 数据，或 `prefix/outdir` 与上游不一致。
- k-point 类型与目标不匹配，例如用路径型 k-list 支撑 DOS。
- band 数量不足导致目标能量窗口缺失。
- output 显示波函数、电荷密度、赝势或结构信息不一致。
- 程序未完成或关键 eigenvalue 数据不可用。

### 允许进入的下游

- `PASS`：DOS、PDOS、Fermi surface、部分 pp.x 后处理、需要 dense eigenvalues 的分析。
- `WARN`：mesh 或 band 数量调试、能量窗口预检查；不得用于最终积分型 observable。
- `BLOCK`：不允许进入 DOS/PDOS/Fermi surface 或依赖 NSCF eigenvalues 的下游。

## Relax

### Output 中要看

- [ ] 每个 ionic step 的内部 SCF 都有可接受收敛状态。
- [ ] final energy、final force、最大力分量、位移趋势和优化停止原因已记录。
- [ ] 约束、固定原子、`ion_dynamics`、force threshold 与 input 目标一致。
- [ ] final atomic positions 已明确来源于 output，而不是旧 input。
- [ ] final structure 是否需要重新做 static SCF 已写入记录。

### WARN 触发

- 几何优化完成但 final force 接近阈值，或离目标精度仍有余量问题。
- ionic step 中个别 SCF 收敛较差，但最终结构只用于继续优化或探索。
- 发生 restart、trust radius 调整、step backtracking 等，需要保留 provenance。
- 约束设置合理但会限制下游解释范围。

### BLOCK 触发

- relax 未达到力收敛，异常中止，或停止原因不是可信收敛。
- 任一关键 ionic step 的 SCF 明显失败并污染后续结构。
- final structure 无法追踪，或 input/output 中原子顺序、单位、坐标系混乱。
- 约束、固定原子或晶胞设置与目标不一致。
- 直接把未审阅 final structure 用作最终性质结论。

### 允许进入的下游

- `PASS`：final static SCF、NSCF、bands、DOS、PDOS、pp.x、phonon 前置结构。
- `WARN`：继续 relax、调整优化参数、粗略结构筛选；不得进入最终 phonon 或高精度性质计算。
- `BLOCK`：不允许进入任何依赖平衡结构的下游。

## vc-relax

### Output 中要看

- [ ] 每个 ionic/cell step 的 SCF 状态、力、应力、压力和晶胞变化已检查。
- [ ] final cell parameters、atomic positions、volume、pressure/stress 与目标阈值已记录。
- [ ] `cell_dynamics`、`cell_dofree`、外压、晶胞约束和对称性处理与 input 目标一致。
- [ ] 晶胞没有非物理突变、体积塌缩、角度异常或单位混淆。
- [ ] final cell + final positions 已用于独立 static SCF 计划或记录。

### WARN 触发

- 力或应力接近阈值但仍可用于继续优化。
- 晶胞变化较大，需要重新评估 cutoff、k-points 或赝势适用性。
- 压力、应力或体积趋势可解释，但尚未做独立 final SCF。
- 晶胞约束合理但限制了下游声子、弹性或相稳定性解释。

### BLOCK 触发

- vc-relax 未达到力/应力收敛，异常中止，或最终停止原因不可信。
- final cell 明显非物理，或由错误单位、错误外压、错误约束造成。
- output 中结构、晶胞、对称性或赝势信息与 input/记录不一致。
- 直接把 vc-relax output 当作最终静态性质结果。
- 未能区分临时继续优化结构与可进入性质计算的平衡结构。

### 允许进入的下游

- `PASS`：final static SCF、NSCF、bands、DOS、PDOS、phonon、应力敏感后处理。
- `WARN`：继续 vc-relax、参数收敛排查、结构趋势探索；不得作为最终相稳定性或声子依据。
- `BLOCK`：不允许进入依赖平衡晶胞和结构的下游。

## Bands

### Output 中要看

- [ ] 上游 SCF 状态可信，bands 计算读取的 `prefix/outdir` 与上游一致。
- [ ] k-path 来源、端点、分段、单位和高对称点标签已记录。
- [ ] band 数量覆盖目标能量窗口。
- [ ] Fermi level、energy zero、plot 或 `bands.x` 输出的参考能量已明确。
- [ ] band crossing、metal/insulator 判断、spin/SOC 分支标记与 input 设置一致。

### WARN 触发

- k-path 来源可追踪但未与具体晶胞标准化流程完全对齐。
- band 数量或能量窗口只够初步看图。
- Fermi level 取自 SCF/NSCF 的方式需要注明，可能影响图上零点。
- 自旋、SOC 或非共线分支标签需要进一步人工核对。

### BLOCK 触发

- 上游 SCF 不可信或文件链不一致。
- k-path 与晶胞、布里渊区或坐标单位不匹配。
- band 数量不足，目标能区缺失。
- energy zero/Fermi level 不明，导致图或结论不可复查。
- `bands.x`、plot 数据或后处理输出缺失。

### 允许进入的下游

- `PASS`：能带图、带隙/带交叉定性分析、与 DOS/PDOS 交叉检查。
- `WARN`：路径或绘图调试、初步电子结构判断；不得作为最终带隙或拓扑类结论。
- `BLOCK`：不允许进入基于 bands 的解释、作图归档或论文级结论。

## DOS

### Output 中要看

- [ ] 上游 NSCF 使用 dense uniform mesh，且读取正确 SCF 数据。
- [ ] `dos.x` output 中 energy grid、smearing/broadening、Fermi level 和积分范围已记录。
- [ ] 总 DOS 文件生成成功，能量零点和单位明确。
- [ ] DOS 积分、电荷数或占据趋势与体系电子数定性一致。
- [ ] 金属/绝缘体判断与 SCF、NSCF、bands 结果交叉检查。

### WARN 触发

- mesh 或 broadening 只适合探索，峰位/带隙不用于最终数值。
- Fermi level 与上游略有差异，需要注明来源。
- 能量窗口偏窄但仍覆盖当前观察目标。
- DOS 曲线噪声较大，需要更密 k-mesh 或调整 broadening。

### BLOCK 触发

- 使用了路径型 bands k-list 或不合适的 NSCF 数据。
- `dos.x` 未完成，总 DOS 文件缺失或能量零点不可复查。
- energy grid、Fermi level、broadening 或单位不明。
- DOS 与电子数、占据、bands 出现无法解释的矛盾。
- 上游 SCF/NSCF 状态为 BLOCK。

### 允许进入的下游

- `PASS`：DOS 图、能隙/费米面附近态密度分析、与 PDOS/bands 交叉检查。
- `WARN`：k-mesh、broadening、能量窗口调试；不得作为最终峰位或带隙数值。
- `BLOCK`：不允许进入 DOS 解释、图件归档或 PDOS 对照结论。

## PDOS

### Output 中要看

- [ ] 上游 SCF/NSCF 状态可信，投影读取的波函数和赝势一致。
- [ ] `projwfc.x` output 中投影通道、原子标签、轨道标签、spin/SOC 分量可解释。
- [ ] PDOS 文件完整生成，文件名与原子/轨道映射可追踪。
- [ ] energy zero、Fermi level、broadening 与 DOS 对齐或差异已说明。
- [ ] 总 PDOS 与 DOS 的关系已做定性 sanity check。

### WARN 触发

- 投影标签复杂或存在简并/混合，需要人工解释。
- PDOS 与 DOS 不完全一致，但差异可能来自投影定义、能量网格或 broadening。
- band 数量、能量窗口或 k-mesh 对高能投影余量不足。
- spin/SOC 分量可读但尚未完成图例和通道命名核对。

### BLOCK 触发

- 上游波函数、赝势或 `prefix/outdir` 不一致。
- 投影文件缺失、通道标签不可解释，或原子顺序无法对应结构。
- energy zero/Fermi level 与 DOS/bands 对不上且无法解释。
- 关键轨道或能量窗口缺失。
- 上游 SCF/NSCF/DOS 状态为 BLOCK。

### 允许进入的下游

- `PASS`：轨道成分分析、PDOS 图、与 bands/DOS 的电子结构解释。
- `WARN`：投影标签清理、绘图调试、通道归并探索；不得作为最终轨道归属结论。
- `BLOCK`：不允许进入轨道成分解释或图件归档。

## pp.x

### Output 中要看

- [ ] 上游 SCF/NSCF 数据可信，`prefix/outdir` 和 plot target 与目标一致。
- [ ] `pp.x` output 中读取的数据类型、plot number、spin 分量、文件格式和网格信息已记录。
- [ ] 输出文件成功生成，单位、坐标系、cell/grid 尺寸和可视化工具需求已明确。
- [ ] charge density、potential、ELF、STM 或其他后处理量的物理含义没有混用。
- [ ] 后处理量与上游计算类型匹配，例如是否需要 spin/SOC/NSCF 数据。

### WARN 触发

- 输出文件可用但网格较粗，只适合预览。
- 可视化坐标、单位或裁剪范围需要进一步核对。
- plot target 正确但物理解释依赖额外归一化或可视化设置。
- 上游状态为 WARN，当前 pp.x 只用于诊断。

### BLOCK 触发

- `pp.x` 读取错误数据、错误 `prefix/outdir`，或输出文件缺失。
- plot number、spin 分量或数据类型选错。
- 单位、坐标系、晶胞网格或文件格式不可复查。
- 把可视化 artifact 当作物理结论，且没有回到 output 和上游数据核对。
- 上游 SCF/NSCF 状态为 BLOCK。

### 允许进入的下游

- `PASS`：电荷密度/势/ELF 等图件、定性空间分布分析、与电子结构结果交叉检查。
- `WARN`：可视化参数调试、网格和格式转换测试；不得作为最终空间分布结论。
- `BLOCK`：不允许进入图件归档、定量解释或下游可视化分析。

## Phonon

### Output 中要看

- [ ] 前置结构来自可信 relax/vc-relax 后的 final static SCF。
- [ ] `ph.x` output 中每个 q-point、irreducible representation、perturbation 都达到收敛。
- [ ] dielectric、effective charge、Raman、electron-phonon 等可选量只在 input 目标要求时解释。
- [ ] dyn 文件、q-point 列表、`q2r.x`、`matdyn.x`、freq 或 phonon DOS 文件链完整。
- [ ] ASR 设置、声学模、负频/虚频、单位和绘图能量零点已记录。

### WARN 触发

- 小负频或声学模偏离零点，但尚未完成 cutoff、k/q mesh、力阈值或 ASR 排查。
- 个别 perturbation 收敛较慢，但 output 完整且只用于诊断。
- q-mesh、smearing 或阈值只适合初步稳定性判断。
- ASR 或 non-analytical correction 设置需要在图和记录中明确。

### BLOCK 触发

- 前置结构未充分 relax，或 final static SCF 不可信。
- `ph.x` 任一关键 q-point/perturbation 未收敛、异常中止或 dyn 文件缺失。
- q-point、dyn、fc、freq 文件链不完整或混用不同结构/参数。
- 负频明显且未能归类为数值误差、声学零点问题或真实不稳定性。
- ASR、单位、q-path 或文件来源不明，导致结果不可复查。

### 允许进入的下游

- `PASS`：phonon dispersion、phonon DOS、热力学/稳定性基础分析、Born charge 或 IR/Raman 后处理。
- `WARN`：收敛排查、ASR/q-mesh 测试、虚频来源诊断；不得作为最终动力学稳定性结论。
- `BLOCK`：不允许进入 phonon 图件归档、稳定性声明或任何依赖力常数的下游。

## Advanced

高级 workflow 包括 spin、SOC、DFT+U、vdW、hybrid、MD、NEB、EPC、Wannier90、赝势生成和其他超出基础路径的计算。它们必须先通过对应基础 workflow 的 output review，再增加高级特有检查。

### Output 中要看

- [ ] 高级功能的上游 SCF/NSCF/relax/phonon 状态为 `PASS`，或明确记录为何只能探索。
- [ ] input 中新增物理模型的参数、适用范围和来源已记录，例如 U、vdW 修正、SOC、hybrid、MD/NEB/EPC/Wannier 设置。
- [ ] output 中高级模块确实启用，且关键迭代、约束、投影、耦合或轨迹信息完整。
- [ ] 相关中间文件链可追踪，没有混用不同结构、赝势、k/q mesh 或 `prefix/outdir`。
- [ ] 结果解释没有超过当前 workflow 的最低学习目标和来源边界。

### WARN 触发

- 高级参数来自初步测试或文献/教程迁移，尚未完成体系内收敛或敏感性检查。
- output 完成但关键 observable 对参数、网格、初始态或投影选择敏感。
- 上游为 WARN，当前只用于学习、复现流程或诊断。
- 官方文档、教程和个人推断的边界已经记录，但还缺少最终验证。

### BLOCK 触发

- 基础上游 workflow 为 BLOCK。
- 高级功能没有在 output 中真正启用，或启用方式与 input 目标不一致。
- 关键参数来源不明、单位不明、版本边界不明，或与赝势/泛函不兼容。
- 中间文件链混用，导致结果无法追溯到明确的结构和参数集合。
- 把高级 workflow 的演示性 output 直接当作材料结论。

### 允许进入的下游

- `PASS`：对应高级 workflow 的下一步分析、图件、复现实验或与基础结果交叉检查。
- `WARN`：教程复现、参数敏感性测试、debugging、来源补充；不得进入最终科研结论。
- `BLOCK`：不允许进入高级分析、图件归档、论文级结论或进一步自动化。
