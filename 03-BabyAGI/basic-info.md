# BabyAGI 项目 - 基本信息

## 项目来源
- GitHub: https://github.com/yoheinakajima/babyagi
- 作者: Yohei Nakajima
- 归档版本: https://github.com/yoheinakajima/babyagi_archive (2024年9月快照)
- 许可证: MIT License

## 项目简介

BabyAGI 是一个实验性的自构建自主 Agent 框架。这是一个由 Yohei 构建的框架，他从未担任过开发人员职位。该仓库的目的是分享想法和激发讨论，供有经验的开发者进行实验。

**重要声明**:
- ⚠️ **不是为生产使用设计的**
- ⚠️ **实验性框架**
- ⚠️ **使用时需要谨慎**

## 核心概念

### 自构建 Agent (Self-Building Agent)
- 早期扩展 BabyAGI 的努力表明，构建通用自主 Agent 的最佳方式是构建能够自我构建的最简单的东西
- 这是一个概念性实验，探索 Agent 如何能够自己生成和改进代码

### Functionz 框架
- 新的函数框架，用于存储、管理和执行数据库中的函数
- 提供基于图的结构来跟踪导入、依赖函数和认证密钥
- 自动加载和全面的日志记录能力
- 附带一个仪表板，用于管理函数、运行更新和查看日志

## 核心特点

### 1. 函数注册和管理
- **register_function()**: 注册函数及其元数据
- **load_functions()**: 加载函数包或自定义函数
- **依赖管理**: 自动解析函数之间的依赖关系

### 2. 元数据系统
- **imports**: 函数依赖的外部库
- **dependencies**: 此函数依赖的其他函数
- **key_dependencies**: 函数所需的密钥
- **metadata["description"]**: 函数功能的描述

### 3. 图结构
- 基于图的结构跟踪导入
- 依赖关系可视化
- 认证密钥管理

### 4. 自动执行环境
- 自动加载必要的函数包
- 管理它们的依赖关系
- 确保无缝的执行环境

### 5. 全面的日志系统
- 执行跟踪（函数开始和结束）
- 错误日志（捕获和记录错误）
- 依赖管理（自动解决和记录）
- 触发日志（记录触发的函数）

### 6. Dashboard（仪表板）
- 函数管理（注册、注销、更新）
- 依赖可视化（查看和管理依赖关系）
- 密钥管理（安全添加和管理密钥）
- 日志和监控（访问全面的执行日志）
- 触发管理（设置自动化执行）

### 7. 自构建能力
- **process_user_input**: 决定使用现有函数还是生成新函数
- **self_build**: 根据用户描述生成任务

## 快速开始

### 安装

```bash
pip install babyagi
```

### 运行 Dashboard

```python
import babyagi

if __name__ == "__main__":
    app = babyagi.create_app('/dashboard')
    app.run(host='0.0.0.0', port=8080)
```

访问: http://localhost:8080/dashboard

### 基本使用

```python
import babyagi

# 注册简单函数
@babyagi.register_function()
def world():
    return "world"

# 注册依赖 'world' 的函数
@babyagi.register_function(dependencies=["world"])
def hello_world():
    x = world()
    return f"Hello {x}!"

# 执行函数
print(babyagi.hello_world())  # Output: Hello world!
```

## 预加载的函数包

### 1. Default Functions (packs/default_functions.py)
- 函数执行（运行、添加、更新、检索函数和版本）
- 密钥管理（添加和检索密钥）
- 触发器（添加基于其他函数的触发器）
- 日志（检索日志，可选过滤）

### 2. AI Functions (packs/ai_generator.py)
- AI 描述和嵌入（自动生成函数描述和嵌入）
- 函数选择（基于提示查找或选择类似函数）

## 关键依赖

### 存储密钥
- 从代码直接存储
- 通过 Dashboard 管理

### 示例

```python
import babyagi
import os

# 添加密钥
babyagi.add_key_wrapper('openai_api_key', os.environ['OPENAI_API_KEY'])
```

## 系统要求

- Python 3.8+
- OpenAI API Key（用于 AI 功能）
- Web 浏览器（用于访问 Dashboard）

## 资源链接

- **GitHub**: https://github.com/yoheinakajima/babyagi
- **归档版本**: https://github.com/yoheinakajima/babyagi_archive
- **Twitter/X**: https://x.com/yoheinakajima
- **介绍帖子**: https://x.com/yoheinakajima/status/1840678823681282228
- **贡献表格**: https://forms.gle/UZLyT75HQULr8XNUA

## 技术栈

- **语言**: Python
- **Web 框架**: Flask（推测）
- **数据存储**: 数据库（未指定具体类型）
- **AI 集成**: OpenAI API
- **许可证**: MIT License

## 注意事项

### 重要警告
1. **实验性质** - 这是实验性框架，不是生产就绪
2. **作者背景** - 作者从未担任过开发人员职位
3. **谨慎使用** - 仅供学习和实验，不应用于生产环境
4. **开发速度** - 开发可能较慢（作者在夜间和周末工作）
5. **代码质量** - 生成的代码可能需要改进

### 限制
- 生成的代码是最小化的，可能需要改进
- 实验性功能可能无法按预期工作
- 需要显著改进才能实用

## 项目历史

### 原始 BabyAGI (2023年3月)
- 引入任务规划作为开发自主 Agent 的方法
- 2024年9月归档，移至 babyagi_archive

### 新 BabyAGI (当前版本)
- 自构建 Agent 的实验框架
- 专注于能够自我构建的最简单的东西
- Functionz 框架为核心

## 核心理念

"构建通用自主 Agent 的最佳方式是构建能够自我构建的最简单的东西。"

这一理念探索 Agent 如何能够：
1. 自我生成代码
2. 改进自身功能
3. 自动扩展能力
