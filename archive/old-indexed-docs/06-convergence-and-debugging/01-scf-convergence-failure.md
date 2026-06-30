# SCF convergence failure

## 错误长什么样

SCF iteration 达到最大步数仍未收敛，estimated accuracy 不下降或振荡。

## 可能原因

结构差、金属 smearing 不合适、`mixing_beta` 太大、赝势/cutoff/k 点问题、初始磁性或电荷态设置错误。

## 优先排查顺序

1. 确认结构和赝势。
2. 检查 occupation/smearing。
3. 降低 `mixing_beta`。
4. 调整 k 点和 cutoff。
5. 必要时改变初始磁性或对角化策略。

## 记录规范

记录失败 input、output 最后 50-100 行、energy/accuracy 走势、尝试过的参数。

