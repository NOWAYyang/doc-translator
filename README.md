# 翻译工具

基于 FastAPI 的文档翻译工具，支持 TXT/PDF/EPUB/DOCX 格式，采用 LLM的OpenAI兼容 API 进行并行翻译，支持联网搜索、实时 SSE 流式输出、进度保存与恢复（修了bug但是不知道稳不稳定（））。

## 功能

- **多格式输入**：直接输入文本或上传 .txt/.pdf/.epub/.docx 文件
- **SSE 流式翻译**：每译完一块立即推送到前端，实时显示进度
- **并行翻译**：asyncio + Semaphore 控制并发数（默认 5 路并行）
- **联网搜索**：勾选后每次 API 调用携带 `enable_search=true`（需 DeepSeek API 支持）
- **实时日志**：记录每块 API 延迟、重试次数
- **终止翻译**：随时停止正在进行的任务，进度自动保存
- **自动保存**：每翻译完 10 块自动保存到 `progress/` 目录
- **恢复翻译**：页面刷新后自动检测上次未完成的进度，可继续翻译
- **导出结果**：TXT / Word（宋体格式化）/ PDF（A4 自动分页，中文支持）

## 安装依赖

启动时会自动用清华镜像（谢谢清华大学开源镜像站！！）检测并安装缺失依赖：

```bash
pip install fastapi uvicorn openai pypdf ebooklib python-docx beautifulsoup4 fpdf2
```

或直接启动（自动安装）：

```bash
uvicorn main:app --reload --port 8000
```

## 运行

```bash
uvicorn main:app --reload --port 8000
```

浏览器打开 `http://localhost:8000`。

## 使用联网搜索（仅限DeepSeek的API

1. 在设置中勾选 **"启用联网搜索"**
2. 后端向 DeepSeek API 发送请求时会在 body 中添加 `"enable_search": true`
3. 请确认你的 DeepSeek API Key 支持该功能

## 恢复翻译

- 翻译过程中关闭或刷新页面，进度自动保存到 `progress/<task_id>.json`
- 重新打开页面后顶部会出现"恢复翻译"横幅
- 点击"恢复翻译"按钮（需要重新输入 API Key）
- 已翻译完成的块会立即恢复显示，继续翻译剩余块
- 点击"忽略"可清除恢复提示

## 文件结构

```
translator/
├── main.py           # FastAPI 应用（全部后端逻辑）
├── static/
│   └── index.html    # Web 前端界面
├── progress/         # 自动保存的翻译进度（JSON）
└── README.md
```

## API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/translate` | POST | 简单文本翻译（同步） |
| `/translate_file` | POST | 文件翻译（轮询模式，已弃用） |
| `/stream_translate` | POST | 文件翻译（SSE 流式，推荐） |
| `/resume_translate/{id}` | POST | 恢复翻译（SSE 流式） |
| `/check_progress/{id}` | GET | 检测是否有保存的进度 |
| `/cancel/{id}` | POST | 终止正在执行的翻译任务 |
| `/export/txt` | POST | 导出 TXT |
| `/export/docx` | POST | 导出 Word 文档 |
| `/export/pdf` | POST | 导出 PDF 文档 |
| `/progress/{id}` | GET | 轮询进度（旧版兼容） |

## 配置

所有设置通过 Web 界面完成，无需环境变量：

- LLM API Key
- 模型名称（默认 `deepseek-chat`）（已弃用，等待修改）
- 最大并发数（默认 5）（可以随意改但是注意API限流与自己的CPU能力）
- 源语言 / 目标语言（必选）
- 联网搜索开关

## 写在最后
这是一个Vibe Coding的产物，初衷是为了让阅读外文文章时稍微快乐一点
以后会尝试封装之，在Web端边翻译边有前端可以看
谢谢你看到这里
:>
