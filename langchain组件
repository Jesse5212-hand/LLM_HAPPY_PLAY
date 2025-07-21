# LangChain 核心组件学习笔记


## 一、模型类型：LLM 与 Chat Model
### 核心区别
- **LLM（语言模型）**：  
  - 功能：文本补全（Text Completion），擅长续写、扩展文本（比如给开头生成完整文章）。  
  - 输入输出：单字符串输入 → 单字符串输出（例如输入“今天天气很好，我打算”，输出“去公园散步”）。  

- **Chat Model（聊天模型）**：  
  - 功能：对话交互（经对话场景专项优化），支持多轮上下文理解。  
  - 输入输出：消息列表（`list`）输入 → 单条消息输出（更贴近人类聊天习惯）。  

### 应用场景
- LLM 适合：文本生成（如写邮件、代码补全）、内容续写。  
- Chat Model 适合：客服机器人、智能助手等对话场景（如 GPT-3.5-turbo、GPT-4 均为此类）。  
- 现状：Chat Model 是主流，对话体验更优，功能更全面。  


## 二、集成 OpenAI 聊天模型（实操步骤）
### 1. 安装依赖
```bash
pip install langchain-openai  # 专门用于集成 OpenAI 模型的库
```

### 2. 导入与初始化
```python
from langchain_openai import ChatOpenAI

# 初始化模型实例
model = ChatOpenAI(
    model="gpt-3.5-turbo",  # 模型名称（必填，如 gpt-4、gpt-4o 等）
    temperature=0.7,        # 随机性（0=严谨，1= creative，常用 0.5-0.7）
    max_tokens=1024         # 单次输出最大 Token 数（防止回答过长）
)
```

### 3. API 密钥配置
- 推荐方式：提前设置环境变量 `OPENAI_API_KEY`（自动读取，避免代码暴露密钥）。  
  - Windows 配置：系统环境变量新增 `OPENAI_API_KEY`，值为 `sk-xxx` 开头的密钥。  
- 临时方式：显式传入（适合调试，不建议生产环境）：  
  ```python
  model = ChatOpenAI(api_key="sk-xxxxxxxxxxxx", model="gpt-3.5-turbo")
  ```

### 4. 进阶参数设置
- 不常用参数（如频率惩罚、存在惩罚）通过 `model_kwargs` 传入：  
  ```python
  model = ChatOpenAI(
      model="gpt-3.5-turbo",
      model_kwargs={
          "frequency_penalty": 0.3,  # 降低重复内容概率（0-2）
          "presence_penalty": 0.2    # 鼓励新话题（0-2）
      }
  )
  ```
- 所有参数参考：[OpenAI 官方文档](https://platform.openai.com/docs/api-reference/chat/create)（LangChain 兼容原生参数）。  


## 三、消息结构：对话的“语言”
Chat Model 用**消息列表**承载对话内容，每条消息需指定“角色”，核心角色有 3 种：

| 角色类型          | 作用                          | 示例                                  |
|-------------------|-------------------------------|---------------------------------------|
| SystemMessage     | 设定 AI 角色/规则（全局指令） | “你是一个专业翻译，只翻译不解释”      |
| HumanMessage      | 用户输入内容                  | “请把‘你好’翻译成英文”                |
| AIMessage         | AI 历史回复（用于上下文）     | （上一轮 AI 输出的“Hello”）           |

### 实操示例：多轮对话
```python
from langchain.schema.messages import SystemMessage, HumanMessage, AIMessage

# 构建消息列表（包含上下文）
messages = [
    SystemMessage(content="你是翻译助手，只输出翻译结果，不额外回复"),
    HumanMessage(content="你好，世界"),
    AIMessage(content="Hello, world"),  # 上一轮AI回复，作为上下文
    HumanMessage(content="那‘再见’呢？")  # 新的用户输入
]

# 调用模型
response = model.invoke(messages)
print(response.content)  # 输出：Goodbye
```
- 关键点：消息列表的顺序就是对话时序，AI 会根据全量上下文生成回复。  


## 四、提示模板（Prompt Template）：动态构建输入
### 为什么需要模板？
AI 输入常需包含动态内容（比如不同用户的提问、变量参数），直接硬编码字符串会很繁琐，模板可实现“固定格式 + 动态变量”的灵活组合。

### 1. 角色专属模板
针对 System/Human/AI 角色分别创建模板，适合角色分工明确的场景：
```python
from langchain.prompts import (
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate
)

# 系统消息模板（含变量：task=任务，source=源语言）
system_template = "你是{task}助手，将{source}翻译成中文，保持简洁"
system_prompt = SystemMessagePromptTemplate.from_template(system_template)

# 人类消息模板（含变量：text=待翻译文本）
human_template = "请翻译：{text}"
human_prompt = HumanMessagePromptTemplate.from_template(human_template)

# 填充变量（动态传入参数）
filled_system_msg = system_prompt.format(task="专业", source="英文")
filled_human_msg = human_prompt.format(text="Hello, LangChain")

# 组合成消息列表
messages = [filled_system_msg, filled_human_msg]

# 调用模型
response = model.invoke(messages)
print(response.content)  # 输出：你好，LangChain
```

### 2. 统一模板（ChatPromptTemplate）
用一个模板管理所有角色消息，适合快速定义简单对话：
```python
from langchain.prompts import ChatPromptTemplate

# 定义消息模板列表：[(角色, 内容模板), ...]
prompt_template = ChatPromptTemplate.from_messages([
    ("system", "你是{role}，回答不超过{max_len}个字"),
    ("human", "请解释：{concept}")
])

# 填充变量（一次传入所有变量）
filled_prompt = prompt_template.invoke({
    "role": "科普专家",
    "max_len": 20,
    "concept": "LangChain"
})

# 转换为消息列表并调用模型
messages = filled_prompt.to_messages()
response = model.invoke(messages)
print(response.content)  # 输出：LangChain是AI应用开发框架，简化组件集成
```
- 优势：一次填充所有变量，无需分别处理不同角色的模板。  


## 五、输出解析器（Output Parser）：规范输出格式
### 问题痛点
AI 输出是自然语言，格式不确定（比如有时用逗号分隔，有时用换行），直接提取信息（如列表、JSON）很麻烦。解析器可解决两个问题：  
1. 告诉 AI“必须按指定格式输出”（给指令）；  
2. 自动将 AI 输出解析为结构化数据（如列表、字典）。

### 1. 逗号分隔列表解析器（CommaSeparatedListOutputParser）
适合提取列表类结果（如关键词、选项）：
```python
from langchain.output_parsers import CommaSeparatedListOutputParser

# 1. 创建解析器
output_parser = CommaSeparatedListOutputParser()

# 2. 获取解析器给AI的格式指令（告诉AI要输出逗号分隔的列表）
format_instructions = output_parser.get_format_instructions()
# 指令内容："你的回应应该是一串以逗号分隔的值，例如：foo, bar, buzz"

# 3. 构建提示模板（包含格式指令）
prompt = ChatPromptTemplate.from_messages([
    ("system", f"生成3个水果名称，{format_instructions}"),
    ("human", "请列出当季水果")
])

# 4. 填充模板并调用模型
filled_prompt = prompt.invoke({})
messages = filled_prompt.to_messages()
response = model.invoke(messages)
# AI输出示例：苹果, 香蕉, 草莓

# 5. 解析结果（直接得到Python列表）
fruits = output_parser.invoke(response)
print(fruits)  # 输出：['苹果', '香蕉', '草莓']
```

### 2. JSON 解析器（StructuredOutputParser）
适合提取结构化数据（如键值对、多字段信息），需提前定义 JSON 格式：
```python
from langchain.output_parsers import StructuredOutputParser, ResponseSchema
from langchain.prompts import ChatPromptTemplate

# 1. 定义输出结构（字段+描述）
response_schemas = [
    ResponseSchema(name="name", description="水果名称"),
    ResponseSchema(name="season", description="当季月份，用逗号分隔")
]

# 2. 创建JSON解析器
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = output_parser.get_format_instructions()  # 告诉AI输出JSON

# 3. 构建提示
prompt = ChatPromptTemplate.from_messages([
    ("system", f"请返回1种水果的信息，{format_instructions}"),
    ("human", "举例说明")
])

# 4. 调用模型并解析
filled_prompt = prompt.invoke({})
response = model.invoke(filled_prompt.to_messages())
result = output_parser.invoke(response)

print(result)  
# 输出：{'name': '西瓜', 'season': '6,7,8'}（字典格式，可直接提取字段）
```


## 六、链（Chain）与 Runnable 接口：组件协同工作
### 核心设计
LangChain 中所有核心组件（提示模板、模型、解析器）都实现了 `Runnable` 接口，统一通过 `invoke()` 方法调用，这使得组件可以像“链条”一样串联起来，自动传递数据。

### 链的组合方式
用 `|`（管道符）连接组件，前一个组件的输出自动作为后一个组件的输入：
```python
from langchain.prompts import ChatPromptTemplate
from langchain.output_parsers import CommaSeparatedListOutputParser
from langchain_openai import ChatOpenAI

# 1. 定义组件
output_parser = CommaSeparatedListOutputParser()  # 解析器
prompt = ChatPromptTemplate.from_messages([  # 提示模板
    ("system", f"生成3个{category}，{output_parser.get_format_instructions()}"),
    ("human", "请举例")
])
model = ChatOpenAI(model="gpt-3.5-turbo")  # 模型

# 2. 组合成链：模板 → 模型 → 解析器
chain = prompt | model | output_parser

# 3. 调用链（传入变量给第一个组件：prompt）
result = chain.invoke({"category": "编程语言"})
print(result)  # 输出：['Python', 'Java', 'JavaScript']
```

### 链的优势
- 简化流程：无需手动传递中间结果（比如模板输出 → 模型输入 → 解析器输入）。  
- 灵活替换：比如将模型从 `gpt-3.5-turbo` 换成 `claude-2`，只需修改模型组件，链的调用方式不变。  
- 可扩展性：支持添加更多组件（如加入存储器、工具调用等）。  


## 七、总结
1. **模型**：LLM 用于补全，Chat Model 用于对话，优先选 Chat Model。  
2. **消息**：用 System/Human/AIMessage 构建上下文，顺序即对话时序。  
3. **模板**：动态生成输入，角色模板适合复杂场景，统一模板适合快速开发。  
4. **解析器**：解决 AI 输出格式混乱问题，支持列表、JSON 等结构化提取。  
5. **链**：通过 `|` 串联组件，实现“输入→处理→输出”全流程自动化。  

核心逻辑：用组件化思维拆解 AI 应用，每个组件负责单一功能，通过链组合实现复杂需求，大幅降低开发难度。
