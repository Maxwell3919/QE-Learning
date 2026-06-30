# Source Index

本页是公开网页来源索引。它用于定位原始资料，不替代 `references/tutorial-sites/` 和 `references/tools/` 中的学习导读。

## 使用规则

- 参数定义优先核对 QE 官方 `INPUT_*` reference。
- 程序链优先核对 QE 官方 guide、Pranab、Kyoto 或 package 文档。
- 工具文档只说明辅助边界，不替代 QE 原生命令行 workflow 学习。
- 本页维护公开网页、教程和工具入口；论文级 canonical source 进入 [canonical-literature.md](canonical-literature.md)，不要塞进本页正文。

## 原始链接

| 来源 | 用途 | 对应导读页 |
|---|---|---|
| [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/) | 官方程序和 package 文档入口 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html) | `pw.x` 参数定义 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html) | `ph.x` 参数定义 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE INPUT_PP](https://www.quantum-espresso.org/Doc/INPUT_PP.html) | `pp.x` 参数定义 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE INPUT_BANDS](https://www.quantum-espresso.org/Doc/INPUT_BANDS.html) | `bands.x` 参数定义 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE INPUT_Q2R](https://www.quantum-espresso.org/Doc/INPUT_Q2R.html) | `q2r.x` 参数定义 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE INPUT_MATDYN](https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html) | `matdyn.x` 参数定义 | [official-qe-docs.md](tutorial-sites/official-qe-docs.md) |
| [QE PHonon user guide](https://www.quantum-espresso.org/Doc/ph_user_guide/) | PHonon 程序链和使用说明 | [kyoto-phonon-dokuwiki.md](tutorial-sites/kyoto-phonon-dokuwiki.md) |
| [Pranab Das QE tutorial](https://pranabdas.github.io/espresso/) | hands-on workflow 拆解 | [pranab-das-qe-tutorial.md](tutorial-sites/pranab-das-qe-tutorial.md) |
| [Kyoto QE phonon DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97) | DFPT phonon 判断和 caveat | [kyoto-phonon-dokuwiki.md](tutorial-sites/kyoto-phonon-dokuwiki.md) |
| [Materials Cloud QE school](https://github.com/materialscloud-org/QuantumESPRESSO-school-2023) | 学校材料和高级专题 | [materials-cloud-qe-schools.md](tutorial-sites/materials-cloud-qe-schools.md) |
| [ENCCS QE tutorial](https://enccs.github.io/efficient-materials-modelling-on-hpc/espresso-tutorial/) | HPC 上运行 QE | [enccs-qe-hpc-tutorials.md](tutorial-sites/enccs-qe-hpc-tutorials.md) |
| [ASE documentation](https://ase.gitlab.io/ase) | 结构脚本和输入生成 | [ase.md](tools/ase.md) |
| [pymatgen documentation](https://pymatgen.org/) | 结构对象和数据处理 | [pymatgen.md](tools/pymatgen.md) |
| [SeeK-path documentation](https://seekpath.readthedocs.io/en/latest/maindoc.html) | high-symmetry k-path | [seekpath.md](tools/seekpath.md) |
| [Phonopy QE interface](https://phonopy.github.io/phonopy/qe.html) | finite-displacement phonon 接口 | [phonopy-qe.md](tools/phonopy-qe.md) |
| [Wannier90](https://wannier.org/) | Wannier interpolation | [wannier90.md](tools/wannier90.md) |
| [EPW documentation](https://docs.epw-code.org/) | electron-phonon coupling 高级工作流 | [epw.md](tools/epw.md) |
| [VESTA](https://jp-minerals.org/vesta/en/) | 可视化 | [vesta-xcrysden.md](tools/vesta-xcrysden.md) |
| [XCrySDen](http://www.xcrysden.org/) | 可视化和费米面 | [vesta-xcrysden.md](tools/vesta-xcrysden.md) |
| [AiiDA-QE tutorials](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/) | 高级 provenance/workflow 参考 | [aiida-qe-optional.md](tools/aiida-qe-optional.md) |


## Canonical literature spine

| 来源 | 用途 | 对应导读页 |
|---|---|---|
| [Canonical Literature](canonical-literature.md) | A0/A1/A2/B/C 文献分级和 Fast Core Loop | [reading-maintenance-policy.md](reading-maintenance-policy.md) |
| [Canonical BibTeX](canonical.bib) | 可复用 BibTeX 起点 | [canonical-literature.md](canonical-literature.md) |
| [Yambo educational wiki](https://wiki.yambo-code.eu/wiki/index.php?title=Main_Page) | GW/BSE 与 excited-state 工具入口 | [tools/yambo.md](tools/yambo.md) |
| [Yambo learn](https://www.yambo-code.eu/learn/) | Yambo 理论、教程和代码文档入口 | [tools/yambo.md](tools/yambo.md) |
