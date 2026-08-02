# Python 项目实战：AI 应用开发

> **学习目标：** 理解模型调用链路，使用 API、提示词、会话记忆和 Streamlit 构建聊天应用。

## 模型与调用链路

大语言模型提供理解和生成能力；AI 应用把能力落地到具体场景。典型路径是：用户界面 -> 应用逻辑 -> 模型 API -> 响应展示 -> 会话持久化。模型可本地部署（如 Ollama），也可调用云端 API；前者数据可控但依赖算力，后者接入快但需关注网络和成本。

HTTP 请求由方法、URL、请求头和请求体组成，响应包含状态码与响应体；模型 API 常用 JSON 传递 `model`、`messages`、`stream` 等字段。

```python
from openai import OpenAI
import os
client = OpenAI(api_key=os.environ["DEEPSEEK_API_KEY"], base_url="https://api.deepseek.com")
```

> **重点：** API Key 必须只从环境变量或密钥服务读取，绝不能写进代码、日志或仓库。对网络错误、限流和空响应都要有处理。

## 提示词与会话记忆

`messages` 通常由 `system`、`user`、`assistant` 角色构成。系统提示词定义边界与格式，历史消息提供上下文。提示词要明确目标、上下文、约束和期望输出。

> **重点：** 会话历史会不断增长，应限制轮次或 Token 并摘要旧消息；模型输出是不可信输入，不能直接执行为代码或数据库命令。

## Streamlit 与持久化

Streamlit 可快速构建交互界面：`st.chat_input` 接收消息，`st.chat_message` 展示对话，`st.session_state` 保存当前会话。侧边栏可提供新建、切换、删除会话。

```python
if "messages" not in st.session_state:
    st.session_state.messages = []
prompt = st.chat_input("请输入问题")
```

会话可用 JSON 持久化，文件操作使用 `with open(..., encoding="utf-8")` 自动关闭文件，并通过 `json.dump`、`json.load` 写入和读取。实现时应处理空输入、流式中断、损坏 JSON 和上下文超限。

![课件图示](../pythonImages/第3章-Python项目实战之AI应用-p56-1cbed2a260ace292.png)
