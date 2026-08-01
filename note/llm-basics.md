# 大模型基础：Token、上下文与消息结构

> 「AI Agent 工程实践」系列 · 基础篇第 1 篇。
> 本系列其他文档反复使用 token、上下文窗口、消息角色这些概念，这篇一次性讲清楚。
> 相关：[手写 Agent](./agent.md) / [流式输出](./streaming.md)

## Token：模型读写的基本单位

Token（词元）是模型处理文本的最小单位。它不是"字"也不是"词"，而是**子词（subword）切分**的结果：

```text
"你好世界"  →  ["你好", "世界"]      （分词器不同，切法不同）
"hello world" → ["hello", " world"]  （英文里空格通常并入下一个 token）
```

为什么必须关心 token：

1. **上下文窗口按 token 计**，不是按字数；
2. **计费按 token**（输入 + 输出分开算）；
3. 模型的"记忆容量"就是窗口里的 token 数。

经验值：英文约 1 token ≈ 0.75 个词；中文约 1 个汉字 ≈ 1~2 token（取决于分词器）。**要精确，用官方 tokenizer 算**，别按"字"估。

## 上下文窗口

上下文窗口（context window）是一次请求能处理的最大 token 数，**输入 + 输出共享**这个预算：

| 部分 | 内容 | 说明 |
| --- | --- | --- |
| 输入 | system + 历史消息 + 用户输入 + 检索资料 | 占窗口大头 |
| 输出 | 模型的回答 / 工具调用 | 受 max_tokens 限制 |

常见窗口规模从 32K 到 200K+ token 不等。窗口大 ≠ 全都用得上：超长上下文里，**中间部分的信息召回质量会下降**（"lost in the middle"），关键信息尽量放开头和结尾。

## messages 消息结构

所有主流 chat API 都用 `messages` 数组对话，角色就四种：

| 角色 | 谁写的 | 作用 |
| --- | --- | --- |
| `system` | 开发者 | 设定角色、行为规则、输出格式（优先级最高） |
| `user` | 用户 | 用户输入；**也可以承载工具结果回填** |
| `assistant` | 模型 | 模型回答；多轮时原样回填；工具调用时含 `tool_calls` |
| `tool` | 程序 | 工具执行结果，配合 `tool_call_id` 与调用一一对应 |

一个带工具调用的完整结构：

```js
const messages = [
  { role: 'system', content: '你是天气助手' },
  { role: 'user', content: '北京今天多少度？' },
  {
    role: 'assistant',
    content: '',
    tool_calls: [{
      id: 'call_1',
      function: { name: 'getWeather', arguments: '{"city":"北京"}' }
    }]
  },
  { role: 'tool', tool_call_id: 'call_1', content: '晴 25℃' }
];
```

关键坑：**assistant 的历史必须原样回填**（包括 `tool_calls` 字段），只回填纯文本会让模型在多轮中"失忆"甚至报错。

## 生成参数

| 参数 | 作用 | 建议 |
| --- | --- | --- |
| `temperature` | 随机性：越高越发散，越低越确定 | 结构化/代码任务 0~0.3；创意 0.7~1.0 |
| `top_p` | 核采样：只从累计概率前 p 的 token 中选 | 与 temperature 二选一调，别同时猛调 |
| `max_tokens` | 输出上限 | 按任务设定，防失控超长 |
| `stop` | 停止词序列 | 结构化输出时可用作终止符 |

注意：`max_tokens` 设太大，输出会挤占输入空间，甚至触发"输入截断"。

## 流式输出

- 非流式：等模型生成完一次性返回（首字可能要几十秒）；
- 流式（stream）：token 增量返回，首字快、体验好。

流式是 Agent 产品的标配，实现细节见[流式输出](./streaming.md)。

## 注意（坑）

1. **中文别按"字"估 token**，用官方 tokenizer 算；
2. **窗口是输入+输出共享的**，max_tokens 设太大等于压缩输入空间；
3. **system 消息不是无限的**：太长会被模型"遗忘"（注意力衰减），关键约束放前面；
4. **计费按 token 累加**：多轮对话成本会滚雪球，需要压缩（见[多轮记忆压缩](./memory-compression.md)）。

## 总结

Token（单位）→ 上下文窗口（预算）→ messages（结构）→ 生成参数（行为）。这四样是理解后面所有 Agent 机制的地基。下一篇讲 Agent 检索能力的引擎——[Embedding 与向量检索](./embedding.md)。
