# Project Realignment Audit

本审计用于把 QE-Learning 重新对齐到一个明确目标：中文 Quantum ESPRESSO 计算参考手册与 DFT 固体计算知识库。它服务科研人员学习和复查 QE 计算，重点是 input/output 阅读、workflow 依赖、数值收敛、DFT 理论边界、物理判断和可复查记录。

本轮审计先读仓库现状，再按信息架构、workflow、最低理论、物理判断、来源边界和 standards 六个工作流检查。审计结论先记录缺口和优先级，后续修改应小批量推进，避免把项目写成自动化执行系统、工具接口规范或具体材料案例库。

## 1. 当前目标聚焦程度

当前仓库主体已经聚焦在“QE 参考手册 + DFT 知识体系”：

- `README.md` 能说明学习主线和六个主目录。
- `learn/` 负责能力路线、完成判据和可验证输出。
- `workflows/` 负责 QE 原生命令行 workflow、input/output 和下游准入。
- `theory-minimum/` 负责正确使用 QE 所需的最低 DFT/DFPT 理论。
- `physics-judgement/` 负责更高层的 DFT/QE 物理判断、失败模式和解释边界。
- `references/` 负责公开来源导读。
- `standards/` 负责文件命名、计算记录、引用政策、output review 和 `PASS / WARN / BLOCK`。

总体方向正确。主要问题不是缺少页面，而是少数页面定位和质量还不够“参考手册化”：入口页需要更直接地强调手册定位，`physics-judgement/` 存在双轨重复，部分 workflow 页面仍偏通用框架，references 有少量失效链接和文献/BibTeX 不一致。

## 2. 偏离目标的内容

| 位置 | 问题 | 处理建议 | 优先级 |
|---|---|---|---|
| 根目录本地工作痕迹 | `task_plan.md`、`findings.md`、`progress.md` 是执行过程文件，不属于手册正文 | 从工作区移除，并加入忽略规则 | P0 |
| `.DS_Store` | 本地系统文件出现在工作区 | 删除并加入忽略规则 | P0 |
| `physics-judgement/CONCLUSION_REFERENCE_BOOK.md` | 开头出现工具协作语境 | 改成“页面维护和审阅分工” | P0 |
| `physics-judgement/README.md` | “authoritative literature loop”“结论型补写页面”偏维护过程叙事 | 改为读者视角：系统主题页、快速判断页、审阅清单 | P1 |
| `learn/09-feature-expansion-map.md` | 容易被误读为基础学习阶段 | 在 `learn/README.md` 明确它是扩展功能地图，不是基础闭环必修阶段 | P1 |

未发现需要引入自动化执行系统、工具接口规范或外部项目执行文档的内容。后续维护应继续避免这些方向进入正文。

## 3. 过于空泛或仍偏模板的页面

workflow 页面整体已经有 output review 和 `PASS / WARN / BLOCK`，但深度不均。

| 页面 | 当前状态 | 建议 |
|---|---|---|
| `workflows/electronic/fermi-surface.md` | 偏短，`fs.x` / BXSF / XCrySDen 边界不够具体 | 补 command boundary、NSCF mesh、Fermi level、BXSF 输出审阅 |
| `workflows/electronic/charge-density.md` | `pp.x` 字段和输出格式边界偏少 | 补 `plot_num`、`iflag`、`output_format`、可视化边界 |
| `workflows/electronic/elf.md` | 可视化解释边界偏短 | 补 ELF 与 charge density 区别、定性/定量边界 |
| `workflows/phonon/ir-raman.md` | 官方支持、response tensor、build/version 边界不够清楚 | 补 `lraman`、`epsil`、Gamma phonon 前提、版本敏感字段 |
| `workflows/phonon/gamma-phonon.md`、`phonon-dos.md`、`dielectric-born-effective-charge.md` | 比主 DFPT dispersion 页面薄 | 补 PHonon output evidence、dyn/fc/freq 文件链和下游准入 |
| `workflows/ground-state/relax.md`、`vc-relax.md` | 可用，但可进一步参考手册化 | 补 force/stress/pressure 的 command boundary 和 final static SCF 准入表 |

高级页面整体符合边界页定位，不应扩写成完整专题教程。优先补“输入字段、output review、版本边界、下游准入”，而不是增加理论篇幅。

## 4. output review 缺口

自动扫描显示，`workflows/` 页面没有完全缺失 output review 的文件。但存在三类问题：

1. compact 页面常用通用审阅模板，缺少该 workflow 特有的 output 证据；
2. 部分页面没有明确的 `Allowed downstream workflows` 字段；
3. version-sensitive 字段只列来源，没有说哪些字段必须按当前官方文档核验。

优先修正方式：

- 每个 workflow 页增加或强化 `Command boundary`：executable、input、output、上游文件、`prefix/outdir`、handoff。
- 每个短页面补一个 page-specific `Output evidence` 表。
- 每个短页面补一个 `Source / version boundary` 表，指向 QE 官方 `INPUT_*` 或工具官方文档。

## 5. 来源边界缺口

`references/` 结构正确，但需要修复以下问题：

| 问题 | 影响 | 处理建议 |
|---|---|---|
| 少量外部链接失效 | 公开来源追踪不稳 | 修复 QE resources、Yambo documentation、ASE espresso IO 链接 |
| `canonical-literature.md` 20 篇，`canonical.bib` 约 14 条 | BibTeX spine 不完整 | 补齐缺失条目、DOI、volume、pages，并对齐 key |
| advanced package map 的部分条目只有“推荐来源”文本 | 用户难以回到公开来源 | 在 `references/source-index.md` 增加 XSpectra、TurboTDDFT、TurboEELS、PWcond、GIPAW、KCW 等入口 |
| 个别 physics judgement 页面来源过于通用 | 主题判断支撑不够精确 | Wannier/EPC/work function 页面优先换成工具或方法专项来源 |

所有涉及 QE 参数、默认值、可选值、程序行为和版本差异的判断，应以官方 `INPUT_*`、package guide 或工具官方文档为准。论文和教程只能支撑物理边界或学习路径，不能替代参数定义。

## 6. DFT 理论支撑缺口

`theory-minimum/` 已经能服务 QE input/output，但还有几个 landing page 缺口：

| 缺口 | 当前位置 | 建议 |
|---|---|---|
| functional choice minimum | 分散在 `dft-ks-scf.md`、`pseudopotentials.md` 和 `physics-judgement/` | 新增或在 README 明确 functional route |
| KS eigenvalue / ground-state boundary | 主要在 `physics-judgement/` | 在 `dft-ks-scf.md` 或 README 加强最低边界提示 |
| finite-size / boundary / work function | 主要在 `physics-judgement/` 和 workflow | 增加 theory-minimum route，或修正链接到现有页面 |
| Wannier / EPC / uncertainty | 主要在 advanced 和 standards | 保持 advanced/physics/standards 分工，不建议放入 theory-minimum 主线 |

`magnetism-soc-dftu.md` 同时承载磁性、SOC、DFT+U，当前可用，但后续如果继续扩写，应拆成查询更细的页面或在内部增加清晰锚点。

## 7. 重复与合并建议

`physics-judgement/` 现在同时有编号系统页和结论型快速页。它们的目标不同，但主题大量重叠。

建议采用“一主题一 canonical 页”的长期策略：

- 编号页作为 canonical topic page；
- 结论型页面逐步合并为编号页顶部的 judgement card；
- `CONCLUSION_REFERENCE_BOOK.md` 保留为快速路由和全局判断内核；
- `18-physics-judgement-checklist.md` 保留为审阅清单；
- standalone 结论页在合并后降级为索引/跳转页，避免长期重复维护。

优先合并候选：

| 重复组 | 建议 |
|---|---|
| `functional-choice-and-sensitivity.md` + `02-*` | 合并成 functional model canonical 页 |
| `band-gap-problem-and-delocalization.md` + `03-*` + `04-*` | 明确 self-interaction、delocalization、gap boundary 的层级 |
| `u-value-provenance-and-boundary.md` + `05-*` | 将 U provenance 作为强关联页 judgement card |
| `soc-and-symmetry-boundary.md` + `06-*` | 将 SOC/symmetry 作为磁性页 judgement card |
| `work-function-and-electrostatic-boundary.md` + `11-*` | 合并表面/电势/work-function 判断 |
| `wannier-validation-and-window-choice.md` + `13-*` | 合并 Wannier validation |
| `epc-data-chain-and-convergence.md` + `14-*` | 合并 EPC data chain |
| `uncertainty-statement-template.md` + `17-*` | physics 页讲判断，fillable template 进入 `standards/` |

## 8. 目录微调建议

不建议新增顶层目录，也不建议新增 case、archive、structure-learning、examples 或 tutorial-cases。

建议的微调只限于：

1. 加 `.gitignore`，忽略 `.DS_Store` 和本地 planning 文件；
2. 清理当前工作区里的 `.DS_Store` 和 planning 文件；
3. 保留六个文档目录；
4. 后续如新增 theory-minimum 页面，应以“QE 使用判断”命名，不以教材章节命名。

## 9. 优先级排序

### P0：项目边界与污染清理

- 移除本地 planning 文件和 `.DS_Store`；
- 删除正文中的自动化执行系统语境；
- 修复最明显 broken links；
- 根 README 强化“QE 计算参考手册 + DFT 知识体系”定位。

### P1：入口页与 standards 一致性

- `physics-judgement/README.md` 改成读者视角；
- `learn/README.md` 标明 feature map 是扩展地图；
- `standards/output-review-checklist.md` 修正“四步/五步”不一致；
- 统一 record schema 的字段名：`Status`、`Reason`、`Evidence`、`Downstream allowed`、`Uncertainty statement`。

### P2：高价值 workflow 精修

- `fermi-surface.md`、`charge-density.md`、`elf.md`、`ir-raman.md`；
- pp.x 相关页面补 `plot_num`、`iflag`、`output_format`、`average.x`；
- phonon 二级页面补 PHonon output evidence 和 version boundary。

### P3：理论与 physics judgement 去重

- 建立 canonical routing table；
- 合并或降级重复结论页；
- 补 functional choice / finite-size route；
- 将 uncertainty fillable template 迁入 standards 或链接到 standards。

### P4：文献与 BibTeX 完整性

- 补齐 `canonical.bib`；
- 为 advanced feature map 增加 source-index 入口；
- 对所有 version-sensitive 字段标出官方文档来源。

## 10. 执行计划

1. Commit 1：提交本审计文件，冻结对齐目标和优先级。
2. Commit 2：修正入口页、清理 planning 痕迹、添加忽略规则、修复 broken links。
3. Commit 3：精修 `standards/` 的 record schema、output checklist 和 source policy。
4. Commit 4：补强高价值 workflow 页面，优先 pp.x、Fermi surface、IR/Raman 和 phonon 二级页面。
5. Commit 5：补 functional/finite-size theory route，整理 physics judgement canonical routing。
6. Commit 6：补 references/canonical BibTeX，运行 QA，提交 `PROJECT_COMPLETION_REPORT.md`。

每个阶段都必须运行：Markdown 本地链接检查、禁用词/本地路径/固定材料案例扫描、README 与目录索引一致性检查、`git diff --check`。
