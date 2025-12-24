# Explore to Evolve: Scaling Evolved Aggregation Logic via Proactive Online Exploration for Deep Research Agents

## 动机
- 开源深度研究代理更偏重信息检索，但信息聚合能力弱，难以形成高质量综合结论。
- 现有数据集多为离线静态网页、聚合逻辑弱，无法充分考验真实网页环境下的深度推理。

## 方法
- Explore to Evolve 流程：主动在线探索 + 自动聚合逻辑合成 + 质量控制。
- 代理从 anchor URL 出发探索真实网页，使用搜索、浏览、动态元素交互、文件与图像处理等工具。
- 聚合逻辑由 12 种高层操作指导，覆盖 Element、Set、Scientific Analysis、Temporal Reasoning 四类。
- 产出 WebAggregatorQA（约 1 万 QA，5.4 万 URL，12 个领域）及人工标注测试集。

## 实验
- 模型：WebAggregator 系列（Qwen2.5/Qwen3 7B/8B/32B），使用 SmolAgents 轨迹进行训练。
- 基准：GAIA-text 与 WebAggregatorQA；指标为 pass@1（GPT-4.1 评判）。
- 结果：WebAggregator-32B 在 GAIA-text 上超过 GPT-4.1 与多种强基线，在 WebAggregatorQA 上也有提升。
- WebAggregatorQA 难度高：GPT-4.1（agent）从 GAIA-text 的 43.7 降至 WebAggregatorQA 的 25.8。
