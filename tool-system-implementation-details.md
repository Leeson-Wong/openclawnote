# 工具系统实现细节 - TypeScript/Python

## 🎯 实现目标

细化工具系统的实现细节：
1. 工具接口的 TypeScript/Python 实现
2. 工具注册和发现机制
3. 工具调用和错误处理
4. 工具测试策略

---

## 🔧 工具接口实现

### 1. TypeScript 工具接口

```typescript
// src/core/ITool.ts

import { z } from 'zod';

/**
 * 工具参数 Schema
 */
export type ToolParameterSchema = {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'object' | 'array';
  description: string;
  required: boolean;
  default?: any;
  enum?: string[];
  properties?: Record<string, ToolParameterSchema>;
};

/**
 * 工具调用结果
 */
export type ToolResult = {
  success: boolean;
  data?: any;
  error?: string;
  error_code?: string;
  metadata?: {
    execution_time: number;
    timestamp: string;
    tool_name: string;
  };
};

/**
 * 工具配置
 */
export type ToolConfig = {
  name: string;
  description: string;
  parameters: ToolParameterSchema[];
  type?: 'function' | 'api' | 'database' | 'external';
  timeout?: number;
  retryable?: boolean;
  max_retries?: number;
};

/**
 * 基础工具接口
 */
export interface ITool {
  /**
   * 工具配置
   */
  readonly config: ToolConfig;
  
  /**
   * 调用工具
   */
  invoke(parameters: Record<string, any>): Promise<ToolResult>;
  
  /**
   * 验证参数
   */
  validate(parameters: Record<string, any>): { valid: boolean; errors: string[] };
  
  /**
   * 获取工具 Schema
   */
  getSchema(): ToolConfig;
  
  /**
   * 工具是否可重试
   */
  isRetryable(): boolean;
}

/**
 * 抽象工具基类
 */
export abstract class BaseTool implements ITool {
  public readonly config: ToolConfig;
  protected readonly logger: Logger;
  
  constructor(config: ToolConfig) {
    this.config = {
      type: 'function',
      timeout: 30000,
      retryable: true,
      max_retries: 3,
      ...config
    };
    
    this.logger = new Logger(`Tool:${this.config.name}`);
  }
  
  /**
   * 调用工具（抽象方法）
   */
  protected abstract execute(parameters: Record<string, any>): Promise<any>;
  
  /**
   * 调用工具（带错误处理和重试）
   */
  public async invoke(parameters: Record<string, any>): Promise<ToolResult> {
    const startTime = Date.now();
    
    try {
      // 1. 验证参数
      const validation = this.validate(parameters);
      if (!validation.valid) {
        return {
          success: false,
          error: `参数验证失败: ${validation.errors.join(', ')}`,
          error_code: 'VALIDATION_ERROR',
          metadata: {
            execution_time: Date.now() - startTime,
            timestamp: new Date().toISOString(),
            tool_name: this.config.name
          }
        };
      }
      
      // 2. 执行工具（带重试）
      let result = await this.executeWithRetry(parameters);
      
      // 3. 返回结果
      return {
        success: true,
        data: result,
        metadata: {
          execution_time: Date.now() - startTime,
          timestamp: new Date().toISOString(),
          tool_name: this.config.name
        }
      };
      
    } catch (error) {
      this.logger.error(`工具执行失败: ${error.message}`, error);
      
      return {
        success: false,
        error: error.message,
        error_code: this.getErrorCode(error),
        metadata: {
          execution_time: Date.now() - startTime,
          timestamp: new Date().toISOString(),
          tool_name: this.config.name
        }
      };
    }
  }
  
  /**
   * 带重试的执行
   */
  protected async executeWithRetry(parameters: Record<string, any>): Promise<any> {
    if (!this.isRetryable()) {
      return this.execute(parameters);
    }
    
    let lastError: Error;
    const maxRetries = this.config.max_retries || 3;
    const baseDelay = 1000; // 1 秒
    
    for (let attempt = 0; attempt < maxRetries; attempt++) {
      try {
        return await this.execute(parameters);
      } catch (error) {
        lastError = error;
        
        // 判断是否可重试的错误
        if (!this.isRetryableError(error)) {
          throw error;
        }
        
        // 计算退避延迟
        const delay = baseDelay * Math.pow(2, attempt);
        
        this.logger.warn(`执行失败，${delay}ms 后重试 (${attempt + 1}/${maxRetries})`);
        
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw lastError;
  }
  
  /**
   * 验证参数
   */
  public validate(parameters: Record<string, any>): { valid: boolean; errors: string[] } {
    const errors: string[] = [];
    
    for (const param of this.config.parameters) {
      // 检查必需参数
      if (param.required && !(param.name in parameters)) {
        errors.push(`缺少必需参数: ${param.name}`);
        continue;
      }
      
      // 如果参数存在，检查类型
      if (param.name in parameters) {
        const value = parameters[param.name];
        
        // 类型检查
        if (!this.validateType(value, param.type)) {
          errors.push(`参数 ${param.name} 类型错误: 预期 ${param.type}, 实际 ${typeof value}`);
        }
        
        // 枚举检查
        if (param.enum && !param.enum.includes(value)) {
          errors.push(`参数 ${param.name} 枚举值错误: 预期 ${param.enum.join(', ')}, 实际 ${value}`);
        }
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
  
  /**
   * 验证类型
   */
  protected validateType(value: any, expectedType: string): boolean {
    if (value === null || value === undefined) {
      return true;
    }
    
    switch (expectedType) {
      case 'string':
        return typeof value === 'string';
      case 'number':
        return typeof value === 'number';
      case 'boolean':
        return typeof value === 'boolean';
      case 'object':
        return typeof value === 'object' && !Array.isArray(value);
      case 'array':
        return Array.isArray(value);
      default:
        return true;
    }
  }
  
  /**
   * 判断错误是否可重试
   */
  protected isRetryableError(error: Error): boolean {
    const retryableErrors = [
      'ECONNRESET',
      'ETIMEDOUT',
      'ECONNREFUSED',
      'ENOTFOUND',
      'EAI_AGAIN',
      'TIMEOUT',
      'NETWORK_ERROR'
    ];
    
    return retryableErrors.some(err => error.message.includes(err));
  }
  
  /**
   * 获取错误码
   */
  protected getErrorCode(error: Error): string {
    if (error instanceof NetworkError) {
      return 'NETWORK_ERROR';
    }
    
    if (error instanceof TimeoutError) {
      return 'TIMEOUT_ERROR';
    }
    
    if (error instanceof ValidationError) {
      return 'VALIDATION_ERROR';
    }
    
    return 'UNKNOWN_ERROR';
  }
  
  /**
   * 获取工具 Schema
   */
  public getSchema(): ToolConfig {
    return { ...this.config };
  }
  
  /**
   * 工具是否可重试
   */
  public isRetryable(): boolean {
    return this.config.retryable ?? true;
  }
}
```

### 2. Python 工具接口

```python
# src/core/base_tool.py

from abc import ABC, abstractmethod
from typing import Any, Dict, List, Optional, Callable
from dataclasses import dataclass
from enum import Enum
import time
import logging
from functools import wraps

class ToolErrorCode(Enum):
    """工具错误码"""
    VALIDATION_ERROR = "VALIDATION_ERROR"
    NETWORK_ERROR = "NETWORK_ERROR"
    TIMEOUT_ERROR = "TIMEOUT_ERROR"
    EXECUTION_ERROR = "EXECUTION_ERROR"
    UNKNOWN_ERROR = "UNKNOWN_ERROR"

@dataclass
class ToolParameter:
    """工具参数定义"""
    name: str
    type: str  # string, number, boolean, object, array
    description: str
    required: bool = False
    default: Any = None
    enum: Optional[List[str]] = None
    properties: Optional[Dict[str, 'ToolParameter']] = None

@dataclass
class ToolResult:
    """工具调用结果"""
    success: bool
    data: Optional[Any] = None
    error: Optional[str] = None
    error_code: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None

@dataclass
class ToolConfig:
    """工具配置"""
    name: str
    description: str
    parameters: List[ToolParameter]
    type: str = "function"  # function, api, database, external
    timeout: int = 30
    retryable: bool = True
    max_retries: int = 3

class BaseTool(ABC):
    """工具基类"""
    
    def __init__(self, config: ToolConfig):
        """
        初始化工具
        
        Args:
            config: 工具配置
        """
        self.config = config
        self.logger = logging.getLogger(f"Tool:{config.name}")
        
        # 设置默认值
        if config.timeout is None:
            self.config.timeout = 30
        if config.retryable is None:
            self.config.retryable = True
        if config.max_retries is None:
            self.config.max_retries = 3
    
    @abstractmethod
    async def _execute(self, parameters: Dict[str, Any]) -> Any:
        """
        执行工具（抽象方法）
        
        Args:
            parameters: 工具参数
        
        Returns:
            执行结果
        """
        pass
    
    async def invoke(self, parameters: Dict[str, Any]) -> ToolResult:
        """
        调用工具（带错误处理和重试）
        
        Args:
            parameters: 工具参数
        
        Returns:
            工具调用结果
        """
        start_time = time.time()
        
        try:
            # 1. 验证参数
            validation = self.validate(parameters)
            if not validation["valid"]:
                return ToolResult(
                    success=False,
                    error=f"参数验证失败: {', '.join(validation['errors'])}",
                    error_code=ToolErrorCode.VALIDATION_ERROR.value,
                    metadata={
                        "execution_time": time.time() - start_time,
                        "timestamp": time.time(),
                        "tool_name": self.config.name
                    }
                )
            
            # 2. 执行工具（带重试）
            result = await self.execute_with_retry(parameters)
            
            # 3. 返回结果
            return ToolResult(
                success=True,
                data=result,
                metadata={
                    "execution_time": time.time() - start_time,
                    "timestamp": time.time(),
                    "tool_name": self.config.name
                }
            )
            
        except Exception as e:
            self.logger.error(f"工具执行失败: {e}", exc_info=True)
            
            return ToolResult(
                success=False,
                error=str(e),
                error_code=self.get_error_code(e),
                metadata={
                    "execution_time": time.time() - start_time,
                    "timestamp": time.time(),
                    "tool_name": self.config.name
                }
            )
    
    async def execute_with_retry(self, parameters: Dict[str, Any]) -> Any:
        """
        带重试的执行
        
        Args:
            parameters: 工具参数
        
        Returns:
            执行结果
        """
        if not self.is_retryable():
            return await self._execute(parameters)
        
        last_error = None
        max_retries = self.config.max_retries
        base_delay = 1.0  # 1 秒
        
        for attempt in range(max_retries):
            try:
                return await self._execute(parameters)
                
            except Exception as e:
                last_error = e
                
                # 判断是否可重试的错误
                if not self.is_retryable_error(e):
                    raise e
                
                # 计算退避延迟
                delay = base_delay * (2 ** attempt)
                
                self.logger.warning(
                    f"执行失败，{delay} 秒后重试 ({attempt + 1}/{max_retries})"
                )
                
                await asyncio.sleep(delay)
        
        raise last_error
    
    def validate(self, parameters: Dict[str, Any]) -> Dict[str, Any]:
        """
        验证参数
        
        Args:
            parameters: 工具参数
        
        Returns:
            验证结果 {"valid": bool, "errors": List[str]]}
        """
        errors = []
        
        for param in self.config.parameters:
            # 检查必需参数
            if param.required and param.name not in parameters:
                errors.append(f"缺少必需参数: {param.name}")
                continue
            
            # 如果参数存在，检查类型
            if param.name in parameters:
                value = parameters[param.name]
                
                # 类型检查
                if not self._validate_type(value, param.type):
                    errors.append(
                        f"参数 {param.name} 类型错误: "
                        f"预期 {param.type}, 实际 {type(value)}"
                    )
                
                # 枚举检查
                if param.enum and value not in param.enum:
                    errors.append(
                        f"参数 {param.name} 枚举值错误: "
                        f"预期 {', '.join(param.enum)}, 实际 {value}"
                    )
        
        return {
            "valid": len(errors) == 0,
            "errors": errors
        }
    
    def _validate_type(self, value: Any, expected_type: str) -> bool:
        """
        验证类型
        
        Args:
            value: 要验证的值
            expected_type: 预期类型
        
        Returns:
            是否匹配预期类型
        """
        if value is None:
            return True
        
        type_mapping = {
            "string": str,
            "number": (int, float),
            "boolean": bool,
            "object": (dict),
            "array": (list, tuple)
        }
        
        expected_python_type = type_mapping.get(expected_type, None)
        
        if expected_python_type is None:
            return True
        
        if isinstance(expected_python_type, tuple):
            return isinstance(value, expected_python_type)
        
        return isinstance(value, expected_python_type)
    
    def is_retryable_error(self, error: Exception) -> bool:
        """
        判断错误是否可重试
        
        Args:
            error: 异常对象
        
        Returns:
            是否可重试
        """
        retryable_errors = [
            "ConnectionResetError",
            "TimeoutError",
            "ConnectionRefusedError",
            "NetworkError",
            "TemporaryError"
        ]
        
        error_name = type(error).__name__
        return any(retryable_error in error_name for retryable_error in retryable_errors)
    
    def get_error_code(self, error: Exception) -> str:
        """
        获取错误码
        
        Args:
            error: 异常对象
        
        Returns:
            错误码
        """
        error_name = type(error).__name__
        
        if "Connection" in error_name:
            return ToolErrorCode.NETWORK_ERROR.value
        
        if "Timeout" in error_name:
            return ToolErrorCode.TIMEOUT_ERROR.value
        
        if "Validation" in error_name:
            return ToolErrorCode.VALIDATION_ERROR.value
        
        return ToolErrorCode.UNKNOWN_ERROR.value
    
    def is_retryable(self) -> bool:
        """
        工具是否可重试
        """
        return self.config.retryable
    
    def get_schema(self) -> ToolConfig:
        """
        获取工具 Schema
        
        Returns:
            工具配置
        """
        return self.config
```

---

## 🗂️ 工具注册和发现机制

### TypeScript 工具注册器

```typescript
// src/core/ToolRegistry.ts

import { ITool } from './ITool';

/**
 * 工具注册器
 */
export class ToolRegistry {
  private tools: Map<string, ITool> = new Map();
  private categories: Map<string, Set<string>> = new Map();
  
  /**
   * 注册工具
   */
  public register(tool: ITool, category: string = 'general'): void {
    const toolName = tool.config.name;
    
    // 检查是否已注册
    if (this.tools.has(toolName)) {
      throw new Error(`工具 ${toolName} 已经注册`);
    }
    
    // 注册工具
    this.tools.set(toolName, tool);
    
    // 添加到分类
    if (!this.categories.has(category)) {
      this.categories.set(category, new Set());
    }
    this.categories.get(category)!.add(toolName);
    
    this.logger.info(`注册工具: ${toolName} (分类: ${category})`);
  }
  
  /**
   * 注销工具
   */
  public unregister(toolName: string): void {
    if (!this.tools.has(toolName)) {
      throw new Error(`工具 ${toolName} 未注册`);
    }
    
    this.tools.delete(toolName);
    
    // 从分类中移除
    for (const [category, tools] of this.categories) {
      if (tools.has(toolName)) {
        tools.delete(toolName);
        break;
      }
    }
    
    this.logger.info(`注销工具: ${toolName}`);
  }
  
  /**
   * 获取工具
   */
  public getTool(toolName: string): ITool | undefined {
    return this.tools.get(toolName);
  }
  
  /**
   * 获取分类工具
   */
  public getCategoryTools(category: string): ITool[] {
    const toolNames = this.categories.get(category);
    if (!toolNames) {
      return [];
    }
    
    return Array.from(toolNames)
      .map(name => this.tools.get(name)!)
      .filter(Boolean);
  }
  
  /**
   * 搜索工具
   */
  public searchTools(query: string): ITool[] {
    const queryLower = query.toLowerCase();
    
    const results: ITool[] = [];
    
    for (const [name, tool] of this.tools) {
      // 搜索工具名称
      if (name.toLowerCase().includes(queryLower)) {
        results.push(tool);
        continue;
      }
      
      // 搜索工具描述
      if (tool.config.description.toLowerCase().includes(queryLower)) {
        results.push(tool);
        continue;
      }
      
      // 搜索参数名称
      for (const param of tool.config.parameters) {
        if (param.name.toLowerCase().includes(queryLower) ||
            param.description.toLowerCase().includes(queryLower)) {
          results.push(tool);
          break;
        }
      }
    }
    
    return results;
  }
  
  /**
   * 列出所有工具
   */
  public listAll(): ITool[] {
    return Array.from(this.tools.values());
  }
  
  /**
   * 列出所有分类
   */
  public listCategories(): string[] {
    return Array.from(this.categories.keys());
  }
  
  /**
   * 自动发现工具（从插件目录）
   */
  public async discoverTools(pluginDir: string): Promise<void> {
    const fs = await import('fs');
    const path = await import('path');
    
    // 读取插件目录
    const files = await fs.readdir(pluginDir);
    
    for (const file of files) {
      if (file.endsWith('.js') || file.endsWith('.ts')) {
        try {
          // 动态导入工具模块
          const modulePath = path.join(pluginDir, file);
          const module = await import(modulePath);
          
          // 查找工具类
          for (const exported of Object.values(module)) {
            if (typeof exported === 'function' && exported.prototype instanceof BaseTool) {
              const tool: ITool = new exported();
              this.register(tool);
            }
          }
          
        } catch (error) {
          this.logger.error(`加载工具失败: ${file}`, error);
        }
      }
    }
  }
}
```

### Python 工具注册器

```python
# src/core/tool_registry.py

from typing import Dict, List, Optional, Callable, Set
import importlib
import os
import logging
from pathlib import Path

class ToolRegistry:
    """工具注册器"""
    
    def __init__(self):
        self.tools: Dict[str, BaseTool] = {}
        self.categories: Dict[str, Set[str]] = {}
        self.logger = logging.getLogger("ToolRegistry")
    
    def register(self, tool: BaseTool, category: str = "general") -> None:
        """
        注册工具
        
        Args:
            tool: 工具实例
            category: 工具分类
        """
        tool_name = tool.config.name
        
        # 检查是否已注册
        if tool_name in self.tools:
            raise ValueError(f"工具 {tool_name} 已经注册")
        
        # 注册工具
        self.tools[tool_name] = tool
        
        # 添加到分类
        if category not in self.categories:
            self.categories[category] = set()
        self.categories[category].add(tool_name)
        
        self.logger.info(f"注册工具: {tool_name} (分类: {category})")
    
    def unregister(self, tool_name: str) -> None:
        """
        注销工具
        
        Args:
            tool_name: 工具名称
        """
        if tool_name not in self.tools:
            raise ValueError(f"工具 {tool_name} 未注册")
        
        del self.tools[tool_name]
        
        # 从分类中移除
        for category, tools in self.categories.items():
            if tool_name in tools:
                tools.remove(tool_name)
                break
        
        self.logger.info(f"注销工具: {tool_name}")
    
    def get_tool(self, tool_name: str) -> Optional[BaseTool]:
        """
        获取工具
        
        Args:
            tool_name: 工具名称
        
        Returns:
            工具实例，如果不存在则返回 None
        """
        return self.tools.get(tool_name)
    
    def get_category_tools(self, category: str) -> List[BaseTool]:
        """
        获取分类下的所有工具
        
        Args:
            category: 分类名称
        
        Returns:
            工具列表
        """
        tool_names = self.categories.get(category, set())
        return [self.tools[name] for name in tool_names if name in self.tools]
    
    def search_tools(self, query: str) -> List[BaseTool]:
        """
        搜索工具
        
        Args:
            query: 搜索查询
        
        Returns:
            匹配的工具列表
        """
        query_lower = query.lower()
        results = []
        
        for name, tool in self.tools.items():
            # 搜索工具名称
            if query_lower in name.lower():
                results.append(tool)
                continue
            
            # 搜索工具描述
            if query_lower in tool.config.description.lower():
                results.append(tool)
                continue
            
            # 搜索参数名称和描述
            for param in tool.config.parameters:
                if (query_lower in param.name.lower() or
                    query_lower in param.description.lower()):
                    results.append(tool)
                    break
        
        return results
    
    def list_all(self) -> List[BaseTool]:
        """
        列出所有工具
        
        Returns:
            所有工具列表
        """
        return list(self.tools.values())
    
    def list_categories(self) -> List[str]:
        """
        列出所有分类
        
        Returns:
            分类列表
        """
        return list(self.categories.keys())
    
    async def discover_tools(self, plugin_dir: str) -> None:
        """
        自动发现工具（从插件目录）
        
        Args:
            plugin_dir: 插件目录路径
        """
        plugin_path = Path(plugin_dir)
        
        if not plugin_path.exists():
            self.logger.warning(f"插件目录不存在: {plugin_dir}")
            return
        
        # 遍历插件目录
        for file_path in plugin_path.glob("*.py"):
            try:
                # 动态导入工具模块
                module_name = file_path.stem
                spec = importlib.util.spec_from_file_location(module_name, file_path)
                module = importlib.util.module_from_spec(spec)
                spec.loader.exec_module(module)
                
                # 查找工具类
                for attr_name in dir(module):
                    attr = getattr(module, attr_name)
                    
                    # 检查是否是工具类
                    if (isinstance(attr, type) and
                        issubclass(attr, BaseTool) and
                        attr != BaseTool):
                        
                        # 实例化工具
                        tool = attr()
                        self.register(tool)
                        
            except Exception as e:
                self.logger.error(f"加载工具失败: {file_path}", exc_info=e)
```

---

## 🧪 工具测试策略

### TypeScript 工具测试

```typescript
// tests/tools/exampleTool.test.ts

import { describe, test, expect, beforeEach, afterEach } from '@jest/globals';
import { ExampleTool } from '../../src/tools/ExampleTool';

describe('ExampleTool', () => {
  let tool: ExampleTool;
  
  beforeEach(() => {
    tool = new ExampleTool({
      name: 'example_tool',
      description: '示例工具',
      parameters: [
        {
          name: 'input',
          type: 'string',
          description: '输入参数',
          required: true
        }
      ]
    });
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('invoke', () => {
    test('should successfully invoke tool with valid parameters', async () => {
      const result = await tool.invoke({
        input: 'test input'
      });
      
      expect(result.success).toBe(true);
      expect(result.data).toBeDefined();
      expect(result.error).toBeUndefined();
    });
    
    test('should fail with invalid parameters', async () => {
      const result = await tool.invoke({
        invalid_param: 'test'
      });
      
      expect(result.success).toBe(false);
      expect(result.error).toContain('参数验证失败');
      expect(result.error_code).toBe('VALIDATION_ERROR');
    });
    
    test('should retry on retryable errors', async () => {
      const mockExecute = jest.spyOn(tool as any, '_execute')
        .mockRejectedValueOnce(new Error('ConnectionError'))
        .mockResolvedValueOnce({ success: true });
      
      const result = await tool.invoke({
        input: 'test input'
      });
      
      expect(result.success).toBe(true);
      expect(mockExecute).toHaveBeenCalledTimes(2);
    });
  });
  
  describe('validate', () => {
    test('should validate valid parameters', () => {
      const validation = tool.validate({
        input: 'test input'
      });
      
      expect(validation.valid).toBe(true);
      expect(validation.errors).toHaveLength(0);
    });
    
    test('should reject missing required parameters', () => {
      const validation = tool.validate({});
      
      expect(validation.valid).toBe(false);
      expect(validation.errors).toContain('缺少必需参数: input');
    });
    
    test('should reject invalid type', () => {
      const validation = tool.validate({
        input: 123  // 应该是 string
      });
      
      expect(validation.valid).toBe(false);
      expect(validation.errors).toContain('参数 input 类型错误');
    });
  });
  
  describe('getSchema', () => {
    test('should return tool schema', () => {
      const schema = tool.getSchema();
      
      expect(schema.name).toBe('example_tool');
      expect(schema.description).toBe('示例工具');
      expect(schema.parameters).toHaveLength(1);
    });
  });
});
```

### Python 工具测试

```python
# tests/tools/test_example_tool.py

import pytest
import asyncio
from src.tools.example_tool import ExampleTool

class TestExampleTool:
    """示例工具测试"""
    
    @pytest.fixture
    def tool(self):
        """工具实例"""
        config = ToolConfig(
            name="example_tool",
            description="示例工具",
            parameters=[
                ToolParameter(
                    name="input",
                    type="string",
                    description="输入参数",
                    required=True
                )
            ]
        )
        return ExampleTool(config)
    
    @pytest.mark.asyncio
    async def test_invoke_success(self, tool):
        """测试成功调用"""
        result = await tool.invoke({"input": "test input"})
        
        assert result.success is True
        assert result.data is not None
        assert result.error is None
    
    @pytest.mark.asyncio
    async def test_invoke_invalid_parameters(self, tool):
        """测试无效参数"""
        result = await tool.invoke({"invalid_param": "test"})
        
        assert result.success is False
        assert "参数验证失败" in result.error
        assert result.error_code == ToolErrorCode.VALIDATION_ERROR.value
    
    @pytest.mark.asyncio
    async def test_retry_on_error(self, tool, monkeypatch):
        """测试错误重试"""
        call_count = [0]
        
        async def mock_execute(parameters):
            call_count[0] += 1
            if call_count[0] == 1:
                raise ConnectionError("ConnectionError")
            return {"result": "success"}
        
        monkeypatch.setattr(tool, "_execute", mock_execute)
        
        result = await tool.invoke({"input": "test input"})
        
        assert result.success is True
        assert call_count[0] == 2
    
    def test_validate_valid(self, tool):
        """测试有效参数验证"""
        validation = tool.validate({"input": "test input"})
        
        assert validation["valid"] is True
        assert len(validation["errors"]) == 0
    
    def test_validate_missing_required(self, tool):
        """测试缺少必需参数"""
        validation = tool.validate({})
        
        assert validation["valid"] is False
        assert "缺少必需参数: input" in validation["errors"]
    
    def test_validate_invalid_type(self, tool):
        """测试无效类型"""
        validation = tool.validate({"input": 123})
        
        assert validation["valid"] is False
        assert "参数 input 类型错误" in validation["errors"]
    
    def test_get_schema(self, tool):
        """测试获取 Schema"""
        schema = tool.get_schema()
        
        assert schema.name == "example_tool"
        assert schema.description == "示例工具"
        assert len(schema.parameters) == 1
```

---

## 📊 研究总结

### 工具接口实现
1. **TypeScript/Python 双语言支持**: 完整的类型系统
2. **参数验证**: 严格的类型检查和枚举验证
3. **错误处理**: 完善的错误码和错误处理
4. **重试机制**: 指数退避的重试策略
5. **异步支持**: 完整的异步调用支持

### 工具注册和发现
1. **动态注册**: 支持运行时注册/注销
2. **分类管理**: 按分类组织工具
3. **智能搜索**: 支持名称、描述、参数搜索
4. **自动发现**: 从插件目录自动发现工具

### 工具测试
1. **单元测试**: 完整的单元测试覆盖
2. **集成测试**: 工具调用的集成测试
3. **参数验证测试**: 边界条件测试
4. **错误处理测试**: 各种错误场景测试

---

## 🚀 应用到我们的设计

### 创建我们的工具系统

```python
# src/core/tool_system.py

from tool_registry import ToolRegistry
from tools.gateway.websocket_manager import WebSocketManager
from tools.gateway.http_handler import HttpHandler

# 创建工具注册器
tool_registry = ToolRegistry()

# 注册 Gateway Agent 的工具
tool_registry.register(WebSocketManager(ToolConfig(...)), category="gateway")
tool_registry.register(HttpHandler(ToolConfig(...)), category="gateway")

# 搜索工具
search_results = tool_registry.search_tools("websocket")

# 自动发现工具
await tool_registry.discover_tools("./plugins")
```

---

**研究状态**: ✅ 工具系统实现细节细化完成（15 分钟）
**研究时间**: 2026-02-03 17:10
**下一研究**: AI Agent 可观测性
