# Fun-ASR 语音识别服务

基于 Fun-ASR 和 Paraformer 模型的本地语音识别 HTTP 服务。

## 环境要求

- Python 3.13
- macOS (Apple Silicon)

## 安装

```bash
# 安装依赖
pip install funasr uvicorn fastapi python-multipart
```

## 启动服务

```bash
source .venv/bin/activate
python app.py
```

服务将在 `http://localhost:8000` 启动。

首次启动时，模型会自动从 ModelScope 下载（约 840MB）。

## API 接口

### 语音识别

**POST** `/asr`

接受音频文件，返回识别文本。

**支持的音频格式**: wav, mp3, m4a

**示例（curl）**:

```bash
# 识别 wav 文件
curl -X POST "http://localhost:8000/asr" \
  -F "file=@/path/to/audio.wav"

# 识别 mp3 文件
curl -X POST "http://localhost:8000/asr" \
  -F "file=@/path/to/audio.mp3"
```

**响应**:

```json
{
  "success": true,
  "text": "识别文本内容",
  "filename": "audio.wav"
}
```

### 健康检查

**GET** `/health`

```bash
curl http://localhost:8000/health
```

**响应**:

```json
{
  "status": "ok",
  "model": "paraformer"
}
```

## API 文档

启动服务后访问:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## 模型说明

- Fun-ASR 的 Paraformer-zh-NANO 轻量模型在当前版本（v1.3.1）注册表中不可用
- 当前使用 `paraformer` 模型（约 840MB）
- 模型来源：[ModelScope](https://www.modelscope.cn) iic/speech_paraformer-large_asr_nat-zh-cn-16k-common-vocab8404-pytorch
- 模型文件会缓存到本地 ~/.cache/modelscope/hub，无需重复下载
