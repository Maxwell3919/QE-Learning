# Pseudopotentials

## 本页解决什么问题

本页解释 pseudopotential 在 QE 中为什么既是物理模型选择，也是数值收敛边界。它帮助用户判断 `ATOMIC_SPECIES`、`.UPF`、`pseudo_dir` 和 output 中的 pseudopotential loading 信息是否足以支撑 SCF、relax、bands、DOS、phonon、Born charge 或高级 workflow。它不评价某个库对某个具体材料必然更好。

## 最低掌握深度

最低需要知道：

- Pseudopotential 用有效势替代显式 core electrons，只把选定 valence electrons 放入计算。
- PP 的 exchange-correlation functional、valence configuration、relativistic treatment、PP type 和 cutoff 建议会影响所有下游。
- PP transferability 是物理判断风险，不是单纯调大 cutoff 就能消除的问题。
- 不同 PP 不应无记录混用；同一元素的不同 UPF 可能对应不同 valence、XC、relativistic 能力和 recommended cutoff。

## 物理图像

在全电子图像中，原子核附近的 core electrons 变化很快，valence electrons 决定多数化学键、能带和响应性质。Plane-wave 方法若直接表示 core region 的快速振荡，需要极高 cutoff。Pseudopotential 的思路是把 core electrons 和原子核的强势场替换成一个对 valence electrons 有效的势，同时让价电子在核外区域保持正确的散射和成键行为。

这个替换既提高效率，也引入模型边界。Core/valence 分离、valence configuration、semicore 是否进入价层、scalar-relativistic 或 fully relativistic 处理、projector 形式和 exchange-correlation functional 都被写入 UPF 文件。两个 UPF 即使元素相同，也可能定义了不同的有效 Hamiltonian；同一个 cutoff、同一条 k mesh 或同一套下游脚本，在换 PP 后也不再自动等价。

Norm-conserving、ultrasoft 和 PAW 可以理解为不同的 core/valence 表示策略。Norm-conserving 形式较直接但可能需要较高 cutoff；ultrasoft 通过放松范数守恒降低 wavefunction cutoff，同时需要更认真审阅 density cutoff；PAW 保留了重构全电子信息的思想，也带来 projector 和 augmentation 相关边界。QE 用户不需要把它们排成绝对优劣顺序，关键是记录来源、功能一致性和目标性质的收敛证据。

## 最低数学结构

Pseudopotential 把全电子问题中的强 Coulomb 势和 core states 约化为 valence-only Hamiltonian：

```text
all-electron nuclei + core + valence problem
  -> effective ionic potential + selected valence states
  -> plane-wave calculation
```

Transferability 指这个有效势能否在目标化学环境、氧化态、磁性状态、压力、响应计算或相对论条件下仍然保持可靠。它不能由一次 SCF convergence 单独证明；需要结合 UPF metadata、functional consistency、cutoff convergence、目标 output 和必要的来源/对照判断。对 phonon、Born charge、stress、SOC、DFT+U 和 PDOS 这类高风险目标，PP 的 projector、semicore 和 relativistic 信息本身就是 output review 的一部分。

## QE 中的对应对象

| 对象 | QE 位置 | 判断意义 | output 证据 |
|---|---|---|---|
| `pseudo_dir` | `&CONTROL` | 指向 UPF 文件目录 | pseudopotential file path / loading message |
| `ATOMIC_SPECIES` | card | 元素标签、质量和 UPF 文件名 | species label、mass、UPF loaded |
| `.UPF` metadata | pseudo 文件 | XC、valence、PP type、relativistic 信息 | output 中 pseudo type、valence、warnings |
| `input_dft` | `&SYSTEM` | functional 选择或覆盖 | XC 回显、PP/XC mismatch warning |
| `ecutwfc/ecutrho` | `&SYSTEM` | PP 类型和增强电荷的数值边界 | cutoff、FFT grid、force/stress/phonon convergence |
| `noncolin/lspinorb` | `&SYSTEM` | SOC / noncollinear 边界 | relativistic PP capability、spinor output |
| `HUBBARD` card | `pw.x` input | DFT+U 作用通道与 valence/projector 相关 | Hubbard 输出、orbital occupation |

相关 workflow：[SCF](../workflows/ground-state/scf.md)、[cutoff convergence](../workflows/ground-state/cutoff-convergence.md)、[phonon dispersion](../workflows/phonon/phonon-dispersion-dfpt.md)。相关标准：[calculation record template](../standards/calculation-record-template.md)。

## 核心概念

Pseudopotential 把 core electrons 的复杂快速振荡部分替换为更适合 plane-wave basis 的有效描述。这样计算更高效，但代价是把 core-valence separation、valence space、relativistic treatment 和 transferability 假设写进 UPF 文件。PP 改变后，Hamiltonian、能量零点、力、应力、band structure、projection 和 response quantities 都可能改变。

## 对 input 的影响

- `ATOMIC_SPECIES` 中的元素标签必须与 `ATOMIC_POSITIONS` 一致；同一化学元素可有不同标签，但必须有明确理由和记录。
- `pseudo_dir` 和 UPF 文件名不足以证明来源；记录应包含库、版本、URL/DOI、XC、valence、PP type 和 relativistic 信息。
- PP 与 `input_dft`/functional 应保持一致；若 output 有 mismatch warning，应进入 WARN/BLOCK，而不是忽略。
- PP 类型影响 `ecutwfc/ecutrho`，尤其 USPP/PAW 相关 density cutoff、augmentation charge、force/stress 和 phonon。
- SOC、noncollinear、DFT+U、Born charge、PDOS 和 Wannier 等下游都依赖当前 PP 的 valence/projector/relativistic 边界。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| pseudopotential loaded | QE 读取了目标 UPF | UPF 来源可信或适合体系 |
| pseudo type / valence electrons | valence space 和 PP 表示可追踪 | transferability 已验证 |
| XC functional warning | PP 与 `input_dft` 是否可能不一致 | mismatch 可忽略 |
| recommended cutoff / actual cutoff | 数值起点和实际设置可复查 | cutoff 已对目标收敛 |
| relativistic information | SOC/noncollinear 能力边界 | SOC 结果已可信 |
| projector / orbital 信息 | PDOS、DFT+U、Wannier 解释边界 | 投影等于唯一化学分解 |
| semicore / valence configuration | 价层空间和响应自由度可追踪 | transferability 对目标环境已充分 |

## 常见误区

- 只要元素相同就混用 PP。
- functional 与 PP 不一致却不记录。
- 只看 SCF 收敛就认为 PP 合适。
- 换 PP 后不重做 cutoff/k mesh/force/stress/phonon 检查。
- 把 NC、USPP、PAW 当成绝对精度排名。
- SOC 使用不具备相对论能力的 PP。
- DFT+U 通道与 PP valence/projector 不匹配。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | UPF 来源、XC、valence、PP type、relativistic 能力、cutoff policy 和 output loading 都可追踪 | 可进入 SCF、relax、bands、DOS、phonon 等对应 workflow |
| WARN | UPF 可追踪但 transferability 或目标 observable 收敛尚未充分检查 | 可探索，不应给高精度结构、phonon 或响应结论 |
| BLOCK | PP 来源不明、XC mismatch 未解释、元素/valence 错配、SOC/DFT+U 依赖不满足、换 PP 后未重测 | 不允许进入下游性质结论 |

## 下游影响

- `SCF/cutoff`：PP 决定 valence 和 cutoff 需求。
- `relax/vc-relax`：force/stress 受 PP transferability 与 cutoff 影响。
- `bands/DOS/PDOS`：band ordering、gap、projection 和 orbital label 依赖 PP。
- `phonon/DFPT`：force constants、Born charge、dielectric response 对 PP 敏感。
- `SOC/DFT+U/Wannier`：需要确认 relativistic capability、Hubbard manifold 和 projector 边界。

## 与 physics-judgement 的边界

本页只说明 PP 的最低使用和记录要求。以下问题应转到 `physics-judgement/`：

- [pseudopotential transferability and functional consistency](../physics-judgement/pseudopotential-transferability-and-functional-consistency.md)
- [numerical vs model error](../physics-judgement/numerical-vs-model-error.md)
- [SOC and symmetry boundary](../physics-judgement/soc-and-symmetry-boundary.md)
- [U value provenance and boundary](../physics-judgement/u-value-provenance-and-boundary.md)

## 来源与边界

- Stable: `ATOMIC_SPECIES`、`pseudo_dir`、`input_dft`、`ecutwfc/ecutrho` 字段含义以 QE `INPUT_PW` 为准。
- Boundary: PP recommendation 不能替代体系内 convergence 和 output review。
- Internal standard: PP provenance 应写入 [calculation record template](../standards/calculation-record-template.md)。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- QE UPF format notes: <https://pseudopotentials.quantum-espresso.org/home/unified-pseudopotential-format>
- Vanderbilt, Ultrasoft Pseudopotentials.
- Blochl, Projector Augmented-Wave Method.
- 本仓库：[source index](../references/source-index.md)
