# NEB reaction path workflow

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、收敛性、结构 relax、端点结构审阅和赝势记录
- 下游用途：反应路径、迁移路径、构型转变能垒和过渡态候选审阅
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)、[standards/output-review-checklist.md](../../standards/output-review-checklist.md)

## 适用问题

NEB workflow 用于估计两个已知端点结构之间的最小能量路径（minimum energy path, MEP）和能垒。它适合回答：

- 初态和终态之间是否存在可比较的路径。
- 路径最高点附近的能量位置和过渡态候选 image。
- 在同一 Hamiltonian、同一结构约束和同一数值参数下，端点与中间 images 的相对能量变化。
- 路径是否已经在力、能量和几何连续性上达到可审阅状态。

本页是 B 级高级边界页：说明进入 NEB 前需要具备的判断、QE 输入字段和 output review 边界，不展开结构构建、路径插值算法细节或具体材料案例。

## 进入该 workflow 前必须已经掌握什么

- 已能独立完成 `pw.x` SCF，并能审阅 cutoff、k-points、pseudopotential、occupation/smearing 和 electronic convergence。
- 已能完成 `pw.x` `relax` 或 `vc-relax` 的 output review，并能判断最终结构是否允许进入下游 workflow。
- 已能区分 input 坐标、relax 后坐标、static SCF 坐标和 restart/scratch 文件的用途。
- 已明确端点结构的物理含义、原子数、元素顺序、晶胞、约束条件和电荷/自旋设定是否可比。
- 已理解 NEB 的结果不是单个结构优化结果；它是一组 images 的路径优化结果，需要逐 image 审阅。

## 输入前提

- 两个端点结构已经分别完成必要的结构审阅；不能把未收敛 relax 输出直接当作可信端点。
- 端点使用同一套赝势、cutoff、k mesh 策略、occupation/smearing、spin 设置和晶胞约束。
- 初始、中间和最终 images 的原子数、元素顺序、晶胞定义和坐标单位一致。
- 已记录路径 images 的来源、生成方式和是否固定端点；本仓库不展开 images 构建流程。
- 已规划独立的 `prefix`、`outdir` 和运行记录，避免与端点 relax/static SCF 或其他 NEB 尝试混用 scratch。

## 计算图

```text
relaxed/checkable endpoint A + relaxed/checkable endpoint B
  + shared SCF policy + ordered path images
  -> neb.x path optimization
  -> image energies + path forces + MEP review
  -> barrier estimate + transition-state candidate review
```

## 关键 QE 程序或输入字段

| 字段或程序 | 所属位置 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `neb.x` | QE NEB / PWneb | 运行路径优化；调用 PWscf engine 计算各 image 的能量和力 | 当作 `pw.x` 输入运行，或误以为会从标准输入读取 | 运行命令、NEB header、生成的 path/PWscf 输入与输出文件 |
| `BEGIN_PATH_INPUT` / `&PATH` | `neb.x` 单文件输入 | 放置 NEB 路径控制参数 | 路径参数和 PWscf engine 参数混在一起 | output 或生成文件中能看到解析后的 path control |
| `BEGIN_ENGINE_INPUT` | `neb.x` 单文件输入 | 放置每个 image 共用的 PWscf engine 参数 | 与端点 SCF/relax 参数不一致导致能量不可比 | output 中 pseudopotential、cutoff、k-points、occupation 等与记录一致 |
| `restart_mode` | `&PATH` | 控制从头运行或从中断路径继续 | 未记录 restart 来源，导致旧路径状态污染新判断 | output 和目录记录显示 from-scratch/restart 来源 |
| `num_of_images` | `&PATH` | 路径离散点数量，包含端点和中间 images | images 太少导致路径粗糙；数量和实际结构块不一致 | output 中 image 数量、每个 image 的能量/力均有记录 |
| 路径 images / positions | NEB positions cards 或 image 输入 | 给出有序路径结构 | 原子顺序错、端点反了、路径不连续、坐标单位混乱 | 逐 image 坐标、路径长度、几何连续性和异常跳变 |
| 端点结构 | first / last image | 定义反应、迁移或构型转变的初态和终态 | 端点未 relax、晶胞/约束不一致、端点能量不可比 | first/last image 的坐标、能量、约束和端点保持状态 |
| `string_method` | `&PATH` | 选择路径方法；NEB workflow 通常使用 `neb` | 混淆 NEB 与其他 string method 设置 | output 中 path method 与记录一致 |
| `nstep_path` | `&PATH` | 限制路径优化步数 | 达到步数上限却误判为收敛 | output 末尾的 path step、收敛状态和停止原因 |
| `path_thr` | `&PATH` | 路径力收敛阈值 | 只看能垒，不看 path force 是否低于阈值 | output 中 path force / convergence message |
| `opt_scheme` | `&PATH` | 路径优化算法，例如 steepest descent、Broyden 或 quick-min 类方案 | 优化不稳定时盲目改参数，不保留前后记录 | output 中 optimizer、step history、是否震荡或停滞 |
| `ds` | `&PATH` | 路径优化步长或 Broyden 初始 Jacobian 尺度 | 步长过大导致震荡，过小导致停滞 | path step history、image 位移和收敛速度 |
| `k_max` / `k_min` | `&PATH` | 弹簧常数边界，影响路径 images 的分布 | 弹簧设置导致 image 聚集、路径折叠或最高点附近采样不足 | image 间距、路径连续性和最高能量附近分辨率 |
| `CI_scheme` | `&PATH` | climbing image 策略，用于改进最高能量 image 附近的鞍点定位 | 过早开启 CI；最高点尚不稳定时误认过渡态 | output 中 CI 状态、climbing image 编号和最高能量 image 演化 |
| `CLIMBING_IMAGES` | NEB card | 手动指定 climbing images | 手动编号与当前路径最高点不符 | output 中被指定 climbing 的 image 与能量排序一致 |
| `first_last_opt` | `&PATH` | 允许端点在路径优化中同步优化 | 端点定义被改变，能垒参考态不再是原审阅端点 | first/last image 坐标和端点状态变化 |
| `minimum_image` | `&PATH` | 对周期性跨胞位移使用 minimum-image 处理 | 用开关掩盖错误路径或坐标跳变 | 相邻 images 的周期性位移和路径连续性 |

## climbing image 和优化参数边界

- climbing image 适合在路径已经基本合理、最高能量区域可识别后使用；过早开启可能放大坏路径或坏 image 的问题。
- `CI_scheme = 'auto'` 依赖当前路径最高能量 image 的识别；`manual` 需要同时记录 `CLIMBING_IMAGES` 的编号和选择理由。
- `path_thr` 约束的是路径优化收敛，不等同于单个端点 relax 的 `forc_conv_thr`；两者不能互相替代。
- `nstep_path` 到达上限只说明运行停止，不说明路径已收敛。
- `opt_scheme`、`ds`、`k_max`、`k_min`、image 数量和初始路径质量相互耦合；修改这些参数后应视为新的 NEB 尝试并保留独立记录。
- `first_last_opt` 会改变端点处理方式；如果端点已经来自可信 relax，通常需要明确说明为什么允许端点继续移动。
- `minimum_image` 可帮助处理周期性位移，但不能替代人工审阅路径 images 是否真的表示同一物理过程。
- 如果端点本身不可比，调节 NEB 优化参数不能修复物理问题。

## 通用输出审阅模板

```markdown
## NEB output review

- QE 程序:
- QE version:
- NEB run command:
- Input mode: single input / input_images
- Endpoint source:
- Image count:
- Shared PWscf policy:
- Pseudopotentials loaded:
- Cutoff reported:
- K-points reported:
- Occupation / spin settings:
- Path optimizer:
- Climbing image setting:
- Path convergence status:
- Max path force / path error:
- Image energy table recorded:
- Highest-energy image:
- Barrier estimate:
- Endpoint status:
- Image continuity review:
- Warnings:
- Scratch / restart status:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream use:
```

## output review 要点

- 确认 `neb.x` 正常解析输入；如果使用单文件输入，应能区分 NEB path input 和 PWscf engine input。
- 检查每个 image 是否都有能量、力和路径位置信息；不能只摘最高能量差。
- 检查 `path_thr` 或等价路径收敛信息；`JOB DONE` 或队列正常结束不等于 NEB 路径可信。
- 检查每个 image 的 PWscf 计算是否电子收敛；单个 image 的 SCF 异常会污染整条路径。
- 检查 first/last image 是否仍对应目标端点；端点漂移、端点替换或端点约束不一致会破坏能垒含义。
- 检查路径连续性：相邻 images 不应出现无法解释的原子交换、坐标跳变、跨晶胞误读或结构断裂。
- 检查最高能量 image 与 climbing image 设置是否一致；如果 CI 结果和能量排序冲突，需要解释。
- 检查 image 间距和能量曲线是否存在明显折叠、尖峰或采样不足。
- 记录能垒时同时记录能量零点、端点选择、最高 image 编号和是否为收敛路径。
- 将结果分类为 PASS / WARN / BLOCK；WARN 可以用于探索性比较，BLOCK 不应进入论文图、数据库或下游动力学解释。

## 常见风险

| 风险 | 表现 | 处理原则 |
|---|---|---|
| 端点不可比 | 初态/终态晶胞、原子顺序、约束、磁态或数值参数不同 | 回到 endpoint relax/static SCF 审阅 |
| images 构造错误 | 路径跳变、原子交换、结构断裂、端点反向 | 重新审阅 images 来源；本页不展开构建 |
| 只看能垒 | output 中没有 path force、image convergence 或 continuity review | 将结果降级为 WARN/BLOCK |
| climbing image 误用 | CI image 不是稳定最高点，或路径尚未粗收敛 | 先获得可解释的 no-CI 或粗路径，再审阅 CI |
| 参数漂移 | 中途修改 cutoff、k mesh、smearing、spin 或赝势 | 视为不同计算，不能直接比较 |
| restart 污染 | 旧 `prefix/outdir` 或残留 image 文件被复用 | 使用独立 scratch；记录 restart 来源 |
| 并行/文件组织错误 | 部分 image 缺输出、编号错乱、生成文件为空 | 先检查输入解析、image 文件、运行命令和目录结构 |
| 端点未收敛 | 路径最高点看似合理但端点力很大 | 端点回到 relax workflow；NEB 不替代端点优化 |

## 与基础 workflow 的关系

- NEB 依赖 SCF workflow 的可比能量基础：同一 pseudopotential、cutoff、k mesh、occupation/smearing、spin 和收敛阈值策略。
- NEB 依赖 relax/vc-relax workflow 的结构可信度：端点结构必须先通过结构和力的 output review。
- NEB 的 engine input 本质上仍是 PWscf 设置；凡是会改变 Hamiltonian 或能量零点的基础字段，都必须继承并记录。
- NEB 结果通常不能直接替代 phonon、bands、DOS 或 transition-state 精修；它提供路径和鞍点候选，需要按下游目标再做专项审阅。
- 如果 NEB 暴露出端点不合理、路径断裂或 SCF 不稳定，应回到基础 workflow 修复，而不是继续堆叠高级开关。
- NEB 默认审阅的是 minimum energy path 和势能面上的 barrier 候选；它不是 free-energy path。有限温或熵相关结论需回查 [physics-judgement/energy-vs-free-energy.md](../../physics-judgement/energy-vs-free-energy.md)。

## 资料来源

- QE INPUT_NEB reference: <https://www.quantum-espresso.org/Doc/INPUT_NEB.html>
- QE PWneb user guide: <https://www.quantum-espresso.org/Doc/neb_user_guide/>
- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
