# QE 工作流生态地图

## 1. 为什么不写成工具大全？

QE 不是单个可执行文件，而是围绕 ground state、response、post-processing 和 workflow provenance 展开的研究栈。学习时应先问“这个工具插入 workflow 的哪里”，再问“它有哪些命令”。

本页基于两份本地调研材料整理：

- `Quantum ESPRESSO as a Modern Research Stack.md`
- `Quantum ESPRESSO Tutorial Website Ecosystem.md`

## 2. 原生 QE 模块

| 工具 | 解决什么问题 | 插入 workflow 的位置 | 初学必需 | 适合自动化 | 常见替代/补充 | 学习优先级 |
|---|---|---|---|---|---|---|
| `pw.x` | SCF、NSCF、bands、relax、vc-relax、MD、力和应力 | 所有主线 workflow 的 ground-state 引擎 | 是 | 是 | AiiDA `PwCalculation` 封装 | 最高 |
| `ph.x` | DFPT phonon、介电张量、Born effective charges、部分响应性质 | 高质量 SCF 之后 | phonon 学习必需 | 是，但耗时且需 restart | phonopy finite displacement | 高 |
| `dos.x` | 从 NSCF/SCF 数据计算总 DOS | 电子结构分支 | 是 | 是 | Python 自写积分/绘图，sumo 部分场景 | 高 |
| `bands.x` | 处理 band path eigenvalues、排序、输出绘图数据 | bands 分支 | 是 | 是 | AiiDA bands workchain、Python 绘图 | 高 |
| `projwfc.x` | 投影态密度、Lowdin charge、轨道贡献 | PDOS 分支 | 是 | 是 | Wannier90 投影分析 | 高 |
| `pp.x` | 电荷密度、势、ELF 等实空间后处理 | SCF/NSCF 后处理 | 中 | 是 | VESTA/XCrySDen/Python 可视化 | 中 |
| `q2r.x` | dynamical matrices -> real-space IFC | q-grid phonon 中间步骤 | phonon dispersion 必需 | 是 | phonopy 的 force constants 流程 | 高 |
| `matdyn.x` | IFC -> phonon dispersion / phonon DOS | `q2r.x` 之后 | phonon dispersion 必需 | 是 | phonopy band/dos | 高 |
| `dynmat.x` | Gamma 点 dynamical matrix 对角化、模式分析、ASR/非解析项相关处理 | Gamma phonon / IR/Raman 分析 | 中 | 是 | matdyn.x 部分场景 | 中 |

核心依赖图：

```text
structure + pseudopotential
  -> pw.x scf
  -> pw.x relax/vc-relax
  -> pw.x scf
     ├─> pw.x bands -> bands.x
     ├─> pw.x nscf -> dos.x / projwfc.x
     ├─> pp.x
     └─> ph.x -> q2r.x -> matdyn.x
```

## 3. 外部工具地图

| 工具 | 解决什么问题 | 插入 QE workflow 的位置 | 初学必需 | 适合自动化 | 常见替代工具 | 学习优先级 |
|---|---|---|---|---|---|---|
| ASE | 结构构建、格式转换、批量生成 input、轻量脚本 | 结构准备、参数扫描、结果解析辅助 | 推荐 | 是 | pymatgen、手写脚本 | 高 |
| pymatgen | 材料结构操作、对称性、材料数据库生态 | 结构标准化、超胞、缺陷前处理 | 非必需 | 是 | ASE、spglib | 中 |
| seekpath | 标准化 primitive cell 与高对称 k-path | bands workflow 的路径生成 | 推荐 | 是 | AFLOW labels、Bilbao 工具 | 高 |
| phonopy | finite-displacement phonon、力常数、声子后处理 | 替代或补充 DFPT phonon | 非初学必需 | 是 | QE `ph.x/q2r/matdyn` | 中 |
| Wannier90 | Wannierization、能带插值、输运前处理 | 高级电子结构分支 | 后置 | 是 | 直接 bands/DOS，BoltzTraP2 | 中-高 |
| AiiDA | provenance、workflow、scheduler、restart、数据库 | 研究级自动化与高通量 | 后置 | 强 | FireWorks、jobflow、自写调度 | 高，但不作为第一周工具 |
| AiiDAlab | AiiDA 之上的 GUI/app 层 | 教学、交互式 workflow 管理 | 非必需 | 是 | 命令行 AiiDA、自写 notebook | 中 |
| sumo | 能带/DOS 绘图工具，VASP 生态更强 | 图像生成，可作为参考 | 非必需 | 部分 | pymatgen/matplotlib | 低-中 |
| VESTA | 结构、体数据、charge density 可视化 | 结构检查和 volumetric data 检查 | 推荐 | 弱 | XCrySDen、OVITO | 高 |
| XCrySDen | 结构、费米面、AXSF 模式等可视化 | QE 传统可视化出口 | 推荐 | 弱 | VESTA、Python | 中 |
| HPC / SLURM scripts | 资源申请、并行运行、日志和 restart | 所有真实大计算 | 必需于集群 | 是 | AiiDA scheduler | 高 |

## 4. 学习优先级

第一阶段：

- QE native：`pw.x`、`bands.x`、`dos.x`、`projwfc.x`
- 外部：VESTA 或 XCrySDen，基础 shell/SLURM

第二阶段：

- QE native：`ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x`、`pp.x`
- 外部：ASE、seekpath、matplotlib

第三阶段：

- AiiDA/AiiDAlab、Wannier90、phonopy、BoltzTraP2、缺陷/超胞工具

## 5. 资料链接

- QE documentation: <https://www.quantum-espresso.org/documentation/>
- QE input docs: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- PH input docs: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- ASE Espresso docs: <https://wiki.fysik.dtu.dk/ase/ase/calculators/espresso.html>
- AiiDA Quantum ESPRESSO plugin: <https://aiida-quantumespresso.readthedocs.io/>
- SeeK-path: <https://seekpath.readthedocs.io/>
- Phonopy QE interface: <https://phonopy.github.io/phonopy/qe.html>
- Wannier90: <https://wannier.org/>

