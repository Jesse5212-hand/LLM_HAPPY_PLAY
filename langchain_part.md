# Learning Notes on Core Components of LangChain

## I. Model Types: LLM and Chat Model
### Core Differences
- **LLM (Language Model)**
  - **Function**: Text completion. It excels at续写 and expanding text (e.g., generating a complete article given a beginning).
  - **Input and Output**: Single string input → single string output (e.g., input "The weather is nice today. I plan to", output "go for a walk in the park").

- **Chat Model**
  - **Function**: Dialogue interaction (specially optimized for dialogue scenarios), supporting multi - round context understanding.
  - **Input and Output**: List of messages (`list`) input → single message output (closer to human chatting habits).

### Application Scenarios
- **LLM is suitable for**: Text generation (e.g., writing emails, code completion), content continuation.
- **Chat Model is suitable for**: Dialogue scenarios such as customer service robots and intelligent assistants (e.g., GPT - 3.5 - turbo and GPT - 4 belong to this category).
- **Current Situation**: Chat Model is the mainstream, offering a better dialogue experience and more comprehensive functions.

## II. Integrating OpenAI Chat Model (Practical Steps)
### 1. Install Dependencies
```bash
pip install langchain-openai  # A library specifically for integrating OpenAI models
```

### 2. Import and Initialize
```python
from langchain_openai import ChatOpenAI

# Initialize the model instance
model = ChatOpenAI(
    model="gpt-3.5-turbo",  # Model name (required, e.g., gpt-4, gpt-4o, etc.)
    temperature=0.7,        # Randomness (0 = rigorous, 1 = creative, commonly used range: 0.5 - 0.7)
    max_tokens=1024         # Maximum number of Tokens for a single output (to prevent overly long answers)
)
```

### 3. API Key Configuration
- **Recommended Method**: Set the environment variable `OPENAI_API_KEY` in advance (automatically read to avoid exposing the key in the code).
  - **Windows Configuration**: Add a new system environment variable `OPENAI_API_KEY` with a value starting with `sk-xxx`.
- **Temporary Method**: Pass the key explicitly (suitable for debugging, not recommended for production environments):
```python
model = ChatOpenAI(api_key="sk-xxxxxxxxxxxx", model="gpt-3.5-turbo")
```

### 4. Advanced Parameter Settings
- Uncommon parameters (e.g., frequency penalty, presence penalty) are passed through `model_kwargs`:
```python
model = ChatOpenAI(
    model="gpt-3.5-turbo",
    model_kwargs={
        "frequency_penalty": 0.3,  # Reduce the probability of repeated content (0 - 2)
        "presence_penalty": 0.2    # Encourage new topics (0 - 2)
    }
)
```
- Refer to all parameters in the [OpenAI official documentation](https://platform.openai.com/docs/api-reference/chat/create) (LangChain is compatible with native parameters).

## III. Message Structure: The "Language" of Dialogue
Chat Model uses a **list of messages** to carry dialogue content. Each message needs to specify a "role", and there are 3 core roles:

| Role Type       | Function                          | Example                               |
|-----------------|-----------------------------------|---------------------------------------|
| SystemMessage   | Set the AI's role/rules (global instructions) | "You are a professional translator, only translate without explanation" |
| HumanMessage    | User input content                | "Please translate 'Hello' into English" |
| AIMessage       | AI's historical responses (for context) | (The "Hello" output by the AI in the previous round) |

### Practical Example: Multi - Round Dialogue
```python
from langchain.schema.messages import SystemMessage, HumanMessage, AIMessage

# Build a list of messages (including context)
messages = [
    SystemMessage(content="You are a translation assistant, only output the translation result without additional replies"),
    HumanMessage(content="Hello, world"),
    AIMessage(content="Hello, world"),  # The AI's response from the previous round, used as context
    HumanMessage(content="What about 'Goodbye'?")  # New user input
]

# Call the model
response = model.invoke(messages)
print(response.content)  # Output: Goodbye
```
- **Key Point**: The order of the message list represents the dialogue sequence, and the AI will generate responses based on the full context.

## IV. Prompt Template: Dynamically Build Input
### Why Do We Need Templates?
AI input often needs to include dynamic content (e.g., different users' questions, variable parameters). Hard - coding strings directly can be cumbersome. Templates enable flexible combinations of "fixed format + dynamic variables".

### 1. Role - Specific Templates
Create templates for System/Human/AI roles respectively, suitable for scenarios with clear role divisions:
```python
from langchain.prompts import (
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate
)

# System message template (including variables: task = task, source = source language)
system_template = "You are a {task} assistant, translate {source} into Chinese concisely"
system_prompt = SystemMessagePromptTemplate.from_template(system_template)

# Human message template (including variable: text = text to be translated)
human_template = "Please translate: {text}"
human_prompt = HumanMessagePromptTemplate.from_template(human_template)

# Fill in variables (dynamically pass parameters)
filled_system_msg = system_prompt.format(task="professional", source="English")
filled_human_msg = human_prompt.format(text="Hello, LangChain")

# Combine into a list of messages
messages = [filled_system_msg, filled_human_msg]

# Call the model
response = model.invoke(messages)
print(response.content)  # Output: Hello, LangChain
```

### 2. Unified Template (ChatPromptTemplate)
Use a single template to manage messages for all roles, suitable for quickly defining simple dialogues:
```python
from langchain.prompts import ChatPromptTemplate

# Define a list of message templates: [(role, content template), ...]
prompt_template = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}, answer within {max_len} characters"),
    ("human", "Please explain: {concept}")
])

# Fill in variables (pass all variables at once)
filled_prompt = prompt_template.invoke({
    "role": "science popularizer",
    "max_len": 20,
    "concept": "LangChain"
})

# Convert to a list of messages and call the model
messages = filled_prompt.to_messages()
response = model.invoke(messages)
print(response.content)  # Output: LangChain is an AI application development framework that simplifies component integration
```
- **Advantage**: Fill in all variables at once without separately handling templates for different roles.

## V. Output Parser: Standardize Output Format
### Problem Pain Points
AI output is in natural language with an uncertain format (e.g., sometimes separated by commas, sometimes by line breaks). Directly extracting information (e.g., lists, JSON) can be troublesome. Parsers can solve two problems:
1. Tell the AI "must output in the specified format" (give instructions).
2. Automatically parse the AI's output into structured data (e.g., lists, dictionaries).

### 1. Comma - Separated List Output Parser (CommaSeparatedListOutputParser)
Suitable for extracting list - type results (e.g., keywords, options):
```python
from langchain.output_parsers import CommaSeparatedListOutputParser

# 1. Create a parser
output_parser = CommaSeparatedListOutputParser()

# 2. Get the format instructions from the parser for the AI (tell the AI to output a comma - separated list)
format_instructions = output_parser.get_format_instructions()
# Instruction content: "Your response should be a comma - separated list of values, e.g., foo, bar, buzz"

# 3. Build a prompt template (including format instructions)
prompt = ChatPromptTemplate.from_messages([
    ("system", f"Generate 3 fruit names, {format_instructions}"),
    ("human", "Please list seasonal fruits")
])

# 4. Fill in the template and call the model
filled_prompt = prompt.invoke({})
messages = filled_prompt.to_messages()
response = model.invoke(messages)
# Example AI output: Apple, Banana, Strawberry

# 5. Parse the result (directly get a Python list)
fruits = output_parser.invoke(response)
print(fruits)  # Output: ['Apple', 'Banana', 'Strawberry']
```

### 2. JSON Parser (StructuredOutputParser)
Suitable for extracting structured data (e.g., key - value pairs, multi - field information). The JSON format needs to be defined in advance:
```python
from langchain.output_parsers import StructuredOutputParser, ResponseSchema
from langchain.prompts import ChatPromptTemplate

# 1. Define the output structure (field + description)
response_schemas = [
    ResponseSchema(name="name", description="Fruit name"),
    ResponseSchema(name="season", description="Seasonal months, separated by commas")
]

# 2. Create a JSON parser
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = output_parser.get_format_instructions()  # Tell the AI to output JSON

# 3. Build a prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", f"Please return information about 1 fruit, {format_instructions}"),
    ("human", "Give an example")
])

# 4. Call the model and parse
filled_prompt = prompt.invoke({})
response = model.invoke(filled_prompt.to_messages())
result = output_parser.invoke(response)

print(result)  
# Output: {'name': 'Watermelon', 'season': '6,7,8'} (dictionary format, fields can be directly extracted)
```

## VI. Chain and Runnable Interface: Components Working Together
### Core Design
All core components (prompt templates, models, parsers) in LangChain implement the `Runnable` interface and are uniformly called through the `invoke()` method. This allows components to be chained together like a "chain" to automatically transfer data.

### Chain Combination Method
Use the `|` (pipe operator) to connect components. The output of the previous component automatically serves as the input of the next component:
```python
from langchain.prompts import ChatPromptTemplate
from langchain.output_parsers import CommaSeparatedListOutputParser
from langchain_openai import ChatOpenAI

# 1. Define components
output_parser = CommaSeparatedListOutputParser()  # Parser
prompt = ChatPromptTemplate.from_messages([  # Prompt template
    ("system", f"Generate 3 {category}, {output_parser.get_format_instructions()}"),
    ("human", "Please give examples")
])
model = ChatOpenAI(model="gpt-3.5-turbo")  # Model

# 2. Combine into a chain: template → model → parser
chain = prompt | model | output_parser

# 3. Call the chain (pass variables to the first component: prompt)
result = chain.invoke({"category": "programming languages"})
print(result)  # Output: ['Python', 'Java', 'JavaScript']
```

### Advantages of Chains
- **Simplify the process**: No need to manually transfer intermediate results (e.g., template output → model input → parser input).
- **Flexible replacement**: For example, changing the model from `gpt-3.5-turbo` to `claude-2` only requires modifying the model component, and the way of calling the chain remains the same.
- **Scalability**: Supports adding more components (e.g., adding a memory store, tool calls, etc.).

## VII. Summary
1. **Models**: Use LLM for completion and Chat Model for dialogue. Prioritize Chat Model.
2. **Messages**: Build context using System/Human/AIMessage. The order represents the dialogue sequence.
3. **Templates**: Dynamically generate input. Role - specific templates are suitable for complex scenarios, and unified templates are suitable for rapid development.
4. **Parsers**: Solve the problem of messy AI output formats, supporting structured extraction such as lists and JSON.
5. **Chains**: Connect components through `|` to automate the entire process of "input → processing → output".

**Core Logic**: Decompose AI applications using a component - based approach. Each component is responsible for a single function, and complex requirements are achieved through chain combinations, significantly reducing development difficulty.
