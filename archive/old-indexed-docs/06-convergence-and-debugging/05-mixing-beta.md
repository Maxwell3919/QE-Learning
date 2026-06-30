# Mixing beta

## 核心问题

`mixing_beta` 控制新旧电荷密度混合比例。过大可能导致振荡，过小可能收敛很慢。

## 常见场景

- 金属、大超胞、缺陷、低维体系常需要更保守的 mixing。
- relax 中每个离子步都可能触发新的 SCF 困难。

## 记录规范

记录修改前后 SCF iteration 数和 estimated accuracy 走势。

