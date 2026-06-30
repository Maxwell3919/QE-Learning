# SCF workflow

## 页面定位

- 对应学习路线：[learn/02-first-scf-loop.md](../../learn/02-first-scf-loop.md)
- 结构学习边界：结构操作学习将作为独立项目展开；本页只说明 QE 对结构输入的要求。
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 1. 计算目标

SCF（self-consistent field）计算的目标是在固定原子结构下求解 Kohn-Sham 方程的自洽基态电子密度、总能、势、特征值和后续 workflow 所需的 scratch 数据。

SCF 是 QE 中最核心的数据生成步骤。bands、DOS、PDOS、phonon、charge density、potential 等 workflow 都依赖一个物理合理且数值收敛的 ground state。

## 2. 输入前提

- 已确定结构来源：实验 CIF、数据库结构、手工构建结构或上游优化结构。
- 已检查晶胞、元素、原子坐标、真空层和周期性边界是否符合问题。
- 已选择同一泛函族、同一来源、适合元素价态的赝势。
- 已有初步 cutoff、k 点、occupation/smearing 策略。

## 3. 计算图

```text
structure + pseudopotentials + numerical policy
  -> pw.x scf
  -> converged charge density / total energy / eigenvalues / scratch state
  -> downstream: relax, bands, DOS, PDOS, pp.x, ph.x
```

## 4. 需要的 QE 程序

- 必需：`pw.x`
- 可选后续：`pp.x`、`bands.x`、`dos.x`、`projwfc.x`、`ph.x`

## 5. 输入文件模板

```fortran
&CONTROL
  calculation = 'scf',
  prefix = 'si',
  outdir = './out',
  pseudo_dir = './pseudo',
/
&SYSTEM
  ibrav = 0,
  nat = 2,
  ntyp = 1,
  ecutwfc = 40,
  ecutrho = 320,
  occupations = 'fixed',
/
&ELECTRONS
  conv_thr = 1.0d-8,
  mixing_beta = 0.7,
/
ATOMIC_SPECIES
  Si 28.0855 Si.upf
ATOMIC_POSITIONS crystal
  Si 0.000000 0.000000 0.000000
  Si 0.250000 0.250000 0.250000
K_POINTS automatic
  8 8 8 0 0 0
CELL_PARAMETERS angstrom
  5.430000 0.000000 0.000000
  0.000000 5.430000 0.000000
  0.000000 0.000000 5.430000
```

## 6. 关键参数解释

| 参数 | 作用 | 错误用法 | 输出中如何验证 |
|---|---|---|---|
| `calculation='scf'` | 固定离子位置求自洽电子基态 | 把 SCF 当结构优化 | output 开头会显示 calculation 类型 |
| `prefix` | 标识 scratch 数据和后续读取对象 | 不同 workflow 混用同一 prefix | 下游程序必须使用同一 prefix |
| `outdir` | 保存 charge density、wavefunction 等中间状态 | 删除或混用旧 outdir | output 会打印 temporary directory |
| `pseudo_dir` | 赝势目录 | 赝势文件名存在但来源混乱 | output 会列出 pseudo 文件与类型 |
| `ecutwfc` | 波函数平面波 cutoff | 只抄赝势建议值，不做收敛测试 | output 中有 kinetic-energy cutoff |
| `ecutrho` | 电荷密度/势 cutoff | 对 USPP/PAW 设得过低 | output 中有 charge density cutoff |
| `occupations` / `smearing` / `degauss` | 控制占据方式，金属体系尤其关键 | 绝缘体/金属策略混用 | output 中有 occupation scheme 和 Fermi energy |
| `conv_thr` | 电子自洽阈值 | 过松导致下游 phonon/力不可信 | SCF iteration 的 estimated accuracy |
| `mixing_beta` | 电荷混合强度 | 金属/大胞中设太大导致振荡 | SCF 是否来回震荡 |
| `K_POINTS` | Brillouin zone 采样 | 用 bands path 做 SCF | output 中有 irreducible k-points |

## 7. 输出文件与判断标准

必须检查：

- 是否有 `JOB DONE`，但这只是程序完成，不等于科学可信。
- SCF 是否达到 `convergence has been achieved`。
- 最后几步 total energy 是否稳定。
- `estimated scf accuracy` 是否低于设定阈值。
- k 点、cutoff、赝势、occupation 是否与 input 预期一致。
- Fermi energy 是否合理；金属体系是否需要 smearing。
- 是否出现明显 warning，例如 charge not converged、wrong pseudo、symmetry 异常。

可记录的最小摘要：

```markdown
- Final total energy:
- SCF iterations:
- Estimated SCF accuracy:
- Fermi energy:
- Cutoff:
- K mesh:
- Pseudopotentials:
- Status: PASS / WARN / BLOCK
```

## 8. 收敛性要求

SCF 本身收敛不等于参数收敛。至少需要：

- `ecutwfc` 收敛测试：观察总能、力、应力或目标性质。
- `ecutrho` 收敛测试：尤其 USPP/PAW、phonon、应力敏感任务。
- k 点收敛测试：金属、低维体系和 DOS 更敏感。
- smearing 测试：金属中比较 `degauss` 对能量和力的影响。

phonon 前的 SCF 通常应比普通电子结构图像更严格，例如更小 `conv_thr`、更充分的 cutoff/k 点验证。

## 9. 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| SCF 不收敛 | `mixing_beta` 太大、金属 smearing 不合适、结构太差、k 点/cutoff 问题 | 先看 energy/accuracy 是否振荡，再调 mixing 和 smearing |
| 能量看似收敛但力很差 | cutoff/k 点不足，结构远离平衡 | 做 relax 或收敛性测试 |
| 下游程序找不到数据 | `prefix/outdir` 不一致或 scratch 被删 | 保持同一 prefix/outdir，记录路径 |
| 结果与教程差异大 | 赝势、晶格常数、k 点、QE 版本不同 | 记录所有输入来源，不只比较数值 |

不能靠调参数解决的情况：

- 结构本身错误，如元素、坐标、晶胞、真空层不对。
- 赝势与目标物理不兼容。
- 金属/绝缘体物理判断错误。

## 10. 最小可运行案例

第一版建议使用 Si diamond 或 Al fcc：

- Si：适合理解 fixed occupations、band gap、bands/DOS。
- Al：适合理解 metallic smearing 和 k 点敏感性。

后续补充要求：

- 提供 input/output 文件。
- 记录 QE 版本、赝势来源、命令和运行环境。
- 附至少一组 cutoff 或 k 点收敛表。

## 11. 与其他 workflow 的关系

- `relax` / `vc-relax`：离子优化内部会反复调用电子 SCF。
- `bands`：需要先有 SCF ground state，再沿 k-path 做 bands 计算。
- `DOS/PDOS`：通常需要 SCF 后接 dense k-mesh NSCF。
- `phonon`：`ph.x` 直接依赖 SCF 产生的数据，且对 SCF 质量更敏感。

## 12. GitHub 记录规范

每个 SCF 目录至少保留：

```text
pw.scf.<system>.in
pw.scf.<system>.out
record.md
pseudo/ 或 pseudo-source.md
README.md
```

`record.md` 必须写清楚结构来源、赝势来源、命令、环境、收敛状态、是否允许下游继续。

## 13. 资料来源

- 已有调研：`Quantum ESPRESSO as a Modern Research Stack.md`
- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- Pranab Das SCF tutorial: <https://pranabdas.github.io/espresso/hands-on/scf>

## 14. 后续完善重点

- Si 学习示例 `pw.scf.Si.in/out`。
- Al 学习示例 metallic SCF + smearing 示例。
- 对 output 中 total energy、SCF accuracy、Fermi energy、cutoff、k-points、pseudopotential 段落的逐项标注。
