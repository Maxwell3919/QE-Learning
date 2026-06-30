# Postprocessing Loop

## 我为什么要学这个？

QE 的许多物理图像不是直接从 `pw.x` output 读出，而是通过 `pp.x`、`average.x`、可视化工具和 Python 脚本得到。

## 它在 QE workflow 中处于哪个位置？

```text
trusted SCF / NSCF
  -> pp.x / average.x / fs.x
  -> volumetric data / potential / ELF / Fermi surface
  -> VESTA / XCrySDen / Python plotting
```

## 我需要读哪些外部资料？

- QE INPUT_PP
- QE PostProc docs
- VESTA / XCrySDen references

## 我需要跑什么最小案例？

Si charge density；slab electrostatic potential / work function 作为后续案例。

## output 应该怎么看？

- 输出数据类型是否和目标物理量一致。
- 网格、单位、坐标和能量零点是否记录。
- 图像是否能追溯到具体 input/output。

## 结果什么时候可以进入下游？

当原始数据文件、图像脚本、单位和解释边界可追踪后，才进入报告或论文图。

## 当前页面还缺哪些验证？

- 还缺 `pp.x` input 模板。
- 还缺 VESTA/XCrySDen/Python 的图像标准。
