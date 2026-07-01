# K-point convergence workflow

## 页面定位

- 对应学习路线：[learn/03-convergence-loop.md](../../learn/03-convergence-loop.md)
- 上游依赖：checked structure, pseudo, cutoff policy
- 下游用途：k-point policy for SCF, DOS, phonon, Fermi surface
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

评估 uniform k mesh、shift 和 symmetry 对目标量的影响，避免把 bands path 误当作 k 点收敛。

## 输入前提

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。

## 计算图

```text
<structure> + <pseudo> + cutoff policy
  -> pw.x scf with k-mesh scan
  -> convergence table
  -> k-point policy
```

## 需要的 QE 程序

`pw.x`

## 通用输入模板

```fortran
&CONTROL
  calculation = 'scf',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
/
&SYSTEM
  ibrav = 0,
  nat = <number_of_atoms>,
  ntyp = <number_of_species>,
  ecutwfc = <wavefunction_cutoff>,
  ecutrho = <charge_density_cutoff>,
  occupations = '<occupation_scheme>',
/
&ELECTRONS
  conv_thr = <scf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>
ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>
K_POINTS automatic
  <nk1> <nk2> <nk3> <sk1> <sk2> <sk3>
CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `K_POINTS automatic` | pw.x | SCF/NSCF 的 uniform k mesh | 用 bands path 做收敛 | output 显示 k points 和 irreducible k-points |
| `shift flags` | pw.x | 记录 shifted/unshifted mesh | 不记录 shift 导致不可复查 | output k 点列表或 input record |
| `nosym/noinv` | pw.x | 影响对称性约化 | 随意开关改变不可约 k 点 | output 显示 symmetry operations |
| `occupations/smearing` | pw.x | 金属 k 点收敛相关 | 改变 k mesh 同时改 smearing | output occupation scheme |

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

- 检查 irreducible k-points 是否随 symmetry 预期变化。
- 记录 total energy、force、stress、Fermi energy、DOS 近 Fermi 区域或 phonon frequency 的变化。
- bands k-path 只用于能带图，不用于 k mesh convergence。

## 收敛性要求

- 固定 cutoff、structure、pseudo 和 smearing，只扫描 k mesh。
- 金属、小带隙和低维体系通常对 k 点更敏感。
- DOS 和 Fermi surface 需要 dense NSCF k mesh，不能只依赖 SCF mesh。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| DOS 锯齿或 Fermi 附近不稳 | k mesh 太稀 | 提高 NSCF dense mesh |
| bands 与 DOS 金属性判断冲突 | path 与 mesh 语义混用 | 分开检查 bands path 和 DOS mesh |
| 同一 mesh 点数结果不同 | shift/symmetry 未记录 | 记录 shift 和 symmetry 设置 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.k<nk>.in
pw.scf.<system>.k<nk>.out
kpoint-convergence.<system>.md
record.md
```

## 与其他 workflow 的关系

- SCF 使用基本 k mesh。
- DOS/PDOS 通常需要更密 NSCF mesh。
- bands 使用 k-path，应单独记录 k-path 来源。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
