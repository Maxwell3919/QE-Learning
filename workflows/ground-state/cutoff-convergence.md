# Cutoff convergence workflow

## 页面定位

- 对应学习路线：[learn/03-convergence-loop.md](../../learn/03-convergence-loop.md)
- 上游依赖：checked structure, pseudo, baseline SCF input
- 下游用途：cutoff policy for SCF, relax, bands, DOS, phonon
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

评估 `ecutwfc` 与 `ecutrho` 对目标量的影响，建立后续 workflow 可复用的 cutoff 策略。

## 输入前提

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。

## 计算图

```text
<structure> + <pseudo> + fixed k mesh
  -> pw.x scf with ecutwfc/ecutrho scan
  -> convergence table
  -> cutoff policy for downstream workflow
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
| `ecutwfc` | pw.x | 波函数平面波 cutoff | 只看单点结果，不做 scan | output 显示 kinetic-energy cutoff |
| `ecutrho` | pw.x | 电荷密度和势 cutoff | USPP/PAW 设置过低 | output 显示 charge density cutoff |
| `ATOMIC_SPECIES` | pw.x | 赝势类型影响 cutoff 需求 | 换赝势后沿用旧 cutoff | output 中 pseudopotential type |
| `K_POINTS` | pw.x | cutoff scan 中应固定 k mesh | 同时改 cutoff 和 k mesh | output 中 k 点数量一致 |

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

- 每个 scan 点都应确认 SCF 已收敛。
- 记录 total energy、force、stress、band gap 或 phonon frequency 等目标量。
- PASS 需要目标量变化低于自定准则；WARN 表示可探索但不足以下游定稿；BLOCK 表示需继续加大 cutoff 或更换策略。

## 收敛性要求

- 固定结构、赝势、k mesh、smearing，只改变 cutoff 变量。
- NC、USPP、PAW 对 `ecutrho` 敏感性不同，需按赝势类型记录。
- phonon、stress、force 任务通常比单纯总能更敏感。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 总能收敛但力不稳定 | 只用 total energy 判断 | 加入 force/stress 目标量 |
| 换赝势后结果漂移 | cutoff policy 未重做 | 从最低 cutoff 重新扫描 |
| phonon 前出现虚频 | cutoff/ecutrho 或结构未足够严格 | 提高 cutoff 并复查 SCF/relax |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.ecut<value>.in
pw.scf.<system>.ecut<value>.out
cutoff-convergence.<system>.md
record.md
```

## 与其他 workflow 的关系

- SCF 使用 cutoff policy。
- relax/vc-relax 应使用通过力/应力审阅的 cutoff。
- phonon 前应复查 cutoff 对频率的影响。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
