---
title: 灵码 + Qwen3-Coder：用 Skill 机制实现代码审核
source: https://mp.weixin.qq.com/s/BY_gZJizglAgvEjNBcAfMQ
author: 荣阳
tags: [Skill机制, 渐进式披露, openskills, 代码审核]
---

# 灵码 + Qwen3-Coder —— 使用 Skill 机制实现代码审核

> **一句话结论**：**Skill 的本质不是可执行代码，而是"基于提示词的动态上下文注入 + 元工具架构"。**
> **它不调用外部函数，而是把领域指令、工作流和知识"展开"注入到对话里。**

---

## ⭐⭐ 一、Skill 的实现机制（讲得最清楚的一篇）

### 1.1 本质

> **"Claude Skills 的本质不是可执行代码，而是基于提示词的动态上下文注入与元工具架构。**
> **与传统观念不同，Skill 并非让模型调用外部函数，**
> **而是将领域特定的指令、工作流和知识'展开'并'注入'到对话中。"**

### 1.2 文件结构

```
skill/
├── SKILL.md          # 核心提示模板
├── scripts/          # 可执行 Python/Bash 脚本
│   ├── validator.py
│   └── processor.sh
├── references/       # 参考文档
│   ├── terminology.md
│   └── examples.md
└── assets/           # 模板和静态资源
    ├── template.docx
    └── config.json
```

**SKILL.md 结构**：
```yaml
---
name: my-skill
description: 清晰描述技能功能和触发场景
allowed-tools: "Read,Write,Bash"
---
# 技能详细指令
## 工作流程
第一步说明
第二步说明
## 输出规范
```

### 1.3 ⭐⭐ 调用流程（**关键：纯 LLM 推理，不是向量检索**）

> **"Claude 使用'纯 LLM 推理'进行 Skill 选择，没有使用算法匹配、向量嵌入或分类器。**
> **系统在 Skill 工具的描述中提供格式化的可用 Skills 列表，**
> **Claude 根据 description 中的关键词和（语义）判断。"**

**四步**：
```
① 搜索匹配：Claude 回顾所有 Skills 的元数据和描述，判断哪些与当前任务相关
② 加载技能说明：匹配到后用 bash 读取该技能文件夹下的 SKILL.md
③ 执行内部流程：按 SKILL.md 说明，按需读取额外文档或运行脚本
④ 串联复用：一个任务需要多个技能协作时，依序调用
```

**⭐⭐ 这解释了一个重要问题**：
> **为什么 description 写得准不准，直接决定 Skill 会不会被调用** ——
> **因为选择完全靠语义匹配，没有别的机制兜底** ⚠️

---

## ⭐ 二、灵码里的落地方案（openskills）

**三步**：
```bash
# 1. 安装
openskills install anthropics/skills
# → 在 .claude 下生成 skill 库

# 2. 同步（在项目根目录生成 agent.md）
openskills sync

# 3. 适配
把 AGENTS.md 里的提示词粘贴到灵码的 project_rules.md
```

**⭐ 关键认知**：
> **"上面生成的文件，就是 Rule，只不过在不同 AI 编程工具里有不同的叫法：
> Claude Code 里叫 claude.md，Codex 里叫 AGENTS.md，灵码里就是 project_rules.md"**

**→ 本质是同一套东西，换个文件名。**

---

## ⭐⭐ 三、自定义 Skill 示例：代码审核

**元信息**：
```yaml
skill: code-reviewer
description: 这个 Skills 帮助进行代码审查，提供代码质量分析/报告生成、最佳实践建议和潜在问题识别。
location: project
```

**SKILL.md 的四个审查重点**：
| # | 维度 | 检查项 |
|:--:|------|-------|
| **1** | **代码质量** | 命名规范、代码复杂度、重复代码 |
| **2** | **安全性** | SQL 注入风险、XSS 漏洞、认证授权问题 |
| **3** | **性能** | 算法效率、资源使用、缓存策略 |
| **4** | **可维护性** | 代码注释、模块化设计、测试覆盖 |

**审查流程**：
```
① 理解代码变更的目的
② 检查代码风格和规范
③ 分析潜在 Bug 和性能问题
④ 验证安全性
⑤ 提供建设性改进建议
```

**⭐⭐ 输出格式（可直接抄，很实用）**：
```markdown
✅ 优点：列出做得好的地方
⚠️ 问题：按严重程度分类
   🔴 严重：需要立即修复
   🟡 中等：建议修复
   🟢 轻微：可选改进
💡 建议：具体改进方案和示例代码
```

---

## 四、对我们的启示

| # | 启示 | 怎么用 |
|:--:|------|-------|
| **1** ⭐⭐⭐ | **Skill 选择靠 description 语义匹配** | **description 必须写清"做什么 + 何时用"**，否则不会被调用 |
| **2** ⭐⭐⭐ | **Skill 不是代码，是"上下文注入"** | 别指望它执行逻辑，它的价值是把专业知识和流程灌进上下文 |
| **3** ⭐⭐ | **四件套结构：SKILL.md / scripts / references / assets** | 我们的 Skill 目录规范 |
| **4** ⭐ | **不同工具只是换个文件名** | claude.md / AGENTS.md / project_rules.md 本质相同 |
| **5** | **输出格式模板可直接复用** | 优点/问题（分严重度）/建议 三段式 |

**⚠️ 一个提醒**：
> **本文说 Skill 含 `assets/` 目录** —— 但**Anthropic 官方文档用的是「Resources（资源）」**，
> `assets` 是社区实现（openskills）的叫法，**对领导/甲方讲时要按官方口径** ⚠️

---
*蒸馏：Hermes ｜ 2026-11-04 ｜ 原文 7,600 字*
