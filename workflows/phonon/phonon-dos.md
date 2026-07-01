# phonon DOS workflow

## 页面定位

- 对应学习路线：[learn/06-phonon-dfpt-loop.md](../../learn/06-phonon-dfpt-loop.md)
- 上游依赖：q-grid phonon dynamical matrices
- 下游用途：phonon density of states
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

从 q-grid dynamical matrices 生成 real-space force constants，再由 `matdyn.x` 在 DOS q mesh 上计算 phonon DOS。

## 输入前提

- 结构已充分优化，并通过 SCF output review。
- SCF 的 cutoff、k 点、smearing 和 `conv_thr` 已按 phonon 目标审阅。
- 已明确该页是 Gamma phonon、q-grid、DOS、响应张量还是 debugging 分支。

## 计算图

```text
pw.x scf
  -> ph.x ldisp=.true. on <q_grid>
  -> q2r.x force constants
  -> matdyn.x dos mode
  -> phonon DOS
```

## 需要的 QE 程序

`ph.x`、`q2r.x`、`matdyn.x`

## 通用输入模板

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  tr2_ph = <phonon_threshold>,
  ldisp = .true.,
  nq1 = <nq1>, nq2 = <nq2>, nq3 = <nq3>,
  fildyn = '<system>.dyn',
/

&INPUT
  fildyn = '<system>.dyn',
  flfrc = '<system>.fc',
  zasr = '<asr_scheme>',
/

&INPUT
  flfrc = '<system>.fc',
  asr = '<asr_scheme>',
  dos = .true.,
  nk1 = <dos_nk1>, nk2 = <dos_nk2>, nk3 = <dos_nk3>,
  fldos = 'phonon-dos.<system>.dat',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `ldisp/nq1/nq2/nq3` | ph.x | 生成 uniform q-grid dyn 文件 | q-grid 不完整 | ph.x q 点列表和 dyn 文件 |
| `fildyn` | ph.x/q2r.x | dyn 文件前缀 | q2r 读取不全 | q2r.x output |
| `flfrc` | q2r.x/matdyn.x | force constants 文件 | matdyn 读错 fc | matdyn.x output |
| `dos/nk1/nk2/nk3` | matdyn.x | phonon DOS mesh | mesh 太粗导致 DOS 不稳 | phonon DOS 输出 |
| `asr/zasr` | q2r.x/matdyn.x | acoustic sum rule / ASR | 掩盖未收敛结构 | 低频 DOS 和 Gamma 声学支 |

## 通用输出审阅模板

```markdown
## output review

- QE 程序:
- 计算类型:
- QE version:
- Input dependency:
- Structure summary:
- Pseudopotentials loaded:
- Cutoff reported:
- K-points reported:
- Convergence status:
- 本 workflow 关键输出:
- Warnings:
- Scratch / restart status:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

## 输出判断标准

- dyn 文件必须覆盖完整 q-grid。
- q2r.x 应成功写出 force constants。
- phonon DOS 低频区域应结合 acoustic modes 和 negative frequencies 审阅。
- frequency range 和负频标记应进入 record。

## 收敛性要求

- phonon DOS 依赖 q-grid 和 DOS mesh 收敛。
- 结构残余力、SCF 阈值和 ASR 会影响低频区域。
- 金属体系还需要 smearing/k 点敏感性检查。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| q2r 报 dyn 缺失 | ph.x q-grid 未完成 | 检查 dyn 文件列表 |
| 低频 DOS 异常 | ASR、结构或 q mesh 问题 | 复查 Gamma acoustic modes |
| DOS 峰形不稳定 | DOS mesh 或 q-grid 太粗 | 加密 q-grid/DOS mesh |

## 通用学习模板

使用 `<system>`、`<q_grid>`、`<q_path>`、`<asr_scheme>`、`<phonon_threshold>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
ph.<system>.in
q2r.<system>.in
matdyn.dos.<system>.in
phonon-dos.<system>.dat
record.md
```

## 与其他 workflow 的关系

- 与 phonon dispersion 共用 `ph.x -> q2r.x -> matdyn.x` 数据链。
- phonon debugging 页面用于处理 negative frequency 和 ASR 异常。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
