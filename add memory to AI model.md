# LangChain 对话记忆机制学习笔记  

## 一、为什么需要对话记忆？  
大模型**本身没有“记忆”**，每次请求是独立的。若想实现多轮对话（如先问“丘吉尔是谁”，再问“他是哪国人”），需手动传递**历史对话上下文**，让模型“记住”之前的内容。  


## 二、手动实现对话记忆（理解原理）  
核心逻辑：**将历史对话存入消息列表，每次请求时连同新问题一起传给模型**。  

### 1. 初始化记忆（ConversationBufferMemory）  
```python
from langchain.memory import ConversationBufferMemory

# 初始化记忆，设置 return_messages=True 以保留消息列表格式
memory = ConversationBufferMemory(return_messages=True)

# 查看初始记忆（空列表）
print(memory.load_memory_variables({}))  
# 输出：{'history': []}
```  

### 2. 保存历史对话  
通过 `save_context` 手动储存一轮对话（用户输入 + AI 回应）：  
```python
# 模拟第一轮对话：用户问“丘吉尔是谁”，AI 回应（假设已知）
memory.save_context(
    {"input": "丘吉尔是谁"}, 
    {"output": "温斯顿·丘吉尔是英国前首相..."},
)

# 查看记忆（已存一轮对话）
print(memory.load_memory_variables({}))  
# 输出：{'history': [HumanMessage(content='丘吉尔是谁'), AIMessage(content='温斯顿·丘吉尔是英国前首相...')]}
```  

### 3. 构建带记忆的提示模板  
需用 `MessagesPlaceholder` 为历史对话预留位置：  
```python
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder

# 提示模板：系统消息 + 历史对话 + 新问题
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个知识问答助手，用简洁语言回答。"),
    MessagesPlaceholder(variable_name="history"),  # 历史对话占位符
    ("human", "{input}"),  # 新问题占位符
])
```  

### 4. 串联模型与记忆（手动实现多轮对话）  
```python
from langchain_openai import ChatOpenAI

# 初始化模型
model = ChatOpenAI(model="gpt-3.5-turbo")

# 第一轮：问“丘吉尔是谁”（手动存记忆）
response = model.invoke(
    prompt.format_messages(
        input="丘吉尔是谁", 
        history=memory.load_memory_variables({})["history"]
    )
)
memory.save_context({"input": "丘吉尔是谁"}, {"output": response.content})

# 第二轮：问“他是哪国人”（自动带历史对话）
response = model.invoke(
    prompt.format_messages(
        input="他是哪国人", 
        history=memory.load_memory_variables({})["history"]
    )
)
print(response.content)  # 输出：他是英国人（模型利用上下文正确回答）
```  


## 三、LangChain 现成工具：ConversationChain（简化开发）  
手动管理记忆繁琐，LangChain 提供 `ConversationChain` 自动处理上下文：  

### 1. 快速创建带记忆的对话链  
```python
from langchain.chains import ConversationChain

# 初始化模型 + 记忆
model = ChatOpenAI(model="gpt-3.5-turbo")
memory = ConversationBufferMemory(return_messages=True)

# 创建对话链
chain = ConversationChain(
    llm=model, 
    memory=memory, 
    verbose=True  # 打印完整提示（含历史对话）
)

# 第一轮对话（自动存记忆）
result = chain.invoke({"input": "丘吉尔是谁"})
print(result["response"])  # 输出：温斯顿·丘吉尔是英国前首相...

# 第二轮对话（自动加载历史）
result = chain.invoke({"input": "他是哪国人"})
print(result["response"])  # 输出：他是英国人（自动带上下文）
```  

### 2. 自定义提示模板（修改 AI 人设）  
```python
# 定义暴躁助手的提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个脾气暴躁、喜欢阴阳怪气的助手。"),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

# 传入自定义模板
chain = ConversationChain(
    llm=model, 
    memory=memory, 
    prompt=prompt, 
    verbose=True
)

# 测试：问“今天天气如何？”
result = chain.invoke({"input": "今天天气如何？"})
print(result["response"])  # 输出：呵，天气好不好关我屁事？自己看天气预报去！
```  


## 四、5 种记忆类型对比（场景化选择）  

| 记忆类型                     | 核心逻辑                                  | 优点                              | 缺点                              | 适用场景                     |  
|------------------------------|-------------------------------------------|-----------------------------------|-----------------------------------|------------------------------|  
| **ConversationBufferMemory** | 完整保存所有历史消息（无截断）            | 简单直接，无信息丢失              | 对话越多，Token 消耗越大，易超限  | 短对话、需完整上下文场景     |  
| **ConversationBufferWindowMemory** | 仅保存最近 `k` 轮对话（按轮数截断）       | 控制 Token 消耗，避免超限         | 超过 `k` 轮后，旧信息直接丢失     | 长对话但需关键上下文场景     |  
| **ConversationSummaryMemory** | 总结历史对话后保存（用大模型压缩内容）    | 减少 Token 消耗，保留关键信息     | 总结可能丢失细节，且需额外 Token | 长对话、需压缩历史的场景     |  
| **ConversationSummaryBufferMemory** | 结合“总结 + 缓存”：短对话存原始，长对话总结 | 平衡细节保留与 Token 消耗         | 逻辑复杂，需调试参数              | 超长对话、需精细控制场景     |  
| **ConversationTokenBufferMemory** | 按 Token 数截断历史消息（保留最近内容）   | 贴合模型上下文窗口（按 Token 限） | 旧信息可能丢失                    | 需严格控制 Token 消耗的场景 |  


## 五、总结  
- **基础逻辑**：多轮对话需手动传递历史上下文，LangChain 记忆类简化实现。  
- **工具推荐**：`ConversationChain` 自动处理记忆，适合快速开发；复杂场景需选合适记忆类型（如 `SummaryMemory` 压缩历史）。  
- **核心权衡**：记忆越完整，Token 消耗越大 → 需根据模型上下文窗口（如 GPT-4 是 8k/128k Token）选择记忆策略。  

通过记忆机制，真正实现“连续对话”，让 AI 像人类一样“记住”之前的交流内容 ✨。  
