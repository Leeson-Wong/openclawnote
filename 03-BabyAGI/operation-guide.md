# BabyAGI 项目 - 操作指南

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

## 核心功能使用

### 1. 注册函数

#### 基本注册

```python
import babyagi

@babyagi.register_function()
def world():
    return "world"
```

#### 带依赖的注册

```python
@babyagi.register_function(dependencies=["world"])
def hello_world():
    x = world()
    return f"Hello {x}!"
```

#### 完整元数据

```python
@babyagi.register_function(
    imports=["math"],
    dependencies=["circle_area"],
    key_dependencies=["openai_api_key"],
    metadata={
        "description": "使用 circle_area 函数计算圆柱体体积"
    }
)
def cylinder_volume(radius, height):
    import math
    area = circle_area(radius)
    return area * height
```

### 2. 加载函数包

#### 加载自定义包

```python
import babyagi

# 加载自定义函数包
babyagi.load_functions("path/to/your/custom_functions.py")
```

#### 加载内置包

```python
# 加载默认函数
babyagi.load_functions("default")

# 加载 AI 函数
babyagi.load_functions("ai_generator")

# 加载代码写作函数
babyagi.load_functions("drafts/code_writing_functions")
```

### 3. 密钥管理

#### 从代码添加密钥

```python
import babyagi
import os

babyagi.add_key_wrapper('openai_api_key', os.environ['OPENAI_API_KEY'])
```

#### 从 Dashboard 添加密钥

1. 访问 http://localhost:8080/dashboard
2. 导航到 "Key Manager"
3. 使用 "Add Key" 功能
4. 安全添加密钥

### 4. 执行函数

#### 直接调用

```python
import babyagi

# 注册函数
@babyagi.register_function()
def greet(name):
    return f"Hello, {name}!"

# 执行函数
result = babyagi.greet("World")
print(result)  # Hello, World!
```

#### 处理依赖

```python
# BabyAGI 自动解析和注入依赖
# 无需手动加载依赖
```

### 5. 使用自构建功能

#### process_user_input

```python
import babyagi
import os

# 设置密钥
babyagi.add_key_wrapper('openai_api_key', os.environ['OPENAI_API_KEY'])

# 加载代码写作函数
babyagi.load_functions("drafts/code_writing_functions")

# 处理用户输入
babyagi.process_user_input("从 ESPN 获取今天的比分并发送给 test@test.com")
```

#### self_build

```python
# 加载自构建包
babyagi.load_functions("drafts/self_build")

# 生成 X 个任务
babyagi.self_build("企业 SaaS 公司的销售人员", 3)
```

### 6. 使用 Dashboard

#### 访问 Dashboard

打开浏览器: http://localhost:8080/dashboard

#### Dashboard 功能

##### Function Manager
- 查看所有注册的函数
- 注册新函数
- 更新或删除函数
- 查看函数元数据

##### Dependency View
- 可视化函数依赖关系
- 理解函数之间的关系
- 检测循环依赖

##### Key Manager
- 添加新密钥
- 管理现有密钥
- 查看密钥使用情况

##### Log Viewer
- 查看函数执行日志
- 过滤日志（按时间、函数、类型）
- 查看输入、输出、执行时间

##### Trigger Manager
- 设置触发器
- 查看现有触发器
- 管理自动化工作流

## 配置管理

### 环境变量

```bash
# 设置 API 密钥
export OPENAI_API_KEY="your-api-key"
```

### Dashboard 配置

通过 Dashboard 界面管理配置：
- 端口配置
- 日志级别
- 函数行为

## 常见问题排查

### 1. 依赖错误

#### 问题
```
ImportError: cannot import 'some_module'
```

#### 解决方案
```python
# 在函数元数据中添加 imports
@babyagi.register_function(imports=["some_module"])
def my_function():
    import some_module
    # ...
```

### 2. 密钥错误

#### 问题
```
KeyError: 'openai_api_key'
```

#### 解决方案
```python
# 添加密钥
babyagi.add_key_wrapper('openai_api_key', 'your-key')

# 或从 Dashboard 添加
```

### 3. 循环依赖

#### 问题
```
CircularDependencyError: Function A depends on B, B depends on A
```

#### 解决方案
- 重新设计函数，避免循环依赖
- 提取共同依赖到第三个函数

### 4. 生成的代码不工作

#### 问题
- AI 生成的代码质量低

#### 解决方案
- 手动审查和改进生成的代码
- 提供更详细的提示
- 迭代改进

## 开发工作流

### 典型开发流程

```
1. 定义函数需求
   ↓
2. 使用 @babyagi.register_function() 注册
   ↓
3. 定义依赖和元数据
   ↓
4. 使用 Dashboard 测试
   ↓
5. 查看日志调试
   ↓
6. 优化和迭代
```

### 创建函数包

```python
# my_functions.py
import babyagi

@babyagi.register_function()
def function1():
    return "result 1"

@babyagi.register_function(dependencies=["function1"])
def function2():
    result = function1()
    return f"result 2: {result}"
```

```python
# main.py
import babyagi

babyagi.load_functions("my_functions.py")
```

## 测试

### 单元测试

```python
import babyagi
import unittest

class TestFunctions(unittest.TestCase):
    def test_world(self):
        @babyagi.register_function()
        def world():
            return "world"
        
        result = world()
        self.assertEqual(result, "world")

if __name__ == '__main__':
    unittest.main()
```

## 社区资源

### 官方资源
- **GitHub**: https://github.com/yoheinakajima/babyagi
- **归档版本**: https://github.com/yoheinakajima/babyagi_archive
- **Twitter/X**: https://x.com/yoheinakajima
- **介绍帖子**: https://x.com/yoheinakajima/status/1840678823681282228

### 贡献
- **贡献表格**: https://forms.gle/UZLyT75HQULr8XNUA
- **注意**: 开发可能较慢，作者在夜间和周末工作

## 最佳实践

### 1. 函数设计
- 单一职责
- 清晰的输入和输出
- 完善的元数据
- 适当的错误处理

### 2. 依赖管理
- 最小化依赖
- 避免循环依赖
- 使用 imports 明确声明

### 3. 密钥安全
- 不硬编码密钥
- 使用环境变量
- 通过 Dashboard 管理

### 4. 测试
- 充分测试函数
- 查看日志调试
- 验证依赖关系

### 5. 使用自构建功能

### 重要警告
⚠️ **实验性质**
- 生成的代码是最小化的
- 可能需要显著改进
- 不适用于生产环境
- 仅用于学习和实验

## 快速参考

### 常用命令

```python
import babyagi

# 注册函数
@babyagi.register_function()

# 加载函数包
babyagi.load_functions("path/to/file.py")

# 添加密钥
babyagi.add_key_wrapper('key_name', 'key_value')

# 处理用户输入
babyagi.process_user_input("task description")

# 自我构建
babyagi.self_build("user description", count)

# 创建 Dashboard
app = babyagi.create_app('/dashboard')
app.run(host='0.0.0.0', port=8080)
```

---

## 分析日期
2026-02-03
