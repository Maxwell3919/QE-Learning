# Relaxation Loop

## 能力目标

理解结构优化在 QE 中的角色，能区分固定晶格优化、晶胞优化和低维材料约束。

## 完成判据

- 能区分 `relax` 和 `vc-relax`。
- 能读 force 和 stress。
- 能说明低维材料为什么不能无脑 `vc-relax`。
- 能说明结构操作学习将作为独立项目展开；本仓库只说明 QE 对结构输入的要求。

## 可验证输出

- `pw.relax.<system>.in/out` 或 `pw.vc-relax.<system>.in/out` 的阅读笔记。
- final structure 的来源说明。
- final static SCF 的准入判断。

## 推荐阅读

- QE INPUT_PW reference 中 `&IONS` 和 `&CELL`。
- Pranab Das structure optimization materials。

## 后续进入哪个 workflow

完成后进入 bands、DOS/PDOS 或 phonon workflow。
