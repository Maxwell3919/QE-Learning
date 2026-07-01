# Dielectric tensor and Born effective charge workflow

## 页面定位

- 对应学习路线：[learn/06-phonon-dfpt-loop.md](../../learn/06-phonon-dfpt-loop.md)
- 上游依赖：insulating relaxed structure + Gamma DFPT
- 下游用途：LO-TO splitting, non-analytic correction, IR branch
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

使用 `epsil=.true.` 在 Gamma limit 计算 dielectric tensor 和 Born effective charge，并审阅它们是否可用于 LO-TO splitting 和 non-analytic correction。

## 输入前提

- 结构已充分优化，并通过 SCF output review。
- SCF 的 cutoff、k 点、smearing 和 `conv_thr` 已按 phonon 目标审阅。
- 已明确该页是 Gamma phonon、q-grid、DOS、响应张量还是 debugging 分支。

## 计算图

```text
pw.x scf
  -> ph.x at Gamma with epsil=.true.
  -> dielectric tensor + Born effective charge
  -> dynmat.x / matdyn.x non-analytic branch when valid
```

## 需要的 QE 程序

`ph.x`、`dynmat.x`、`matdyn.x`

## 通用输入模板

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  tr2_ph = <phonon_threshold>,
  trans = .true.,
  epsil = .true.,
  fildyn = '<system>.dynG',
/
0.0 0.0 0.0
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `epsil` | ph.x | 请求 dielectric tensor 和 Born effective charge | 金属或不适用体系中解释 | ph.x output 中张量段落 |
| `trans` | ph.x | 同时计算 phonon perturbations | 只要电场响应不审阅声子 | perturbation 输出 |
| `fildyn` | ph.x | Gamma dynamical matrix | 后处理读错文件 | dynmat/matdyn output |
| `asr/non-analytic options` | dynmat.x/matdyn.x | LO-TO 和 ASR 后处理 | 条件不满足仍解释 splitting | 后处理输出 |

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

- 确认体系被作为绝缘体响应问题处理；金属情形应 BLOCK 或单独说明。
- dielectric tensor 和 Born effective charge 张量应完整输出。
- electric-field perturbations 和 phonon perturbations 都应收敛。
- LO-TO splitting 解释需要记录方向、非解析项和 ASR 设置。

## 收敛性要求

- SCF/phonon 阈值、结构残余力和 cutoff/k 点都影响响应张量。
- `epsil` 分支不替代 q-grid phonon dispersion。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 没有 dielectric tensor | `epsil` 未开启或物理条件不适用 | 检查 ph.x input/output warning |
| Born effective charge 异常 | 结构、赝势或响应未收敛 | 复查 SCF/phonon convergence |
| LO-TO 解释混乱 | 未记录方向和 non-analytic 设置 | 补充 dynmat/matdyn 记录 |

## 通用学习模板

使用 `<system>`、`<q_grid>`、`<q_path>`、`<asr_scheme>`、`<phonon_threshold>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
ph.epsil.<system>.in
ph.epsil.<system>.out
dynmat.<system>.in
dynmat.<system>.out
record.md
```

## 与其他 workflow 的关系

- 依赖 Gamma phonon。
- IR/Raman 和 polar phonon 分析会使用该分支。
- phonon dispersion 中的 non-analytic correction 需要记录这些张量。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
