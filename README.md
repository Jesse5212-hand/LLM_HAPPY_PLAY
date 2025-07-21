# LLM_PLAY

## Project Overview
LLM_HAPPY_PLAY is a repository that aggregates multiple projects based on Large Language Models (LLMs). It encompasses a variety of practical tools and applications developed using the OpenAI API and the LangChain framework. These projects primarily utilize Streamlit to build user interfaces, offering users a convenient interactive experience. Through these projects, users can effortlessly generate video scripts, popular Xiaohongshu-style copywriting, clone the ChatGPT chat assistant, and conduct intelligent Q&A on PDF documents.

## Project List

### 1. One-Click Video Script Generator
- **Functionality**: Users input the video topic, approximate duration, and script creativity level. The system supplements real-time knowledge through the Wikipedia API and uses LangChain to streamline the process, generating an engaging video title and a video script that meets the specified duration.
- **Technology Stack**: LangChain, Streamlit, Wikipedia API
- **Running Steps**:
  1. Obtain the dependencies from `06 项目1：视频脚本一键生成器/requirements.txt` and run `pip install -r requirements.txt` to install them.
  2. Run `python 06 项目1：视频脚本一键生成器/main.py` to start the application.

### 2. Popular Xiaohongshu Copywriting Generator
- **Functionality**: Users input the topic and their OpenAI API key. The system generates five Xiaohongshu-style titles and a paragraph of body content based on pre-set title and body writing techniques.
- **Technology Stack**: LangChain, Streamlit, OpenAI API
- **Running Steps**:
  1. Obtain the dependencies from `07 项目2：爆款小红书文案生成器/requirements.txt` and run `pip install -r requirements.txt` to install them.
  2. Run `python 07 项目2：爆款小红书文案生成器/main.py` to start the application.

### 3. Clone ChatGPT
- **Functionality**: Implements an AI chat assistant website similar to ChatGPT, supporting users to input their own OpenAI API key in the sidebar, enabling multi-round continuous conversations, and displaying chat records in real-time.
- **Technology Stack**: LangChain, Streamlit, OpenAI API
- **Running Steps**:
  1. Obtain the dependencies from `09 项目3：克隆ChatGPT/requirements.txt` and run `pip install -r requirements.txt` to install them.
  2. Run `python 09 项目3：克隆ChatGPT/main.py` to start the application.

### 4. Intelligent PDF Q&A Tool
- **Functionality**: Users upload a PDF file and input a question. The system conducts Q&A by integrating the OpenAI API and LangChain, supporting multi-round conversations and displaying historical messages.
- **Technology Stack**: LangChain, Streamlit, OpenAI API
- **Running Steps**:
  1. Obtain the dependencies from `11 项目4：智能PDF问答工具/requirements.txt` and run `pip install -r requirements.txt` to install them.
  2. Run `python 11 项目4：智能PDF问答工具/main.py` to start the application.

## Installation and Running

### Install Dependencies
Each project has a corresponding `requirements.txt` file. You can run the following command in the project directory to install the dependencies:
```bash
pip install -r requirements.txt
```

### Run the Project
After installing the dependencies, navigate to the corresponding project directory and run the `main.py` file to start the application:
```bash
streamlit run main.py
```

## Notes
- Before running the projects, ensure that you have obtained a valid OpenAI API key and input it in the sidebar of the application.
- Some projects use third-party APIs (such as the Wikipedia API). Please ensure that your network environment can access these APIs properly.

## Expansion Directions

### One-Click Video Script Generator
- Support multiple models, such as Wenxin Yiyan and Tongyi Qianwen. Simply modify `ChatOpenAI` to the LangChain integration class of the corresponding model.
- Add detailed requirements such as storyboards, characters, and dialogues to the prompt template to achieve complex script customization.
- Deploy the application through Streamlit Cloud or Docker.

### Popular Xiaohongshu Copywriting Generator
- Add more title writing techniques and body writing style options.
- Support integration with the Xiaohongshu platform to achieve automatic publishing functionality.

### Clone ChatGPT
- Integrate more large language models to provide a diverse range of conversation experiences.
- Add management functions for chat records, such as export and deletion.

### Intelligent PDF Q&A Tool
- Support Q&A for multiple file formats, such as DOCX and PPTX.
- Optimize the Q&A algorithm to improve the accuracy and efficiency of answers.

## Contribution
If you have any suggestions or contributions to this project, please feel free to submit an issue or a pull request. We welcome the participation and contributions of the community!

## License
This project is licensed under the [MIT License](LICENSE).
