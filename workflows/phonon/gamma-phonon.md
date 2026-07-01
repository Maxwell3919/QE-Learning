# Gamma phonon workflow

## 页面定位

- 对应学习路线：[learn/06-phonon-dfpt-loop.md](../../learn/06-phonon-dfpt-loop.md)
- 上游依赖：relaxed structure + strict SCF
- 下游用途：Gamma modes, IR/Raman preconditions, dielectric/Born branch
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

在 q=0 计算 Gamma phonon 模式，审阅 perturbation 收敛、acoustic modes、`dynmat.x` 模式输出，以及 `epsil` 分支是否物理有效。

## 输入前提

- 结构已充分优化，并通过 SCF output review。
- SCF 的 cutoff、k 点、smearing 和 `conv_thr` 已按 phonon 目标审阅。
- 已明确该页是 Gamma phonon、q-grid、DOS、响应张量还是 debugging 分支。

## 计算图

```text
pw.x scf
  -> ph.x at q = 0
  -> <system>.dynG
  -> dynmat.x
  -> Gamma phonon modes
```

## 需要的 QE 程序

`ph.x`、`dynmat.x`

## 通用输入模板

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  tr2_ph = <phonon_threshold>,
  trans = .true.,
  epsil = <true_or_false>,
  fildyn = '<system>.dynG',
  recover = .true.,
/
0.0 0.0 0.0

&DYNMAT
  fildyn = '<system>.dynG',
  asr = '<asr_scheme>',
  filout = 'dynmat.<system>.out',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `tr2_ph` | ph.x | DFPT perturbation 收敛阈值 | 过松导致频率漂移 | ph.x 每个 perturbation 收敛 |
| `trans` | ph.x | 计算声子扰动 | 关闭后没有 phonon modes | ph.x perturbation 列表 |
| `epsil` | ph.x | 介电张量与 Born effective charge 分支 | 金属或不适用条件下解释 | output 中 dielectric/Born 段落 |
| `fildyn` | ph.x/dynmat.x | Gamma dynamical matrix 文件 | dynmat 读取错误文件 | dynmat.x output |
| `asr` | dynmat.x | acoustic sum rule / ASR 后处理 | 把 ASR 当作收敛替代 | Gamma acoustic modes |

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

- 所有 irreducible perturbations 应达到 `tr2_ph`。
- Gamma acoustic modes 应接近零；偏离需要结合 ASR、结构残余力和数值参数判断。
- `epsil` 分支只在有效物理条件下解释，并记录 dielectric tensor 与 Born effective charge 是否出现。
- metal vs insulator 的电场响应边界需要在 record 中说明。

## 收敛性要求

- Gamma phonon 对 SCF `conv_thr`、结构残余力、cutoff/k 点敏感。
- ASR 只能处理平移不变性相关数值误差，不能修复未优化结构。
- `epsil`/Born effective charge 分支需要额外物理条件审阅。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| ph.x 不收敛 | SCF 太松、mixing 或 smearing 问题 | 先复查 SCF 和 `tr2_ph` |
| acoustic modes 明显偏离零 | 结构残余力、ASR 或 cutoff/k 点问题 | 比较 ASR 前后并复查 relax |
| dielectric/Born 缺失 | `epsil` 未开或物理条件不适用 | 检查 ph.x input 和 output warning |

## 通用学习模板

使用 `<system>`、`<q_grid>`、`<q_path>`、`<asr_scheme>`、`<phonon_threshold>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.in
ph.gamma.<system>.in
ph.gamma.<system>.out
dynmat.<system>.in
dynmat.<system>.out
record.md
```

## 与其他 workflow 的关系

- Gamma phonon 是 dielectric/Born、IR/Raman 的前置分支。
- q-grid dispersion 需要 `ldisp=.true.`，不能由单个 Gamma 结果替代。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
