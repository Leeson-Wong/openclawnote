# 媒体处理 Skill

## 功能定位
这个技能定义了 OpenClaw 的媒体处理系统，使用户能够管理和操作各种媒体文件。

## 核心功能与操作指导

### 媒体上传
- **上传文件**
  - 命令：`/media upload <file_path>`
  - 实现：`MediaUploader.upload()`

- **支持的格式**
  - 包括：图片、视频、音频、文档

### 媒体下载
- **下载文件**
  - 命令：`/media download <url>`
  - 实现：`MediaDownloader.fetch()`

### 图像处理
- **调整大小**
  - 命令：`/image resize <width> <height>`
  - 实现：`ImageProcessor.resize()`

- **格式转换**
  - 命令：`/image convert <format>`
  - 支持格式：JPEG, PNG, WebP

### 视频处理
- **剪辑视频**
  - 命令：`/video trim <start_time> <end_time>`
  - 实现：`VideoEditor.trim()`

- **提取音频**
  - 命令：`/video extract-audio`
  - 输出：MP3 格式的音频文件

### 音频处理
- **转录音频**
  - 命令：`/audio transcribe <file_path>`
  - 实现：`AudioTranscriber.process()`

- **格式转换**
  - 命令：`/audio convert <format>`
  - 支持格式：MP3, WAV

## 使用场景
- 管理媒体文件
- 处理图像和视频
- 转录音频内容

## 注意事项
- 大文件处理可能需要较长时间
- 不同格式的支持程度不同
- 监控媒体处理性能