# Learn Claude Code 学习指南

> **目标**：从零开始理解 AI Agent 的工作原理，通过 12 个递进式课程掌握 Claude Code 的核心机制。

---

## 目录

1. [项目概述](#项目概述)
2. [学习前的准备](#学习前的准备)
3. [学习方法论](#学习方法论)
4. [四阶段学习路径](#四阶段学习路径)
5. [Windows 用户特别说明](#windows-用户特别说明)
6. [课程速查表](#课程速查表)
7. [推荐学习资源](#推荐学习资源)
8. [常见问题](#常见问题)

---

## 项目概述

### 这是什么？

**Learn Claude Code** 是一个开源教育项目，通过 12 个递进式的 Python 实现，教你如何从零构建一个类似 Claude Code 的 AI 编程 Agent。

### 核心理念

> **"The model is the agent. Our job is to give it tools and stay out of the way."**
> （模型就是智能体。我们的工作就是给它工具，然后让开。）

### 最小 Agent 循环

```
User --> messages[] --> LLM --> response
                          |
                stop_reason == "tool_use"?
               /                          \
             yes                           no
              |                             |
        execute tools                    return text
        append results
        loop back -----------------> messages[]
```

---

## 学习前的准备

### 环境要求

| 项目 | 版本/说明 |
|------|----------|
| Python | 3.8+ |
| Anthropic API Key | 需要 Claude API 访问权限 |
| 操作系统 | macOS / Linux / Windows + WSL 或 Git Bash |

### 安装步骤

```bash
# 1. 克隆仓库
git clone https://github.com/shareAI-lab/learn-claude-code
cd learn-claude-code

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置环境变量
cp .env.example .env
# 编辑 .env，填入你的 ANTHROPIC_API_KEY

# 4. 测试运行
python agents/s01_agent_loop.py
```

---

## 学习方法论

### 核心原则：理解而非复刻

**不推荐**：
- ❌ 逐行复制所有代码
- ❌ 试图一次性掌握所有12个课程
- ❌ 在 Windows 上直接运行而不做适配

**推荐**：
- ✅ 按顺序阅读代码，理解设计思想
- ✅ 运行每个 session，观察 Agent 行为
- ✅ 重点理解核心循环（`agent_loop` 函数）
- ✅ 选择性复刻最感兴趣的部分

### 推荐的三种学习方式

#### 方式一：快速浏览（2-3小时）

适合想要快速了解 Agent 工作原理的人。

1. 阅读本文档的【四阶段学习路径】
2. 阅读 s01、s03、s07、s09 的代码和文档
3. 理解每个阶段的核心问题和解法

#### 方式二：递进学习（1-2周）

适合想要系统掌握 Agent 开发的人。

1. 按顺序 s01 → s12，每天学习 1-2 个 session
2. 每个 session：阅读文档 → 运行代码 → 观察输出 → 理解原理
3. 完成 s12 后，阅读 s_full.py 了解生产级实现

#### 方式三：问题导向（按需）

适合有具体目标的人。

| 你的目标 | 重点学习 |
|---------|---------|
| 构建自己的编程 Agent | s01-s02, s07, s12, s_full |
| 理解任务规划 | s03, s07, s11 |
| 实现多 Agent 协作 | s09-s12 |
| 优化上下文管理 | s06, s12 |

---

## 四阶段学习路径

### 第一阶段：核心循环（The Loop）

**目标**：理解 Agent 的最小工作循环

| Session | 主题 | 格言 | 新增工具 |
|---------|------|------|---------|
| **s01** | The Agent Loop | "One loop & Bash is all you need" | 1 |
| **s02** | Tool Use | "Adding a tool means adding one handler" | 4 |

**核心问题**：如何用最小循环让 LLM 调用工具并迭代？

**关键概念**：
- `stop_reason == "tool_use"` 是循环继续的条件
- 工具调用结果通过 `messages` 数组回传给 LLM
- `dispatch map` 模式：工具名称 → 处理函数的映射

**学习建议**：
1. 仔细阅读 `agent_loop` 函数（第 67-87 行）
2. 运行 s01，输入简单命令（如 `ls`）观察循环
3. 对比 s01 和 s02，理解如何添加新工具

---

### 第二阶段：规划与知识（Planning & Knowledge）

**目标**：理解如何让 Agent 有计划地工作，并动态加载知识

| Session | 主题 | 格言 | 新增工具 |
|---------|------|------|---------|
| **s03** | TodoWrite | "An agent without a plan drifts" | 5 |
| **s04** | Subagents | "Break big tasks down; each subtask gets a clean context" | 6 |
| **s05** | Skills | "Load knowledge when you need it, not upfront" | 7 |
| **s06** | Context Compact | "Context will fill up; you need a way to make room" | 7 |

**核心问题**：如何让 Agent 有规划地工作，并管理有限的上下文？

**关键概念**：
- **TodoWrite（s03）**：任务列表是 Agent 规划的基础，nag 提醒防止 Agent "跑偏"
- **Subagents（s04）**：子 Agent 拥有独立的 `messages[]`，避免污染主对话
- **Skills（s05）**：通过 `tool_result` 动态注入知识，而非放在 system prompt
- **Context Compact（s06）**：三层压缩策略（summarize → checkpoint → archive）

**学习建议**：
1. s03：修改任务列表，观察 Agent 如何跟随计划
2. s04：对比主 Agent 和子 Agent 的 `messages` 内容
3. s05：阅读 `skills/` 目录下的 SKILL.md 文件格式
4. s06：思考你自己的会话管理策略

---

### 第三阶段：持久化（Persistence）

**目标**：理解如何让 Agent 记住任务状态，并在后台运行

| Session | 主题 | 格言 | 新增工具 |
|---------|------|------|---------|
| **s07** | Tasks | "Break big goals into small tasks, order them, persist to disk" | 12 |
| **s08** | Background Tasks | "Run slow operations in the background; the agent keeps thinking" | 14 |

**核心问题**：如何让任务持久化，并在后台异步执行？

**关键概念**：
- **Tasks（s07）**：文件化的任务图（CRUD + 依赖关系），为多人协作打基础
- **Background Tasks（s08）**：守护线程执行命令，完成后通过通知队列注入结果

**学习建议**：
1. s07：查看任务文件的存储格式（JSON），理解依赖图
2. s08：运行一个耗时命令（如 `sleep 5`），观察前台交互不被阻塞

---

### 第四阶段：团队协作（Teams）

**目标**：理解如何让多个 Agent 协同工作

| Session | 主题 | 格言 | 新增工具 |
|---------|------|------|---------|
| **s09** | Agent Teams | "When the task is too big for one, delegate to teammates" | 19 |
| **s10** | Team Protocols | "Teammates need shared communication rules" | 21 |
| **s11** | Autonomous Agents | "Teammates scan the board and claim tasks themselves" | 21 |
| **s12** | Worktree Isolation | "Each works in its own directory, no interference" | 22 |

**核心问题**：如何让多个 Agent 有效协作？

**关键概念**：
- **Agent Teams（s09）**：持久化的团队成员 + 异步 JSONL 邮箱通信
- **Team Protocols（s10）**：统一的请求-响应模式，shutdown 和 plan approval 状态机
- **Autonomous Agents（s11）**：Agent 主动扫描任务板，自动认领任务
- **Worktree Isolation（s12）**：每个 Agent 在独立的工作目录工作，互不干扰

**学习建议**：
1. s09：创建两个 Agent，让它们相互发送消息
2. s10：观察请求-响应模式的完整流程
3. s11：创建任务，观察 Agent 如何自动认领
4. s12：理解 worktree 如何隔离文件系统状态

---

## Windows 用户特别说明

### 问题

项目代码使用类 Unix 命令（如 `cat`, `ls`, `rm`），在 Windows PowerShell 中直接运行会有兼容性问题。

### 解决方案

#### 方案 1：使用 Git Bash（推荐）

1. 安装 [Git for Windows](https://git-scm.com/download/win)
2. 在 Git Bash 中运行：

```bash
cd /c/projects/learn-claude-code
python agents/s01_agent_loop.py
```

#### 方案 2：使用 WSL

如果你有 WSL 环境：

```bash
cd /mnt/c/projects/learn-claude-code
python agents/s01_agent_loop.py
```

#### 方案 3：仅阅读不运行

项目的核心价值在于**理解代码逻辑**，而非运行结果。你可以：
1. 在 IDE 中阅读代码
2. 使用项目的 Web 交互平台（`web/` 目录）

---

## 课程速查表

| 阶段 | Session | 主题 | 新增机制 | 格言 |
|------|---------|------|---------|------|
| **第一阶段** | s01 | Agent Loop | 核心循环 | "One loop & Bash is all you need" |
| | s02 | Tool Use | 工具分派 | "Adding a tool means adding one handler" |
| **第二阶段** | s03 | TodoWrite | 任务列表 | "An agent without a plan drifts" |
| | s04 | Subagents | 子 Agent | "Break big tasks down; each subtask gets a clean context" |
| | s05 | Skills | 技能加载 | "Load knowledge when you need it, not upfront" |
| | s06 | Context Compact | 上下文压缩 | "Context will fill up; you need a way to make room" |
| **第三阶段** | s07 | Tasks | 任务持久化 | "Break big goals into small tasks, order them, persist to disk" |
| | s08 | Background Tasks | 后台任务 | "Run slow operations in the background; the agent keeps thinking" |
| **第四阶段** | s09 | Agent Teams | Agent 团队 | "When the task is too big for one, delegate to teammates" |
| | s10 | Team Protocols | 团队协议 | "Teammates need shared communication rules" |
| | s11 | Autonomous Agents | 自主 Agent | "Teammates scan the board and claim tasks themselves" |
| | s12 | Worktree Isolation | 工作树隔离 | "Each works in its own directory, no interference" |

---

## 推荐学习资源

### 官方资源

| 资源 | 说明 | 位置 |
|------|------|------|
| **主文档** | 英文、中文、日文 | `docs/{en,zh,ja}/` |
| **代码实现** | 12个递进式 Python 文件 | `agents/s*.py` |
| **完整实现** | 综合所有机制的生产级代码 | `agents/s_full.py` |
| **Web 平台** | 交互式学习，带可视化动画 | `web/` |

### 扩展资源

| 项目 | 说明 | 链接 |
|------|------|------|
| **Kode CLI** | 开源编程 Agent CLI，基于本项目原理构建 | `npm i -g @shareai-lab/kode` |
| **Kode Agent SDK** | 可嵌入后端/浏览器的 Agent SDK | GitHub: shareAI-lab/Kode-agent-sdk |
| **claw0** | 从"按需会话"到"始终在线助手" | GitHub: shareAI-lab/claw0 |

---

## 常见问题

### Q1: 需要多少 Python 基础？

**基础要求**：
- 熟悉 Python 基础语法（函数、类、列表、字典）
- 了解装饰器、生成器等进阶特性
- 有使用 API 的经验（了解 HTTP、JSON）

**不需要**：
- 精通异步编程（本项目使用同步代码）
- 深入了解 LLM 原理

### Q2: 每个 session 需要花多长时间？

| 学习方式 | 每个 session 时间 | 总学习时间 |
|---------|------------------|-----------|
| 快速浏览（只看代码+文档） | 15-30 分钟 | 4-6 小时 |
| 正常学习（阅读+运行+思考） | 1-2 小时 | 2-3 周 |
| 深入学习（复刻+扩展） | 3-4 小时 | 1-2 个月 |

### Q3: 需要按顺序学习吗？

**强烈建议按顺序**，原因：
- 每个 session 都建立在之前的机制之上
- s02 使用 s01 的核心循环
- s07 依赖 s03 的任务列表概念
- s09-s12 依赖前面所有机制

**例外**：如果你只想了解特定主题，可以直接跳转到对应 session，但需要先阅读前置 session 的文档。

### Q4: 这个项目和真正的 Claude Code 有什么区别？

**本项目**：
- 教学目的，代码简洁易懂
- 省略了生产环境的复杂机制（权限、钩子、生命周期控制）
- 专注于核心原理的教学

**真正的 Claude Code**：
- 完整的权限系统和信任工作流
- 完整的事件/钩子总线
- 会话生命周期控制（恢复/分叉）
- 高级 worktree 生命周期控制

### Q5: 学习完之后能做什么？

**基础能力**：
- 理解 AI Agent 的工作原理
- 能够阅读和修改类似项目

**进阶方向**：
1. **构建自己的 Agent**：基于本项目原理，开发定制化的编程 Agent
2. **使用 Kode SDK**：将 Agent 能力嵌入你的应用
3. **贡献开源**：向 Kode CLI 或本项目贡献代码
4. **深入研究**：学习更复杂的机制（MCP、心跳、Cron 等），参考 claw0 项目

---

## 下一步行动

1. **确认环境**：检查 Python 版本，准备 API Key
2. **选择学习方式**：快速浏览 / 递进学习 / 问题导向
3. **开始第一课**：阅读 `docs/zh/s01-the-agent-loop.md`，运行 `agents/s01_agent_loop.py`
4. **保持节奏**：每天 1-2 个 session，不要急于求成

---

**祝学习愉快！**

> *"The model is the agent. Our job is to give it tools and stay out of the way."*
