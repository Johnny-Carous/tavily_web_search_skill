# Tavily 搜索集成

## 概述

本项目提供Tavily API的集成方案，包含优化版命令行脚本、直接API集成模块和OpenClaw技能。

## 文件结构

```
tavily/
├── config/                    # 配置文件
│   ├── tavily-config.example.json  # 配置示例（复制为tavily-config.json并修改）
│   └── tavily-config.json.template # 配置模板
├── deps/                     # 依赖管理文件
│   ├── package.json               # Node.js包配置
│   └── package-lock.json          # 依赖锁定文件
│   └── package_README.md          # 依赖说明文档
├── scripts/                  # 脚本文件
│   ├── tavily-v2.js               # 优化版主脚本（推荐使用）
│   └── tavily-integration.js      # 直接API集成模块
├── skill/                    # OpenClaw技能
│   └── tavily-search-skill/
│       └── SKILL.md              # 技能文档
└── docs/                     # 文档
│   └── README.md                  # 项目说明文档
```

## 快速开始

### ⚠️ 重要提示：确保在正确目录下运行

**所有命令必须在工作空间根目录下执行：**
```bash
cd "C:\Users\***\.openclaw\workspace"
```

如果收到错误 `Cannot find module 'C:\***\tavily-v2.js'`，说明你在错误目录下运行命令。请先进入正确目录。

### 1. 配置API密钥

获取Tavily API密钥：https://app.tavily.com

配置方式（三选一）：

**方式一：配置文件**（推荐）
```bash
# 复制示例配置文件
copy config\tavily-config.example.json config\tavily-config.json
# 编辑config\tavily-config.json，填入你的API密钥
```

**方式二：环境变量**
```bash
# Windows
set TAVILY_API_KEY=你的密钥

# Linux/macOS
export TAVILY_API_KEY=你的密钥
```

**方式三：命令行参数**
```bash
node scripts\tavily-v2.js --query "test" --apiKey 你的密钥
```

### 2. 使用脚本

**推荐使用包装器（避免路径错误）**
```bash
# 基本搜索 - 使用包装器（推荐）
node tools\tavily-v2.js --query "人工智能发展"

# 新闻搜索，包含AI答案，Markdown格式
node tools\tavily-v2.js --query "最新科技新闻" --topic news --includeAnswer true --outputFormat markdown

# 学术搜索，更多结果，更长超时
node tools\tavily-v2.js --query "机器学习研究" --topic academic --maxResults 10 --timeout 60000

# 显示帮助
node tools\tavily-v2.js --help
```

**备选方式：直接调用脚本（需在正确目录下）**
```bash
# 如果使用直接调用，请确保在 workspace\tavily 目录下，路径需要自行修改
cd "C:\Users\***\.openclaw\workspace\tavily"
node scripts\tavily-v2.js --query "人工智能发展"
```

### 3. 在OpenClaw中使用

**推荐方式：使用包装器（自动处理路径）**
```javascript
await exec({
  command: 'node tools\\tavily-v2.js --query "搜索词"',
  workdir: 'C:\\Users\\***\\.openclaw\\workspace'
})
```

**备选方式：直接调用脚本**
```javascript
await exec({
  command: 'node scripts\\tavily-v2.js --query "搜索词"',
  workdir: 'C:\\Users\\***\\.openclaw\\workspace\\tavily'
})
```

**直接API集成：**
```javascript
const { TavilySearch } = require('./scripts/tavily-integration.js');
const searcher = new TavilySearch();
const result = await searcher.search('搜索词', { maxResults: 5 });
```

## 依赖管理

Tavily 集成依赖于 `@tavily/core` 包，相关配置文件位于 `tavily/deps/` 目录：

- **package.json** - 包含 `@tavily/core` 依赖定义
- **package-lock.json** - 依赖版本锁定文件

依赖已安装在 workspace 根目录的 `node_modules/` 中。如需重新安装依赖：

```bash
# 在 workspace 根目录运行
cd "C:\Users\***\.openclaw\workspace"
npm install

# 或在 tavily/deps 目录运行
cd tavily\deps
npm install --prefix ../..
```

## 向后兼容性

为保持原有使用习惯，在workspace根目录的`tools/`文件夹中提供了包装器：

- `tools/tavily-v2.js` → 指向 `tavily/scripts/tavily-v2.js`
- `tools/tavily.js` → 指向 `tavily/scripts/tavily-v2.js`（现在默认使用优化版本）
- `tools/tavily-integration.js` → 重新导出 `tavily/scripts/tavily-integration.js`

原有调用方式不变：
```bash
# 仍然可以这样调用
cd C:\Users\***\.openclaw\workspace
node tools\tavily-v2.js --query "test"
```

## 优化特性

### tavily-v2.js 新增功能

1. **重试机制**：自动重试失败请求（默认3次，可配置）
2. **超时控制**：可设置请求超时（默认30秒）
3. **多种输出格式**：JSON、Markdown、纯文本
4. **搜索主题**：general、news、academic
5. **搜索深度**：basic（快速）、advanced（深入）
6. **详细错误处理**：友好的错误信息和调试建议
7. **配置文件支持**：支持多场景配置预设

### tavily-integration.js API模块

1. **面向对象设计**：`TavilySearch` 类
2. **批量搜索**：支持多个查询同时处理
3. **灵活格式化**：动态选择输出格式
4. **错误恢复**：完善的错误包装和结果处理

## 故障排除

### 常见问题

1. **API密钥错误**
   ```
   错误: TAVILY_API_KEY not found
   解决方案: 按照上述三种方式之一配置API密钥
   ```

2. **网络超时**
   ```
   错误: 请求超时 (30000ms)
   解决方案: 增加超时时间 --timeout 60000 或检查网络连接
   ```

3. **模块加载失败**
   ```
   错误: Cannot find module '@tavily/core'
   解决方案: 在workspace根目录运行 npm install @tavily/core
   ```

4. **路径错误（C:\\Users\\***\\scripts\\tavily-v2.js）**
   ```
   错误: Cannot find module 'C:\Users\***\scripts\tavily-v2.js'
   解决方案: 
     - 确保在正确目录下运行: cd "C:\Users\***\.openclaw\workspace"
     - 使用包装器脚本: node tools/tavily-v2.js 或 node tools/tavily.js
     - 包装器会自动设置正确的工作目录和路径
   ```

### 调试模式

```bash
# 显示详细日志
set DEBUG=tavily* && node scripts\tavily-v2.js --query "test"
```

## 更新日志

### 2026-02-17
- **文件结构优化**：将所有Tavily相关文件整理到统一的 `tavily/` 目录
- **依赖管理**：将 `package.json` 和 `package-lock.json` 移动到 `tavily/deps/` 目录
- **包装器系统**：创建 `tools/` 目录下的包装器脚本，保持100%向后兼容
- **脚本优化**：`tavily-v2.js` 添加重试机制、超时控制、多种输出格式等高级功能
- **路径错误修复**：修复 `Cannot find module 'C:\Users\***\scripts\tavily-v2.js'` 错误，增强包装器健壮性
- **文档清理**：删除重复和备份文件，统一文档管理

### 2026-02-16
- **初始版本**：基础搜索功能，包含 `tavily.js` 脚本和简单集成

## 仅供学习交流，严禁用于商业用途。
