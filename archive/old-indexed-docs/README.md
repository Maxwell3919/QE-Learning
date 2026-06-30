# QE 中文学习笔记入口

本目录是 QE 学习笔记主体。它按真实科研 workflow 组织，而不是按软件命令或理论教材章节机械排列。

## 学习主线

```text
structure + pseudopotential
  -> pw.x scf
  -> convergence tests
  -> relax / vc-relax
  -> electronic workflows: bands / DOS / PDOS
  -> phonon workflows: ph.x / q2r.x / matdyn.x / dynmat.x
  -> post-processing / visualization
  -> HPC / restart / provenance
```

## 第一版重点

- 先建立目录、模板和路线图。
- 先写 `SCF`、`bands`、`phonon dispersion` 三篇 workflow 初稿。
- 先形成文件命名规范和计算记录模板。
- 未展开章节保留清晰占位，避免后续扩展时失去结构。

## 资料状态标记

- `已有调研支撑`：已由两份本地 Markdown 调研材料支持。
- `需官方核对`：后续写细节前必须查 QE 官方 input reference 或 user guide。
- `需案例验证`：需要实际最小例子跑通后再补全。

