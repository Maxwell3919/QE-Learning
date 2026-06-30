# Smearing and metal

## 错误长什么样

SCF 振荡、Fermi level 附近 DOS 不稳定、不同 `degauss` 得到不同能量/力趋势。

## 排查重点

- 确认体系是否为金属或小带隙。
- 比较 smearing 类型与 `degauss`。
- 区分结构优化设置和最终 DOS 设置。

## 不能靠调参数解决的情况

如果结构、磁态或电荷态错误，smearing 只能掩盖问题。

