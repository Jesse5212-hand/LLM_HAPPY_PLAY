# 视频脚本生成器项目学习笔记  

## 一、项目核心功能  
### 1. 用户输入  
- **OpenAI API 密钥**：支持替换为文心一言、通义千问等其他大模型密钥（需修改后端模型调用逻辑）。  
- **视频主题**：任意主题（如“Sora 模型”“美食制作”）。  
- **视频时长**：分钟级精度（支持小数，如 `0.5` 分钟）。  
- **脚本创造力**：滑块调节（0~1），对应 OpenAI 的 `temperature` 参数（值越高，输出越开放有创意）。  

### 2. 核心能力  
- **维基百科 API 补充知识**：解决大模型训练数据滞后问题（如 Sora 模型 2024 年发布，旧模型无相关信息），确保内容真实。  
- **LangChain 串联流程**：整合“标题生成 → 维基知识获取 → 脚本生成”全流程。  
- **前端交互优化**：加载态提示、结果折叠展示，提升用户体验。  


## 二、技术栈与环境搭建  
### 1. 关键技术  
| 技术/库          | 作用                                  |  
|------------------|---------------------------------------|  
| **LangChain**    | 串联大模型、提示模板、维基百科 API，实现自动化流程 |  
| **Streamlit**    | 快速搭建 Web 界面（纯 Python 编写，无需前端知识） |  
| **Wikipedia API**| 补充实时知识，避免 AI 输出“胡编乱造”          |  
| **虚拟环境**     | 隔离项目依赖（可选，小项目可跳过，避免版本冲突）  |  


### 2. 环境搭建步骤  
#### （1）创建虚拟环境（可选）  
```bash  
# Windows  
python -m venv venv  
venv\Scripts\activate  

# Linux/Mac  
python -m venv venv  
source venv/bin/activate  
```  

#### （2）安装依赖  
从课程获取 `requirements.txt`，运行：  
```bash  
pip install -r requirements.txt  
```  
**核心依赖**：`langchain-openai`、`langchain-community`、`streamlit`、`wikipedia-api`  


## 三、后端逻辑：`generate_script` 函数解析  
### 1. 功能拆解  
1. **生成视频标题**：通过提示模板让 AI 生成吸睛标题。  
2. **获取维基百科知识**：调用维基百科 API，补充实时信息（支持中文）。  
3. **生成视频脚本**：结合标题、时长、维基知识，让 AI 生成完整脚本。  


### 2. 代码核心片段  
```python  
from langchain.prompts import ChatPromptTemplate  
from langchain_openai import ChatOpenAI  
from langchain_community.utilities import WikipediaAPIWrapper  

def generate_script(subject, duration, creativity, api_key):  
    # 1. 定义标题生成模板  
    title_template = ChatPromptTemplate.from_messages([  
        ("human", "为「{subject}」生成一个吸睛的视频标题")  
    ])  

    # 2. 定义脚本生成模板（结合标题、时长、维基知识）  
    script_template = ChatPromptTemplate.from_messages([  
        ("system", "你是专业视频脚本创作者，结合以下信息生成脚本："),  
        ("human", "标题：{title}\n时长：{duration}分钟\n维基百科知识：{wikipedia}\n请生成详细脚本")  
    ])  

    # 3. 初始化 OpenAI 模型（使用用户密钥）  
    model = ChatOpenAI(  
        api_key=api_key,  
        temperature=creativity  # 创造力直接映射 temperature 参数  
    )  

    # 4. 生成标题（通过 Chain 串联模板和模型）  
    title_chain = title_template | model  
    title = title_chain.invoke({"subject": subject}).content  

    # 5. 获取维基百科知识（中文）  
    wikipedia = WikipediaAPIWrapper(lang="zh").run(subject)  

    # 6. 生成脚本（同样通过 Chain 串联）  
    script_chain = script_template | model  
    script = script_chain.invoke({  
        "title": title,  
        "duration": duration,  
        "wikipedia": wikipedia  
    }).content  

    return wikipedia, title, script  
```  


## 四、前端搭建：Streamlit 实现  
### 1. 核心组件与交互逻辑  
#### （1）侧边栏：API 密钥输入  
```python  
import streamlit as st  

st.set_page_config(page_title="视频脚本生成器")  
st.title("🎬 视频脚本生成器")  

# 侧边栏 - API 密钥输入  
with st.sidebar:  
    api_key = st.text_input("请输入OpenAI API密钥", type="password")  
    st.markdown("[获取OpenAI API密钥](https://platform.openai.com/account/api-keys)")  # 官方链接  
```  

#### （2）主界面：用户参数输入  
```python  
# 主界面 - 视频参数输入  
subject = st.text_input("请输入视频的主题")  
duration = st.number_input("视频时长（分钟）", min_value=0.1, step=0.1, value=1.0)  
creativity = st.slider("脚本创造力（0=严谨，1=多样）", 0.0, 1.0, 0.5)  
generate_btn = st.button("生成脚本")  
```  

#### （3）交互逻辑：校验 + 加载态 + 结果展示  
```python  
if generate_btn:  
    # 校验输入（无密钥、无主题、时长过短）  
    if not api_key:  
        st.error("请先输入 OpenAI API 密钥！")  
        st.stop()  # 终止后续执行  
    if not subject:  
        st.error("请输入视频主题！")  
        st.stop()  
    if duration < 0.1:  
        st.error("时长需≥0.1分钟！")  
        st.stop()  

    # 加载态提示（AI 思考时显示）  
    with st.spinner("AI 思考中..."):  
        wikipedia, title, script = generate_script(subject, duration, creativity, api_key)  

    # 展示结果  
    st.success("视频脚本已生成！")  
    st.subheader("🔥 标题：")  
    st.write(title)  
    st.subheader("📃 视频脚本：")  
    st.write(script)  

    # 维基知识（折叠展示，可选看）  
    with st.expander("维基百科参考内容"):  
        st.write(wikipedia)  
```  


## 五、关键细节与扩展方向  
### 1. 温度参数（creativity）的意义  
- `temperature=0`：输出保守、严谨（适合教程、科普类脚本）。  
- `temperature=1`：输出开放、有创意（适合故事、剧情类脚本）。  

### 2. 维基百科 API 的价值  
- 解决大模型“知识滞后”问题（如 Sora 模型 2024 年发布，旧模型无数据），确保 AI 输出基于真实信息。  

### 3. 扩展方向  
- **多模型支持**：替换为文心一言、通义千问等，只需修改 `ChatOpenAI` 为对应模型的 LangChain 集成类。  
- **复杂脚本定制**：在提示模板中加入分镜、角色、台词等细节要求。  
- **部署上线**：通过 Streamlit Cloud 或 Docker 部署（参考课程“应用部署”章节）。  


## 六、学习总结  
- **核心流程**：用户输入 → 维基补全 → LangChain 串联模型 → 结果展示。  
- **LangChain 优势**：通过“链（Chain）”简化复杂流程，让非专业开发者也能快速搭建 AI 应用。  
- **Streamlit 优势**：纯 Python 代码搭建 Web 界面，适合 AI 应用快速原型开发。  

后续可尝试扩展功能（如多语言支持、脚本格式定制），或替换为其他大模型，深化对 LangChain 生态的理解！  
