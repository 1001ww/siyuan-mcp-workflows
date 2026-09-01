# 思源 MCP 工作流 Skill

面向思源笔记（SiYuan Note）的 MCP 优先工作流 Skill。

本项目综合了以下两个项目中可复用的操作策略：

- https://github.com/dazexcl/siyuan-skill
- https://github.com/purefkh/siyuan-skills

它不提供另一套思源 CLI、API 客户端、向量索引、向量数据库集成或凭据配置。这些属于已连接的 MCP 服务职责。Skill 只定义让 Agent 稳定、安全地使用 MCP 的工作纪律：工具发现、编辑前检索、最小块级修改、写后验证，以及破坏性操作的显式确认。

## 内容

- `skills/siyuan-mcp-workflows/SKILL.md`：可移植的 Skill 定义。

## 设计取舍

| 保留 | 原因 |
|---|---|
| 编辑/删除前检索 | 避免改到同名笔记或错误的内容块。 |
| 块级优先编辑 | 保留文档身份、引用关系和未涉及的内容。 |
| 显式删除确认 | 让破坏性操作的范围和影响保持可见、可控。 |
| 写后回读验证 | MCP 写入成功不代表结果符合预期。 |
| 思源链接与元数据约定 | 保持知识库内部结构一致。 |
| 批量操作预览与核对 | 避免静默修改或漏掉目标笔记。 |

| 移除 | 原因 |
|---|---|
| Node/Python 命令封装 | 与已连接的 MCP 服务重复。 |
| 特定实现的工具名称 | 不同 SiYuan MCP 服务暴露的工具和参数不同。 |
| Qdrant、Ollama、FAISS 或 OpenAI 配置 | 语义搜索属于可选的服务端基础设施，不属于工作流策略。 |
| 冗长的接口与 CLI 命令表 | MCP 工具 Schema 可在运行时动态发现。 |
| 自动同步、自动格式化、自动设置图标 | 这些是需要用户确认的偏好型状态变更。 |

## 安装

使用 Agent 的 Skill 安装机制安装此目录，然后开启新会话以刷新 Skill 索引。使用前需已配置并连接可用的 SiYuan MCP 服务。

也可以通过 `skills.sh` CLI 安装：

```bash
npx skills add 1001ww/siyuan-mcp-workflows --skill siyuan-mcp-workflows
```

## 适用范围

这是一个仅包含 Skill 的项目，假设你已拥有功能完整的思源 MCP 服务。它规范 MCP 的使用流程，不替代 MCP 服务本身。

## 许可证

MIT
