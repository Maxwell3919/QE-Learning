# PDOS workflow

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 上游依赖：SCF -> NSCF with wavefunctions
- 下游用途：orbital and atomic projected density of states
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

用 `projwfc.x` 将电子态投影到原子和轨道通道，分析元素/轨道贡献，并与 total DOS 对照。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
<structure> + <pseudo>
  -> pw.x scf
  -> pw.x nscf
  -> projwfc.x
  -> pdos files by atom/orbital
```

## 需要的 QE 程序

`pw.x`、`projwfc.x`

## 通用输入模板

```fortran
&PROJWFC
  prefix = '<system>',
  outdir = '<scratch_dir>',
  filpdos = 'pdos.<system>',
  Emin = <energy_min>,
  Emax = <energy_max>,
  DeltaE = <energy_step>,
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `prefix/outdir` | projwfc.x | 读取 NSCF wavefunctions | NSCF 数据不完整 | output 显示读取状态 |
| `filpdos` | projwfc.x | PDOS 文件前缀 | 无法追踪参数 | 生成分通道文件 |
| `Emin/Emax/DeltaE` | projwfc.x | 能量窗口和步长 | 与 total DOS 不一致 | PDOS 文件范围 |
| `atomic labels` | projwfc.x output | 投影通道标识 | 轨道解释与赝势不一致 | output 中 state labels |

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

- projection 文件是否完整生成。
- 轨道标签、原子序号和赝势通道是否一致。
- total DOS 与 PDOS 求和趋势应能互相对照，但投影不是严格完备性的保证。
- Lowdin charge 可作辅助信息，不应直接当作充分价态证据。
- PDOS/Lowdin/轨道标签只是证据链的一部分，不能单独证明价态、局域化、电荷转移或化学键强度。
- 若 PDOS 用于判断 delocalization、defect level、DFT+U projector 或 Wannier projection，应回查 [physics-judgement/band-gap-problem-and-delocalization.md](../../physics-judgement/band-gap-problem-and-delocalization.md) 与 [physics-judgement/wannier-validation-and-window-choice.md](../../physics-judgement/wannier-validation-and-window-choice.md)。

## 收敛性要求

- NSCF 需保留 wavefunctions。
- PDOS 对 `nbnd`、能量窗口、k mesh 和 smearing 敏感。
- 轨道解释要结合赝势投影通道。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| PDOS 文件缺失 | NSCF wavefunctions 不完整或 prefix 错误 | 检查 outdir 和 disk IO |
| 轨道标签误读 | 未核对 projwfc output labels | 以 output 标签为准 |
| PDOS 与 total DOS 差异大 | 投影通道、能量窗口或 broadening 不一致 | 对照 total DOS 设置 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.in
pw.nscf.<system>.in
projwfc.<system>.in
projwfc.<system>.out
pdos.<system>*
record.md
```

## 与其他 workflow 的关系

- 依赖 NSCF。
- 与 DOS 配合解释态密度来源。
- 与 Wannier 投影选择有概念联系，但目标不同。

## 资料来源

- QE INPUT_PROJWFC reference: <https://www.quantum-espresso.org/Doc/INPUT_PROJWFC.html>
- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
