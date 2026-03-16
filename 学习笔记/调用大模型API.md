## OpenAI 接口协议

## 1.  为什么要先讲OpenAI协议

- 原因很简单，OpenAI的Chat Completions API已经成为大模型API的事实标准。就像HTTP是Web的通用协议一样，OpenAI的接口格式是大模型API世界的"普通话"。
## 2. 请求格式详解

- 调用大模型 API，本质上就是发一个 HTTP POST 请求。请求体是一个 JSON，长这样：
``` json
{
	"model": "Qwen/Qwen3-32B",
	"messages": [
		{
			"role": "system",
			"content": "你是一个专业的电商客服助手，只回答和退货、换货、物流相关的问题。"
		},
		{
			"role": "user",
			"content": "买了一周的东西还能退吗？"
		}
	],
	 "temperature": 0.1,    
	 "max_tokens": 512,    
	 "stream": false
}
```
### 2.1 model: 指定要调用的模型

**model**字段告诉平台你要用哪个模型来处理此次请求

| 平台          | 模型 ID 示例         | 说明        |
| ----------- | ---------------- | --------- |
| SiliconFlow | `Qwen/Qwen3-32B` | 厂商/模型名 格式 |
| OpenAI      | `gpt-4o`         | 直接用模型名    |
| DeepSeek    | `deepseek-chat`  | 直接用模型名    |
### 2.2 messages: 对话消息数组
`messages`是整个请求中最核心的字段。它是一个数组，里面的每条消息都有两个属性：**`role`**(角色)和`content`(内容)。
模型不是只看你这一句话，而是看整个`messages`数组。你可以把它理解为一段对话记录——模型会根据这段完整的对话来生成回答。

### 2.3 messages 中的角色机制
`messages` 数组中的每条消息都有一个 `role`，一共有三种角色：

**system（系统角色）**

系统消息用来定义模型的行为规则，相当于给模型一份工作手册。模型会始终遵守系统消息中的指令。

比如你设置了 `"role": "system", "content": "你是一个专业的电商客服助手，只回答和退货、换货、物流相关的问题"`，那么当用户问“今天天气怎么样”时，模型会拒绝回答，因为这不在它的“工作范围”内。

system 消息在 RAG 系统中非常重要——后续我们会通过 system 消息告诉模型"根据以下参考资料回答用户的问题，如果资料中没有相关信息，请如实告知"。

**user（用户角色）**
用户消息就是用户的输入，也就是用户问的问题。

**assistant（助手角色）**

助手消息是模型之前的回答。它的作用是构建多轮对话的上下文。
举个例子，一段多轮对话的 `messages` 数组长这样：
``` json
{    
	"messages": [       
		{"role": "system", "content": "你是一个电商客服助手"},       
		{"role": "user", "content": "你们支持七天无理由退货吗？"},       
		{"role": "assistant", "content": "支持的。自签收之日起7天内，商品未使用且不影响二次销售的，可以申请七天无理由退货。"},       
		{"role": "user", "content": "那运费谁出？"}   
	] 
}
```

> 大模型本身是没有记忆的。每次 API 调用都是独立的，模型不会记住上一次调用的内容。所谓的多轮对话，其实是你在每次请求中把之前的对话历史都带上，让模型看到完整的上下文。这也是为什么对话越长，消耗的 Token 越多——因为每次请求都要把历史消息重新发一遍。

来看一个更直观的例子——同一个问题，不同的 system 消息会导致完全不同的回答风格：

|system 消息|用户问题|模型回答风格|
|---|---|---|
|你是一个专业的技术顾问|Java 和 Python 哪个好？|客观分析两种语言的优劣势和适用场景|
|你是一个 Java 狂热粉丝|Java 和 Python 哪个好？|疯狂吹 Java，贬低 Python|
|你是一个五岁小孩|Java 和 Python 哪个好？|用幼稚的语气回答，可能说"我不知道这是什么"|

>**关于 OpenAI 的 developer 角色**：OpenAI 在新版 API 中引入了 `developer` 角色来替代 `system` 角色（参考 [OpenAI 官方文档](https://developers.openai.com/api/docs/guides/text)）。两者的功能类似，都是用来定义模型的行为规则。不过目前大多数兼容 OpenAI 协议的平台（包括 SiliconFlow、DeepSeek 等）仍然使用 `system` 角色，所以本系列统一使用 `system`。如果你直接调用 OpenAI 官方 API，可以用 `developer` 替换 `system`，效果是一样的。

### 2.4 temperature、max_tokens、top_p 等参数

除了 `model` 和 `messages`，还有几个常用的可选参数：

| 参数            | 类型      | 说明                                     | RAG 场景推荐值            |
| ------------- | ------- | -------------------------------------- | -------------------- |
| `temperature` | float   | 控制回答的随机性，0~2 之间。上一篇详细讲过                | 0~0.3                |
| `max_tokens`  | int     | 模型最多生成多少个 Token。超过这个数就会被截断             | 512~2048（根据预期回答长度设置） |
| `top_p`       | float   | 另一种控制随机性的方式，0~1 之间。和 temperature 二选一即可 | 0.7~0.9              |
| `stream`      | boolean | 是否启用流式返回。true 为流式，false 为非流式           | 根据场景选择               |
### 2.5 stream：流式开关

`stream` 参数决定了模型的回答是一次性返回还是逐块推送：

- `stream: false`（默认）：模型生成完所有内容后，一次性返回完整的 JSON 响应。适合后台处理场景

- `stream: true`：模型每生成一小段内容就立刻推送给客户端，客户端可以实时展示。适合面向用户的对话场景
## 3. 响应格式详解

发出请求后，模型会返回一个 JSON 响应（这里先看非流式的响应格式）：

``` json
{    
	"id": "chatcmpl-abc123",    
	"object": "chat.completion",    
	"created": 1700000000,    
	"model": "Qwen/Qwen3-32B",    
	"choices": [       
		{            
			"index": 0,            
			"message": {                
				"role": "assistant",                
				"content": "支持的。根据我们的退货政策，自签收之日起7天内，商品未经使用且不影响二次销售的，您可以申请七天无理由退货。请在订单详情页点击\"申请退货\"按钮，按照提示操作即可。"           
			},            
			"finish_reason": "stop"       
		}   
	],    
	"usage": {        
		"prompt_tokens": 42,        
		"completion_tokens": 68,        
		"total_tokens": 110   
	} 
}
```
关键字段说明：

- `id`：这次请求的唯一标识，用于日志追踪
- `choices`：模型的回答数组。通常只有一个元素（除非你设置了 `n` 参数要求生成多个回答）
- `choices[0].message`：模型的回答，格式和请求中的 message 一样，`role` 是 `assistant`，`content` 是回答内容
- `choices[0].finish_reason`：模型停止生成的原因
- `usage`：Token 用量统计