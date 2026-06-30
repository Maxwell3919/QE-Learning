# pw.x input overview

## 文档目的

建立 `pw.x` 输入文件的全景：namelist、cards、结构、赝势、k 点和 scratch 数据如何共同定义一个计算。

## 适合阶段

第一周核心阅读。

## 前置知识

SCF、平面波 cutoff、赝势、k 点。

## 应包含内容

- `&CONTROL`、`&SYSTEM`、`&ELECTRONS`、`&IONS`、`&CELL`
- `ATOMIC_SPECIES`、`ATOMIC_POSITIONS`、`K_POINTS`、`CELL_PARAMETERS`
- `prefix/outdir/pseudo_dir` 的数据流作用

## 资料状态

已有调研支撑；需核对 QE `INPUT_PW.html`。

