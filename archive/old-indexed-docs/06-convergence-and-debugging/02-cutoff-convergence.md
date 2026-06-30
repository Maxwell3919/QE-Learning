# Cutoff convergence

## 核心问题

cutoff 不收敛会污染总能、力、应力、phonon 和相对稳定性。

## 排查重点

- 不只看总能，也看目标性质。
- USPP/PAW 需要认真检查 `ecutrho`。
- phonon 和 stress 通常比普通 SCF 更敏感。

## GitHub 记录

使用 `09-templates-and-checklists/convergence-report-template.md`。

