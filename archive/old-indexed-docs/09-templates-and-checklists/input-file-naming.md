# QE 文件命名规范

## 1. 命名目标

QE workflow 依赖许多 side-effect 文件。命名规范的目的不是美观，而是让文件名直接表达：

- 使用哪个 QE 程序。
- 属于哪个计算阶段。
- 对应哪个体系。
- 是 input、output 还是中间数据。

## 2. 基本格式

```text
<program>.<calculation>.<system>.<ext>
```

其中：

- `<program>`：`pw`、`ph`、`q2r`、`matdyn`、`dynmat`、`dos`、`projwfc`、`bands`、`pp`
- `<calculation>`：`scf`、`relax`、`vc-relax`、`bands`、`nscf`、`dos`、`pdos`、`phonon`、`density`
- `<system>`：短体系名，例如 `Si`、`GaAs`、`Na-HfCl2-P1A`
- `<ext>`：`in`、`out`、`dat`、`freq`、`fc`、`dyn`

## 3. 推荐命名

```text
pw.scf.<system>.in
pw.scf.<system>.out
pw.relax.<system>.in
pw.relax.<system>.out
pw.vc-relax.<system>.in
pw.vc-relax.<system>.out
pw.nscf.<system>.in
pw.nscf.<system>.out
pw.bands.<system>.in
pw.bands.<system>.out
bands.<system>.in
bands.<system>.out
bands.<system>.dat
dos.<system>.in
dos.<system>.out
dos.<system>.dat
projwfc.<system>.in
projwfc.<system>.out
ph.<system>.in
ph.<system>.out
q2r.<system>.in
q2r.<system>.out
matdyn.<system>.in
matdyn.<system>.out
dynmat.<system>.in
dynmat.<system>.out
pp.density.<system>.in
pp.density.<system>.out
```

## 4. 目录建议

```text
calculations/
  <system>/
    00-structure/
    01-scf/
    02-convergence/
    03-relax/
    04-electronic/
      bands/
      dos/
      pdos/
    05-phonon/
      gamma/
      dispersion/
    06-postprocessing/
    record.md
```

## 5. `prefix` 与文件名的关系

`prefix` 是 QE scratch 数据的逻辑标识，文件名是 GitHub 记录的可读标识。二者可以相同，但不要混淆：

- 文件名用于人和 Git 追踪。
- `prefix/outdir` 用于 QE 程序之间读取数据。

建议：

- 探索性计算用独立 `prefix/outdir`。
- 生产计算不要复用旧 scratch，除非明确是 restart。
- 下游 workflow 的 `record.md` 必须写清楚读取的是哪个 SCF 的 `prefix/outdir`。

