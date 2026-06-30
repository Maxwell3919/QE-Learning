# Charge density and potential workflow

## 1. 计算目标

从 SCF 数据提取 charge density、potential、ELF 或相关实空间数据。

## 3. 计算图

```text
pw.x scf
  -> pp.x
  -> volumetric data
  -> VESTA / XCrySDen / Python visualization
```

## 后续扩展

补 `pp.x` 参数、cube/xsf 输出和功函数/电势平台判断。

