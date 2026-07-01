# ELF workflow

## 页面定位

- 对应学习路线：[learn/07-postprocessing-loop.md](../../learn/07-postprocessing-loop.md)
- 上游依赖：SCF ground-state density
- 下游用途：ELF grid and visualization
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

用 `pp.x` 输出 electron localization function，用于辅助判断电子局域化和成键特征。ELF 是可视化/定性分析工具，不应被直接当作定量键强度。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
pw.x scf
  -> pp.x ELF plot_num
  -> ELF grid file
  -> visualization
```

## 需要的 QE 程序

`pp.x`

## Command boundary

| 对象 | 要求 |
|---|---|
| 上游命令 | 已审阅的 `pw.x scf` |
| 本步命令 | `pp.x -in pp.elf.<system>.in > pp.elf.<system>.out` |
| 必须读取 | SCF charge density 和对应的 `prefix/outdir` |
| 主要输出 | ELF grid 或切片文件 |
| 不应混用 | 不同等值面、不同色标、不同网格的可视化结果作为定量比较 |

## 通用输入模板

```fortran
&INPUTPP
  prefix = '<system>',
  outdir = '<scratch_dir>',
  plot_num = <elf_plot_num>,
  filplot = 'elf.<system>.dat',
/
&PLOT
  iflag = 3,
  output_format = <output_format>,
  fileout = 'elf.<system>.<format>',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `plot_num` | pp.x | 选择 ELF 输出 | 未核对官方 plot_num | pp.x output |
| `iflag` | pp.x / &PLOT | 输出维度 | 只输出切片却解释整体空间 | 输出网格维度 |
| `output_format` | pp.x / &PLOT | 可视化格式 | 格式不被工具支持 | 生成文件 |
| `prefix/outdir` | pp.x | 读取 SCF 密度 | 未收敛或旧密度 | pp.x output |

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

- 确认 ELF 文件来自目标 SCF。
- `pp.x` output 应显示对应 `plot_num`、`filplot`、`iflag`、`output_format` 和 `fileout` 已处理。
- 可视化前记录等值面、切片方向、色标、单位和软件版本。
- ELF 解释应结合 charge density、PDOS 或结构信息。
- ELF 只支持“电子局域化图像”的定性讨论；不能单独给出键强度、键级或电荷转移的定量结论。

## 收敛性要求

- ELF 对 SCF 密度质量和网格分辨率敏感。
- 不同可视化等值面不能直接比较为定量键强度。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| ELF 图像异常 | plot_num 或输出格式错误 | 核对 INPUT_PP |
| 解释过度 | 把 ELF 当定量键级 | 回到 PDOS/charge density 对照 |
| 不同体系图不可比 | 等值面/色标/网格不同 | 统一可视化设置并记录 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.out
pp.elf.<system>.in
pp.elf.<system>.out
elf.<system>.<format>
record.md
```

## 与其他 workflow 的关系

- 依赖 SCF。
- 与 charge density、PDOS 配合解释电子局域化。

## 资料来源

- QE INPUT_PP reference: <https://www.quantum-espresso.org/Doc/INPUT_PP.html>

## Source / version boundary

| 项目 | 边界 |
|---|---|
| ELF `plot_num` | 以当前 QE `INPUT_PP` 为准 |
| 文件格式 | 以当前 QE PostProc 文档和可视化工具支持为准 |
| 物理解释 | 属于定性后处理，必须与结构、charge density、DOS/PDOS 或其他证据交叉审阅 |
