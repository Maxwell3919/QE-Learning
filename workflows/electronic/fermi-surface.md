# Fermi surface workflow

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 上游依赖：SCF -> dense NSCF for metallic system
- 下游用途：BXSF or visualization data for Fermi surface
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

从金属体系 dense k mesh eigenvalues 生成 Fermi surface 数据，用于与 bands/DOS 交叉验证 Fermi-level 附近电子结构。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
pw.x scf
  -> pw.x nscf on dense uniform k mesh
  -> fs.x
  -> <system>.bxsf
  -> XCrySDen / viewer
```

## 需要的 QE 程序

`pw.x`、`fs.x`、XCrySDen 或其他可视化工具

## Command boundary

| 对象 | 要求 |
|---|---|
| 上游命令 | 已审阅的 `pw.x scf`，随后是 dense uniform mesh 的 `pw.x nscf` |
| 本步命令 | `fs.x -in fs.<system>.in > fs.<system>.out` |
| 必须读取 | NSCF 的 eigenvalues、Fermi energy、band occupations 和 `prefix/outdir` |
| 主要输出 | BXSF 文件，例如 `fermi-surface.<system>.bxsf` |
| 不应混用 | bands high-symmetry path、稀疏 SCF mesh 或不同 Fermi-level reference |

## 通用输入模板

```fortran
&FERMI
  prefix = '<system>',
  outdir = '<scratch_dir>',
  filfermi = 'fermi-surface.<system>.bxsf',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `prefix/outdir` | fs.x | 读取 dense NSCF 数据 | 读取旧或稀疏数据 | fs.x output |
| `filfermi` | fs.x | BXSF 输出文件 | 文件名无法追踪 | 生成 BXSF |
| `K_POINTS automatic` | pw.x nscf | dense uniform k mesh | 用 bands path 生成费米面 | NSCF output k 点数量 |
| `Fermi energy` | pw.x output | 费米面能量参考 | 与 DOS/bands 零点混用 | output Fermi energy |

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

- 仅对有 Fermi surface 的体系有意义。
- `fs.x` output 和 BXSF 文件应能追踪到正确的 dense NSCF 数据。
- 确认 NSCF k mesh 足够密，并记录 shifted/unshifted mesh 与 Fermi energy 来源。
- BXSF 可视化应与 bands crossing 和 DOS at Fermi level 对照。
- BXSF 只是一种可视化数据格式；费米面拓扑或嵌套判断需要额外 k mesh sensitivity。

## 收敛性要求

- Fermi surface 对 k mesh、smearing 和 Fermi level 敏感。
- 应先完成 bands 和 DOS 的基本一致性检查。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| BXSF 不显示或破碎 | k mesh 太稀或文件格式问题 | 加密 NSCF 并检查 fs.x 输出 |
| 与 bands 不一致 | 能量零点或 band index 混乱 | 对照 Fermi energy 和 bands crossing |
| 用于非金属体系 | 物理前提错误 | 回到 DOS/bands 判断 |
| 费米面形状随 smearing 大幅变化 | k mesh 或占据设置不稳 | 回到 smearing/k-point convergence |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.in
pw.nscf.<system>.in
fs.<system>.in
fs.<system>.out
fermi-surface.<system>.bxsf
record.md
```

## 与其他 workflow 的关系

- 依赖 NSCF dense k mesh。
- 与 bands 和 DOS 共同判断金属性。

## 资料来源

- QE PostProc guide: <https://www.quantum-espresso.org/Doc/pp_user_guide/>
- XCrySDen: <http://www.xcrysden.org/>

## Source / version boundary

| 项目 | 边界 |
|---|---|
| `fs.x` 输入字段 | 以当前 QE PostProc 文档为准 |
| BXSF 可视化 | XCrySDen 或其他工具只负责显示，不判断物理可信度 |
| 金属判断 | 必须与 bands、DOS、occupation 和 smearing review 共同成立 |
