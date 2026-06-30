# Pseudopotentials

## QE 中对应的问题

赝势决定 QE 在平面波基组中实际求解的价电子问题。对 `pw.x` 来说，赝势文件同时固定了：

- 元素标签和价电子数；
- 交换关联泛函生成条件；
- norm-conserving、ultrasoft 或 PAW 的数据结构；
- scalar-relativistic 或 fully relativistic 能力；
- 推荐或最低可用的 `ecutwfc` / `ecutrho` 起点；
- DFT+U、SOC、vdW、DFPT、后处理等高级 workflow 的前置兼容性。

因此 pseudopotential review 要回答的是：当前 input 指向的 UPF 文件是否与本次计算的泛函、物理开关、cutoff policy 和下游 workflow 一致。本仓库不对具体赝势库做排名；学习记录必须能回溯到文件来源、版本、元信息和输出回显。

在 QE input 中，赝势通过 `pseudo_dir` 和 `ATOMIC_SPECIES` 进入计算；在 output 中，QE 会回显实际加载的 pseudopotential 文件、pseudo type、valence electrons、cutoff/mesh 相关信息和可能的 warning。只有 input 预期、UPF 元信息和 output 回显三者一致，后续 SCF、relax、phonon 或电子结构结果才有可审阅基础。

## 输入字段表

| 字段 | 所属位置 | 作用 | 需要成组固定/记录的对象 | output 中如何复核 | 常见误用 |
|---|---|---|---|---|---|
| `pseudo_dir` | `&CONTROL` / `pw.x` | 指向 UPF 文件所在目录 | 目录路径、文件来源、库/版本、下载 URL 或 DOI、校验方式 | output 开头的 pseudo 文件读取信息；运行目录和 temporary directory | 目录中有同名文件但来源不同；只记录本地路径不记录来源 |
| `ATOMIC_SPECIES` | card / `pw.x` | 建立元素标签、原子质量和 UPF 文件映射 | 元素标签、质量、UPF 文件名、结构中的原子标签 | output 中每种 species 加载的 pseudopotential、atomic species 信息 | 元素标签与 `ATOMIC_POSITIONS` 不一致；复制 input 后文件名未改 |
| `input_dft` | `&SYSTEM` / `pw.x` | 显式指定交换关联泛函；通常应与赝势生成泛函一致 | UPF header 中的 XC 信息、所有 species 的泛函族、是否混入 dispersion/hybrid 处理 | output 中 exchange-correlation functional 回显和相关 warning | PBE、PBEsol、LDA 等混用；手动设 `input_dft` 覆盖了 pseudo 元信息但未说明 |
| `ecutwfc` | `&SYSTEM` / `pw.x` | 波函数平面波 cutoff；受赝势硬度、价电子和 PP 类型影响 | pseudo 文件、`ecutrho`、k mesh、smearing、目标性质、收敛表 | `kinetic-energy cutoff`；SCF 是否收敛；目标量随 cutoff 的变化 | 直接照搬推荐值；换赝势后沿用旧 cutoff |
| `ecutrho` | `&SYSTEM` / `pw.x` | 电荷密度和势 cutoff；USPP/PAW 通常需要更高密度 cutoff | `ecutwfc`、PP type、FFT grid、force/stress/phonon 目标 | `charge density cutoff`、FFT grid、smooth grid、force/stress 噪声 | 认为省略即可；只收敛 `ecutwfc` 不复查 `ecutrho` |
| `noncolin` / `lspinorb` | `&SYSTEM` / `pw.x` | 启用非共线自旋和 SOC | fully relativistic UPF、初始磁性设置、对称性、下游 bands/phonon 可比性 | output 中 noncollinear/SOC 回显、spin-orbit 相关信息、pseudo relativistic 类型 | SOC 用 scalar-relativistic 赝势；开 SOC 后沿用非 SOC 收敛结论 |
| `nspin` / `starting_magnetization` | `&SYSTEM` / `pw.x` | 自旋极化和初始磁矩；依赖价电子空间和磁性通道 | UPF valence 配置、初始磁矩、occupation/smearing、最终磁矩 | total/absolute magnetization、atomic magnetic moments、SCF 收敛 | 未记录初始条件；把不同 pseudo 下磁矩直接比较 |
| `HUBBARD` card | `pw.x` | 指定 DFT+U 通道、投影和 U/J 参数 | 元素价层、投影通道、U 值来源、是否更换 pseudo | Hubbard 参数回显、投影通道、SCF 稳定性 | U 通道与 pseudo valence 不匹配；换 pseudo 后不复查 U 和收敛 |
| `vdw_corr` 等 vdW 相关字段 | `&SYSTEM` / `pw.x` | 添加色散修正或非局域 vdW 泛函 | 基础 XC、赝势 XC、是否使用非局域相关项、下游可比性 | output 中 XC/vdW 设置回显 | 把 vdW 当作后处理标签；与 pseudo 泛函边界不清 |

## UPF 元信息最低要读什么

UPF 文件名只能作为线索，不能替代元信息审阅。每次计算至少要能从 UPF header 或来源页面记录以下内容：

- 元素和 species 标签：确认与 `ATOMIC_SPECIES`、`ATOMIC_POSITIONS` 使用同一标签体系。
- 交换关联泛函：确认所有元素使用同一泛函族，且与 `input_dft`、vdW 或 hybrid 设置的边界一致。
- valence electrons：确认 QE output 回显的价电子数与预期一致；总价电子数会影响 occupation、磁矩和带填充判断。
- PP type：区分 NC、USPP、PAW；这会改变 `ecutrho`、augmentation charge、FFT grid、force/stress 和 DFPT 审阅重点。
- relativistic 信息：区分 non-relativistic、scalar-relativistic 和 fully relativistic；SOC/noncollinear 计算必须单独复核。
- 推荐 cutoff 或测试说明：只能作为 scan 起点；不能替代本 workflow 的 `ecutwfc` / `ecutrho` 收敛表。
- 生成器、版本和来源：记录库名、版本、发布日期或 DOI/URL；本地文件重命名后仍要能追溯来源。

## NC / USPP / PAW 的 workflow 含义

| 类型 | QE 中的含义 | input/output 重点 | 常见误判 |
|---|---|---|---|
| NC | norm-conserving pseudopotential；通常形式更简单但可能需要较高 `ecutwfc` | 重点看 `ecutwfc` 收敛、valence 配置和 relativistic 能力 | 只因为 `ecutrho` 比例较低就认为总体更便宜 |
| USPP | ultrasoft pseudopotential；通过增强电荷降低波函数 cutoff，但密度 cutoff 更敏感 | 同时审阅 `ecutwfc`、`ecutrho`、augmentation charge、force/stress | 只收敛总能，不复查 `ecutrho` 对力和应力的影响 |
| PAW | projector augmented-wave 数据；在 QE 中也依赖 augmentation 相关信息 | 复核 PAW type、`ecutrho`、投影通道、DFPT/后处理兼容性 | 把 PAW 与 NC 使用同一 cutoff policy；忽略下游程序支持边界 |

这些类型不是精度排名。它们改变的是数值表示、cutoff 需求和 output review 的敏感点。选择任一类型后，都必须用目标 workflow 的收敛证据支持，而不是用类型名称本身支持结论。

## cutoff 建议如何使用

赝势来源页或 UPF 元信息中的 cutoff 建议只适合作为初始测试点。最低做法是：

- 先按来源建议设定第一组 `ecutwfc` / `ecutrho`，确认 input 能正常运行并加载预期 UPF。
- 固定结构、k mesh、smearing、电子收敛阈值和赝势文件，单独做 cutoff convergence。
- SCF 探索可先看 total energy，但进入 relax、vc-relax、phonon、弹性、压力或响应性质前，必须看 force、stress 或目标响应量。
- USPP/PAW 需要特别复查 `ecutrho`；密度 cutoff 不足可能在 total energy 上不明显，却污染 force、stress、phonon 或电荷密度后处理。
- 换赝势、换 valence 配置、换泛函、开 SOC、开 DFT+U 或改变下游目标，都应视为新的数值体系，不能直接继承旧 cutoff 表。

PASS / WARN / BLOCK 判断应写成目标量可复查的记录，而不是写成“按推荐 cutoff 已足够”。

## output review

### 基本一致性

- output 是否实际读取了 `pseudo_dir` 中预期的 UPF 文件，而不是同名旧文件或错误目录文件。
- 每个 `ATOMIC_SPECIES` 的元素标签、质量、UPF 文件名是否与 input 和结构标签一致。
- output 中回显的 pseudo type、valence electrons、relativistic 信息是否与 UPF 元信息和计算目标一致。
- output 中 exchange-correlation functional 是否与所有 UPF 文件的生成泛函一致；若出现 warning，应进入记录并解释。
- `ecutwfc`、`ecutrho`、FFT grid 和 smooth grid 是否与当前 pseudo 类型和收敛策略一致。

### 高级依赖

- SOC / noncollinear：确认 `noncolin` / `lspinorb` 与 fully relativistic 赝势匹配，并重新审阅 k mesh、cutoff、磁矩和下游 bands 可比性。
- DFT+U：确认 Hubbard 通道存在于当前 valence 空间，U/J 参数来源明确；更换 pseudo 后不能自动沿用旧 U 结论。
- vdW / dispersion：确认基础 XC 与 pseudo 泛函边界写清楚；vdW 设置改变能量、力和结构优化可比性。
- DFPT / phonon：确认 pseudo 类型、cutoff、SCF 阈值、结构残余力和 q 点 workflow 均满足 response 计算要求；不要只用普通 SCF 通过作为 phonon 准入。
- 后处理：bands、DOS、PDOS、charge density、Wannier 等步骤必须引用同一 pseudo/cutoff/provenance 记录，否则结果不可直接比较。

### PASS / WARN / BLOCK

- PASS：UPF 来源、版本、XC、valence、PP type、relativistic 能力、cutoff scan 和 output 回显均可复核；下游 workflow 所需的高级依赖已匹配。
- WARN：来源和 output 基本一致，但只有推荐 cutoff、缺少 force/stress/phonon 目标量收敛，或高级开关的重新收敛尚未完成；可用于探索，不应用于定稿判断。
- BLOCK：泛函混用无解释、元素/价电子不匹配、SOC 赝势能力不足、UPF 来源不可追溯、换赝势后未重做收敛，或 output warning 指出 pseudo/XC/读取问题。

## 常见错误

| 错误 | 为什么危险 | 应如何修正 |
|---|---|---|
| 只记录 UPF 文件名 | 文件名可能被重命名，同名文件也可能来自不同版本 | 记录来源页、库/版本、下载日期或 DOI/URL，并保留 output 回显 |
| 混用 PBE、PBEsol、LDA 等泛函族 | 总能、力、结构和电子结构不再属于同一个 Hamiltonian | 所有 species 使用同一 XC 边界；若有特殊设置，写明理由和来源 |
| 把 `input_dft` 当作万能修正 | `input_dft` 不能把不一致的赝势变成一致数据 | 优先选用与目标泛函一致的 UPF；output warning 必须处理 |
| 忽略 valence electrons | 价电子数影响带填充、磁矩、投影和 DFT+U 通道 | 从 UPF 和 output 同时记录每种元素 valence，计算总价电子数 |
| 把 NC、USPP、PAW 当作精度排名 | 类型不同代表表示方式和 cutoff 需求不同，不是自动优劣 | 按目标性质做 convergence 和 output review |
| 直接使用推荐 cutoff | 推荐值只是起点，不能证明当前结构和目标性质已收敛 | 建立 `ecutwfc` / `ecutrho` scan，并用目标量给出 PASS/WARN/BLOCK |
| 换赝势后不重做收敛 | valence、投影子、augmentation charge 和推荐 cutoff 都可能改变 | 把换 pseudo 视作新数值设定，重新跑 cutoff/k 点/目标性质检查 |
| SOC 使用非 fully relativistic 赝势 | Hamiltonian 和赝势能力不匹配，SOC 结果不可用 | 先确认 UPF relativistic 信息，再启用 `noncolin` / `lspinorb` |
| DFT+U 通道与 pseudo 不匹配 | U 作用的轨道可能不在当前 valence 或投影定义不一致 | 复核 valence 配置、Hubbard card、U 值来源和 output 回显 |
| vdW/hybrid/高级开关沿用旧记录 | 高级开关改变能量、力、结构和可比性 | 重新记录 XC 边界、cutoff、k mesh 和下游准入 |

## 最低掌握深度

读完本页后，最低应能完成以下判断：

- 能在 `pw.x` input 中指出 `pseudo_dir`、`ATOMIC_SPECIES`、`input_dft`、`ecutwfc`、`ecutrho`、`noncolin/lspinorb`、`HUBBARD` card 与赝势的联动关系。
- 能从 UPF 或来源页记录元素、XC、valence electrons、PP type、relativistic 能力、推荐 cutoff、版本和来源。
- 能从 output 复核实际加载的 pseudopotential、valence、pseudo type、XC 回显、cutoff、FFT grid 和 warning。
- 能解释为什么换赝势、换泛函、开 SOC、开 DFT+U、开 vdW 或进入 phonon/DFPT 后，需要重新审阅收敛和下游可比性。
- 能把赝势问题写成 PASS / WARN / BLOCK，而不是写成“文件能跑所以可用”。

## 对应 workflow

- [SCF workflow](../workflows/ground-state/scf.md)
- [Cutoff convergence workflow](../workflows/ground-state/cutoff-convergence.md)
- [K-point convergence workflow](../workflows/ground-state/kpoint-convergence.md)
- [Relax workflow](../workflows/ground-state/relax.md)
- [Phonon dispersion DFPT workflow](../workflows/phonon/phonon-dispersion-dfpt.md)
- [Spin polarization workflow](../workflows/advanced/spin-polarization.md)
- [Noncollinear SOC workflow](../workflows/advanced/noncollinear-soc.md)
- [DFT+U workflow](../workflows/advanced/dft-plus-u.md)
- [vdW corrections workflow](../workflows/advanced/vdW-corrections.md)
- [Pseudopotential generation overview](../workflows/advanced/pseudopotential-generation-overview.md)

## 资料来源

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- QE UPF format notes: <https://pseudopotentials.quantum-espresso.org/home/unified-pseudopotential-format>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
