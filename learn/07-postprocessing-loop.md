# Postprocessing Loop

## 能力目标

理解 QE 后处理如何从已验证的 ground state 或 NSCF 状态生成可解释数据。

## 覆盖内容

- `pp.x`
- charge density
- electrostatic potential
- ELF
- work function
- visualization

## 完成判据

- 能说明后处理读取哪个上游 `prefix/outdir`。
- 能说明输出数据类型、单位、网格和可视化工具。
- 能说明图像是否可追溯到具体 input/output。

## 推荐阅读

- QE INPUT_PP reference
- QE PostProc documentation
- VESTA / XCrySDen documentation

## 后续进入哪个 workflow

进入 `workflows/electronic/charge-density.md`、`electrostatic-potential.md`、`elf.md`、`work-function.md`。
