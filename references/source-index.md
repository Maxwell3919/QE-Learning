# Source Index

本页整理公开网页来源，作为 QE-Learning 的来源索引。它不是工具排名，也不是教程替代品。

## Quantum ESPRESSO 官方来源

- [Quantum ESPRESSO official documentation](https://www.quantum-espresso.org/documentation/)
- [QE ecosystem](https://www.quantum-espresso.org/ecosystem/)
- [QE pseudopotentials](https://www.quantum-espresso.org/pseudopotentials/)
- [QE INPUT_PW reference](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [QE INPUT_PH reference](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [QE INPUT_PP reference](https://www.quantum-espresso.org/Doc/INPUT_PP.html)
- [QE INPUT_BANDS reference](https://www.quantum-espresso.org/Doc/INPUT_BANDS.html)
- [QE INPUT_Q2R reference](https://www.quantum-espresso.org/Doc/INPUT_Q2R.html)
- [QE INPUT_pw2wannier90 reference](https://www.quantum-espresso.org/Doc/INPUT_pw2wannier90.html)
- [QE PHonon user guide](https://www.quantum-espresso.org/Doc/ph_user_guide/)
- [QE user guide: parallelization](https://www.quantum-espresso.org/Doc/user_guide/node20.html)

## 优秀教程与学校材料

- [Pranab Das QE tutorial](https://pranabdas.github.io/espresso/)
- [Pranab Das SCF tutorial](https://pranabdas.github.io/espresso/hands-on/scf)
- [Pranab Das phonon tutorial](https://pranabdas.github.io/espresso/hands-on/phonon/)
- [Pranab Das GaAs phonon repository directory](https://github.com/pranabdas/espresso/tree/master/src/GaAs-phonon)
- [Kyoto University QE phonon calculation DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97)
- [Kyoto University QE phonon DOS and dispersion DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E7%8A%B6%E6%85%8B%E5%AF%86%E5%BA%A6%E3%81%A8%E5%88%86%E6%95%A3)
- [Materials Cloud Quantum ESPRESSO school 2023](https://github.com/materialscloud-org/QuantumESPRESSO-school-2023)
- [ENCCS QE tutorial](https://enccs.github.io/efficient-materials-modelling-on-hpc/espresso-tutorial/)
- [ENCCS QE phonons tutorial](https://enccs.github.io/efficient-materials-modelling-on-hpc/espresso-tutorial-phonons/)

## 辅助工具文档

- [ASE documentation](https://ase.gitlab.io/ase)
- [ASE espresso IO source](https://ase-lib.org/_modules/ase/io/espresso.html)
- [pymatgen-io-espresso](https://github.com/oashour/pymatgen-io-espresso)
- [SeeK-path documentation](https://seekpath.readthedocs.io/en/latest/maindoc.html)
- [Phonopy QE interface](https://phonopy.github.io/phonopy/qe.html)
- [Wannier90](https://wannier.org/)
- [EPW documentation](https://docs.epw-code.org/)
- [AiiDA-QE tutorials](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/)
- [AiiDA-QE workflow logic](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/workflow_logic.html)

## 使用规则

- 官方 input reference 用于核对参数定义和适用条件。
- Pranab / Kyoto 等教程用于学习工作流拆解和 output 判断。
- ASE、pymatgen、SeeK-path、phonopy、Wannier90 等只作为辅助参考，不替代 QE 原生命令行学习。
- AiiDA / AiiDA-QE 只作为高级 workflow/provenance 参考。
