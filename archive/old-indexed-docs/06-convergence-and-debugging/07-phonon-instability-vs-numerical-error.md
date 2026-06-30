# Phonon instability vs numerical error

## 核心问题

negative frequency 可能是真实动力学不稳定，也可能是数值误差。

## 优先排查

1. 结构是否真正 relax。
2. SCF `conv_thr` 是否足够严格。
3. cutoff/k 点/smearing 是否对 phonon 收敛。
4. q-grid 是否太粗。
5. ASR 前后变化是否合理。
6. 2D 或极性材料是否需要特殊处理。

## 不能做的判断

不能凭一次粗糙 q-grid 或未充分 relax 的结构宣布材料稳定/不稳定。

