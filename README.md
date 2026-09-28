# Embedded Project Memory

[English](#english-tldr) ｜ [中文说明](#中文说明)

<a name="english-tldr"></a>

## English (TL;DR)

An [Agent Skill](https://www.anthropic.com/news/agent-skills) (open `SKILL.md` standard) for embedded firmware teams. It does two things:

1. **Create** — from a project name, builds a team-standard project skeleton: numbered Chinese directories, `git init`, `.gitkeep` in every leaf dir. No placeholder memory.
2. **Remember** — for long-running repos, maintains lightweight *pointer-based* project memory (`.agents/project-map.md` + `.agents/current-state.md`) across tasks, instead of ever-growing context dumps.

Memory stores **WHAT and WHERE**; facts stay in code and docs where they self-update — except the few high-frequency facts (build/flash commands, current versions) that get short, dated inline copies. Built and battle-tested on real STM32/Keil multi-project repositories (300k+ files, 10+ mixed sub-projects).

Works with WorkBuddy, Claude Code, and Codex — any client that loads Agent Skills from a `skills/` directory.

---

<a name="中文说明"></a>

## 中文说明

### 它解决什么问题

嵌入式仓库里最常见的三类"记忆腐化"（全部来自真实项目教训）：

| 症状 | 真实案例 | 本 skill 的对策 |
|---|---|---|
| 目录形态漂移，每次进仓都要重新摸底 | 资料目录混装 10+ 个子工程 | 固定目录标准 + 遗留形态"识别不迁移" |
| 流水账记忆越写越长，塞满过时状态 | 一个 PROJECT_CONTEXT.md 长到 242KB | 记忆只存 what + where，每文件 ~80 行目标、150 行硬顶 |
| 悬空指针——记忆指向"应该存在"的文档 | 9 个子工程 0 个有 AGENTS.md | 写指针前必须验证目标实存；没处放就内联进 Quick Facts |

### 它做什么

**Mode 1 — 从 0 创建工程**：说一句工程名即可。建标准目录树 + 6 个叶子目录各放 `.gitkeep`（git 不跟踪空目录，否则 clone 后骨架散架）+ 默认 `git init`。**骨架期不建 `.agents/`** —— 空壳记忆会伪造"已调研"信号，导致真正的首进调研被跳过。空记忆比没记忆更坏。

**Mode 2 — 首进工程 / 记忆引导**：在该工程的第一个真实任务上运行。有界调研（不递归扫描），然后只写两个记忆文件：

- `.agents/project-map.md` —— 根结构、子工程表、**Build & Flash Quick Facts**、禁区（no-go zones）、硬件指针、既有文档指针
- `.agents/current-state.md` —— 每个活跃工作流一节：任务、分支/回滚点、上次验证、下一步

**日常任务 — Lightweight path**：触发 skill ≠ 全量执行。记忆已存在时，普通任务只读两个记忆文件，做完活，仅在维护触发条件命中时更新。

**指针分档**（频率 × 波动性）：

- 低频稳定事实（引脚表、协议规格、手册位置）→ 纯指针，绝不抄进记忆
- 高频事实（构建命令 + 成功判据、烧录入口、当前固件/算法版本）→ 内联 5–10 行短拷贝进 Quick Facts，带 `source` 路径和 `verified` 日期；动源时顺手更新拷贝
- **Quick Facts self-heal**：构建命令失败时，先怀疑记忆拷贝过期（对着源验证、更新日期、重试），再怀疑代码

### 目录标准

```
<工程名>/
├── 1.代码/
│   └── Main_project/          # 主工程（Keil uvprojx 等）
├── 2.原理图PCB/
├── 3.APP或上位机/
└── 4.相关资料/
    ├── 1.需求或功能/          # 客户给的需求/功能说明
    ├── 2.相关技术资料/        # 现成的厂商文档（手册、SDK；无论谁下载）
    └── 3.技术分析/            # AI 或团队生成的分析/对比/笔记
```

三个资料子目录按**谁产出**划分，这条规则贯穿整个工程生命周期——AI 生成的分析永远进 `3.技术分析/`，不进 `1.代码/`、不进仓库根目录。

### 安装

复制整个文件夹（除 `_user_meta.json`，那是 WorkBuddy 宿主私有元数据）到对应客户端的 skills 目录，无需构建步骤：

| 客户端 | 路径 |
|---|---|
| WorkBuddy | `~/.workbuddy/skills/embedded-project-memory/` |
| Claude Code / Claude Desktop | `~/.claude/skills/embedded-project-memory/` |
| Codex | `~/.codex/skills/embedded-project-memory/`（自定义了 CODEX_HOME 的放其 skills 下） |

```bash
# 示例：装到 Claude Code
git clone https://github.com/<you>/embedded-project-memory.git
cp -r embedded-project-memory ~/.claude/skills/   # Windows: %USERPROFILE%\.claude\skills\
```

### 触发方式（使用示例）

- 「新建一个工程，叫 MZL024-智能门禁」→ Mode 1 建骨架
- 「第一次进 SLV-270 这个工程，先把项目记忆建起来」→ Mode 2 调研 + 写记忆
- 日常调试/看代码任务 → 自动走 Lightweight path，不重复摸底

### 设计决策速览

| 决策 | 原因 |
|---|---|
| 骨架期不建 `.agents/` | 空壳记忆伪造"已调研"信号；记忆从第一个有真实事实的任务诞生 |
| 6 个叶子目录放 `.gitkeep` | git 不跟踪空目录，否则 push/clone 后骨架只剩空壳目录名都保不住 |
| 记忆只存 what + where | 事实留在代码/文档里会自更新；抄进记忆的那份必然开始腐化 |
| 高频事实例外，内联带日期短拷贝 | 每个任务都要用，抄写成本低于每次验证的成本；日期让过期可检 |
| 悬空指针比没指针糟 | 子工程常常没有任何文档可指；没处放就内联或问用户 |
| `current-state` 完结即删 | 归档式状态积累就是记忆腐化本身，不 archive 只 delete |
| 超 ~150 行 / 6KB 触发压缩 | 合并重复、清完结状态、淘汰死子工程的 Quick Facts |

### 验证方法

1. **触发验证**：新会话说「新建一个工程」→ 应进入 Mode 1 并先查父目录同名/近似名冲突
2. **骨架验证**：确认目录树、6 个 `.gitkeep`、`git init` 齐全，且**没有** `.agents/`
3. **clone 测试**：`git clone` 到临时目录，确认空目录经 `.gitkeep` 全部存活
4. **记忆验证**：在既有工程上跑 Mode 2，确认两个记忆文件各 ~80 行内、指针目标全部实存

### 兼容性说明

- 遵循 Agent Skills 开放标准：YAML frontmatter（`name` + `description` 触发面）+ 正文 < 500 行 + 渐进披露
- 正文为英文以保证跨客户端稳定性；团队目录名保留中文原文
- `description` 同时声明了排除面（纯硬件分析/已有记忆的调试任务不触发全量流程）

## License

MIT
