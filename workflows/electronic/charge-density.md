# Charge density workflow

## 页面定位

- 对应学习路线：[learn/07-postprocessing-loop.md](../../learn/07-postprocessing-loop.md)
- 上游依赖：SCF ground-state density
- 下游用途：charge density data and visualization
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

用 `pp.x` 从 SCF ground state 提取 charge density 网格数据，用于可视化、电荷差分或后续定性分析。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
pw.x scf
  -> pp.x with charge-density plot_num
  -> <charge_density_file>
  -> visualization / analysis
```

## 需要的 QE 程序

`pp.x`

## Command boundary

| 对象 | 要求 |
|---|---|
| 上游命令 | `pw.x -in pw.scf.<system>.in > pw.scf.<system>.out` |
| 本步命令 | `pp.x -in pp.density.<system>.in > pp.density.<system>.out` |
| 必须读取 | 已审阅 SCF 的 `prefix/outdir` 和 charge density |
| 主要输出 | `filplot` 中间文件、`fileout` 可视化/分析文件 |
| 不应混用 | 不同结构、不同 FFT grid、不同 `prefix/outdir` 的密度文件 |

## 通用输入模板

```fortran
&INPUTPP
  prefix = '<system>',
  outdir = '<scratch_dir>',
  plot_num = <charge_density_plot_num>,
  filplot = 'charge-density.<system>.dat',
/
&PLOT
  iflag = <plot_dimension>,
  output_format = <output_format>,
  fileout = 'charge-density.<system>.<format>',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `plot_num` | pp.x | 选择 charge density 等输出量 | 未核对 plot_num 含义 | pp.x output 和生成文件 |
| `filplot` | pp.x | 中间 plot 文件 | 文件覆盖或来源不明 | 生成 filplot |
| `iflag` | pp.x / &PLOT | 选择 1D/2D/3D 输出 | 维度与分析目标不符 | 输出网格维度 |
| `output_format` | pp.x / &PLOT | 选择可视化格式 | 目标软件无法读取 | 生成 fileout |

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

- 确认读取的是目标 SCF `prefix/outdir`。
- `pp.x` output 中应能确认 `filplot` 和 `fileout` 已生成，且输出维度与 `iflag` 一致。
- 输出格式和网格维度符合可视化或分析目标；3D charge density 用于体数据，1D/2D 输出只能解释所选方向或切片。
- 图像解释必须回到 SCF input/output，不能脱离收敛性。
- 电荷差分或跨计算对比必须确认 cell、原子顺序、FFT grid、单位和参考密度一致。

## 收敛性要求

- charge density 依赖 SCF 收敛质量。
- 网格分辨率受 FFT grid 和 cutoff 影响。
- 电荷差分需要多次计算的结构、网格和参考一致。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 图像为空或格式错误 | output_format/iflag 不适配 | 检查 pp.x &PLOT 设置 |
| 读取旧密度 | prefix/outdir 混用 | 核对 pp.x output |
| 电荷差分不可比 | 网格或结构不一致 | 统一 cell 和 FFT/grid 设置 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.out
pp.density.<system>.in
pp.density.<system>.out
charge-density.<system>.<format>
record.md
```

## 与其他 workflow 的关系

- 依赖 SCF。
- 与 ELF、electrostatic potential 同属 pp.x 后处理。
- 可视化工具只辅助审阅。

## 资料来源

- QE INPUT_PP reference: <https://www.quantum-espresso.org/Doc/INPUT_PP.html>
- QE PostProc guide: <https://www.quantum-espresso.org/Doc/pp_user_guide/>

## Source / version boundary

| 项目 | 边界 |
|---|---|
| `plot_num` | 具体编号含义以当前 QE `INPUT_PP` 为准 |
| `output_format` | 可用格式和可视化软件支持以当前 PostProc 文档为准 |
| 图像解释 | 属于后处理判断，不能替代 SCF 收敛和物理模型审阅 |
