# Theory Minimum

本目录记录正确使用 Quantum ESPRESSO 所需的最低充分 DFT / DFPT / 固体计算理论背景。它面向 input/output 阅读、workflow 准入和结果可靠性判断，回答一个问题：为了读懂 QE input/output，并判断一次计算是否可以进入下游，最低需要知道哪些概念。

Phase 3-A 的核心页已经与 ground-state、electronic 和 phonon 三类 workflow 建立回链：SCF、cutoff、pseudopotential、k-points、smearing、force/stress、DFPT phonon、dielectric tensor 和 Born effective charge 都应作为 output review 的理论回查层使用。

## 阅读方式

先从 `learn/` 的能力路线进入具体 workflow，再到 `workflows/` 阅读 QE 原生命令、输入模板、output review 和 `PASS / WARN / BLOCK` 判断。读 workflow 时如果遇到参数选择、输出行含义或准入结论无法解释，再回到本目录查对应理论页。

理论页只作为 workflow 的回查层使用：

- 看不懂 input 字段时，查对应概念说明，再回到 workflow 检查字段是否成组设置。
- 看不懂 output review 时，查对应物理量或数值判据，再回到 output 判断 `PASS / WARN / BLOCK`。
- 遇到常见错误时，先确认它属于概念误解、输入不一致、收敛不足还是下游依赖错误，再回到 workflow 修复。

阅读本目录时建议按 workflow 问题回查，而不是按时间顺序或某一种材料故事阅读。理论页按 QE workflow 中反复出现的判断问题组织。

## 页面写作要求

每篇理论页都应落到四类可复查对象：

- QE input 字段：相关 namelist、card、关键参数及其联动关系。
- Output review：需要检查的输出位置、警告、收敛信息和下游准入证据。
- 常见错误：典型误读、参数不一致、数值未收敛或文件依赖错误。
- 对应 workflow：应回链到 `workflows/` 中使用该理论判断的计算步骤。

如果一个理论点不能帮助解释 input 字段、output 证据、收敛判断或 workflow 准入，它就不属于本目录的最低理论层；更深的模型边界和物理可信度问题应进入 `physics-judgement/`。

本目录中的核心页不替代 workflow 页面：workflow 决定如何运行和审阅；theory-minimum 解释为什么这些 input/output 证据会影响准入判断。

## 与 physics-judgement/ 的关系

本目录回答“正确使用 QE 的最低理论”；[physics-judgement/](../physics-judgement/) 回答“如何形成更可靠的 DFT 物理判断”。如果问题已经超出参数含义和最低 output review，例如 band gap 的物理解释、DFT+U 的模型边界、SOC/vdW 是否改变结论、phonon 虚频是否代表真实不稳定，应回到 `physics-judgement/` 进一步审阅。

## 页面列表

- [DFT, Kohn-Sham and SCF](dft-ks-scf.md)
- [Plane-wave cutoff](plane-wave-cutoff.md)
- [Exchange-correlation functional choice](functional-choice.md)
- [Pseudopotentials](pseudopotentials.md)
- [K-points, symmetry and k-path](kpoints-symmetry-kpath.md)
- [Reciprocal space and Brillouin zone](reciprocal-space-and-brillouin-zone.md)
- [Band structure and DOS](band-structure-and-dos.md)
- [Crystal symmetry, space group and point group](crystal-symmetry-space-group-point-group.md)
- [Smearing and metals](smearing-metals.md)
- [Force, stress and relaxation](force-stress-relaxation.md)
- [Magnetism, SOC and DFT+U](magnetism-soc-dftu.md)
- [DFPT and phonons](dfpt-phonons.md)
- [Dielectric tensor and Born effective charge](dielectric-born-charge.md)

## 资料来源

- [learn/README.md](../learn/README.md)
- [workflows/README.md](../workflows/README.md)
- [references/source-index.md](../references/source-index.md)
