# 语音功能 Skill

## 功能定位
这个技能定义了 OpenClaw 的语音处理系统，使用户能够进行文本转语音、语音识别和语音通话。

## 核心功能与操作指导

### 文本转语音 (TTS)
- **转换文本**
  - 命令：`/tts "要转换的文本"`
  - 实现：`TextToSpeechEngine.convert()`

- **支持的格式**
  - 输出格式：MP3, WAV
  - 配置路径：`config/tts/formats.json`

### 语音识别
- **识别语音**
  - 命令：`/speech recognize <audio_file>`
  - 实现：`SpeechRecognizer.process()`

- **支持的语言**
  - 包括：中文、英文等
  - 配置路径：`config/speech/languages.json`

### 语音播放
- **播放语音**
  - 命令：`/voice play <file_path>`
  - 实现：`VoicePlayer.playFile()`

### 语音通话
- **发起通话**
  - 命令：`/call start <target>`
  - 实现：`VoiceCallManager.startCall()`

- **结束通话**
  - 命令：`/call end`
  - 实现：`VoiceCallManager.endCall()`

## 使用场景
- 将文字转换为语音
- 识别语音内容
- 播放语音文件
- 进行语音通话

## 注意事项
- 需要正确配置音频设备
- 不同语言的识别效果可能不同
- 监控语音处理性能