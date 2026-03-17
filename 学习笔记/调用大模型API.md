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
### 3.1 usage：Token 用量统计
`usage` 字段告诉你这次调用消耗了多少 Token：

|字段|含义|
|---|---|
|`prompt_tokens`|你发送的内容（system + user + assistant 历史消息）消耗的 Token 数|
|`completion_tokens`|模型生成的回答消耗的 Token 数|
|`total_tokens`|总 Token 数 = prompt_tokens + completion_tokens|
API 按 Token 计费，所以这个字段帮你监控成本。上面的例子中，输入 42 个 Token，输出 68 个 Token，总共 110 个 Token。
### 3.2 finish_reason：模型为什么停下来了

`finish_reason` 有两个常见的值：

|值|含义|说明|
|---|---|---|
|`stop`|正常结束|模型认为回答已经完整，主动停止|
|`length`|达到长度上限|回答被 `max_tokens` 截断了，内容可能不完整|

如果你经常看到 `finish_reason: "length"`，说明你的 `max_tokens` 设小了，模型的回答被截断了。把 `max_tokens` 调大一些就行。
## 4. 为什么国内厂商都兼容这个协议

OpenAI 是最早定义这套 Chat Completions API 协议的，围绕它已经形成了庞大的生态：

- 各种 SDK 和框架（LangChain、Spring AI、LlamaIndex）都默认支持 OpenAI 协议
- 开发者社区的教程、示例代码大多基于这个协议
- 各种工具（Postman 模板、VS Code 插件、命令行工具）都兼容这个格式

国内厂商兼容这个协议，开发者就能零成本迁移。你用 SiliconFlow 写的代码，想切换到 DeepSeek 官方 API，只需要改两个东西：
``` Java
// SiliconFlow 
baseURL = "https://api.siliconflow.cn/v1/chat/completions" apiKey  = "你的 SiliconFlow API Key" ​ 
// 切换到 DeepSeek 
baseURL = "https://api.deepseek.com/v1/chat/completions" apiKey  = "你的 DeepSeek API Key"
```

代码逻辑一行不用改。这就是兼容协议的好处。
# SiliconFlow 平台：注册与获取 API Key

## 1. 为什么选择 SiliconFlow

上一篇已经提过技术选型的理由，这里再简单汇总一下：
- 国内平台，网络访问没有障碍，不需要特殊网络环境
- 注册简单，手机号即可注册，新用户有免费额度
- 支持多种主流开源模型（Qwen、DeepSeek、GLM、Llama 等），一个平台就能用多种模型
- API 完全兼容 OpenAI 协议，学会了这一套，切换到其他平台零成本
- 部分模型可免费调用，足够学习和实验使用

## 2. 注册步骤

- 1.访问 SiliconFlow 官网：`https://siliconflow.cn`
- 2.点击右上角的“注册”按钮，使用手机号注册账号
- 3.注册完成后登录，进入控制台
- 4.在左侧菜单中找到“API 密钥”，点击进入
- 5.点击“新建 API 密钥”按钮，系统会生成一个以 `sk-` 开头的密钥字符串
- 6.把这个密钥复制下来，保存到安全的地方

>其它平台也是类似的操作

>API Key 是你调用 API 的身份凭证，相当于密码。不要把它提交到 Git 仓库、发到群聊里、或者写在公开的代码中。后面的代码示例中，我们会用 `YOUR_API_KEY` 作为占位符，你需要替换成自己的真实 Key。

## 3. 本系列用到的模型
| 模型 ID                     | 类型           | 用途         | 本系列使用场景                   |
| ------------------------- | ------------ | ---------- | ------------------------- |
| `Qwen/Qwen3-32B`          | Chat 模型      | 对话、问答、文本生成 | 本篇的 API 调用实战，后续 RAG 的生成环节 |
| `BAAI/bge-m3`             | Embedding 模型 | 文本向量化      | 后续 RAG 的向量化环节             |
| `BAAI/bge-reranker-v2-m3` | Reranker 模型  | 检索结果重排序    | 后续 RAG 的检索环节              |

>模型的可用性和价格可能会变化，以 SiliconFlow 官网的实际展示为准。充值少量金额（几块钱就够实验很久）。

# 非流式调用：发一个问题，拿一个完整回答
## 1. Maven 依赖

在你的 `pom.xml` 中添加以下依赖：
``` json
<dependencies>
    <!-- OkHttp：HTTP 客户端 -->
    <dependency>
        <groupId>com.squareup.okhttp3</groupId>
        <artifactId>okhttp</artifactId>
        <version>4.12.0</version>
    </dependency>
    <!-- Gson：JSON 处理 -->
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.13.1</version>
    </dependency>
</dependencies>
```

## 2. 完整代码实现

``` Java
public class NonStreamingChat {
​
    // SiliconFlow API 地址
    private static final String API_URL = "https://api.siliconflow.cn/v1/chat/completions";
    // 替换成你自己的 API Key
    private static final String API_KEY = "YOUR_API_KEY";
​
    public static void main(String[] args) throws IOException {
        // 1. 构建请求体 JSON
        JsonObject requestBody = new JsonObject();
        requestBody.addProperty("model", "Qwen/Qwen3-32B");
        requestBody.addProperty("temperature", 0);
        requestBody.addProperty("max_tokens", 1024);
        requestBody.addProperty("stream", false);
​
        // 构建 messages 数组
        JsonArray messages = new JsonArray();
​
        // system 消息：定义模型的行为规则
        JsonObject systemMsg = new JsonObject();
        systemMsg.addProperty("role", "system");
        systemMsg.addProperty("content", "你是一个专业的电商客服助手，回答要简洁明了。");
        messages.add(systemMsg);
​
        // user 消息：用户的问题
        JsonObject userMsg = new JsonObject();
        userMsg.addProperty("role", "user");
        userMsg.addProperty("content", "买了一周的东西还能退吗？");
        messages.add(userMsg);
​
        requestBody.add("messages", messages);
​
        // 2. 创建 OkHttp 客户端（设置超时时间，大模型响应可能较慢）
        OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(30, TimeUnit.SECONDS)
                .readTimeout(60, TimeUnit.SECONDS)
                .build();
​
        // 3. 构建 HTTP 请求
        Request request = new Request.Builder()
                .url(API_URL)
                .addHeader("Authorization", "Bearer " + API_KEY)
                .addHeader("Content-Type", "application/json")
                .post(RequestBody.create(
                        requestBody.toString(),
                        MediaType.parse("application/json")
                ))
                .build();
​
        // 4. 发送请求并处理响应
        try (Response response = client.newCall(request).execute()) {
            if (!response.isSuccessful()) {
                System.out.println("请求失败，状态码：" + response.code());
                System.out.println("错误信息：" + response.body().string());
                return;
            }
​
            // 5. 解析 JSON 响应
            String responseBody = response.body().string();
            Gson gson = new Gson();
            JsonObject jsonResponse = gson.fromJson(responseBody, JsonObject.class);
​
            // 提取模型的回答
            String answer = jsonResponse
                    .getAsJsonArray("choices")
                    .get(0).getAsJsonObject()
                    .getAsJsonObject("message")
                    .get("content").getAsString();
​
            // 提取 finish_reason
            String finishReason = jsonResponse
                    .getAsJsonArray("choices")
                    .get(0).getAsJsonObject()
                    .get("finish_reason").getAsString();
​
            // 提取 Token 用量
            JsonObject usage = jsonResponse.getAsJsonObject("usage");
            int promptTokens = usage.get("prompt_tokens").getAsInt();
            int completionTokens = usage.get("completion_tokens").getAsInt();
            int totalTokens = usage.get("total_tokens").getAsInt();
​
            // 6. 打印结果
            System.out.println("=== 模型回答 ===");
            System.out.println(answer);
            System.out.println();
            System.out.println("=== 调用信息 ===");
            System.out.println("结束原因：" + finishReason);
            System.out.println("输入 Token：" + promptTokens);
            System.out.println("输出 Token：" + completionTokens);
            System.out.println("总 Token：" + totalTokens);
        }
    }
}
```

## 3. 运行效果

把 `YOUR_API_KEY` 替换成你自己的 API Key，运行这段代码，控制台输出大概长这样：
# 流式调用：像打字一样逐字输出

## 1. 为什么需要流式调用

非流式调用有一个体验上的问题：模型要把所有内容都生成完，才一次性返回给你。如果回答比较长（比如几百、几千字），用户可能要等 3~30 秒才能看到任何内容。这段时间里，界面上什么都没有，用户会觉得是不是卡了。

>如果是深度思考，这个首包响应时间会更长。注意是首包。

