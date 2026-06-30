# PDOS workflow

## 1. 计算目标

计算投影态密度，用于分析元素和轨道贡献。

## 3. 计算图

```text
pw.x scf
  -> pw.x nscf
  -> projwfc.x
  -> PDOS / Lowdin charge
```

## 后续扩展

补投影解释边界、轨道标签、投影归一化和图像规范。

