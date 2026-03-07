
# 大模型
## 关于大模型必须知道的概念
---
### Token: 大模型内部的最小计量单位
### 上下文窗口: 一次对话能看到的文本总量上限
### Temperature: 用来控制回答的随机性
- Temperature = 0：模型每次都会选择概率最高的那个词，回答最确定、最稳定，但可能比较死板
- Temperature = 0.7：模型会在高概率的词里随机选择，回答更自然、更有变化
- Temperature = 1.0 或更高：模型的选择更加随机，回答更有创意，但也更容易胡说八道
### MoE: 混合专家系统: 将大模型内部拆分成多个专家子网络，每次推理只调用部分专家
- 好处：用更低的计算成本获取更强的推理能力
## 目前主流大模型一览-2026.3.6
---
### 1. 国际主流

| 模型系列（举例）                                   | 厂商        | 特点                                           | 是否开源    |
| ------------------------------------------ | --------- | -------------------------------------------- | ------- |
| GPT 系列（GPT-5.2 / GPT-5.2 pro 等）            | OpenAI    | 综合能力与生态最成熟；推理、工具调用、Agent 工作流能力强              | 否       |
| Claude 系列（Claude Opus 4.6 / Sonnet 4.6）    | Anthropic | 代码与长上下文能力突出；“computer use/代理”能力强；安全与对齐投入大    | 否       |
| Gemini 系列（Gemini 3.1 Pro / Gemini 3 Flash） | Google    | 原生多模态（文/图/音/视频）；超长上下文（最高 1M）；与 Google 生态结合紧密 | 否       |
| Llama 系列（Llama 4 Scout / Maverick）         | Meta      | 开放权重生态最强之一；社区活跃、部署与微调方案成熟；MoE + 多模态          | 是（开放权重） |
### 2. 国内主流

| 模型系列（最新）                          | 厂商                | 特点                                                                         | 是否开源                                                                                                          |
| --------------------------------- | ----------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| DeepSeek 系列（V3 / R1 等）            | 深度求索（DeepSeek）    | 性价比高；推理能力强（R1）；开放权重、可商用；社区生态活跃                                             | 是（MIT）（[Hugging Face](https://huggingface.co/deepseek-ai/DeepSeek-R1)）                                        |
| 通义千问 Qwen 系列（Qwen3.5）             | 阿里云 / 阿里巴巴        | **Agent 生态强**、工具调用/多模态路线清晰；尺寸覆盖广；开源生态完善                                    | 是（Apache-2.0，开放权重）（[Hugging Face](https://huggingface.co/Qwen/Qwen3.5-397B-A17B)）                             |
| 智谱 GLM 系列（GLM-5）                  | 智谱 AI / Z.ai      | **面向复杂系统工程与长程 Agent 任务**；推理/编码/Agentic 能力强化；GLM-5 已开源且 MIT 许可              | 是（MIT） ([Hugging Face](https://huggingface.co/zai-org/GLM-5))                                                 |
| MiniMax 系列（MiniMax-M1 / abab 6.5） | MiniMax           | M1 主打**超长上下文 + 高效推理**（百万级输入）；abab 6.5 系列面向通用对话/长上下文                        | 部分开源（M1 开源 Apache-2.0；abab 多为服务型） ([MiniMax](https://www.minimaxi.com/news/minimaxm1?utm_source=chatgpt.com)) |
| Kimi 系列（Kimi K2.5）                | 月之暗面（Moonshot AI） | **长文档/多模态 + Agent**；K2.5 开源权重，Modified MIT 许可；并提供兼容 OpenAI/Anthropic 的 API | 是（Modified MIT） ([Hugging Face](https://huggingface.co/moonshotai/Kimi-K2.5))                                 |
### 3. 模型对比表
| 模型                             | 厂商          | 参数量              | 上下文窗口               | 开源            | 主要特点                                       |
| ------------------------------ | ----------- | ---------------- | ------------------- | ------------- | ------------------------------------------ |
| GPT-5.2 pro                    | OpenAI      | 未公开              | 400K                | 否             | 强推理 + 工具/多轮交互；旗舰 API 能力                    |
| Claude Sonnet 4.6              | Anthropic   | 未公开              | 200K（默认）/ 1M（Beta）  | 否             | 代码与长上下文强；computer-use/Agent 能力突出           |
| Gemini 3.1 Pro                 | Google      | 未公开              | 1M                  | 否             | 原生多模态；长上下文；适合复杂资料/代码库                      |
| Llama 4 Maverick（17Bx128E）     | Meta        | 400B（17B 激活，MoE） | 1M                  | 是（开放权重）       | 多模态 + MoE；本地部署/微调生态成熟                      |
| DeepSeek-V3.2（deepseek-chat）   | 深度求索        | 671B（37B 激活，MoE） | 128K                | 是（MIT）        | 通用 + Agent 取向；支持 Thinking in Tool-Use；性价比高 |
| DeepSeek-R1（deepseek-reasoner） | 深度求索        | 671B（37B 激活，MoE） | 128K                | 是（MIT）        | 深度思考推理；数学/代码/规划强                           |
| Qwen3.5-397B-A17B              | 阿里巴巴        | 397B（17B 激活，MoE） | 262K                | 是（Apache-2.0） | 旗舰开放权重；偏 agentic；长上下文更强                    |
| Qwen3-32B                      | 阿里巴巴        | 32B              | 32K（原生）/ 131K（YaRN） | 是（Apache-2.0） | 中大型开源；适合私有化、RAG、微调                         |
| Qwen3-8B                       | 阿里巴巴        | 8B               | 32K（原生）/ 131K（YaRN） | 是（Apache-2.0） | 轻量部署；成本敏感/边缘侧更友好                           |
| GLM-5                          | 智谱 AI（Z.ai） | 744B（40B 激活，MoE） | 200K                | 是（MIT）        | 面向 Coding/Agent；长上下文 + 高输出上限（128K）         |

## Chat模型：我们真正要调用的东西


你在网页端用的 DeepSeek、通义千问，或者通过 API 调用的模型，其实都是一种特定类型的大模型——Chat 模型。但大模型并不是天生就会聊天的，它需要经过专门的训练才能变成一个合格的对话助手。
---
### 1. 基座模型（Base Model）vs Chat 模型
大模型的训练分两个阶段：

- 第一阶段是预训练（Pre-training）。模型阅读海量文本数据，学习语言的规律。训练完成后得到的就是基座模型（Base Model）。基座模型有一个特点：它只会续写。你给它一句话，它会接着往下写，但它不会回答问题。
- 第二阶段是对齐训练（Alignment）。在基座模型的基础上，通过指令微调（Instruction Tuning）和人类反馈强化学习（RLHF）等技术，教会模型理解人类的指令，并按照指令给出有用、安全的回答。训练完成后得到的就是 Chat 模型（也叫 Instruct 模型）。
![[Pasted image 20260307144043.png]]
#### 1.1 模型命名规则解读
	