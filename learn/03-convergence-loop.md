# Convergence Loop

## 能力目标

理解 cutoff、`ecutrho`、k-points、smearing / occupations 对 QE 数值结果的影响。

## 完成判据

- 能说明 `ecutwfc` 和 `ecutrho` 的区别。
- 能说明 k-point mesh 与 bands path 的区别。
- 能说明金属为什么需要 smearing。
- 能写出最小 convergence 记录表。
- 能判断一个 SCF 是否允许进入 relax、bands、DOS 或 phonon。

## 可验证输出

| tested parameter | fixed parameters | observed output | decision |
|---|---|---|---|
| `ecutwfc` | structure, pseudo, k mesh | total energy / force / stress | PASS/WARN/BLOCK |
| k mesh | structure, pseudo, cutoff | total energy / Fermi level | PASS/WARN/BLOCK |
| smearing | metallic system setup | SCF stability / energy trend | PASS/WARN/BLOCK |

## 推荐阅读

- QE INPUT_PW reference
- Pranab Das convergence materials
- `standards/output-review-checklist.md`

## 后续进入哪个 workflow

完成后进入 `learn/04-relaxation-loop.md` 或电子结构 workflow。
