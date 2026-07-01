# Theory Minimum

本目录只记录正确使用 Quantum ESPRESSO 所需的最低理论背景。它不是 DFT 教材，也不追求从历史、定理或完整推导建立理论体系；它只回答一个问题：为了读懂 QE input/output，并判断一次计算是否可以进入下游，最低需要知道哪些概念。

## 阅读方式

先从 `learn/` 的能力路线进入具体 workflow，再到 `workflows/` 阅读 QE 原生命令、输入模板、output review 和 `PASS / WARN / BLOCK` 判断。读 workflow 时如果遇到参数选择、输出行含义或准入结论无法解释，再回到本目录查对应理论页。

理论页只作为 workflow 的回查层使用：

- 看不懂 input 字段时，查对应概念说明，再回到 workflow 检查字段是否成组设置。
- 看不懂 output review 时，查对应物理量或数值判据，再回到 output 判断 `PASS / WARN / BLOCK`。
- 遇到常见错误时，先确认它属于概念误解、输入不一致、收敛不足还是下游依赖错误，再回到 workflow 修复。

不要按时间顺序阅读本目录，也不要把它写成具体材料案例库。理论页应按 QE workflow 中反复出现的判断问题组织，而不是按某一种材料、体系或研究故事组织。

## 页面写作要求

每篇理论页都应落到四类可复查对象：

- QE input 字段：相关 namelist、card、关键参数及其联动关系。
- Output review：需要检查的输出位置、警告、收敛信息和下游准入证据。
- 常见错误：典型误读、参数不一致、数值未收敛或文件依赖错误。
- 对应 workflow：应回链到 `workflows/` 中使用该理论判断的计算步骤。

## 与 physics-judgement/ 的关系

本目录回答“正确使用 QE 的最低理论”；[physics-judgement/](../physics-judgement/) 回答“如何形成更可靠的 DFT 物理判断”。如果问题已经超出参数含义和最低 output review，例如 band gap 的物理解释、DFT+U 的模型边界、SOC/vdW 是否改变结论、phonon 虚频是否代表真实不稳定，应回到 `physics-judgement/` 进一步审阅。

## 页面列表

- [DFT, Kohn-Sham and SCF](dft-ks-scf.md)
- [Plane-wave cutoff](plane-wave-cutoff.md)
- [Exchange-correlation functional choice](functional-choice.md)
- [Pseudopotentials](pseudopotentials.md)
- [K-points, symmetry and k-path](kpoints-symmetry-kpath.md)
- [Smearing and metals](smearing-metals.md)
- [Force, stress and relaxation](force-stress-relaxation.md)
- [Magnetism, SOC and DFT+U](magnetism-soc-dftu.md)
- [DFPT and phonons](dfpt-phonons.md)
- [Dielectric tensor and Born effective charge](dielectric-born-charge.md)

## 资料来源

- [learn/README.md](../learn/README.md)
- [workflows/README.md](../workflows/README.md)
- [references/source-index.md](../references/source-index.md)
