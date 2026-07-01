# Phonons, Soft Modes and Dynamical Stability

## 1. 这个主题解决什么判断问题？

本页帮助判断 phonon 虚频、软模和动力学稳定性结论是否可信。它把 phonon output 中的 negative/imaginary frequency 放回 Born-Oppenheimer 能量面、DFPT response、q-grid interpolation 和 workflow evidence 中审阅，避免把数值伪影、结构未平衡或文件链错误误写成真实动力学不稳定。

## 物理图像

声子频率来自 Born-Oppenheimer 能量面对原子位移的二阶曲率。把 force constants 质量归一化并转换到 q 空间后得到 dynamical matrix；它的本征值对应 `omega^2(q)`。正的 `omega^2` 表示该振动方向附近是稳定曲率，接近零的 acoustic modes 来自整体平移自由度，负的 `omega^2` 在 output 中通常显示为 imaginary 或 negative frequency。

Soft mode 的物理含义是某个集体位移方向的能量面很平。它可能预示结构相变、畸变或真实动力学不稳定，也可能只是 SCF、force/stress、cutoff、k/q mesh、smearing、ASR、IFC 插值或低维/极性长程相互作用没有处理好。判断虚频时要同时看位置、大小、分支、eigenvector、参数敏感性和上游结构质量。

Gamma acoustic 的小虚频、单个 q 点的局部软模和整条 branch 的系统性不稳定需要不同处理。ASR 能帮助检查平移不变性并改善 acoustic branch 行为，但它不会让未优化结构、错误文件链或不收敛 response 变成可信 phonon。最终判断应落到 mode pattern 和完整 workflow evidence：这个模式在哪里出现、强度多大、属于 acoustic 还是 optical branch、eigenvector 指向什么集体位移、参数变化后是否稳定存在。

## 2. 最低理论结构

在 harmonic phonon 中，dynamical matrix 的本征值对应 `omega^2(q)`。若本征值为正，频率是实数；若本征值为负，程序通常把它显示为 imaginary frequency、negative frequency 或带符号的频率值。这个符号约定提醒我们：输出中的“虚频”不是一个新的振动频率，而是某个集体位移方向上的二阶曲率为负。

判断虚频时应区分三类证据：

| 证据层 | 典型对象 | 判断意义 |
|---|---|---|
| 上游能量面 | relaxed structure、final SCF、force/stress | 结构是否真的在所用模型下接近平衡点 |
| response 计算 | `ph.x` q point、irrep、`tr2_ph` convergence | dynamical matrix 是否由收敛的线性响应得到 |
| 后处理/插值 | `q2r.x` IFC、`matdyn.x` path/DOS、ASR | 虚频是否可能来自有限 q-grid、IFC 截断或后处理约束 |

Mode eigenvector 是 triage 的关键证据。频率告诉我们曲率，eigenvector 告诉我们沿哪个原子位移模式发生软化。没有 eigenvector 或 displacement pattern 的虚频判断，只能停留在数值症状层，不能直接进入相变、畸变机制或稳定性结论。

## 3. DFT/QE 中的近似来源

近似来源包括 SCF 精度、结构残余力、cutoff、k/q sampling、smearing、ASR、IFC 插值、二维/极性长程相互作用和 harmonic approximation。对金属或小带隙体系，Fermi surface sampling 和 smearing 会直接影响力常数；对极性体系，Born charge、dielectric tensor 和 non-analytic correction 会影响 Gamma 附近的 LO/TO 行为；对低维体系，周期镜像和长程相互作用可能让低 q 区域特别敏感。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| tr2_ph | ph.x | 响应方程收敛 | 默认粗略设置进入软模判断 |
| ldisp/nq1/nq2/nq3 | ph.x | q-grid 与 IFC 范围 | q-grid 太粗解释 dispersion |
| `zasr` | `q2r.x` | 在 IFC 转换层施加 acoustic sum rule | 用 ASR 掩盖未收敛问题 |
| `asr` | `matdyn.x/dynmat.x` | 在频率/模式后处理层施加 acoustic sum rule | 未区分后处理修正和真实收敛 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| Gamma acoustic modes 明显偏离零 | 结构、ASR 或数值收敛问题 | relax、SCF、ASR |
| 局部 q 点小虚频 | q-grid、k mesh、smearing 或真实软模 | 参数扫描和模式分析 |
| 整支频率漂移 | cutoff/IFC 插值不足 | cutoff 和 q-grid convergence |
| eigenvector / displacement pattern 指向明确畸变 | 可能是真实软模，也可能暴露结构未平衡 | direct q-point 复算、结构沿模式扰动、force/stress |
| 只在插值曲线出现的虚频 | q-grid/IFC 截断或 path 插值伪影风险 | `ph.x` direct q-point、`q2r.x` 文件链 |

### 虚频分型

| 类型 | 常见含义 | 优先回查 |
|---|---|---|
| Gamma acoustic 小虚频 | 平移不变性、ASR、结构残余力或数值噪声 | final forces/stress、ASR 前后、cutoff/k mesh |
| 单个 q 点小虚频 | 近软模、插值误差或局部采样问题 | direct q-point 复算、eigenvector、q-grid |
| optical branch 明显虚频 | 结构可能偏离局部稳定点，也可能是模型/收敛问题 | eigenvector、结构沿模式扰动、functional/PP |
| 整条 branch 系统性虚频 | 真实不稳定或全链条数值/结构错误 | 上游结构、SCF、q-grid、IFC、smearing |
| polar/2D Gamma 附近异常 | 长程相互作用或边界条件处理不充分 | Born/dielectric、non-analytic、低维边界 |

## 6. 对 workflow 的影响

- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 结构、SCF、cutoff、k/q grid、smearing、ASR、direct q-point 或插值来源、eigenvector 和模式分析均支持频率结论 | 进入 phonon DOS、受限热力学或稳定性讨论 |
| WARN | 小虚频、声学支漂移或局部 soft mode 对参数敏感；eigenvector 可疑但 direct q-point 或结构沿模式复查尚不完整 | 只做待复查软模记录 |
| BLOCK | 未经复查直接把虚频判为真实不稳定；用 ASR 替代收敛测试；文件链混用；只从插值图读出虚频却未回查 direct q-point/IFC | 停止动力学稳定性结论 |

## 8. 常见误区

- 把任何虚频都当真实不稳定
- 用 ASR 替代收敛测试
- 未优化结构就解释 soft mode
- q-grid 太粗却解释细节
- 不看 eigenvector 或 displacement pattern 就宣称相变路径
- 把插值曲线中的孤立虚频当作 direct DFPT 证据
- 忽略有限温度和 anharmonicity 边界

## 9. 与前沿常识的关系

动力学稳定性不同于热力学稳定性；0 K harmonic phonon 不能自动代表有限温度自由能稳定。软模需要结合模式、收敛和物理边界判断。

## 10. 最低掌握深度

- 能从 Gamma acoustic modes、ASR 前后、残余力、cutoff、k/q mesh 和 smearing 判断虚频风险来源。
- 能区分数值噪声、结构未平衡、q-grid 插值不足和真实软模。
- 能说明 dynamical stability 与 thermodynamic stability 的区别。
- 能把 negative frequency 结论写成带证据的 PASS/WARN/BLOCK，而不是直接下物理结论。

## 11. 对应仓库页面

- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Baroni et al., Rev. Mod. Phys. 73, 515 (2001)](https://doi.org/10.1103/RevModPhys.73.515)
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [QE PHonon user guide](https://www.quantum-espresso.org/Doc/ph_user_guide/)
