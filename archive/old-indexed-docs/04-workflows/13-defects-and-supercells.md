# Defects and supercells workflow

## 1. 计算目标

用 QE 作为计算引擎处理缺陷超胞、局域态和电荷密度差等问题。

## 3. 计算图

```text
bulk structure
  -> supercell / defect generation
  -> charged or neutral relax
  -> scf / dos / pdos / density analysis
  -> external correction and thermodynamics
```

## 后续扩展

补超胞收敛、带电缺陷修正、对齐、有限尺寸效应和工具边界。

