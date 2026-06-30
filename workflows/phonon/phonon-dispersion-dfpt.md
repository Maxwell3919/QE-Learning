# Phonon dispersion workflow

## 页面定位

- 对应学习路线：[learn/05-phonon-dfpt-loop.md](../../learn/05-phonon-dfpt-loop.md)
- 对应案例：`cases/si/06-phonon/`、`cases/gaas/03-phonon-dispersion/`
- 结构学习边界：本页要求结构已经充分 relax；2D vacuum、supercell、defect 和结构标准化细节后续见 [structure-learning/README.md](../../structure-learning/README.md)。
- 当前验证状态：保留第一版技术框架；还缺真实 `ph.x/q2r.x/matdyn.x` output、ASR 前后对照和 negative frequency 诊断样例。

## 1. 计算目标

Phonon dispersion workflow 的目标是计算晶体在 Brillouin zone 中不同 q 点上的声子频率和模式，用于判断动力学稳定性、软模、声子带隙、低维材料异常、极性材料 LO-TO splitting 以及后续热学/电声耦合分析的前置条件。

phonon dispersion 不是普通后处理。它依赖 DFPT、严格 SCF、接近平衡的结构和 q-grid interpolation。

## 2. 输入前提

- 结构已经充分 relax；若关心应力，应使用合适的 `vc-relax` 或实验晶格并说明理由。
- SCF cutoff、k 点、smearing 已做过收敛检查。
- phonon 前的 SCF 阈值通常要比普通 bands/DOS 更严格。
- 已决定 Gamma 点 phonon 还是 q-grid phonon dispersion。
- 对 2D、极性、金属、磁性体系已确认特殊处理策略。

## 3. 计算图

q-grid phonon dispersion：

```text
relaxed structure + pseudopotential
  -> pw.x scf
  -> ph.x with ldisp=.true., nq1/nq2/nq3
  -> dynamical matrices (*.dyn*)
  -> q2r.x
  -> real-space force constants (*.fc)
  -> matdyn.x on high-symmetry q path
  -> phonon frequencies / modes / plot data
```

Gamma 点 phonon：

```text
relaxed structure + pseudopotential
  -> pw.x scf
  -> ph.x at q = 0
  -> dynmat.x
  -> Gamma modes / IR-Raman related quantities / LO-TO analysis when valid
```

## 4. 需要的 QE 程序

- `pw.x`：生成 SCF ground state。
- `ph.x`：DFPT 计算 dynamical matrices。
- `q2r.x`：把 q-grid dynamical matrices Fourier transform 为 real-space IFC。
- `matdyn.x`：从 IFC 插值任意 q-path 或 phonon DOS。
- `dynmat.x`：Gamma 点 dynamical matrix 对角化与模式分析。

## 5. 输入文件模板

SCF：

```fortran
&CONTROL
  calculation = 'scf',
  prefix = 'gaas',
  outdir = './out',
  pseudo_dir = './pseudo',
/
&SYSTEM
  ibrav = 0,
  nat = 2,
  ntyp = 2,
  ecutwfc = 60,
  ecutrho = 480,
  occupations = 'fixed',
/
&ELECTRONS
  conv_thr = 1.0d-10,
/
ATOMIC_SPECIES
  Ga 69.723 Ga.upf
  As 74.922 As.upf
ATOMIC_POSITIONS crystal
  ...
K_POINTS automatic
  8 8 8 0 0 0
CELL_PARAMETERS angstrom
  ...
```

`ph.x` q-grid：

```fortran
&INPUTPH
  prefix = 'gaas',
  outdir = './out',
  fildyn = 'gaas.dyn',
  tr2_ph = 1.0d-14,
  ldisp = .true.,
  nq1 = 4,
  nq2 = 4,
  nq3 = 4,
  recover = .true.,
/
```

`q2r.x`：

```fortran
&INPUT
  fildyn = 'gaas.dyn',
  flfrc = 'gaas.fc',
  zasr = 'crystal',
/
```

`matdyn.x`：

```fortran
&INPUT
  asr = 'crystal',
  flfrc = 'gaas.fc',
  flfrq = 'gaas.freq',
  q_in_band_form = .true.,
/
4
  0.0000 0.0000 0.0000 30
  0.5000 0.0000 0.5000 30
  0.5000 0.2500 0.7500 30
  0.3750 0.3750 0.7500 1
```

## 6. 关键参数解释

| 参数 | 所属程序 | 作用 | 错误用法 | 输出中如何验证 |
|---|---|---|---|---|
| `tr2_ph` | `ph.x` | phonon 自洽响应阈值 | 设得过松导致虚频/声学支漂移 | ph output 中每个 perturbation 收敛情况 |
| `ldisp` | `ph.x` | 是否计算 uniform q-grid | 想要 dispersion 却只算 Gamma | output 会列 q 点列表 |
| `nq1/nq2/nq3` | `ph.x` | q-grid 尺寸 | q-grid 太粗直接解释细节 | dyn 文件数量和 q 点覆盖 |
| `recover` | `ph.x` | 中断后恢复 | 不保存 scratch 导致无法续算 | restart 文件与 output |
| `fildyn` | `ph.x/q2r.x` | dynamical matrix 文件前缀 | 与 q2r 输入不一致 | q2r 是否读取全部 dyn |
| `zasr` | `q2r.x` | IFC 级 ASR 处理 | 把 ASR 当作收敛替代品 | q2r output |
| `asr` | `matdyn.x` | 插值时 acoustic sum rule | 无脑选择不适合维度的 ASR | Gamma 附近声学支 |
| `epsil` | `ph.x` | Gamma 极限介电张量/Born charges | 金属或非 Gamma 场景误用 | output 中 dielectric/Born 数据 |

## 7. 输出文件与判断标准

`ph.x`：

- 每个 q 点、不可约扰动是否收敛。
- 是否产生完整 `*.dyn*` 文件。
- 是否有 symmetry、electric field、metal/insulator 相关 warning。

`q2r.x`：

- 是否读取完整 q-grid 的 dynamical matrices。
- 是否成功写出 `*.fc`。
- ASR 设置是否记录清楚。

`matdyn.x`：

- `*.freq` 或 plot 文件是否覆盖目标 q-path。
- Gamma 附近三条声学支是否接近 0。
- 是否出现 negative frequency。

negative frequency 判断：

- 小的 Gamma 附近负频可能来自 ASR、FFT/cutoff/k/q 网格、结构未完全平衡。
- 明显且稳定存在的负频可能是真实动力学不稳定。
- 不能只凭一次粗 q-grid phonon 就宣布材料不稳定或稳定。

## 8. 收敛性要求

phonon 比普通 SCF 更敏感。建议至少检查：

- 结构残余力是否足够小。
- SCF `conv_thr`、`ecutwfc`、`ecutrho` 是否足够严格。
- k 点对 phonon 的影响，金属体系尤其重要。
- q-grid 收敛：`2x2x2`、`3x3x3`、`4x4x4` 或低维体系相应网格。
- smearing 对金属 phonon 的影响。
- 2D 材料是否使用合适的 Coulomb 截断/真空/偶极修正策略。

## 9. 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| `ph.x` 不收敛 | SCF 太松、mixing 问题、金属 smearing 不合适 | 先收紧 SCF 和 ph 阈值，检查 smearing/k 点 |
| Gamma 声学支不为 0 | ASR 破坏、数值误差、结构未平衡 | 比较无 ASR/有 ASR，检查残余力 |
| 大范围虚频 | 真实不稳定或结构/参数严重问题 | 用更严格参数复算关键 q 点 |
| `q2r.x` 报 dyn 不全 | `ph.x` q-grid 未完成或 fildyn 不匹配 | 检查 dyn 文件列表 |
| 极性材料 LO-TO 不对 | `epsil`、Born charges、非解析项处理不完整 | 仅在绝缘体 Gamma 极限使用并记录方向 |
| 2D 材料 Gamma 附近异常 | 周期镜像、Coulomb 长程、真空不足 | 检查 `assume_isolated='2D'` 等策略 |

## 10. 最小可运行案例

建议第一版使用 Si 或 GaAs：

- Si：非极性、适合入门 q-grid phonon。
- GaAs：可引入极性材料、Born effective charge、LO-TO splitting 的后续扩展。

目录建议：

```text
phonon-dispersion/
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

## 11. 与其他 workflow 的关系

- 依赖 SCF 和 relax/vc-relax。
- phonon DOS 是 `matdyn.x` 的另一个输出模式。
- IR/Raman、Born charge、dielectric tensor 是 Gamma DFPT 分支。
- EPC/EPW、热输运、自由能等高级 workflow 依赖更严格 phonon 数据。

## 12. GitHub 记录规范

phonon 记录必须额外写明：

- 结构是否 fully relaxed，残余力是多少。
- SCF 参数与普通电子结构计算相比是否加严。
- q-grid 大小与选择理由。
- ASR 设置：无 ASR、`simple`、`crystal`、低维 ASR 等。
- negative frequency 的判断：数值误差、待复查、还是物理不稳定。
- 是否使用 `epsil`、Born effective charge、non-analytic correction。

## 13. 资料来源

- 已有调研：`Quantum ESPRESSO as a Modern Research Stack.md`
- 已有调研：`Quantum ESPRESSO Tutorial Website Ecosystem.md`
- QE `ph.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE `q2r.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE `matdyn.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE `dynmat.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- Phonopy QE interface: <https://phonopy.github.io/phonopy/qe.html>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- 本轮 `.docx` 来源：`Quantum ESPRESSO as a Modern Research Stack.docx`、`Quantum ESPRESSO Tutorial Website Ecosystem.docx`

## 14. 当前页面还缺哪些验证？

- Si 或 GaAs q-grid phonon 的真实 dyn/fc/freq 文件链。
- Gamma acoustic branch 与 ASR 对照。
- polar GaAs 的 dielectric/Born charge 分支说明。
