# Phonon DFPT Loop

## 能力目标

掌握 DFPT phonon 的基本路线，理解 phonon 不是普通后处理。

## 完成判据

- 能区分 Gamma phonon 与 q-grid phonon。
- 能说明 `ph.x`、`dynmat.x`、`q2r.x`、`matdyn.x` 分工。
- 能说明 ASR 的作用与局限。
- 能初步判断 negative frequency 是数值问题还是可能的物理不稳定。
- 能知道 `epsil`、Born charge、dielectric tensor 只在特定物理条件下有意义。

## 可验证输出

- `ph.x` perturbation convergence 阅读笔记。
- `*.dyn* -> *.fc -> *.freq` 文件链说明。
- ASR 和 negative frequency 判断记录。

## 推荐阅读

- QE INPUT_PH, INPUT_Q2R, INPUT_MATDYN
- Pranab Das phonon tutorial
- Kyoto University QE phonon DokuWiki

## 后续进入哪个 workflow

完成后可进入 phonon DOS、dielectric/Born charge、IR/Raman 或 electron-phonon overview。
