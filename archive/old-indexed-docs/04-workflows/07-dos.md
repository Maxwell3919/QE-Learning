# DOS workflow

## 1. 计算目标

计算总态密度，用于判断带隙、金属性、Fermi level 附近态密度。

## 3. 计算图

```text
pw.x scf
  -> pw.x nscf on dense uniform k mesh
  -> dos.x
  -> DOS plot
```

## 后续扩展

补 NSCF 参数、tetrahedron/smearing 策略和能量零点规范。

