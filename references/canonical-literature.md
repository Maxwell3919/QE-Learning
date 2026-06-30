
# Canonical Literature

本页维护 QE-Learning 的 canonical literature spine。它不是 DOI 清单，也不替代逐篇精读；它用于说明哪些公开文献支撑仓库中的物理判断结论。

## 分级规则

| 等级 | 定义 | 仓库用途 |
|---|---|---|
| A0 | 不可跳过；缺它会误解 DFT/QE 基础 | `theory-minimum/` 与 `physics-judgement/` 的基础判断 |
| A1 | 方法核心；缺它会误解某类 workflow | `workflows/` 和高级边界页的主要来源 |
| A2 | 判断必读；缺它会误判 failure mode | `physics-judgement/` 的 failure-mode 支撑 |
| B1 | 前沿入口；进入专题研究前先读 | `references/` 与 `workflows/advanced/` 的 gateway |
| B2 | 专题深入；有具体科研问题再读 | 方向性补充，不进入基础路线 |
| C | 收藏级背景 | 历史和拓展阅读 |

## Fast Core Loop

该列表控制在 20 篇内，用于建立 QE/DFT 物理判断力。

| 序 | 文献 | 等级 | 支撑的判断 |
|---|---|---|---|
| 1 | Hohenberg & Kohn, Inhomogeneous Electron Gas | A0 | ground-state density 边界 |
| 2 | Kohn & Sham, Self-Consistent Equations Including Exchange and Correlation Effects | A0 | KS 辅助体系 |
| 3 | Kohn, Nobel Lecture | A0 | density vs wavefunction 方法边界 |
| 4 | Perdew, Burke & Ernzerhof, GGA Made Simple | A1 | 常用 GGA functional 起点 |
| 5 | Kim, Sim & Burke, Understanding and Reducing Errors in DFT | A2 | functional error / density-driven error |
| 6 | Mori-Sanchez, Cohen & Yang, Current Limitations of DFT | A2 | delocalization、gap、charge-transfer 边界 |
| 7 | Vanderbilt, Ultrasoft Pseudopotentials | A1 | USPP 与 cutoff 关系 |
| 8 | Blochl, Projector Augmented-Wave Method | A1 | PAW 思想与边界 |
| 9 | Monkhorst & Pack, Special Points for Brillouin-Zone Integrations | A1 | k mesh 是积分对象 |
| 10 | Methfessel & Paxton, High-Precision Sampling for Brillouin-Zone Integration | A1 | smearing 的数值角色 |
| 11 | Blochl, Jepsen & Andersen, Improved Tetrahedron Method | A1 | DOS/Fermi-surface integration |
| 12 | Giannozzi et al., Quantum ESPRESSO code paper | A1 | QE 方法和模块总入口 |
| 13 | Giannozzi et al., Advanced capabilities for materials modelling with QE | A1 | QE 高级能力边界 |
| 14 | Baroni et al., Phonons and related crystal properties from DFPT | A0 | phonon/DFPT response 总入口 |
| 15 | Gonze & Lee, Dynamical matrices, Born charges, dielectric tensors and IFCs | A1 | polar phonon / LO-TO 边界 |
| 16 | Cococcioni & de Gironcoli, Linear response approach to DFT+U | A2 | U provenance |
| 17 | Marzari & Vanderbilt, Maximally localized generalized Wannier functions | A1 | Wannier validation 起点 |
| 18 | Giustino, Electron-phonon interactions from first principles | A1 | EPC data chain |
| 19 | Onida, Reining & Rubio, Electronic excitations | A0 | excited-state boundary |
| 20 | Lejaeghere et al., Reproducibility in density functional theory calculations of solids | A2 | uncertainty / reproducibility |

## 使用规则

- 页面正文只引用结论，不堆 DOI。
- 参数和程序行为仍以 QE 官方 `INPUT_*`、PHonon guide、Wannier90/EPW/Yambo 官方文档为准。
- 任何 version-sensitive 字段都不得只靠文献摘要确认。
- 新增文献进入 A 级前，必须说明它改变了哪个 output review 或 PASS/WARN/BLOCK 判断。

## 原始链接

- [QE documentation](https://www.quantum-espresso.org/documentation/)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [EPW documentation](https://docs.epw-code.org/)
- [Wannier90 documentation](https://wannier.org/)
- [Yambo documentation](https://www.yambo-code.eu/wiki/index.php/Documentation)
