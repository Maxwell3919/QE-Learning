# QE Project Layout

这是个人计算项目建议结构，不是本仓库目录结构。本仓库只保存学习笔记和规范，不要求保存具体材料体系的 input/output、scratch 或结果文件。

该结构适合放在个人计算工作区、课题目录或集群同步目录中，用于把结构来源、赝势来源、输入输出、脚本、记录、图像和临时 scratch 数据分开。目标是让每次计算都能通过 record 找回关键证据、运行环境和下游依赖，而不要求长期保存所有 QE 生成文件。

## 1. 通用目录模板

```text
<project>/
  README.md
  structures/
  pseudo-source/
  inputs/
  outputs/
  scripts/
  records/
  figures/
  notes/
```

这是建议模板，可以按实际项目缩减或增加子目录。目录名保持通用含义，不应依赖某个具体材料案例。

## 2. 各目录保存什么

### `README.md`

保存项目级说明：

- 研究或学习目标。
- 主要 workflow，例如 SCF、relax、bands、DOS、phonon。
- 目录结构说明。
- 当前最可信 record 的索引。
- 数据保留策略，例如哪些 scratch 在集群上保留，哪些只保留摘要。

`README.md` 不替代单次计算记录。它只说明项目整体状态和入口。

### `structures/`

保存长期可追溯的结构输入和结构变换记录，例如：

- 原始结构文件。
- 标准化、超胞、约束、坐标转换后的结构文件。
- relax 或 vc-relax 后准备进入下游 workflow 的结构文件。
- 结构来源说明和变换脚本输出摘要。

不建议把 QE scratch 中的临时结构副本当作长期结构来源。进入下游计算的结构应明确写入 record 的 `Structure source` 字段。

### `pseudo-source/`

保存赝势来源和选择依据，而不是只保存一组匿名 `.UPF` 文件。建议包括：

- 使用的赝势库、版本和下载来源。
- exchange-correlation、PP type、推荐 cutoff。
- 实际使用的 `.UPF` 文件清单或校验信息。
- 与赝势选择相关的备注。

如果 `.UPF` 文件较小，可以随项目长期保存；如果由统一赝势库集中管理，也可以只在此处保存来源、路径和校验信息。对应信息应能填入 record 的 `Pseudopotential source` 字段。

### `inputs/`

保存人工维护、可复查的 QE input 文件。命名应遵循 [input-file-naming.md](input-file-naming.md)，例如：

```text
pw.scf.<system>.in
pw.relax.<system>.in
ph.<system>.in
dos.<system>.in
```

`inputs/` 保存的是可读输入，不是 QE 运行时写出的 scratch。若同一 workflow 有多轮参数修改，应让文件名或子目录能区分轮次，并在 record 的 `Input files` 和 `Parameter changes from previous run` 中说明。

### `outputs/`

保存值得长期复查的文本 output 和轻量结果文件，例如：

- `pw.x`、`ph.x`、`dos.x`、`bands.x` 等程序的 `.out`。
- 关键 `.dat`、`.freq`、`.fc`、`.dyn` 等小型结果文件。
- output review 所需的错误、warning、收敛和最终物理量证据。

`outputs/` 不应无选择地混入 QE 的大型 scratch 树。长期保存 output 的目的，是让 record 中的 `Output summary`、`Convergence status`、`Physical result` 和 `Problems encountered` 可以被复查。

### `scripts/`

保存项目中实际使用过、或计划重复使用的脚本，例如：

- 批量提交或本地运行脚本。
- 结构转换、输入生成、结果提取脚本。
- 绘图和后处理脚本。
- 校验文件清单、hash 或记录一致性的脚本。

脚本应尽量可重复执行，并在 record 的 `Command`、`Environment` 或 `Next action` 中说明何时使用。一次性手工命令也应写入 record，不应只留在 shell history。

### `records/`

保存按 [calculation-record-template.md](calculation-record-template.md) 书写的单次或单阶段计算记录。每个可复查计算至少应能回答：

- 使用了哪个结构来源。
- 使用了哪些赝势。
- 使用了哪些 input。
- 在什么环境、目录、`prefix` 和 `outdir` 下运行。
- output 中有哪些关键证据。
- 当前状态是 `PASS`、`WARN` 还是 `BLOCK`。
- 下游 workflow 是否可以继续。

`records/` 是长期记录的中心。即使某些 input/output 或 scratch 不保存在同一个仓库，record 也必须写清楚它们的位置、版本和保留策略。

### `figures/`

保存由计算结果生成、可用于讨论或汇报的图像，例如能带、DOS、声子谱、收敛曲线和结构示意图。图像应能追溯到对应 record、脚本和输入输出。

不建议把未标注来源的截图作为充分证据。截图可以辅助阅读，但最终判断应回到 output、record 和生成脚本。

### `notes/`

保存项目内的非正式备注，例如：

- 临时想法。
- 参数选择讨论。
- 文献和教程摘记。
- 失败尝试的解释。

`notes/` 可以帮助回忆上下文，但不替代 calculation record。凡是影响下游判断的事实，都应整理进 `records/`。

## 3. 不应混入长期记录的大文件

QE 会产生大量 side-effect 文件。以下内容通常不应直接混入长期记录目录，也不应默认提交到文档仓库：

- `outdir` 下的 `.save/` 目录。
- wavefunction、charge-density、restart 文件。
- `_ph0/`、临时 dynamical matrix scratch、并行 I/O 中间文件。
- 大型 scheduler log、core dump、临时 stdout/stderr。
- 可由 input、赝势、结构和命令重新生成的大型中间文件。

这些文件是否保留取决于计算成本和复现需求。建议做法是：

- scratch 大文件留在本地磁盘、集群 scratch 或归档存储。
- 在 record 的 `Environment` 中写清 `prefix`、`outdir`、working directory 和 scheduler job id。
- 在 record 的 `Problems encountered` 或 `Next action` 中说明是否需要保留 restart。
- 对需要长期复查但体积较大的文件，保存路径、文件清单、hash 或压缩包索引，而不是把它们混入 `records/`。

## 4. 与 calculation record 的对应关系

一个完整 record 应把目录结构中的证据串起来：

```text
Structure source        -> structures/
Pseudopotential source  -> pseudo-source/
Input files             -> inputs/
Command / Environment   -> scripts/ and actual working directory
Output summary          -> outputs/
Convergence status      -> outputs/ plus review notes
Physical result         -> outputs/ and figures/
Problems encountered    -> outputs/, scheduler logs, notes/
Next action             -> records/ and downstream inputs/
```

最低要求：每次可复查计算都应有 input、output、record、赝势来源、运行命令和 output review。record 不要求所有文件都与它位于同一仓库，但必须让读者知道关键文件在哪里、哪些文件是长期证据、哪些只是 scratch。

如果某个计算进入下游 workflow，例如从 SCF 进入 bands、DOS 或 phonon，下游 record 必须写清楚读取的是哪个上游计算的 `prefix/outdir`，以及上游状态是否达到 `PASS`。程序完成不等于结果可用；只有 record 中的证据支持下游，才应继续作为可靠输入。
