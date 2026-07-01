# phonon debugging workflow

## 页面定位

- 对应学习路线：[learn/06-phonon-dfpt-loop.md](../../learn/06-phonon-dfpt-loop.md)
- 上游依赖：failed or suspicious phonon workflow
- 下游用途：rerun strategy and PASS / WARN / BLOCK decision
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

按症状排查 phonon workflow：`ph.x` 不收敛、acoustic modes 不为零、negative frequency、q-grid 太粗、结构未充分优化、SCF threshold 太松、金属 smearing 和 2D 长程相互作用问题。

## 输入前提

- 结构已充分优化，并通过 SCF output review。
- SCF 的 cutoff、k 点、smearing 和 `conv_thr` 已按 phonon 目标审阅。
- 已明确该页是 Gamma phonon、q-grid、DOS、响应张量还是 debugging 分支。

## 计算图

```text
suspicious phonon output
  -> identify symptom
  -> check SCF / relax / ph.x / q2r.x / matdyn.x
  -> decide PASS / WARN / BLOCK
  -> rerun with isolated changed variable
```

## 需要的 QE 程序

`pw.x`、`ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x`

## 通用输入模板

```fortran
# Debugging is record-driven. Keep the original input/output unchanged.
# Create a new run with one controlled change at a time:
&INPUTPH
  prefix = '<system_debug_run>',
  outdir = '<fresh_scratch_dir>',
  tr2_ph = <stricter_phonon_threshold>,
  recover = .true.,
  <same_q_point_or_q_grid> = <same_or_controlled_change>,
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `tr2_ph` | ph.x | 响应收敛阈值 | 只调阈值不看 SCF | perturbation convergence |
| `conv_thr` | pw.x | 上游 SCF 阈值 | SCF 太松进入 phonon | SCF estimated accuracy |
| `forc_conv_thr` | pw.x relax | 结构残余力 | 未优化结构导致虚频 | relax output final forces |
| `asr/zasr` | q2r.x/matdyn.x/dynmat.x | acoustic sum rule / ASR | 用 ASR 掩盖真实问题 | ASR 前后频率对比 |
| `nq1/nq2/nq3` | ph.x | q-grid 尺寸 | 粗 q-grid 解释细节 | dyn 文件和插值结果 |

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

- ph.x 不收敛先回查 SCF、smearing、mixing 和 perturbation 输出。
- acoustic modes 不为零时比较 ASR 前后，并复查结构残余力。
- negative frequency 要记录位置、大小、参数敏感性和是否稳定存在。
- 2D 体系需检查真空、长程库仑和边界处理设置。

## 收敛性要求

- 每次 debug 只改变一个变量。
- 新测试使用 fresh `prefix/outdir`，避免旧 scratch 污染。
- PASS / WARN / BLOCK 应写明证据和下一步，不用单次粗结果定性稳定性。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| ph.x 不收敛 | SCF 太松、smearing/mixing 不合适 | 先复查 SCF output 和 ph perturbation |
| acoustic modes 不为零 | 结构残余力、ASR、cutoff/k 点问题 | 检查 relax、ASR 前后、收敛参数 |
| negative frequency | 数值误差或真实不稳定 | 做参数和关键 q 点复查 |
| q2r dyn 不全 | q-grid 未完成 | 检查 dyn 文件列表 |
| 2D Gamma 附近异常 | 周期镜像或长程相互作用 | 检查真空和 2D 边界策略 |

## 通用学习模板

使用 `<system>`、`<q_grid>`、`<q_path>`、`<asr_scheme>`、`<phonon_threshold>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
debug-notes.<system>.md
original-inputs/
controlled-rerun-inputs/
controlled-rerun-outputs/
record.md
```

## 与其他 workflow 的关系

- 服务 Gamma phonon、phonon dispersion、phonon DOS 和 dielectric/Born 分支。
- SCF、relax 和 convergence 页面是 phonon debug 的上游。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
