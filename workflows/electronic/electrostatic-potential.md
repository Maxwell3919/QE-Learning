# Electrostatic potential workflow

## 页面定位

- 对应学习路线：[learn/07-postprocessing-loop.md](../../learn/07-postprocessing-loop.md)
- 上游依赖：SCF ground state and pp.x potential output
- 下游用途：potential profile and work-function pre-analysis
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

用 `pp.x` 提取势相关网格，再用 `average.x` 得到方向平均电势，用于界面、slab 和功函数分析前置检查。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
pw.x scf
  -> pp.x potential output
  -> average.x along <direction>
  -> planar/macro averaged potential profile
```

## 需要的 QE 程序

`pp.x`、`average.x`

## 通用输入模板

```fortran
&INPUTPP
  prefix = '<system>',
  outdir = '<scratch_dir>',
  plot_num = <potential_plot_num>,
  filplot = 'potential.<system>.dat',
/
&PLOT
  iflag = 3,
  output_format = <output_format>,
  fileout = 'potential.<system>.<format>',
/

&AVERAGE
  input_file = 'potential.<system>.dat',
  output_file = 'potential-average.<system>.dat',
  idir = <average_direction>,
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `plot_num` | pp.x | 选择势相关物理量 | 混淆 electrostatic/local/total potential | pp.x output |
| `idir` | average.x | 平均方向 | 方向与 slab 法向不一致 | average profile |
| `output_format` | pp.x | 输出格式 | average.x 或可视化工具无法读取 | 生成文件 |
| `prefix/outdir` | pp.x | 读取 SCF 数据 | 旧 scratch | pp.x output |

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

- 确认势类型、单位和方向。
- 平均方向应与目标表面/界面法向一致。
- slab/work function 分析需看到足够平坦的真空平台。
- 若电势 profile 用于 work function、band alignment、slab 或 interface 判断，应同时记录边界条件、真空平台和能量零点；物理边界见 [physics-judgement/work-function-and-electrostatic-boundary.md](../../physics-judgement/work-function-and-electrostatic-boundary.md)。

## 收敛性要求

- 势 profile 对 slab 厚度、真空、偶极修正和 SCF 收敛敏感。
- work function 前应先完成 potential profile 审阅。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 没有真空平台 | 真空不足或 slab 设置不合理 | 回到结构和 SCF 输入检查 |
| 方向错误 | idir 与表面法向不一致 | 检查 CELL_PARAMETERS 和平均方向 |
| 势类型混淆 | plot_num 未核对 | 查 INPUT_PP 并记录定义 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.out
pp.potential.<system>.in
pp.potential.<system>.out
average.<system>.in
average.<system>.out
potential-average.<system>.dat
record.md
```

## 与其他 workflow 的关系

- work-function 依赖本页 profile。
- charge-density/ELF 共享 pp.x 数据边。

## 资料来源

- QE INPUT_PP reference: <https://www.quantum-espresso.org/Doc/INPUT_PP.html>
- QE PostProc guide: <https://www.quantum-espresso.org/Doc/pp_user_guide/>
