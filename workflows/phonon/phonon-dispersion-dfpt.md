# Phonon dispersion workflow

## 页面定位

- 对应学习路线：[learn/06-phonon-dfpt-loop.md](../../learn/06-phonon-dfpt-loop.md)
- 结构输入边界：本页要求结构已经充分优化；结构构建与标准化在独立项目中展开。
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

Phonon dispersion workflow 计算晶体在 Brillouin zone 不同 q 点上的声子频率和模式，用于判断动力学稳定性、软模、声子带隙、低维材料异常、极性材料 LO-TO splitting，以及热学和电声耦合分析的前置数据质量。

phonon 属于 DFPT response workflow，对 SCF、结构优化和收敛性要求高于普通电子结构后处理。

## 输入前提

- `<structure>` 已充分优化；若使用固定晶格或实验晶格，应记录理由。
- SCF 的 `<ecutwfc>`、`<ecutrho>`、`<k_mesh>`、smearing 已做过收敛检查。
- phonon 前的 SCF 阈值通常应比普通 bands/DOS 更严格。
- 已决定 Gamma phonon、q-grid dispersion、phonon DOS 或 dielectric/Born effective charge 分支。
- 对 2D、极性、金属、磁性体系已确认特殊处理策略。

## 计算图

q-grid phonon dispersion：

```text
<relaxed_structure> + <pseudo>
  -> pw.x scf
  -> ph.x with ldisp=.true., nq1/nq2/nq3
  -> dynamical matrices (<system>.dyn*)
  -> q2r.x
  -> real-space force constants (<system>.fc)
  -> matdyn.x on <q_path>
  -> phonon frequencies / modes / plot data
```

Gamma phonon：

```text
<relaxed_structure> + <pseudo>
  -> pw.x scf
  -> ph.x at q = 0
  -> dynmat.x
  -> Gamma modes / dielectric-Born branch when valid
```

## 需要的 QE 程序

- `pw.x`：生成 SCF ground state。
- `ph.x`：DFPT 计算 dynamical matrices。
- `q2r.x`：把 q-grid dynamical matrices Fourier transform 为 real-space IFC。
- `matdyn.x`：从 IFC 插值任意 q-path 或 phonon DOS。
- `dynmat.x`：Gamma 点 dynamical matrix 对角化与模式分析。

## 通用输入模板

SCF：

```fortran
&CONTROL
  calculation = 'scf',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
/
&SYSTEM
  ibrav = 0,
  nat = <number_of_atoms>,
  ntyp = <number_of_species>,
  ecutwfc = <wavefunction_cutoff>,
  ecutrho = <charge_density_cutoff>,
  occupations = '<occupation_scheme>',
/
&ELECTRONS
  conv_thr = <strict_scf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>

ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>

K_POINTS automatic
  <nk1> <nk2> <nk3> <sk1> <sk2> <sk3>

CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

`ph.x` q-grid：

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  fildyn = '<system>.dyn',
  tr2_ph = <phonon_threshold>,
  ldisp = .true.,
  nq1 = <nq1>,
  nq2 = <nq2>,
  nq3 = <nq3>,
  recover = .true.,
/
```

`q2r.x`：

```fortran
&INPUT
  fildyn = '<system>.dyn',
  flfrc = '<system>.fc',
  zasr = '<asr_scheme>',
/
```

`matdyn.x`：

```fortran
&INPUT
  asr = '<asr_scheme>',
  flfrc = '<system>.fc',
  flfrq = '<system>.freq',
  q_in_band_form = .true.,
/
<number_of_q_path_points>
  <qx_1> <qy_1> <qz_1> <npoints_to_next>
  <qx_2> <qy_2> <qz_2> <npoints_to_next>
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `tr2_ph` | `ph.x` | phonon 自洽响应阈值 | 设得过松导致虚频或声学支漂移 | ph output 中每个 perturbation 收敛情况 |
| `ldisp` | `ph.x` | 是否计算 uniform q-grid | 想要 dispersion 却只算 Gamma | output 会列 q 点列表 |
| `nq1/nq2/nq3` | `ph.x` | q-grid 尺寸 | q-grid 太粗直接解释细节 | dyn 文件数量和 q 点覆盖 |
| `recover` | `ph.x` | 中断后恢复 | 不保存 scratch 导致无法续算 | restart 文件与 output |
| `fildyn` | `ph.x/q2r.x` | dynamical matrix 文件前缀 | 与 q2r 输入不一致 | q2r 是否读取全部 dyn |
| `zasr` | `q2r.x` | IFC 级 ASR 处理 | 把 ASR 当作收敛替代品 | q2r output |
| `asr` | `matdyn.x` | 插值时 acoustic sum rule | 不分析维度就套用 | Gamma 附近声学支 |
| `epsil` | `ph.x` | Gamma 极限介电张量/Born effective charges | 金属或非 Gamma 场景误用 | output 中 dielectric/Born 数据 |

## 通用输出审阅模板

```markdown
## Output Review

- Program chain:
- QE version:
- Structure relaxation status:
- SCF dependency:
- SCF threshold:
- Cutoff reported:
- K-points reported:
- ph.x q-grid:
- ph.x perturbation convergence:
- Dynamical matrix files:
- q2r.x force-constant file:
- ASR setting:
- matdyn.x q-path:
- Frequency file:
- Gamma acoustic branches:
- Negative frequencies:
- Dielectric tensor / Born effective charge branch:
- Warnings:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

## 输出判断标准

`ph.x`：

- 每个 q 点、不可约扰动是否收敛。
- 是否产生完整 `<system>.dyn*` 文件。
- 是否有 symmetry、electric field、metal/insulator 相关 warning。

`q2r.x`：

- 是否读取完整 q-grid 的 dynamical matrices。
- 是否成功写出 `<system>.fc`。
- ASR 设置是否记录清楚。

`matdyn.x`：

- `<system>.freq` 是否覆盖目标 `<q_path>`。
- Gamma 附近三条声学支是否接近 0。
- negative frequency 的位置、大小和参数敏感性是否已记录。

negative frequency 判断：

- 小的 Gamma 附近负频可能来自 ASR、FFT/cutoff/k/q 网格、结构残余力或插值误差。
- 明显且稳定存在的负频可能是真实动力学不稳定。
- 一次粗 q-grid phonon 只能给出复查线索，不能直接作为最终稳定性结论。

## 收敛性要求

- 结构残余力和应力应满足 phonon 目标精度。
- SCF `conv_thr`、`ecutwfc`、`ecutrho` 应比普通电子结构后处理更严格。
- k 点对 phonon 的影响需要复查，金属体系尤其敏感。
- q-grid 需要针对目标性质收敛，而不是只依赖默认网格。
- smearing 对金属 phonon 的影响需要记录。
- 2D 材料需要检查真空、长程库仑相互作用和边界条件设置。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| `ph.x` 不收敛 | SCF 太松、mixing 问题、smearing 不合适 | 收紧 SCF/ph 阈值，检查 smearing/k 点 |
| Gamma 声学支不为 0 | ASR、数值误差、结构未平衡 | 比较无 ASR/有 ASR，检查残余力 |
| 大范围虚频 | 真实不稳定或结构/参数严重问题 | 用更严格参数复算关键 q 点 |
| `q2r.x` 报 dyn 不全 | `ph.x` q-grid 未完成或 fildyn 不匹配 | 检查 dyn 文件列表 |
| LO-TO 分裂异常 | `epsil`、Born effective charges、非解析项处理不完整 | 仅在有效物理条件下使用并记录方向 |
| 2D Gamma 附近异常 | 周期镜像、长程库仑、真空不足 | 检查 2D 边界处理策略 |

## 通用学习模板

学习时应选择结构简单、已充分优化、赝势来源可靠、物理状态清楚的体系，并完整记录 SCF 参数、q-grid、ASR、频率文件和 negative frequency 判断。具体材料体系不写入本仓库主文档。

个人学习记录中应保留 input、output、运行命令、dyn/fc/freq 文件路径和 output review。

## 记录模板

```text
pw.scf.<system>.in
pw.scf.<system>.out
ph.<system>.in
ph.<system>.out
q2r.<system>.in
q2r.<system>.out
matdyn.<system>.in
matdyn.<system>.out
record.md
```

`record.md` 必须额外写明结构优化状态、SCF 参数是否加严、q-grid 大小、ASR 设置、negative frequency 判断、是否使用 `epsil`、Born effective charge 和 non-analytic correction。

## 与其他 workflow 的关系

- 依赖 SCF 和 relax/vc-relax。
- phonon DOS 是 `matdyn.x` 的另一个输出模式。
- IR/Raman、Born effective charge、dielectric tensor 是 Gamma DFPT 分支。
- EPC/EPW、热输运、自由能等高级 workflow 依赖更严格 phonon 数据。

## 后续完善重点

- 补充 Gamma acoustic branch 与 ASR 的通用审阅规范。
- 补充 `epsil`、Born effective charge、dielectric tensor 的适用条件。
- 补充 negative frequency 的参数复查流程。

## 资料来源

- QE `ph.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE `q2r.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE `matdyn.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE `dynmat.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- Phonopy QE interface: <https://phonopy.github.io/phonopy/qe.html>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
