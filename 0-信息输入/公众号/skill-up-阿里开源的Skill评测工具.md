---
title: skill-up：阿里开源的 Agent Skill 评测工具
source: https://mp.weixin.qq.com/s/5Tvv8g20CybjbT0a7iUfHw
author: 狂师
tags: [Skill评测, 工具, skill-up, 断言, CI]
---

# 阿里开源：skill-up，一款 Agent Skill 评测工具

> **一句话结论**：**把软件测试那套方法论，完整地平移到 Skill 上** ——
> 用 YAML 声明用例、隔离多引擎运行、三种断言、结构化报告，**本地或 CI 都能跑**。

---

## ⭐ 开篇的痛点（几乎每个做 Skill 的人都中）

> **"每次迭代更新一个 Skill，我心里都在打鼓。**
> **改了几行提示词，行为有没有变？跑一遍 demo 没报错，就敢发出去吗？下个版本还会好吗？"**

**⭐ 行业现状（很扎心）**：
> **"在测试领域，有一项铁规：绝不会允许一个没有用例、没有回归、没有质量门禁的系统直接上线。
> 但到了 Skill 这件事上，几乎所有人都在裸奔。写完跑两下，感觉没问题，发布。"**

**⚠️ 根本原因**：
> **"Skill 的本质是提示词工程，SKILL.md 改一个字，行为就可能漂移。"**

**⭐⭐ 一句话回答"Skill 好坏由谁定义"**：
> **应该由「能不能在真实 Agent Engine 里跑对」来定义**，
> 而不是"demo 跑通了"。

---

## ⭐⭐ 一、标准目录结构（照着建就能用）

```
my-skill/
├── SKILL.md                  # Skill 定义（被测对象）
└── evals/
    ├── eval.yaml             # 评测入口配置（必须）
    ├── cases/                # 用例目录
    │   ├── basic-success.yaml
    │   ├── edge-case-null.yaml
    │   └── regression-001.yaml
    ├── fixtures/             # 测试资源（可选）
    │   ├── repos/            # 仓库模板
    │   ├── diffs/            # 补丁文件
    │   └── scripts/          # 评估脚本
    └── mcp/                  # MCP 工具配置
        └── github.json
```

**⭐ 这个结构的巧妙之处**：
> **"这就是一个标准的测试工程目录。"**
> `cases/` = 测试用例集，`fixtures/` = 测试数据和脚手架，`eval.yaml` = 全局配置（定义**在什么环境、用什么 Engine、怎么评估**）。

---

## 二、eval.yaml 的关键字段

```yaml
schema_version: v1alpha1
environment: ...
engine:
  name: claude_code        # 可以是 Claude Code / Codex / Qoder CLI
  model:
    provider: anthropic
    name: claude-sonnet-4-8
cases:
  files:
    - evals/cases/my-test.yaml
```

**⭐ 关键**：**支持真实 Agent Engine**（不是模拟环境）——
**"验证 Skill 在真实 Agent Engine（Claude Code、Codex、Qoder CLI）中的功能正确性。"**

---

## ⭐⭐ 三、三种断言（对应测试的三种校验）

| 断言类型 | 做什么 | 对应测试概念 |
|---------|-------|------------|
| **rule_based** | **规则匹配**：文件是否存在、内容是否包含关键字 | 断言 |
| **script** | **跑自定义脚本**：校验产物结构、跑语法检查 | 脚本校验 |
| **agent_judge** | **用另一个模型当裁判**，按标准给输出打分 | **LLM-as-a-Judge** |

**⭐ 第三个值得注意**：
> **"agent_judge 就是我们在大模型评测里反复讲的 LLM-as-a-Judge，用模型评模型的输出 ——
> 这在 Agent 评测体系里已经是标配手段，现在直接内置到 Skill 测试工具里了。"**

---

## 四、一个用例长什么样

```yaml
id: find-null-bug
title: 应该识别出空指针 bug
prompt: <发给 agent 的话>
judge:
  - type: rule_based
    ...
```

**⭐ 设计理念**：**每个 `.yaml` 定义"发什么 prompt" + "怎么验证结果"** ——
**大多数场景单轮 prompt 即可。**

---

## ⭐ 五、它的定位：把 Skill 从"裸奔"拉进工程

**能力组合**：
```
声明式 YAML 评测 + 隔离的多引擎运行 + 灵活的 Judge + 结构化报告
        ↓
质量可度量
        ↓
skill-upper 把失败转化为改进（自动修复或补充 eval 用例）
        + 持续回归
```

**兼容与输出**：
- 同一套流程**既可在本地运行，也可接入 CI**
- **兼容 Anthropic `evals.json` 导入**
- 输出 **JSON、JUnit 和 HTML 报告**

---

## ⭐⭐ 六、对我们的直接价值（**最重要**）

**⭐ 我们现在的评测正是"裸奔"状态** ——
P1 任务写着"对 skill 进行评测"，但**没有用例、没有回归、没有门禁**。

**⭐ skill-up 可以直接用**：

| 我们的需求 | skill-up 怎么解 |
|-----------|----------------|
| 每个 Skill 建 3 个以上用例 | `cases/*.yaml` 一文件一用例 |
| 有无 Skill 对照实验 | `engine` 配置切换，跑两遍对比 |
| 缺料测试（材料不全） | `edge-case-*.yaml` |
| 回归（改完不退化） | `regression-*.yaml` |
| 接 CI | 原生支持 |
| 结构化报告 | JSON / JUnit / HTML |

**⭐⭐ 落地建议（三步）**：
```
① 先给 1 个 Skill（建议 prd-writer）建 evals/ 目录，写 3 个用例
② 本地跑一遍，看报告能不能指出"哪里失败"
③ 跑通后接 CI，作为 Skill 变更的准出门禁
```

**⚠️ 注意**：它是 Go 写的，需要 Go 环境；如果内网装不了，**照它的目录结构和断言设计自己实现一个也能覆盖 80% 需求**。

---

## 七、一句话

> **"Skill 的本质是提示词工程，SKILL.md 改一个字，行为就可能漂移。"** ⚠️
>
> **⭐⭐ 核心价值**：
> **把 Skill 评测变成"标准测试工程"** ——
> **用例（cases）+ 资源（fixtures）+ 配置（eval.yaml）+ 三种断言 + 结构化报告 + CI**
>
> **⭐ 最该抄的一点**：
> **断言分三类：规则匹配 / 脚本校验 / LLM 当裁判 —— 确定性优先，LLM 兜底。**

---
*蒸馏：Hermes ｜ 2026-11-04 ｜ 原文 7,374 字*
