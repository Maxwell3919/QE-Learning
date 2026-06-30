
# Reading Maintenance Policy

本页说明 canonical literature 如何维护。目标是让文献服务于计算判断，而不是让仓库变成论文清单。

## 1. 收录原则

- 只有能改变 QE input、output review、PASS/WARN/BLOCK 或 uncertainty statement 的来源，才进入 canonical 列表。
- A0/A1/A2 列表保持小而稳定；Fast Core Loop 不超过 20 篇。
- B1/B2/C 可以扩展，但必须说明进入哪个专题入口。

## 2. 写入页面的方式

- 正文先写计算判断结论。
- 页末列 `资料来源`。
- 如果一个结论来自摘要级调研，强度标为 `Moderate` 或 `Boundary`。
- 涉及公式、QE 字段、默认值、程序限制、版本差异时，必须回查官方文档或正文，强度标为 `Version-sensitive`。

## 3. 分级变更规则

- 升级到 A0：必须是基础理论或不可跳过的方法原点。
- 升级到 A1：必须直接支撑一个 workflow 或程序链。
- 升级到 A2：必须能防止一个常见 failure mode。
- 降级：如果来源只提供背景而不改变判断，移入 B/C。

## 4. 维护检查

新增或修改 physics judgement 页面时检查：

- 是否有核心判断结论。
- 是否有 QE input 字段。
- 是否有 output review 迹象。
- 是否有 PASS / WARN / BLOCK。
- 是否有 workflow 和 theory-minimum 回链。
- 是否标注结论强度。
- 是否无本地路径和 tracking URL。

## 原始链接

- [canonical-literature.md](canonical-literature.md)
- [citation-and-source-policy.md](../standards/citation-and-source-policy.md)
- [source-index.md](source-index.md)
