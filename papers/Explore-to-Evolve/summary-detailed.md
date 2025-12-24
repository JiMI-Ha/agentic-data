# Explore to Evolve: Scaling Evolved Aggregation Logic via Proactive Online Exploration for Deep Research Agents

## 动机
- 深度研究代理不仅要检索事实，更需要对证据进行聚合与分析。
- 现有数据集多模拟静态网页检索，对复杂聚合的覆盖不足。
- 真实网页任务涉及动态页面、文件、图像等异构来源，且需要多步推理。

## 方法
- Explore to Evolve 的可扩展、可验证数据构建流程。
- Explore：从 anchor URL 出发，使用搜索、访问、点击、文件读取、截图等工具主动浏览真实网页。
- Evolve：将高层聚合指导演化为具体操作，合成问答对。
- 聚合操作体系：12 种子类，归入 Element、Set、Scientific Analysis、Temporal Reasoning 四组。
- 质量控制：自动一致性检查（问题-答案-来源），并对领域与聚合操作分布做均衡。
- 产出数据：WebAggregatorQA 含 9,883 个任务、54,064 个唯一 URL、12 个领域；测试集为人工标注。

## 实验
- 模型：WebAggregator 系列（Qwen2.5-7B/32B、Qwen3-8B/32B），基于 SmolAgents。
- 训练：使用轨迹数据进行 SFT（问题、动作、观测、答案），对问题与观测做遮蔽。
- 基准：GAIA-text（103 条）与 WebAggregatorQA；指标 pass@1（GPT-4.1 评判）。
- 结果（pass@1）：WebAggregator-32B 在 GAIA-text 为 56.3，在 WebAggregatorQA 为 26.4。
- 强基线（agentic）：GPT-4.1 在 GAIA-text 为 43.7、WebAggregatorQA 为 25.8；Claude-3.7-sonnet 为 60.2、28.3。
- 结论：WebAggregatorQA 更强调聚合能力；微调显著提升 Qwen 系列并逼近强基线。

## 局限
- 自动探索仍可能漏证据或引入噪声；质检过滤约 11.7% 样本。
- 所有模型在该基准上仍表现不足，表明聚合能力仍是瓶颈。
- 人工测试集规模相对较小（159 条）。

## 个人笔记
- 适合训练代理从“检索”走向“综合推断”。
- 可探索对聚合步骤的可执行程序/可验证性。
- 可扩展到多模态聚合与更强的工具执行轨迹。
