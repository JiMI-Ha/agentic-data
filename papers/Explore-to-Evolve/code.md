论文方法与代码映射
  - Explore（主动探索网页、工具多步调用）：核心在 WebAggregator/web_tools.py 和 WebAggregator/
    run_agent.py。run_agent.py 里用 CodeAgent + 一组 Web/文件工具（visit_webpage、MixedSearchTool、
    perform_click、perform_input、DownloadTool 等）进行主动浏览、检索、点击、下载与阅读，体
    现“proactive online exploration”。
  - Evolve（从高层聚合逻辑演化为具体QA）：体现在 WebAggregator/prompt.py 的
    question_construct_format。它要求多跳、多个网页、多个聚合/统计操作，且必须给出可验证解；这就是论
    文里“aggregation taxonomy + evolve”的落地。
  - 自动质检/过滤：WebAggregator/web_tools.py 中 simple_check_constructed_question 用 LLM 对构造问题
    做合规检查；WebAggregator/convert.py 会过滤部分失败/无效轨迹。
  - 轨迹采样（用于SFT训练）：WebAggregator/run_agent.py 记录 agent memory 和轨迹（traj_save_path）；
    WebAggregator/convert.py 的 formattingTrajectory 把轨迹转成训练消息。
  - LLM-as-judge 评测：WebAggregator/eval.py 使用 langchain 的 cot_qa 评估器，配合 config.py 的
    evaluation_llm。

  关键流程与脚本

  - QA构造（论文的“自动生成WebAggregatorQA”）：WebAggregator/run/QA_building.sh → WebAggregator/
    run_agent.py（--task qa_construct + question_construct_format）。输出在 WebAggregator/
    output/...。
  - 评测（GAIA-text & WebAggregatorQA）：WebAggregator/run/test.sh → WebAggregator/
    run_agent.py（--task eval + eval_prompt）→ WebAggregator/eval.py。
  - 轨迹采样（用于训练）：
  WebAggregator/run/traj_sampling.sh →
  WebAggregator/run_agent.py 生成轨迹
  →
    WebAggregator/convert.py 转成训练
  格式。

  模型与工具接线
  - 你要配置的入口：WebAggregator/
  config.py（基础模型、工具LLM、judge
  LLM、数据路径）。
  - 自定义模型接入：WebAggregator/
    Azure），返回 OpenAIServerModel 给
  smolagents。
  - 浏览器与可视化工具：WebAggregator/web_tools.py 用 Selenium + Helium + 辅助截图/HTML转Markdown，支
  - “可验证”= eval.py 的 LLM-as-judge + simple_check_constructed_question 的构造检查。
  - “训练数据/轨迹”= traj 目录 + convert.py 输出的训练消息。


tools:WebAggregator/web_tools.py
  - MixedSearchTool（名为 web_search）：DuckDuckGo 搜索 + LLM 重排结果。
  - visit_webpage：打开网页并转为 Markdown，同时返回可访问性树。
  - perform_click：按可访问性树索引点击元素。
  - perform_input：按可访问性树索引向输入框写入文本。
  - search_item_ctrl_f：在当前页面查找文本并跳转到第 n 个匹配。
  - go_back：浏览器后退。
  - scroll_down_window / scroll_up_window：页面滚动。
  - DownloadTool（名为 download_file）：下载文件并做简要内容描述。
  - visualizer：对图片做视觉问答/描述。
  - generate_a11y_tree：返回页面可访问性树（文本格式）。
  - simple_check_constructed_question：对生成的 QA 做规则检查（构造阶段用）。

Evolve 这一步主要体现在“把探索到的信息，按一套聚合逻辑模板，生成可验证的多跳 QA”。在代码里对应的是
  这些地方：

  - 核心模板：WebAggregator/prompt.py 的 question_construct_format。它强制要求
      - 访问 5–8 个网页
      - 至少 3 类聚合/推理操作（统计、集合、元素运算、时间推理等）
      - 给出可验证的 solution 与 urls
        这就是论文里 “aggregation taxonomy + evolved aggregation logic” 的落地。
  - 触发位置：WebAggregator/run_agent.py 里 args.task == "qa_construct" 时，args.prompt =
    question_construct_format。
    在 run/QA_building.sh 里调用 run_agent.py --task qa_construct，即走 Evolve 流程。
  - 质检：WebAggregator/web_tools.py 的 simple_check_constructed_question 会在构造 QA 时检查是否满足
    上述约束。

轨迹保存：
在 WebAggregator/run_agent.py 里，轨迹是这样记录的：

  - 保存路径：create_agent_hierarchy() 里给 CodeAgent 传 traj_save_path = f"{args.traj_path}/
    {save_name}/{agent_name}/{agent_id}"，所以每个任务会落在 traj/<run_name>/<model_id>/<task_id>/ 这
    样的目录。
  - 保存时机：answer_single_question() 里调用
      - agent.write_memory_to_messages(summary_mode=True) 取总结
      - agent.save_messages(deepcopy(agent.write_memory_to_messages(summary_mode=False))) 真正把完整
        消息序列写成 JSON（通常是 -1.json）。
        位置在 WebAggregator/run_agent.py。

  在 WebAggregator/convert.py 里，formattingTrajectory() 把轨迹转成训练消息的流程是：

  - 读入轨迹文件：遍历 path 目录下每个子目录的 -1.json。
  - 角色标准化：把 smolagents 的 role 映射成 user/assistant/system/tool-call 等。
  - 清理长文本：对包含 “The accessibility tree is” 的消息做截断，避免超长。
  - 去掉无效项：过滤 “unable to determine” 或评估失败的样本。
  - 拼装消息：
      - 非 planning：跳过第 3 条消息（通常是计划步骤）。
      - planning 模式：合并 plan 与后续内容。
  - 输出格式：生成 {"messages": [...]} 的训练样本列表，并通过 write_json 写到输出文件。

  你可以直接看这两个文件的具体实现：

  - 轨迹保存：WebAggregator/run_agent.py
  - 轨迹转训练：WebAggregator/convert.py