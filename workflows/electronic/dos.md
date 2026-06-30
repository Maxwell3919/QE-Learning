# DOS workflow

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 上游依赖：SCF -> NSCF on dense uniform k mesh
- 下游用途：total DOS plot and Fermi-level analysis
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

从 dense uniform k mesh 的 NSCF eigenvalues 计算 total density of states，判断能隙、Fermi 附近态和总体能量分布。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
<structure> + <pseudo>
  -> pw.x scf
  -> pw.x nscf on <dense_k_mesh>
  -> dos.x
  -> dos.<system>.dat
```

## 需要的 QE 程序

`pw.x`、`dos.x`

## 通用输入模板

```fortran
&DOS
  prefix = '<system>',
  outdir = '<scratch_dir>',
  fildos = 'dos.<system>.dat',
  Emin = <energy_min>,
  Emax = <energy_max>,
  DeltaE = <energy_step>,
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `prefix/outdir` | dos.x | 读取 NSCF 数据 | 读取 SCF 或旧 scratch | dos.x output 显示读取目录 |
| `fildos` | dos.x | DOS 输出文件 | 文件名无法追溯 | 生成指定数据文件 |
| `Emin/Emax` | dos.x | 能量窗口 | 没有覆盖目标能区 | DOS 文件能量范围 |
| `DeltaE` | dos.x | 能量步长 | 过粗丢失结构或过细放大噪声 | DOS 文件步长 |

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

- 确认 `dos.x` 读取的是目标 NSCF 数据。
- 能量范围覆盖 VBM/CBM、Fermi 附近或关注能区。
- DOS 金属性判断应与 bands 和 Fermi energy 一致。
- 记录能量零点采用 Fermi energy、VBM 还是原始 QE 输出。

## 收敛性要求

- NSCF k mesh 通常要比 SCF 更密。
- 金属 DOS 对 k mesh 和 smearing/broadening 敏感。
- 能量步长应与目标能区分辨率匹配。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| DOS 锯齿严重 | k mesh 太稀或 DeltaE 太小 | 加密 NSCF k mesh并调整 DeltaE |
| DOS 与 bands 矛盾 | 能量零点或 k mesh 不一致 | 对齐 Fermi reference 并复查 bands |
| dos.x 读不到数据 | prefix/outdir 不一致 | 核对 SCF/NSCF/dos.x 输入 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.in
pw.scf.<system>.out
pw.nscf.<system>.in
pw.nscf.<system>.out
dos.<system>.in
dos.<system>.out
dos.<system>.dat
record.md
```

## 与其他 workflow 的关系

- 依赖 NSCF dense k mesh。
- PDOS 应与 total DOS 对照。
- Fermi surface 需要更密且适合金属的 NSCF 数据。

## 资料来源

- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- Pranab Das QE tutorial: <https://pranabdas.github.io/espresso/>
