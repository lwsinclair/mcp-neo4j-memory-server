[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/jovanhsu-mcp-neo4j-memory-server-badge.png)](https://mseep.ai/app/jovanhsu-mcp-neo4j-memory-server)

# MCP Neo4j Knowledge Graph Memory Server

[![npm version](https://img.shields.io/npm/v/@izumisy/mcp-neo4j-memory-server.svg)](https://www.npmjs.com/package/@izumisy/mcp-neo4j-memory-server)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-blue)](https://www.typescriptlang.org/)
[![Neo4j](https://img.shields.io/badge/Neo4j-5.x-brightgreen)](https://neo4j.com/)

## 简介

MCP Neo4j Knowledge Graph Memory Server是一个基于Neo4j图数据库的知识图谱记忆服务器，用于存储和检索AI助手与用户交互过程中的信息。该项目是[官方Knowledge Graph Memory Server](https://github.com/modelcontextprotocol/servers/tree/main/src/memory)的增强版本，使用Neo4j作为后端存储引擎。

通过使用Neo4j作为存储后端，本项目提供了更强大的图查询能力、更好的性能和可扩展性，特别适合构建复杂的知识图谱应用。

## 功能特点

- 🚀 基于Neo4j的高性能图数据库存储
- 🔍 强大的模糊搜索和精确匹配能力
- 🔄 实体、关系和观察的完整CRUD操作
- 🌐 与MCP协议完全兼容
- 📊 支持复杂的图查询和遍历
- 🐳 Docker支持，便于部署

## 安装

### 前提条件

- Node.js >= 22.0.0
- Neo4j数据库（本地或远程）

### 通过npm安装

```bash
# 全局安装
npm install -g @jovanhsu/mcp-neo4j-memory-server

# 或作为项目依赖安装
npm install @jovanhsu/mcp-neo4j-memory-server
```

### 使用Docker

```bash
# 使用docker-compose启动Neo4j和Memory Server
git clone https://github.com/JovanHsu/mcp-neo4j-memory-server.git
cd mcp-neo4j-memory-server
docker-compose up -d
```

### 环境变量配置

服务器使用以下环境变量进行配置：

| 环境变量 | 描述 | 默认值 |
|----------|------|--------|
| NEO4J_URI | Neo4j数据库URI | bolt://localhost:7687 |
| NEO4J_USER | Neo4j用户名 | neo4j |
| NEO4J_PASSWORD | Neo4j密码 | password |
| NEO4J_DATABASE | Neo4j数据库名称 | neo4j |

## 与Claude集成

### 在Claude Desktop中配置

在`claude_desktop_config.json`中添加以下配置：

```json
{
  "mcpServers": {
    "graph-memory": {
      "command": "npx",
      "args": [
        "-y",
        "@izumisy/mcp-neo4j-memory-server"
      ],
      "env": {
        "NEO4J_URI": "neo4j://localhost:7687",
        "NEO4J_USER": "neo4j",
        "NEO4J_PASSWORD": "password",
        "NEO4J_DATABASE": "memory"
      }
    }
  }
}
```

### 在Claude Web中使用MCP Inspector

1. 安装[MCP Inspector](https://github.com/modelcontextprotocol/inspector)
2. 启动Neo4j Memory Server：
   ```bash
   npx @jovanhsu/mcp-neo4j-memory-server
   ```
3. 在另一个终端启动MCP Inspector：
   ```bash
   npx @modelcontextprotocol/inspector npx @jovanhsu/mcp-neo4j-memory-server
   ```
4. 在浏览器中访问MCP Inspector界面

## 使用方法

### Claude自定义指令

在Claude的自定义指令中添加以下内容：

```
Follow these steps for each interaction:

1. User Identification:
   - You should assume that you are interacting with default_user
   - If you have not identified default_user, proactively try to do so.

2. Memory Retrieval:
   - Always begin your chat by saying only "Remembering..." and search relevant information from your knowledge graph
   - Create a search query from user words, and search things from "memory". If nothing matches, try to break down words in the query at first ("A B" to "A" and "B" for example).
   - Always refer to your knowledge graph as your "memory"

3. Memory
   - While conversing with the user, be attentive to any new information that falls into these categories:
     a) Basic Identity (age, gender, location, job title, education level, etc.)
     b) Behaviors (interests, habits, etc.)
     c) Preferences (communication style, preferred language, etc.)
     d) Goals (goals, targets, aspirations, etc.)
     e) Relationships (personal and professional relationships up to 3 degrees of separation)

4. Memory Update:
   - If any new information was gathered during the interaction, update your memory as follows:
     a) Create entities for recurring organizations, people, and significant events
     b) Connect them to the current entities using relations
     b) Store facts about them as observations
```

### API示例

如果您想在自己的应用程序中使用本服务器，可以通过MCP协议与其通信：

```typescript
import { McpClient } from '@modelcontextprotocol/sdk/client/mcp.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

// 创建客户端
const transport = new StdioClientTransport({
  command: 'npx',
  args: ['-y', '@izumisy/mcp-neo4j-memory-server'],
  env: {
    NEO4J_URI: 'bolt://localhost:7687',
    NEO4J_USER: 'neo4j',
    NEO4J_PASSWORD: 'password',
    NEO4J_DATABASE: 'neo4j'
  }
});

const client = new McpClient();
await client.connect(transport);

// 创建实体
const result = await client.callTool('create_entities', {
  entities: [
    {
      name: '用户',
      entityType: '人物',
      observations: ['喜欢编程', '使用TypeScript']
    }
  ]
});

console.log(result);
```

## 为什么选择Neo4j？

相比于原始版本使用的JSON文件存储和DuckDB版本，Neo4j提供了以下优势：

1. **原生图数据库**：Neo4j是专为图数据设计的数据库，非常适合知识图谱的存储和查询
2. **高性能查询**：使用Cypher查询语言可以高效地进行复杂的图遍历和模式匹配
3. **关系优先**：Neo4j将关系作为一等公民，使得实体间的关系查询更加高效
4. **可视化能力**：Neo4j提供了内置的可视化工具，方便调试和理解知识图谱
5. **扩展性**：支持集群部署，可以处理大规模知识图谱

## 实现细节

### 数据模型

知识图谱在Neo4j中的存储模型如下：

```
(Entity:EntityType {name: "实体名称"})
(Entity)-[:HAS_OBSERVATION]->(Observation {content: "观察内容"})
(Entity1)-[:RELATION_TYPE]->(Entity2)
```

### 模糊搜索实现

本实现结合了Neo4j的全文搜索功能和Fuse.js进行灵活的实体搜索：

- 使用Neo4j的全文索引进行初步搜索
- Fuse.js提供额外的模糊匹配能力
- 搜索结果包括精确和部分匹配，按相关性排序

## 开发

### 环境设置

```bash
# 克隆仓库
git clone https://github.com/JovanHsu/mcp-neo4j-memory-server.git
cd mcp-neo4j-memory-server

# 安装依赖
pnpm install

# 构建项目
pnpm build

# 开发模式（使用MCP Inspector）
pnpm dev
```

### 测试

```bash
# 运行测试
pnpm test
```

### 发布

```bash
# 准备发布
npm version [patch|minor|major]

# 发布到NPM
npm publish
```

## 贡献指南

欢迎贡献代码、报告问题或提出改进建议！请遵循以下步骤：

1. Fork本仓库
2. 创建您的特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交您的更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建一个Pull Request

## 相关项目

- [Model Context Protocol](https://github.com/modelcontextprotocol/mcp)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [Claude Desktop](https://github.com/anthropics/claude-desktop)

## 许可证

本项目采用MIT许可证 - 详见[LICENSE](LICENSE)文件。

## 联系方式

- GitHub: [https://github.com/JovanHsu/mcp-neo4j-memory-server](https://github.com/JovanHsu/mcp-neo4j-memory-server)
- NPM: [https://www.npmjs.com/package/@jovanhsu/mcp-neo4j-memory-server](https://www.npmjs.com/package/@jovanhsu/mcp-neo4j-memory-server)
- 作者: JovanHsu