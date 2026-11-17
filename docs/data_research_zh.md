# SmartDoc 数据调研与构建方案

调研日期：2026-11-18。重点检索 2024-2026 年论文，并保留少量更早的基础数据集/方法作为基线。论文状态以链接页面为准；预印本、正式发表论文和数据集许可不能混为一谈。

## 1. 项目目标与数据问题

SmartDoc 的数据工作要支持一个本地优先的文档智能体：读 PDF、Word、Excel 等输入，利用解析、检索、表格读取等工具，完成总结、单/多文档问答、比较、字段抽取和段落解释，并返回能核验的页码/区域/表格位置。公开网页搜索只补充公开背景知识，不用于上传或检索用户的保密文档。

数据至少要回答四个不同问题：

1. **答案对不对**：答案与可核验参考答案是否一致，缺证据时是否拒答。
2. **证据找得准不准**：是否找到支持答案的文档、页、段落、表格/单元格；证据是否真的蕴含结论。
3. **智能体过程是否有效**：是否选对工具、参数是否合理、是否能根据工具反馈修正、是否有不必要的调用或陷入循环。
4. **资源是否可用**：端到端耗时、工具调用数、解析/检索成本、峰值内存，以及小模型下的成功率。

传统 QA 的 `(文档, 问题, 答案)` 三元组主要覆盖第 1 项；它无法充分训练或评估智能体的执行策略。建议将 benchmark、SFT 示范、RL 环境/轨迹和过程评估标注分开建设，并在数据清单中标明每条数据的用途和许可。

## 2. 论文与数据集调研

### 2.1 长文档、多模态文档与证据定位

| 工作 | 年份 | 可借鉴点 | 建议用途 |
|---|---:|---|---|
| [MMLongBench-Doc: Benchmarking Long-context Document Understanding with Visualizations](https://arxiv.org/abs/2407.01523) | 2024 | 长文档理解，问题需要定位视觉页面中的信息；标注形式可启发页面级证据 | 只作为评测来源/设计参考；核对数据许可和版本 |
| [LongDocURL: A Comprehensive Multimodal Long Document Benchmark Integrating Understanding, Reasoning, and Locating](https://doi.org/10.18653/v1/2025.acl-long.57) | 2025 | 把理解、推理、定位放在同一长文档评测中，和 SmartDoc 的证据定位目标很接近 | 评测对照；借鉴定位标注，不直接混入训练集 |
| [MMDocIR: Benchmarking Multimodal Retrieval for Long Documents](https://doi.org/10.18653/v1/2025.emnlp-main.1576) | 2025 | 长文档的多模态检索问题，可用于拆分“检索没找到”和“找到后答错” | 检索子任务的评测对照 |
| [MEBench: Benchmarking Large Language Models for Cross-Document Multi-Entity Question Answering](https://doi.org/10.18653/v1/2025.emnlp-main.77) | 2025 | 跨文档、多实体关联；适合多文档比较和聚合类问题 | 多文档评测参考 |
| [DocAgent: An Agentic Framework for Multi-Modal Long-Context Document Understanding](https://doi.org/10.18653/v1/2025.emnlp-main.893) | 2025 | 直接研究多模态长文档的 agent 化处理，是本项目最贴近的系统工作之一 | 重点复现其任务分解、工具和评测设计；查代码/数据许可 |
| [MMLongBench-Doc-V2](https://arxiv.org/abs/2608.03397) | 2026 | 对原基准做标注修订和语义层面对齐，提醒我们对公开 benchmark 的答案和证据要抽检 | 评测前核对修订版和原始版的差异 |
| [Towards Agentic Document-Grounded Information Seeking](https://doi.org/10.18653/2026.emnlp-main.954) | 2026 | 将文档作为信息源、通过 agent 多步检索获取信息，贴近文档检索轨迹与流程级监督 | 借鉴任务与轨迹构造；确认论文发布版及代码数据状态 |
| [DocLens: A Tool-Augmented Multi-Agent Framework for Long Visual Document Understanding](https://doi.org/10.18653/v1/2026.acl-long.1234) | 2026 | 多工具/多 agent 处理长视觉文档，可用于比较单 agent 与工作流方案 | 后续系统设计参考，不预设多 agent 一定优于单 agent |
| [LongBench](https://doi.org/10.18653/v1/2024.acl-long.172) / [LongBench v2](https://doi.org/10.18653/v1/2025.acl-long.183) | 2024/25 | 双语长上下文、多任务与更深层推理评测 | 通用长上下文基线，不代替文档页码/视觉定位评测 |
| [OmniDocBench](https://arxiv.org/abs/2412.07626) | 2024 | 文档解析/版面结构评估，可帮助把解析器质量从问答质量中分离 | 解析工具基线；不作为 agent QA 主 benchmark |

**结论**：最适合作为直接评测参考的是 LongDocURL、MMLongBench-Doc（优先检查 V2）、MMDocIR、MEBench 和 DocAgent。它们覆盖长文档、视觉证据、多文档和 agent 流程的不同部分，不能简单合并成一个分数。当前应先做公开集的许可/可获得性核验，再锁定 2-3 个主评测集和一个内部留出集。

### 2.2 文档问答及领域数据

| 数据集/工作 | 年份 | 适用范围 | 使用建议 |
|---|---:|---|---|
| [DocVQA](https://www.docvqa.org/) | 2020/21 | 文档图像问答、表单/版面信息 | 适合视觉文档问答基线；遵守官方访问和许可条款 |
| [DUDE](https://arxiv.org/abs/2305.08455) | 2023 | 多样化文档理解与问答 | 可做跨文档类型测试；先查数据条款 |
| [QASPER](https://allenai.org/data/qasper) | 2021 | 科学论文问答，含不可回答问题 | 训练/评测科学文档任务；论文来源与再分发需核查 |
| [FinQA](https://github.com/czyssrs/FinQA) | 2021 | 财报中的数值推理与程序化计算 | 可作金融表格/计算子集，不代表通用文档智能 |
| [TAT-QA](https://github.com/NExTplusplus/TAT-QA) | 2021 | 财务文本+表格的混合推理 | 用于表格证据和计算题；检查原始数据许可 |
| [MultiHiertt](https://github.com/psunlpgroup/MultiHiertt) | 2022 | 多跳表格/文本推理 | 用于多步证据组合和计算 |
| [Doc2Dial](https://doc2dial.github.io/) | 2021 | 文档引导的对话 | 可借鉴基于文档的多轮追问；不是 agent 工具轨迹数据 |
| [MMLongBench](https://arxiv.org/abs/2407.01523) | 2024 | 长文档多模态理解 | 适合上下文长度压力评测 |

这些公开集更适合做**基线、预训练式任务或外部测试**。公开测试集不能并入训练；即使训练集可下载，也要确认原文、图表和附件的版权/许可是否允许再训练、再分发。金融/法律/医疗内容尤其不能因为论文作者公开了数据就假定可商用或可公开镜像。

**可先拿来试跑的具体清单**：

| 优先级 | 数据 | 获取方式/当前可见许可信息 | 建议怎么用 |
|---|---|---|---|
| P0 | [QASPER](https://huggingface.co/datasets/allenai/qasper) | HF 数据集卡可访问，卡片标记 CC-BY-4.0；论文/源论文的再分发权仍需单独确认 | 先跑科学论文 QA 和不可回答能力；按文档隔离 split，测试部分只评测 |
| P0 | [FinQA](https://github.com/czyssrs/FinQA) | GitHub 仓库公开，仓库标记 MIT；底层财报/问题数据的具体许可应以随数据发布的 terms 为准 | 表格/文本数值推理与可验证计算基线 |
| P0 | [TAT-QA](https://github.com/NExTplusplus/TAT-QA) | GitHub 仓库公开，仓库标记 MIT；确认仓库许可对数据文件也适用后再训练/再分发 | 混合表格+文本的证据及算术基线 |
| P1 | [DocVQA](https://www.docvqa.org/) | 官方站提供数据/挑战入口，数据可能需要按官方流程申请；本次未确认一个统一、无门槛的重分发许可 | 测视觉文档 QA 与 OCR/版面工具，不先镜像到仓库 |
| P1 | [LongDocURL](https://doi.org/10.18653/v1/2025.acl-long.57)、[MMDocIR](https://doi.org/10.18653/v1/2025.emnlp-main.1576)、[MEBench](https://doi.org/10.18653/v1/2025.emnlp-main.77) | 论文已发表；代码/数据入口和许可要从论文项目页逐个核验 | 作为长文档定位、检索、多文档基准；先不假定能训练或再分发 |
| P1 | [MMLongBench-Doc / V2](https://arxiv.org/abs/2608.03397) | V2 为 2026 预印本修订；需要确认数据下载和使用条款 | 先对照原版与修订版标注，避免将旧版误差当成模型错误 |
| P2 | 合成文档集（项目自建） | 自己生成源文档、答案和页级证据，仓库可只公开生成器和小型 CC0 样例 | 作为首选 SFT/RL 训练源；便于生成失败案例和可验证奖励，也最容易控制泄漏 |

上述许可状态是调研时对官方/托管页面的快照，不是法律意见；下载前应保存原始 license/terms 文件和版本哈希。仓库里只提交小型合成样例，不提交上述基准原始数据。

### 2.3 工具调用、智能体轨迹与 Agentic RL

| 工作 | 年份 | 核心启发 | 对 SmartDoc 的用途 |
|---|---:|---|---|
| [τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains](https://arxiv.org/abs/2406.12045) | 2024 | 工具、用户状态和环境反馈共同构成多轮任务 | 借鉴可重放环境、状态变化和端到端成功判定 |
| [ToolACE: Winning the Points of LLM Function Calling](https://arxiv.org/abs/2409.00920) | 2024 | 规模化合成工具调用指令数据及过滤流程 | 借鉴“工具 schema + 用户意图 + 有效调用”的 SFT 组织；不可把其通用工具数据当文档域数据 |
| [ToolACE-DEV](https://arxiv.org/abs/2505.07512) | 2025 | 通过任务分解与演化扩展工具学习数据 | 借鉴边界案例和复杂意图合成，需自行加入文档证据约束 |
| [Search-R1](https://arxiv.org/abs/2503.09516) | 2025 | RL 学习何时搜索、如何交替推理与检索，并用结果奖励 | 借鉴文档/网页检索决策的奖励设计；区分本地文件搜索与外网搜索 |
| [R1-Searcher++](https://arxiv.org/abs/2505.17005) | 2025 | 用 RL 激励动态知识获取 | 借鉴检索必要性、查询质量和答案正确性的联合奖励 |
| [WebAgent-R1](https://arxiv.org/abs/2505.16421) | 2025 | 多轮环境交互下端到端训练 web agent | 借鉴 rollout、工具返回和终止条件的训练结构 |
| [Agentic Reasoning and Tool Integration for LLMs via Reinforcement Learning](https://arxiv.org/abs/2505.01441) | 2025 | RL 与工具交互结合的整体方法 | 对比训练算法、奖励粒度和基线设置 |
| [An Empirical Study on RL for Reasoning-Search Interleaved LLM Agents](https://arxiv.org/abs/2505.15117) | 2025 | 分析检索与推理交替的训练/评测因素 | 借鉴消融：无搜索、固定检索、学到的检索策略 |
| [The Landscape of Agentic Reinforcement Learning for LLMs: A Survey](https://arxiv.org/abs/2509.02547) | 2025 | 梳理环境、数据、奖励和训练算法 | 作为 Agentic RL 术语与方法综述入口 |
| [TRAJECT-Bench](https://arxiv.org/abs/2510.04550) | 2025 | 以完整轨迹而非最终答案评估工具 agent | 借鉴步骤级错误分类和轨迹评测设计 |
| [Process Reward Models for LLM Agents: Practical Framework and Directions](https://arxiv.org/abs/2502.10325) | 2025 | 讨论 agent 过程奖励的标注和建模实践 | 借鉴过程奖励信号，注意代理评分需人工抽查与校准 |
| [GTA: A Benchmark for General Tool Agents](https://arxiv.org/abs/2407.08713) | 2024 | 通用工具智能体基准 | 补充一般工具选择与执行能力基线 |
| [InjecAgent](https://doi.org/10.18653/v1/2024.findings-acl.624) | 2024 | 工具 agent 的间接提示注入风险 | 为读取文档中的恶意指令、外链和提示注入建立安全测试 |

**结论**：通用 function-call 数据可以预热格式和工具调用能力，但它不能教会模型如何解析真实 PDF/表格、引用正确页码、处理解析失败或拒答。RL 数据应来自能重放的任务环境，而不是只有一份答案的静态 QA 文件。先有 SFT 的可执行示范和确定性奖励，再尝试 PPO/GRPO 等算法；不要把“有几万行 JSON”当作 RL 环境已经齐备。

### 2.4 CoT、过程监督与数据生成

| 工作 | 年份 | 启发与边界 |
|---|---:|---|
| [Let's Verify Step by Step (PRM800K)](https://arxiv.org/abs/2305.20050) | 2023 | 经典过程监督数据，说明步骤级正误标签可用于推理训练；其数学推理格式不能直接套到文档 agent |
| [ProcessBench](https://arxiv.org/abs/2412.06559) | 2024 | 评估模型识别推理过程中的首个错误步骤；可借鉴“错误发生在哪一步”的标注方式 |
| [The Tulu 3 Report](https://arxiv.org/abs/2411.15124) | 2024 | 现代指令微调/偏好/安全数据和评测流程，可借鉴混合数据治理，不是文档 agent 专用方案 |
| [Nemotron-Research-Tool-N1](https://arxiv.org/abs/2505.00024) | 2025 | 研究工具使用与强化推理训练；参考轨迹筛选和可验证奖励 |
| [ToolACE](https://arxiv.org/abs/2409.00920) / [ToolACE-DEV](https://arxiv.org/abs/2505.07512) | 2024/25 | 合成工具使用任务后做执行校验、过滤和多样化，可借鉴数据流水线 |
| [R1-Searcher++](https://arxiv.org/abs/2505.17005) | 2025 | 通过可验证检索任务和奖励学习检索行为，适合借鉴“问题-搜索-证据-回答”链路 |

会议里的 CoT 建议落到**可监督的外显过程记录**：任务分解/简短计划、工具名与参数、工具原样返回、引用的证据 ID、关键字段抽取和结果校验。不要把私有的、冗长的隐藏思维链作为必须生成或蒸馏的标签。若确需 rationale，仅保留简短、可核验的决策说明，并和工具日志/证据标注分字段保存。工具返回本身不是模型要学习“预测”的标签；训练目标是选择工具、调用参数、解释观察结果及决定下一步。

## 3. 需要准备的数据

### A. 文档源库与许可清单（基础设施）

为每份文档保存 `doc_id`、来源 URL/机构、获取时间、语言、文档类型、页数、许可/授权、是否可用于训练、是否可再分发、内容哈希、解析器版本。优先用开放许可、公共领域或项目组自制的合成文档。保密文档只能在获授权的本地环境处理，不应上传到第三方模型/API，也不应放入公开仓库。

建议文档类型配比先均衡覆盖：文本 PDF、扫描 PDF、表格密集 PDF、含图表 PDF、DOCX、XLSX；领域覆盖论文/报告、合同模板、制度说明、财务报表等。不要一开始承诺金融/法律真实业务数据，先用公开或合成材料验证流程。

### B. 评测集（先准备，禁止训练污染）

建议首版 600-1,000 个任务，按任务/文档族分层并保留私有留出集：

| 类型 | 比例建议 | 标注 |
|---|---:|---|
| 单文档精确问答/字段抽取 | 25% | 答案、规范化值、证据页/区域/表格单元格 |
| 长文档多跳问答/总结 | 20% | 要点、必要证据集合、可接受变体、不可回答标记 |
| 表格数值推理 | 15% | 输入单元格、运算过程的结构化表达、答案和单位 |
| 多文档比较/实体关联 | 15% | 文档集合、实体对齐、各文档证据和比较结论 |
| 文档定位/解析鲁棒性 | 10% | 页码、bbox、行列坐标、OCR/版面干扰类型 |
| 澄清/拒答/冲突/恶意指令 | 10% | 应追问/拒答/指出冲突；安全理由标签 |
| 汇总与格式化导出 | 5% | 必要字段、JSON Schema/表格目标和完整性 |

每条 QA 至少含：问题、参考答案、答案判定规则、一个或多个支持证据 span、文档版本/页码、不可回答/歧义标签、来源许可、难度和任务类型。证据最好用稳定页图和归一化 bbox（0-1000 坐标）定位；文字解析结果中的页码/字符偏移也一并留存。评测中分别报告 exact match/F1 或数值误差、证据 Recall@k/定位 IoU、引用蕴含正确率、拒答准确率、完成率、工具调用数和延迟。

标注从 100 条双人试标开始，制定指南并算一致率；修订后再扩到 600-1,000。高风险任务和所有“不可回答/冲突/提示注入”样例人工复核。按源文档/模板/论文切分 train/dev/test，不能只随机切问题，避免同一文档或模板泄漏。

### C. SFT 示范集

建议首轮目标 3,000-8,000 条高质量轨迹，先做 500-1,000 条人工/半自动种子，再用规则合成和教师模型扩展。训练用量可逐步增长到一至数万，不以条数替代覆盖率。

覆盖意图识别、澄清问题、选择解析器/搜索工具、页级检索、表格读取、计算、跨文档对齐、证据引用、格式化输出、失败恢复、拒答和工具安全。每条示范应能在固定工具版本下重放；工具调用参数必须符合 JSON Schema，工具观察要保存实际结果，最终回答要引用证据。工具输出可直接从环境记录生成，不让教师模型杜撰。

正例之外，保留约 15%-25% 的困难/反例场景，如错误页、OCR 噪声、表格合并单元格、解析为空、证据冲突、工具超时、检索无结果和恶意文档指令。标注正确恢复动作或停止/拒答行为。可用强模型合成候选任务和答案，再用工具执行校验、答案程序校验、规则过滤及人工抽检；同一个教师写出 rationale 不等于真实过程正确。

### D. Agentic RL 环境和轨迹

先实现可重放的本地环境：任务包固定文档及状态；只开放白名单工具；工具 I/O 确定、结构化、可记录；环境可重置。训练 episode 可以从 SFT 示范变体、程序生成文档和公开许可文档构造。奖励拆成可验证项，避免单一“看起来合理”分数：

- 终局：答案正确、必需字段齐全、schema 合法、证据真实且支持答案。
- 步骤：正确工具/参数、有效证据检索、必要时重试/改查、冲突发现、恰当停止/拒答。
- 成本：无效调用、重复检索、循环、超时、过长上下文；采用小幅惩罚，避免牺牲正确率换短轨迹。
- 安全：拒绝文档内的越权指令；任何敏感文档不触发外发工具。

冷启动可先准备 1,000-3,000 个任务模板，每个模板生成多个文档/问题实例。每个实例可有多个 rollout（成功和失败），保留完整状态和失败原因。RL 运行数据需记录 policy/model/checkpoint、prompt/tool schema 版本、随机种子、环境快照、每步 observation/action/reward、终止原因和终局验证结果。训练/评测按模板和文档族隔离。

### E. 过程评估与偏好数据

先标注 500-1,000 条轨迹的步骤质量，而非一开始训练 PRM。对每一步标：动作是否必要、工具/参数是否正确、证据是否有效、对观察的解释是否忠实、是否应继续/停止；并标出第一个导致失败的步骤及错误类别。抽取成功/失败轨迹对可形成偏好数据。过程奖励模型只有在标签一致性、与成功率相关性及对抗性测试都过关后才值得训练。

## 4. 采集与构造流程

1. **资产盘点**：列出现有公开 benchmark、解析器、模型和实验日志；保存来源、许可、版本、校验和，明确“可评测/可训练/可再分发”三种权限。
2. **定义工具协议**：统一解析、页图、文本检索、表格查询、计算、网页搜索的输入输出 JSON Schema；给证据分配稳定 ID。
3. **建种子任务**：从公开且许可明确的文档，或人工撰写的合成文档，制作 100 条双人标注样例；覆盖各任务和不可回答情况。
4. **生成候选**：模板和程序生成问题、表格变体、多跳依赖；教师模型只提案，禁止它假造工具返回、页码或证据。
5. **确定性校验**：真实运行工具，验证引用存在、计算可复现、答案和 schema 正确、文档 split 无泄漏；程序校验不了的样本送人工复核。
6. **生成轨迹**：先由规则/教师策略产出可重放 SFT 轨迹；再由当前 policy 多次 rollout，按终局验证结果和步骤检查生成 RL 成功/失败经验。
7. **质量抽检与版本化**：每一批留存生成器版本、过滤原因、审阅记录和数据哈希；定期检查重复、模板泄漏、答案可从模型先验直接猜出、对坏 OCR/长文档的覆盖不足等问题。

公开语料建议候选：科学论文可从许可清楚的开放论文及其公开问答资源开始；财务/表格任务先用 FinQA、TAT-QA、MultiHiertt 做基线，商用/再分发前核对许可；文档解析用 OmniDocBench/DocVQA 等独立测解析或视觉问答，不将 benchmark 测试样本挪入训练。自建数据建议先用合成合同、制度、报告和电子表格，确保文档生成器可程序化产生已知答案及证据，再逐步加入授权真实材料。

## 5. 推荐数据格式

用 JSONL 存储，统一 UTF-8；图片/文档放在访问受控的对象存储或本地目录，数据行引用 `asset_uri` 和 SHA-256，避免重复内嵌大二进制。建议字段：

- `id`, `split`, `task_type`, `source`, `license`, `doc_ids`
- `messages`: 对话内容；assistant tool call 与 tool observation 分开记录
- `evidence`: `doc_id`, `page`, `bbox`, `text_span`, `table/cell` 等稳定引用
- `trajectory`: `step`, `action`, `observation_ref`, `step_label`, `reward`（RL/过程数据）
- `target`: `answer`, `abstain`, `output_schema`, `validation`
- `metadata`: 语言、文档类型、难度、噪声条件、模型/工具/提示版本

仓库中的 [JSONL 示例](../data/examples/README.md) 给出 benchmark、SFT 和 RL 三种最小记录形态。公开仓库只放可公开的 schema、合成小样本和脚本；内部或受限文档只留 manifest/哈希，不提交原文、扫描页、日志中的个人信息或密钥。

## 6. 推荐优先级与阶段交付

**第 1 周：调研冻结。** 建立论文/数据集矩阵；逐项确认许可、下载地址、代码和可复现环境；和系统负责人确认工具 schema。

**第 2 周：种子集。** 完成 100 条双人试标、标注指南、数据 schema 和基准切分草案；跑一个小型公开集基线。

**第 3 周：可执行方案。** 提交 600-1,000 条内部评测集设计、3,000-8,000 条 SFT 的生成与审核计划、RL 环境/奖励规格，以及数据治理表。先验证 50-100 个 end-to-end rollout 能正确重放，再扩规模。

建议优先级：

1. 先做数据 schema、许可表和评测集，保证项目知道“做好”是什么。
2. 再做 SFT 示范和确定性工具验证，尽早得到端到端可用系统。
3. 然后把同一批任务包装成可重放 RL 环境，保留成功/失败轨迹。
4. 最后再依据过程标签质量决定是否做 PRM/过程奖励；不要先造大量自由文本 CoT。

当前阶段以完成调研、数据定义和小规模验证为准。公开数据许可、代码可用性和 2026 年新发布工作的状态，应在实际使用前再次核对。

## 7. 参考论文清单（按方向）

长文档/文档 agent：

- [MMLongBench-Doc](https://arxiv.org/abs/2407.01523), 2024
- [OmniDocBench](https://arxiv.org/abs/2412.07626), 2024
- [LongDocURL](https://doi.org/10.18653/v1/2025.acl-long.57), 2025
- [MMDocIR](https://doi.org/10.18653/v1/2025.emnlp-main.1576), 2025
- [MEBench](https://doi.org/10.18653/v1/2025.emnlp-main.77), 2025
- [DocAgent](https://doi.org/10.18653/v1/2025.emnlp-main.893), 2025
- [LongBench v2](https://doi.org/10.18653/v1/2025.acl-long.183), 2025
- [MMLongBench-Doc-V2](https://arxiv.org/abs/2608.03397), 2026
- [Towards Agentic Document-Grounded Information Seeking](https://doi.org/10.18653/2026.emnlp-main.954), 2026
- [DocLens](https://doi.org/10.18653/v1/2026.acl-long.1234), 2026

工具调用/Agentic RL/轨迹评估：

- [GTA: A Benchmark for General Tool Agents](https://arxiv.org/abs/2407.08713), 2024
- [τ-bench](https://arxiv.org/abs/2406.12045), 2024
- [ToolACE](https://arxiv.org/abs/2409.00920), 2024
- [InjecAgent](https://doi.org/10.18653/v1/2024.findings-acl.624), 2024
- [Search-R1](https://arxiv.org/abs/2503.09516), 2025
- [R1-Searcher++](https://arxiv.org/abs/2505.17005), 2025
- [WebAgent-R1](https://arxiv.org/abs/2505.16421), 2025
- [Agentic Reasoning and Tool Integration via RL](https://arxiv.org/abs/2505.01441), 2025
- [Reasoning-Search Interleaved Agents: Empirical Study](https://arxiv.org/abs/2505.15117), 2025
- [ToolACE-DEV](https://arxiv.org/abs/2505.07512), 2025
- [Agentic RL Landscape Survey](https://arxiv.org/abs/2509.02547), 2025
- [TRAJECT-Bench](https://arxiv.org/abs/2510.04550), 2025
- [Process Reward Models for LLM Agents](https://arxiv.org/abs/2502.10325), 2025
- [Nemotron-Research-Tool-N1](https://arxiv.org/abs/2505.00024), 2025

过程监督及长上下文基础：

- [PRM800K / Let's Verify Step by Step](https://arxiv.org/abs/2305.20050), 2023
- [ProcessBench](https://arxiv.org/abs/2412.06559), 2024
- [The Tulu 3 Report](https://arxiv.org/abs/2411.15124), 2024
- [LongBench](https://doi.org/10.18653/v1/2024.acl-long.172), 2024
- [LongBench v2](https://doi.org/10.18653/v1/2025.acl-long.183), 2025
