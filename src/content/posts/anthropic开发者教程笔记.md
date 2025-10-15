---
title: claude开发教程笔记
published: 2025-10-15
description: "简略介绍anthropic开发者教程内容"
image: "https://blog-daoge.oss-cn-beijing.aliyuncs.com/Snipaste_2025-10-15_18-54-15.png"
tags: ["sky-take-out"]
category: demo
draft: false
---
# Claude 开发教程笔记

> 本文是对 [Anthropic 开发者教程](https://anthropic.skilljar.com/claude-with-the-anthropic-api) 的笔记
---


## 📚 课程概述

### 🎯 学习目标

完成本课程后，您将掌握：

- 向 Claude 模型发送 API 请求并处理响应
- 实现多轮对话、流式传输和结构化输出生成
- 使用自动化测试流程系统地构建和评估提示词
- 创建自定义工具并将 Claude 与外部服务集成
- 设计和实现具有混合搜索和重新排序功能的 RAG 系统
- 使用 MCP（模型上下文协议）将 Claude 连接到各种数据源
- 理解常见的工作流和代理架构

---

## 📖 课程大纲

### 第一部分：Claude 入门

#### 完整请求生命周期（五个阶段）

1. **客户端 → 服务器**：用户发起请求
2. **服务器 → Anthropic API**：服务器转发请求
3. **模型处理**：Claude 处理请求
4. **API → 服务器**：返回响应
5. **服务器 → 客户端**：显示结果

**⚠️ 安全提示**：永远不要从客户端直接调用 API，API 密钥必须保存在安全的服务器端

#### API 必须参数

![必须参数截图](https://blog-daoge.oss-cn-beijing.aliyuncs.com/Snipaste_2025-10-15_19-37-58.png)

---

### 核心概念解析

#### Token（词元）

Token 是文本处理的基本单位，可以是：

- ✅ **完整的单词**：`"hello"`
- ✅ **单词的一部分**：`"running"` 可能被分为 `"run"` + `"ning"`
- ✅ **空格**：词与词之间的空格
- ✅ **标点符号**：`"."` `","` `"!"`
- ✅ **特殊字符**：`"@"` `"#"` `"$"`

#### Embedding（嵌入）

Embedding 是将 token 转换为数值向量的过程。这些向量能够数学化地表示词语的含义和语义关系，使计算机能够"理解"文本内容。

---

### 模型工作原理简述

**1. 分词（Tokenization）**  
将文本分解为更小的单元（token）

**2. 嵌入（Embedding）**  
将每个 token 转换为数值向量，表示其语义含义

**3. 语境化（Contextualization）**  
根据上下文调整向量值，确定词语在特定语境中的确切含义

**4. 生成（Generation）**  
计算下一个词的概率分布，结合概率和随机性选择输出，逐词生成响应

#### 生成停止条件

Claude 在每个 token 后检查是否满足以下条件：

- 是否达到最大 token 数限制
- 是否生成了结束序列标记
- 是否遇到预定义的停止标记

---

### Temperature 参数详解

#### 作用

Temperature 参数用于控制模型生成时的创造性：

- **较低的 temperature**：输出更加确定性和一致性，适合严谨的任务（如文本引用、数据提取）
- **较高的 temperature**：输出更加多样化和创意性，适合创作性任务（如写作、头脑风暴）

#### 工作原理

Claude 的文本生成过程包含三个关键步骤：

1. **Tokenization（分词处理）**  
   将输入内容分解为更小的片段

2. **Prediction（概率预测）**  
   计算可能出现的下一个词汇的概率（这些概率会受到 temperature 影响）

3. **Sampling（采样）**  
   根据这些概率选择 token

![Temperature 示意图](https://blog-daoge.oss-cn-beijing.aliyuncs.com/instructor_a46l9irobhg0f5webscixp0bs_public_1748623338_03_-_008_-_Temperature_00.1748623338635.png)

**Temperature 的影响：**

- **低温（接近 0）**：Claude 变得极为确定性，几乎总是选择概率最高的 token
- **高温（接近 1）**：概率更均匀地分布在各个选项中，产生更多样化且富有创意的输出

![Temperature 对比图](https://blog-daoge.oss-cn-beijing.aliyuncs.com/instructor_a46l9irobhg0f5webscixp0bs_public_1748623340_03_-_008_-_Temperature_06.1748623340446.png)

---

### 结构化输出解决方案：助手消息预填充 + 停止序列

#### 概念说明

**消息预填充**：让 Claude 以为自己已经说了填充的那些话，从而引导其输出方向。

**示例：**

- 直接问"面包和馒头哪个好吃"：Claude 大概率会输出中肯的答案
- 预填充"面包好吃，因为"：Claude 会以为自己已经说了这些话，并沿着引导继续生成

#### 实际问题

默认情况下，当要求 Claude 生成 JSON 时，可能会得到：
````markdown
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
```
This rule captures EC2 instance state changes when instances start running.
````

JSON 格式正确，但被包裹在 Markdown 格式中并包含解释性文本，降低了用户体验的流畅度。

#### 解决方案实现
````python
messages = []
add_user_message(messages, "Generate a very short event bridge rule as json")
add_assistant_message(messages, "```json")
text = chat(messages, stop_sequences=["```"])
````

**工作原理：**

1. 用户消息告诉 Claude 需要生成什么内容
2. 预填充的助手消息让 Claude 以为它已经开始了一个 markdown 代码块
3. Claude 直接续写 JSON 内容
4. 当 Claude 试图用三个反引号关闭代码块时，停止序列立即终止生成

这样就能获得纯净的 JSON 输出，无需额外的格式处理。



---

### 第二部分：提示工程与评估

将提示词的评估简单分为三个步骤

第一


---

### 第三部分：Claude的工具使用

---

### 第四部分：检索增强生成 (RAG)

---

### 第五部分：Claude的特性

---

### 第六部分：模型上下文协议 (MCP)

---

### 第七部分：Anthropic应用 - Claude Code和计算机使用

---

### 第八部分：代理和工作流