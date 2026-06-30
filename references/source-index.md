# Source Index

本索引来自两个本轮要求优先读取的 `.docx` 文件：

- `/Users/paquette/Downloads/Quantum ESPRESSO Tutorial Website Ecosystem.docx`
- `/Users/paquette/Downloads/Quantum ESPRESSO as a Modern Research Stack.docx`

本页保留网页索引、工具名称、教程站点、QE 原生模块和适合转化为 GitHub 笔记的内容单元。旧 `.md` 提取文件不作为本轮优先来源。

## 1. 文档目录结构

### Quantum ESPRESSO Tutorial Website Ecosystem.docx

- Ecosystem map
- Deep analysis of the reference sites
  - Pranab Das QE tutorial and repository
  - Kyoto University QE phonon DokuWiki
  - What the two reference sites reveal together
- Workflow representation comparison
- Learning path synthesis
  - Minimal SCF learning path
  - Electronic-structure workflow path
  - Phonon DFPT learning path
  - Advanced workflow integration path
- Ecosystem gaps and opportunities
- Synthesis

### Quantum ESPRESSO as a Modern Research Stack.docx

- Native architecture of the QE ecosystem
- External tooling that actually extends QE workflows
- Canonical QE workflow graphs used in research practice
- A modern learning roadmap for becoming productive with QE
- Theory that QE users often miss but cannot safely skip
- What the modern QE research stack actually looks like

## 2. 抽取到的网页链接

### Tutorial Website Ecosystem.docx

- [AiiDA-QE PwBandsWorkChain](https://aiida-quantumespresso.readthedocs.io/en/latest/howto/run_pwbands.html)
- [AiiDA-QE tutorials](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/)
- [AiiDAlab QE app docs](https://aiidalab-qe.readthedocs.io/)
- [ENCCS QE tutorial](https://enccs.github.io/efficient-materials-modelling-on-hpc/espresso-tutorial/)
- [ENCCS QE phonons tutorial](https://enccs.github.io/efficient-materials-modelling-on-hpc/espresso-tutorial-phonons/)
- [Materials Cloud Quantum ESPRESSO school 2023](https://github.com/materialscloud-org/QuantumESPRESSO-school-2023)
- [Pranab Das GaAs phonon repository directory](https://github.com/pranabdas/espresso/tree/master/src/GaAs-phonon)
- [Phonopy QE interface](https://phonopy.github.io/phonopy/qe.html)
- [Pranab Das QE tutorial](https://pranabdas.github.io/espresso/)
- [Pranab Das phonon tutorial](https://pranabdas.github.io/espresso/hands-on/phonon/)
- [Pranab Das SCF tutorial](https://pranabdas.github.io/espresso/hands-on/scf)
- [Pranab Das Jupyter setup](https://pranabdas.github.io/espresso/setup/jupyter)
- [Pranab GaAs phonon run.sh](https://raw.githubusercontent.com/pranabdas/espresso/main/src/GaAs-phonon/run.sh)
- [QE PW user guide PDF](https://www.quantum-espresso.org/Doc/user_guide_PDF/pw_user_guide.pdf)
- [QE documentation](https://www.quantum-espresso.org/documentation/)
- [Kyoto phonon DOS and dispersion DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E7%8A%B6%E6%85%8B%E5%AF%86%E5%BA%A6%E3%81%A8%E5%88%86%E6%95%A3)
- [Kyoto phonon calculation DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97)

### Modern Research Stack.docx

- [AiiDA-QE PwBandsWorkChain](https://aiida-quantumespresso.readthedocs.io/en/latest/howto/run_pwbands.html)
- [AiiDA-QE protocols](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/protocol.html)
- [AiiDA-QE workflow logic](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/workflow_logic.html)
- [AiiDA plugin registry: aiida-quantumespresso](https://aiida.net/plugin-registry/aiida-quantumespresso/)
- [AiiDAlab QE app paper](https://arxiv.org/abs/2303.12465?utm_source=chatgpt.com)
- [ASE espresso IO source](https://ase-lib.org/_modules/ase/io/espresso.html)
- [ASE documentation](https://ase.gitlab.io/ase)
- [VESTA reference page](https://docs.mat3ra.com/reference/software-directory/analysis/vesta/?utm_source=chatgpt.com)
- [QEF/q-e GitHub mirror](https://github.com/QEF/q-e)
- [pymatgen-io-espresso](https://github.com/oashour/pymatgen-io-espresso)
- [Kohn-Sham 1965](https://link.aps.org/doi/10.1103/PhysRev.140.A1133?utm_source=chatgpt.com)
- [Baroni et al. DFPT review](https://link.aps.org/doi/10.1103/RevModPhys.73.515?utm_source=chatgpt.com)
- [SeeK-path documentation](https://seekpath.readthedocs.io/en/latest/maindoc.html)
- [AiiDA paper](https://www.nature.com/articles/s41597-020-00638-4?utm_source=chatgpt.com)
- [QE INPUT_BANDS](https://www.quantum-espresso.org/Doc/INPUT_BANDS.html)
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [QE INPUT_PP](https://www.quantum-espresso.org/Doc/INPUT_PP.html)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [QE INPUT_Q2R](https://www.quantum-espresso.org/Doc/INPUT_Q2R.html?utm_source=chatgpt.com)
- [QE INPUT_pw2wannier90](https://www.quantum-espresso.org/Doc/INPUT_pw2wannier90.html)
- [QE PHonon user guide: q2r/matdyn section](https://www.quantum-espresso.org/Doc/ph_user_guide/node6.html)
- [QE PHonon user guide: troubleshooting/FAQ section](https://www.quantum-espresso.org/Doc/ph_user_guide/node9.html?utm_source=chatgpt.com)
- [QE user guide: parallelization](https://www.quantum-espresso.org/Doc/user_guide/node20.html)
- [QE PW user guide PDF](https://www.quantum-espresso.org/Doc/user_guide_PDF/pw_user_guide.pdf?utm_source=chatgpt.com)
- [QE ecosystem](https://www.quantum-espresso.org/ecosystem/)
- [QE pseudopotentials](https://www.quantum-espresso.org/pseudopotentials/)

## 3. 教程网站名称

- Pranab Das QE tutorial
- Kyoto University QE phonon DokuWiki
- QE official documentation
- QE Learn / official hands-on materials
- Materials Cloud Quantum ESPRESSO school
- ENCCS Efficient Materials Modelling on HPC QE tutorials
- AiiDA-QE tutorials
- AiiDAlab QE app documentation
- Phonopy QE interface

## 4. 工具名称与生态角色

- ASE：轻量结构与 input/output 脚本层。
- pymatgen：材料结构和分析生态；QE IO 主要依赖外部 `pymatgen-io-espresso`。
- SeeK-path：高对称 k-path 与结构标准化。
- phonopy：有限位移 phonon 生态，与 QE 力计算对接。
- Wannier90：Wannierization、插值和输运分支。
- EPW：电声耦合和 Wannier 插值相关高级 workflow。
- AiiDA / aiida-quantumespresso：workflow、provenance、scheduler、restart。
- AiiDAlab QE app：AiiDA 之上的 GUI workflow。
- VESTA / XCrySDen：结构、体数据、费米面、模式可视化。
- BoltzWann / BoltzTraP2：输运后处理。
- SLURM / PBS：HPC 调度脚本层。
- PWgui / BURAI：GUI input/计算辅助。
- FireWorks / custodian / jobflow：泛用 workflow 或错误处理生态，QE 语义不如 AiiDA-QE 集中。

## 5. QE 原生模块

- Ground state：`pw.x`
- Linear response：`ph.x`
- Phonon interpolation：`q2r.x`、`matdyn.x`、`dynmat.x`
- Electronic post-processing：`bands.x`、`dos.x`、`projwfc.x`、`pp.x`、`fs.x`
- Wannier bridge：`pw2wannier90.x`
- Dynamics：`cp.x`、`cppp.x`
- NEB：`neb.x` / PWneb
- Spectroscopy / response：`turbo_*`、XSPECTRA、GWL
- Transport：PWcond
- Pseudopotential generation：`atomic` / `ld1.x`
- Specialized packages listed by official docs: GIPAW、EPW、WANNIER90 interface、WanT、YAMBO interface、PLUMED、KCW/Koopmans-related workflows、Environ/ESM/RISM-related branches.

## 6. 推荐学习路线抽象

`.docx` 中的路线应改写为能力阶段：

- SCF ground-state loop。
- convergence loop。
- relaxation loop。
- electronic observables loop。
- DFPT phonon loop。
- provenance / HPC / automation loop。

不得按未经验证的时间长度组织。

## 7. 当前仓库尚未覆盖的 QE 功能

详见 [learn/08-feature-expansion-map.md](../learn/08-feature-expansion-map.md)。主要缺口包括 NSCF、smearing、spin/SOC/DFT+U/vdW/hybrid、Fermi surface、work function、Gamma phonon、phonon DOS、dielectric/Born charge、IR/Raman、EPC/EPW、XSpectra、TDDFPT、PWcond、pseudopotential generation/validation 等。

## 8. 适合转化为 GitHub 笔记的内容单元

- Pranab tutorial -> SCF、convergence、relax、bands、DOS、PDOS、phonon、Wannier、SOC、DFT+U 案例页。
- Kyoto DokuWiki -> Gamma phonon、finite-q phonon、q-grid phonon dispersion、phonon DOS、ASR、`epsil`、Born charge、negative frequency 判断。
- Official QE docs -> 每个 workflow 页面末尾的 input reference 和程序边界。
- AiiDA-QE -> provenance、workflow graph、restart、protocol 页面。
- Phonopy QE -> finite-displacement phonon 对照页。
- ASE / pymatgen / SeeK-path -> structure-learning 入口和 bands/structure 标准化引用。
