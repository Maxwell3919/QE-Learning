# K-point convergence

## 核心问题

k 点不足会影响金属体系、DOS、Fermi level、力和 phonon。

## 排查重点

- 低维体系应避免对真空方向过密采样。
- bands path 不能替代 DOS/SCF uniform mesh。
- 金属需要同时检查 smearing。

## GitHub 记录

记录 k mesh、irreducible k-points、目标性质变化和最终选择理由。

