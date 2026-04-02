# 🎙️ Fun-ASR 本地语音识别服务

基于阿里巴巴 **Fun-ASR** 框架的本地语音识别 HTTP 服务，支持 **ASR 自动语音识别 + 标点恢复** 联合推理，开箱即用。

## ✨ 功能特性

- **语音识别（ASR）**：基于 Paraformer-large 模型，中文识别精度高
- **标点恢复**：自动为识别结果添加句号、逗号、问号等标点符号
- **多格式支持**：支持 wav / mp3 / m4a 音频格式
- **本地部署**：模型和数据全部在本地，无需联网调用 API
- **RESTful API**：提供 HTTP 接口，方便集成到各类系统

## 📦 环境要求

- **操作系统**：macOS (Apple Silicon) / Linux
- **Python**：3.13+
- **内存**：推荐 8GB+（模型加载占用约 2GB）
- **磁盘**：预留 2GB+ 空间（模型文件约 1.9GB）

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone git@github.com:LiangJunChan/fun-asr-deploy.git
cd fun-asr-deploy
```

### 2. 创建虚拟环境（推荐）

```bash
python3 -m venv .venv
source .venv/bin/activate      # macOS/Linux
# .venv\Scripts\activate      # Windows
```

### 3. 安装依赖

```bash
pip install -r requirements.txt
```

> 如果使用 Apple Silicon Mac，遇到 torch 安装问题可参考：[PyTorch Apple Silicon 安装指南](https://pytorch.org/get-started/locally/)

### 4. 启动服务

```bash
python app.py
```

首次启动时，模型会自动从 [ModelScope](https://www.modelscope.cn) 下载（约 1.9GB），请耐心等待。

```
INFO:     Uvicorn running on http://127.0.0.1:8000
模型加载完成，服务已就绪
```

## 📖 API 文档

服务启动后，可通过浏览器访问交互式 API 文档：

| 文档类型 | 地址 |
|---------|------|
| **Swagger UI** | http://localhost:8000/docs |
| **ReDoc** | http://localhost:8000/redoc |

## 🔌 API 接口

### 1. 语音识别

**POST** `/asr`

上传音频文件，返回识别文本（带标点）。

**请求示例（curl）**：

```bash
# 识别 wav 文件
curl -X POST "http://localhost:8000/asr" \
  -F "file=@./audio.wav"

# 识别 mp3 文件
curl -X POST "http://localhost:8000/asr" \
  -F "file=@./audio.mp3"

# 识别 m4a 文件
curl -X POST "http://localhost:8000/asr" \
  -F "file=@./audio.m4a"
```

**响应示例**：

```json
{
  "success": true,
  "text": "今天天气很好，我们下午去逛街吧，然后去看电影，怎么样？",
  "filename": "audio.wav",
  "punctuation": true
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | bool | 是否识别成功 |
| `text` | string | 识别文本（已带标点） |
| `filename` | string | 原文件名 |
| `punctuation` | bool | 标点恢复是否启用 |

### 2. 健康检查

**GET** `/health`

```bash
curl http://localhost:8000/health
```

**响应示例**：

```json
{
  "status": "ok",
  "model": "paraformer",
  "punctuation": true
}
```

## 🧪 测试

### 使用 macOS 内置语音测试

```bash
# 启动服务后，另开终端生成测试音频并识别
say -v "Mei-Jia" -o /tmp/test.m4a "今天天气很好，我们下午去逛街吧，然后去看电影，怎么样？"
curl -X POST "http://localhost:8000/asr" -F "file=@/tmp/test.m4a"
```

### 使用 Python 客户端调用

```python
import requests

url = "http://localhost:8000/asr"
with open("audio.wav", "rb") as f:
    files = {"file": ("audio.wav", f, "audio/wav")}
    resp = requests.post(url, files=files)
    print(resp.json())
```

## 🏗️ 项目结构

```
fun-asr-deploy/
├── app.py              # FastAPI 服务主程序
├── README.md           # 本文档
├── requirements.txt     # 依赖列表（如需）
└── .venv/              # Python 虚拟环境（不提交到git）
```

## 📦 模型说明

| 模型 | 用途 | 大小 | 来源 |
|------|------|------|------|
| `paraformer` | ASR 语音识别 | ~840MB | ModelScope |
| `ct-punc` | 标点恢复 | ~1.05GB | ModelScope |

模型文件缓存在 `~/.cache/modelscope/hub/`，首次下载后无需重复下载。

## 🔧 常见问题

### Q: 标点恢复没生效？

确保 `punctuation: true`（健康检查接口可查看）。如果是 `false`，可能是标点模型未加载成功，查看启动日志确认"ASR + 标点恢复模型加载成功"。

### Q: 内存不足？

Paraformer-large 模型较大，如果内存不够，可以考虑使用更小的模型或关闭标点恢复（注释掉 `punc_model` 参数）。

### Q: 识别结果为空？

- 确认音频文件是有效格式（wav/mp3/m4a）
- 确认音频中有可识别的人声（不是纯音乐）
- 尝试用系统语音生成测试音频验证

### Q: Apple Silicon Mac 安装 torch 失败？

```bash
# 使用苹果官方版本
pip install torch --extra-index-url https://download.pytorch.org/whl/unstable
```

## 📄 License

MIT License

## 🙏 致谢

- [Fun-ASR](https://github.com/modelscope/Fun-ASR) — 阿里巴巴开源语音识别框架
- [ModelScope](https://www.modelscope.cn) — 模型托管平台
