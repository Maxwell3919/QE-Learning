# VC-relax workflow

## 1. 计算目标

同时优化原子位置和晶胞，使力与应力达到目标阈值。

## 3. 计算图

```text
initial cell + SCF policy
  -> pw.x vc-relax
  -> optimized cell + positions
  -> final pw.x scf
```

## 后续扩展

重点补 `cell_dofree`、2D 材料限制、压力设置和应力收敛。

