---
title: "Obsidian + Hermes Agent 完全体：会思考、会记忆、自动运行的个人操作系统"
date: 2026-06-10
source: "https://www.toutiao.com/article/7649625038043300392/"
author: "代码拆解师"
tags:
  - Hermes-Agent
  - Obsidian
  - 知识管理
  - AI-Agent
  - 自动化
workspace: "01-收件箱"
type: reference
status: 待整理
---

# Obsidian + Hermes Agent 完全体

> 基于 @cyrilXBT 的完整搭建指南和 @DamiDefi 的实测体验整理。

## 核心问题

大多数个人知识管理系统有一个致命问题：**记忆**。

- Obsidian 存下了你知道的一切，但不去翻它什么都不会告诉你
- AI 助手能执行任务，但会话一结束就全忘了

**把两者连起来，问题彻底消失。**

## 组合逻辑

| 工具 | 角色 | 优势 | 短板 |
|------|------|------|------|
| Obsidian | 永久知识层 | 纯文本、本地、可搜索、可链接 | 被动，不会主动产出 |
| Hermes Agent | 智能层 | 持久记忆、技能执行、MCP、定时自动化 | 记忆存 SQLite，人类不可读 |

**组合效果：** 笔记库成为 Hermes 的操作环境，Hermes 产出写回笔记库，互相增益。

---

## 四层架构

1. **笔记库层（Obsidian）** — 纯文本 Markdown，人类可浏览 + 自动化可处理
2. **连接层（Filesystem MCP / Obsidian Provider）** — Hermes v0.14 内置 Obsidian Provider
3. **智能层（Hermes + Claude）** — 跨笔记推理，产出写回笔记库
4. **自动化层（Hermes 调度器）** — 定时触发工作流

---

## 搭建步骤

1. 安装 Hermes Agent（从 GitHub 克隆）
2. 配置 Claude 作为模型（Anthropic API）
3. 连接 Obsidian 笔记库（Filesystem MCP 或 Obsidian Provider）
4. 安装 MCP 服务器（Filesystem MCP + Brave Search MCP）
5. 启动并验证（让它描述笔记库顶层结构）

关键命令：
```bash
hermes memory setup --provider obsidian --path ~/path/to/your/vault
hermes memory status
```

---

## 推荐笔记库结构

```
VAULT/
├── 00 - INBOX/          # 原始捕获，等待处理
├── 01 - NOTES/
│   ├── permanent/       # 原子笔记，自己的话
│   ├── daily/           # 每日笔记 [YYYY-MM-DD].md
│   └── meetings/        # 会议笔记
├── 02 - PROJECTS/       # 活跃项目
│   └── [project-name]/
│       ├── overview.md
│       ├── tasks.md
│       └── notes/
├── 03 - RESOURCES/      # 参考资料
├── 04 - HERMES-OUTPUTS/ # ★关键新增：自动化产出
│   ├── briefings/       # 晨报
│   ├── analyses/        # 分析报告
│   ├── syntheses/       # 周综合
│   └── reviews/         # 项目健康报告
├── 05 - ARCHIVE/        # 已完成/过时材料
└── 06 - SYSTEM/         # CLAUDE.md + 技能文件
```

---

## CLAUDE.md 关键指令

放在 `06 - SYSTEM/CLAUDE.md`：
- 你是谁、你做什么、笔记库结构
- 活跃项目列表（状态 + 下一步）
- 本周优先级、思考话题、写作风格

**记忆指令：** 执行前读取 Hermes 记忆 + 笔记库相关笔记；执行后存回两者。

**输出指令：** 先读笔记库相关部分 → 文件路径引用 → 保存到 `04 - HERMES-OUTPUTS/` → `YYYY-MM-DD-[类型]-[主题].md` → 含 type/date 的 frontmatter。

---

## 七个核心技能

### 1. 晨报（每日 6:00 自动）
读取 CLAUDE.md → 今天日记 → 项目 overview → 过期任务 → 上周周回顾 → Brave Search 外部动态 → 综合生成。
**产出：** `04 - HERMES-OUTPUTS/briefings/`

### 2. 收件箱处理器（每日 20:00 自动）
读取 INBOX → 分类（永久想法/项目笔记/参考资料/日记/任务）→ 搜索已有笔记 → 归档+链接 → 原始移 ARCHIVE。
**质量标准：** 每条永久笔记至少有一条链接到已有笔记。

### 3. 项目健康监控（每周一 7:00）
评估每个活跃项目：正常/有风险/停滞/阻塞，引用笔记库实际内容，标注 7 天未修改的停滞项目。

### 4. 笔记连接发现器（每周日）
扫描过去 7 天笔记 → 搜索永久笔记语义关联 → 发现未建立的 wikilink → 评估强度 → 建议链接。

### 5. 周笔记综合（每周日）
综合 7 天日记+永久笔记+Hermes产出+项目健康报告 → 跨七天洞察。

### 6. 研究转笔记（手动触发）
研究材料 → 提取核心观点+证据+新颖内容 → 搜索已有笔记 → 文献笔记 → 新永久笔记。
**标准：** 每条文献笔记至少产出一条新永久笔记。

### 7. 思考伙伴（手动触发）
读取 14 天永久笔记 + 7 天日记 → 发现张力、未充分展开的想法、缺失连接、未回答问题 → 质疑和追问。
**语气：** 聪明同事，推动而非附和。

---

## 一天运行节奏

| 时间 | 事件 |
|------|------|
| 06:00 | 晨报已就绪 |
| 白天 | 只管往 INBOX 捕获 |
| 20:00 | 收件箱自动处理归档 |
| 周一 07:00 | 项目健康报告 |
| 周日晚上 | 连接报告 + 周综合 |

---

## 90 天后的复利效应

1. **Obsidian 复利：** 笔记连接越来越密，连接发现器涌现不显然的关系
2. **Hermes 复利：** 记忆跨会话积累，晨报读过三个月日记，知道你的模式
3. **交互复利：** Obsidian 知道越多 → Hermes 推理越好 → 笔记库越丰富 → 下次产出质量越高

> 三个月后，它像一个在你身边工作了好几个月的思考伙伴，读过你写的所有东西，涌出你自己工作记忆装不下的连接和洞察。

---

*原文发布于 2026-06-10 今日头条 · 作者：代码拆解师*
