# 2D materials workflow

## 1. 计算目标

处理二维材料的真空层、晶胞自由度、电场/偶极和长程库仑问题。

## 3. 计算图

```text
2D structure + vacuum
  -> boundary-condition strategy
  -> relax / scf / electronic / phonon workflows
```

## 后续扩展

补 `assume_isolated='2D'`、`cell_dofree='2Dxy'`、dipole correction 和 phonon 特殊风险。

