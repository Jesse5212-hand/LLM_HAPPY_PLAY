# LLM_PLAY

## 项目概述
LLM_HAPPY_PLAY 是一个基于大语言模型（LLM）的项目集合仓库，涵盖了多个利用 OpenAI API 和 LangChain 框架开发的实用工具和应用。这些项目主要使用 Streamlit 构建用户界面，为用户提供了便捷的交互体验。通过这些项目，用户可以轻松生成视频脚本、小红书爆款文案、克隆 ChatGPT 聊天助手以及进行智能 PDF 问答等操作。

## 项目列表
### 1. 视频脚本一键生成器
- **功能**：用户输入视频主题、大致时长和脚本创造力，系统结合维基百科 API 补充实时知识，利用 LangChain 串联流程，生成吸引人的视频标题和符合时长要求的视频脚本。
- **技术栈**：LangChain、Streamlit、Wikipedia API
- **运行步骤**：
  1. 从 `06 项目1：视频脚本一键生成器/requirements.txt` 获取依赖，运行 `pip install -r requirements.txt` 安装。
  2. 运行 `python 06 项目1：视频脚本一键生成器/main.py` 启动应用。

### 2. 爆款小红书文案生成器
- **功能**：用户输入主题和 OpenAI API 密钥，系统根据预设的标题创作技巧和正文创作技巧，生成 5 个小红书标题和一段正文内容。
- **技术栈**：LangChain、Streamlit、OpenAI API
- **运行步骤**：
  1. 从 `07 项目2：爆款小红书文案生成器/requirements.txt` 获取依赖，运行 `pip install -r requirements.txt` 安装。
  2. 运行 `python 07 项目2：爆款小红书文案生成器/main.py` 启动应用。

### 3. 克隆 ChatGPT
- **功能**：实现一个仿 ChatGPT 的 AI 聊天助手网站，支持侧边栏输入用户自己的 OpenAI API 密钥，多轮连续对话和实时展示对话记录。
- **技术栈**：LangChain、Streamlit、OpenAI API
- **运行步骤**：
  1. 从 `09 项目3：克隆ChatGPT/requirements.txt` 获取依赖，运行 `pip install -r requirements.txt` 安装。
  2. 运行 `python 09 项目3：克隆ChatGPT/main.py` 启动应用。

### 4. 智能 PDF 问答工具
- **功能**：用户上传 PDF 文件并输入问题，系统结合 OpenAI API 和 LangChain 进行问答，支持多轮对话并展示历史消息。
- **技术栈**：LangChain、Streamlit、OpenAI API
- **运行步骤**：
  1. 从 `11 项目4：智能PDF问答工具/requirements.txt` 获取依赖，运行 `pip install -r requirements.txt` 安装。
  2. 运行 `python 11 项目4：智能PDF问答工具/main.py` 启动应用。

## 安装与运行
### 安装依赖
每个项目都有对应的 `requirements.txt` 文件，你可以在项目目录下运行以下命令安装依赖：
```bash
pip install -r requirements.txt
```

### 运行项目
在安装完依赖后，进入相应项目的目录，运行对应的 `main.py` 文件即可启动应用：
```bash
streamlit run main.py
```

## 注意事项
- 运行项目前，请确保你已经获取了有效的 OpenAI API 密钥，并在应用的侧边栏输入。
- 部分项目使用了第三方 API（如 Wikipedia API），请确保你的网络环境可以正常访问这些 API。

## 扩展方向
### 视频脚本一键生成器
- 支持多模型，如文心一言、通义千问等，只需修改 `ChatOpenAI` 为对应模型的 LangChain 集成类。
- 在提示模板中加入分镜、角色、台词等细节要求，实现复杂脚本定制。
- 通过 Streamlit Cloud 或 Docker 部署应用。

### 爆款小红书文案生成器
- 增加更多的标题创作技巧和正文写作风格选项。
- 支持与小红书平台的集成，实现自动发布功能。

### 克隆 ChatGPT
- 集成更多的大语言模型，提供多样化的对话体验。
- 增加聊天记录的管理功能，如导出、删除等。

### 智能 PDF 问答工具
- 支持多种文件格式的问答，如 DOCX、PPTX 等。
- 优化问答算法，提高回答的准确性和效率。

## 贡献
如果你对本项目有任何建议或贡献，请随时提交 issue 或 pull request。我们欢迎社区的参与和贡献！

## 许可证
本项目采用 [MIT 许可证](LICENSE)。

