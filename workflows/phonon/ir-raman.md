# IR and Raman workflow

## 页面定位

- 对应学习路线：[learn/06-phonon-dfpt-loop.md](../../learn/06-phonon-dfpt-loop.md)
- 上游依赖：Gamma phonon and response tensors
- 下游用途：IR/Raman activity interpretation boundary
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

整理 IR/Raman 相关响应性质的最低学习边界：依赖 Gamma phonon、dielectric/Born effective charge 或 Raman 分支，输出解释必须核对 PHonon 文档。

## 输入前提

- 结构已充分优化，并通过 SCF output review。
- SCF 的 cutoff、k 点、smearing 和 `conv_thr` 已按 phonon 目标审阅。
- 已明确该页是 Gamma phonon、q-grid、DOS、响应张量还是 debugging 分支。

## 计算图

```text
pw.x scf
  -> ph.x Gamma response
  -> dynmat.x mode analysis
  -> IR/Raman-related quantities when supported and valid
```

## 需要的 QE 程序

`ph.x`、`dynmat.x`

## Command boundary

| 对象 | 要求 |
|---|---|
| 上游命令 | 已审阅的 `pw.x scf` 和 Gamma phonon 前置条件 |
| 本步命令 | `ph.x -in ph.ir-raman.<system>.in > ph.ir-raman.<system>.out`，必要时接 `dynmat.x` |
| 必须读取 | SCF `prefix/outdir`、Gamma dynamical matrix、响应张量输出 |
| 主要输出 | Gamma modes、dielectric/Born branch、IR/Raman 相关响应量 |
| 不应混用 | q-grid dispersion 的插值结果与 Gamma response tensor 解释 |

## 通用输入模板

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  tr2_ph = <phonon_threshold>,
  trans = .true.,
  epsil = <true_for_ir_related_branch>,
  lraman = <true_if_raman_branch_is_supported>,
  fildyn = '<system>.dynG',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `epsil` | ph.x | IR 相关介电/Born 分支 | 未检查适用条件 | ph.x dielectric/Born output |
| `lraman` | ph.x | Raman 相关响应请求 | 未核对编译和适用条件 | ph.x Raman output |
| `fildyn` | ph.x/dynmat.x | Gamma modes 文件 | 读取错误 dyn | dynmat.x output |

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

- 先确认 Gamma phonon modes 和 perturbations 收敛。
- `ph.x` output 应明确显示 `epsil`、`lraman` 或相关响应分支是否被执行；缺少分支输出时不能补写强度结论。
- IR/Raman 输出的单位、模式编号、张量和选择定则解释应回到官方 PHonon 文档。
- 没有响应张量或强度输出时，不应从频率表直接推断活性。
- 金属、smearing 较强或电场响应条件不成立时，应将 dielectric/Born/IR/Raman 解释降级为 WARN 或 BLOCK。

## 收敛性要求

- IR/Raman 对结构、SCF 和 Gamma phonon 收敛敏感。
- Raman 分支应按当前 QE build 和官方文档核对能力。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 没有强度输出 | 未开启对应分支或功能不支持 | 核对 INPUT_PH 和 build |
| 模式编号混乱 | dynmat 与 ph 输出未对应 | 对照 dynmat mode list |
| 把频率当强度 | 输出解释越界 | 回到官方 PHonon 文档 |

## 通用学习模板

使用 `<system>`、`<q_grid>`、`<q_path>`、`<asr_scheme>`、`<phonon_threshold>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
ph.ir-raman.<system>.in
ph.ir-raman.<system>.out
dynmat.<system>.out
record.md
```

## 与其他 workflow 的关系

- 依赖 Gamma phonon。
- dielectric/Born effective charge 页面提供 IR 相关前提。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>

## Source / version boundary

| 项目 | 边界 |
|---|---|
| `epsil` / `lraman` | 字段含义、适用条件和输出内容以当前 QE `INPUT_PH` 与 PHonon guide 为准 |
| Raman 支持 | 与 QE build、版本和方法分支有关，不能只凭 input 字段存在判断 |
| 强度解释 | 需要响应张量或强度输出；频率表本身不足以支持活性结论 |
