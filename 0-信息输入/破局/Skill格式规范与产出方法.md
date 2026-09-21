# 训练营 Skill 体系 · 格式规范与产出方法（全文档归纳）

> 归纳自《12期打卡训练营课件：产品经理AI提效》（26万字）——覆盖第1/2/3/4/5/6/7/8/9/12章的 Skill 教学。
> 目标读者：要自己写 skill / 拆解课程 skill 结构的产品经理。


> ⚠️ **平台适用性提示（2026-09 补）**
> 本文写于 **Claude Code** 语境：文中路径 `~/.claude/skills/` 与「**触发词**」均为 **Claude Code CLI 专属**。
> **跨平台通用做法**：把 Skill 放在 **`.agents/skills/<名称>/SKILL.md`**（各平台均识别）；触发**靠 `description` 自动匹配**，不存在「触发词」机制。
> 若在 **WorkBuddy** 使用：走**技能页 → 导入本地文件/技能包**，无需关心目录路径。

---

## 〇、Skill 的本质（贯穿全营的一句话）

> **Skill 不是"更复杂的提示词"，而是"可调用的流程"。** 封装一次，永久复用；触发一次，自动完成一套流程。
> 类比：提示词=每次手写完整菜谱；Skill=把菜谱存成"一键下单"，改一道菜只改一处。

---

## 一、Skill 的存放位置（两级）

| 级别 | 路径（Claude Code） | **跨平台推荐** ⭐ | 作用范围 |
|------|-------------------|-----------------|---------|
| **USER 级** | `~/.claude/skills/<skill名>/SKILL.md` | **`~/.agents/skills/<skill名>/SKILL.md`** | 当前用户所有项目 |
| **项目级** | `<项目根>/.claude/skills/<skill名>/SKILL.md` | **`<项目根>/.agents/skills/<skill名>/SKILL.md`** | 仅该项目（可进 git 共享） |

> ⚠️ **上表第一列是 Claude Code 专属**；**`.agents/skills/` 才是各平台通用目录**（Trae / Qoder CN / CodeBuddy / Kimi / Claude Code 均识别）。
> **WorkBuddy**：不用管路径，走**技能页 → 导入本地文件/技能包**。

**其他平台路径**：Trae `.trae/skills/`｜Qoder CN `~/.lingma/skills/`｜CodeBuddy `.codebuddy/skills/`｜ZCode `~/.zcode/skills/`

- **优先级**：项目级覆盖 USER 级同名 Skill
- **口诀**：USER 级管通用，项目级管专属


---

## 二、SKILL.md 的格式规范（三种形态，由简到繁）

### 形态 A：单体 Skill（最简单，第 1-3 章教学）

```plaintext
~/.claude/skills/<skill-name>/
└── SKILL.md          ← 唯一文件
```

**SKILL.md = YAML frontmatter + 正文 6 模块**：

```markdown
---
name: <skill名>            # 全小写英文+中划线；= 触发词去掉/；CLI靠它识别
description: <一句话说清"做什么+何时用"，含触发关键词提升命中率>
---

# Skill：<skill名称>

## 触发词（⚠️ 仅 Claude Code CLI 适用）
/<skill-name>               # 用户输入即触发（其他平台无此机制，靠 description 自动匹配）

## 一句话描述
{做什么 + 产出长什么样}      # 给 AI 执行时读（比 frontmatter 简）

## 输入约定
- 必填：{用户必须提供什么：文件路径/粘贴内容}
- 可选：{增强项，如参与人名单/产品背景}

## 执行步骤
1. {按顺序做什么}           # 3-10步，越具体越好
2. …

## 输出契约
{输出长什么样：文件数、板块、字段、路径}   # 必须固定！防每次格式不同

## 约束
- {不能做什么/必须满足什么}  # 清洗规则/粒度/优先级/不编造等
```

**6 模块写法要点（每个都有反例教训）**：
| 模块 | 反例（❌） | 正例（✅） |
|------|-----------|-----------|
| 触发词 | "请用会议纪要Skill整理" | `/meeting-minutes` |
| 一句话描述 | "一个用来整理纪要的Skill"（太抽象） | "把转写文本整理成单文档结构化纪要（含议题/决策/行动项/风险）" |
| 输入约定 | "请提供会议文本"（不知传什么） | 必填：转写文本路径；可选：参与人文件 |
| 执行步骤 | "读取文本并整理"（一句话） | 1.读文本识别发言人 2.清洗 3.切议题 4.提炼决策 5.提取行动项 6.SMART校验… |
| 输出契约 | "一份会议纪要"（AI自由发挥） | 固定 4 大板块+字段+路径+末尾统计行 |
| 约束 | "整理清楚点"（无标准） | 单文档/口语词删除/行动项1人1天/责任人只从名单匹配/截止到日期/P0P1P2定义 |

> **frontmatter 是生死线**：第一行必须是 `---`，第二行 `name:`——缺了 CLI 直接加载不到（报 skill not found）。

**典型案例**：
- `meeting-minutes`（第2章）：转写文本→4板块纪要（元信息/议题决策/行动项SMART表/风险待定）
- `feedback-analysis`（第3章）：100+条反馈→5板块报告（元信息/TopN高频/情感分析/洞察5-8条/机会清单P0-P2+抽样校验记录）；执行 10 步含"自动抽10条给用户人工校验"

### 形态 B：复杂 Skill（SKILL.md 当调度中心 + 外挂资源，第 4/9 章教学）

```plaintext
~/.claude/skills/competitive-analysis/
├── SKILL.md                 ← 调度中心：只写总流程+白名单+约束+分流规则（很短！）
└── references/
    └── template.md          ← 报告模板（章节结构/层级）

~/.claude/skills/pm-diagrams/
├── SKILL.md                 ← 只写：按什么流程走、支持哪5种图、不能做什么
├── templates/               ← 每种图一套模板+填写说明+审查checklist
│   ├── product-architecture.ascii.md / .drawio
│   ├── business-process.ascii.md / business-process.md (mermaid)
│   ├── swimlane / sequence / er …
│   └── _subagent_review_prompts.md
└── examples/                ← 每种图的纯成品示例（AI照抄风格用）
    ├── product-architecture.ascii.md …
```

**设计意图**：SKILL.md 是"调度中心"不是"仓库"——AI 用到哪种图才加载哪个模板，按需加载省上下文、方便单独维护。**从"一个 SKILL.md 打天下"到"SKILL.md + 外挂资源目录"= 单文件 skill 升级成工程化 skill 的分水岭。**

**典型案例**：
- `competitive-analysis`（第4章，深度战略级"重武器"）：8 阶段 Subagent 编排（数据收集[并行]→事实核查[隔离并行]→行业分析[串行]→产品拆解[并行]→财务团队[隔离并行]→报告转写[串行]→多视角独立评审[隔离并行→3评审各写独立文件]→修改[串行]+输出修改说明）；约束含"数据必须标【来源：URL】、8000-15000字、模板合规、评审输出隔离"。⚠️ token 消耗大，非深度需求慎用
- `pm-diagrams`（第9章）：5种图分流（架构/泳道→drawio，流程/时序/ER→mermaid）；两段式=先 ASCII 草图审结构→再转代码审语法，换个 AI 独立审查

### 形态 C：项目型 / 二段式 Skill（skill 只管生、项目自己长，第 7/12 章教学）

```plaintext
~/.claude/skills/prototype-design/     ← skill 本体（全程不动）
├── SKILL.md                ← 只负责 init 那一下
└── assets/scaffold/        ← 脚手架：三栏查看器+8示例页+8模板+7维护脚本+项目CLAUDE.md

初始化后长出的项目（在用户工作目录）：
jianan-wms-prototype/
├── index.html              # 三栏查看器（左目录/中页面/右描述）
├── nav.json                # 目录唯一权威（PM 改它）
├── nav.js                  # nav.json 同步产物，勿手改
├── CLAUDE.md               # 项目工作守则（红线+常用工作流+陷阱）
├── assets/  components/    # 框架与共享资源（不碰）
├── templates/              # 页面模板源（pc×4 + mobile×4，可沉淀自己的模板）
├── pages/                  # 原型页 HTML（每页一个）
├── desc/                   # 四段描述 md（PM 写：页面目的/交互说明/业务规则/异常情况）
└── scripts/                # add/remove/rename/sync_nav/build_desc/validate.py
```

**核心准则**：
- **二段式**：skill 只 init（一次）；初始化后"加页/改描述/删节点"由项目自带 CLAUDE.md + scripts 接管
- **分层红线**：项目改动只碰业务内容（pages/desc/nav.json/项目CLAUDE.md/项目templates），不碰查看器框架与脚本；skill 本体永不随业务漂移
- **什么时候动 skill？** 只对"当前项目"的改动→项目内做；对所有项目成立的通用改进→才回 skill 本体（且先在项目验证）
- **只能 init 一次**：目标目录必须为空；手删 nav.json/CLAUDE.md 再 init = 禁止操作（防覆盖业务文件）

---

## 三、全营 Skill 一览表

| 章节 | Skill 名 | 形态 | 输入 → 输出 | 类型 |
|------|---------|------|------------|------|
| 第2章 | meeting-minutes 会议纪要 | A 单体 | 转写文本 → 4板块纪要(元信息/议题决策/行动项SMART/风险) | 自建 |
| 第3章 | feedback-analysis 反馈分析 | A 单体 | 100+条反馈 → 5板块报告+机会清单P0-P2 | 自建 |
| 第4章 | competitive-analysis 竞品分析 | B 复杂(Subagent编排) | 竞品名+信息 → 8000-15000字深度报告 | 自建 |
| 第5章 | 需求分析 skill | 用现成 | 用户反馈/原始材料 → 需求分析文档 | 调用 |
| 第6章 | PRD 撰写 skill | 用现成 | 需求分析 → PRD(输出到 content/第5章-08/prd/) | 调用 |
| 第7章 | demo-design 工程Demo | C 项目型 | PRD/需求 → 可运行工程 Demo(移动端) | 调用+构建 |
| 第8章 | 项目管理/排期 skill | 用现成 | 任务范围 → 排期初稿(校验落盘) | 调用 |
| 第9章 | pm-diagrams 5种图 | B 复杂(调度中心+外挂) | 需求描述+图类型 → ASCII草图→drawio/mermaid | 调用 |
| 第12章 | prototype-design 原型 | C 项目型(二段式) | PRD → 静态三栏原型(离线双击即开) | 调用+构建 |

---

## 四、每份 Skill 如何产出（通用生产流程）

### 路线 1：自建一个 Skill（第 2/3/4 章标准 4 步）

```
步骤1 场景与结构设计（15-25 min）
  · 判断要不要封装：每周≥2次？流程稳定？产出可模板化？→ 是才做
  · 画"单文档结构图"：输入什么 → 处理什么 → 输出哪几大板块
  · 会议纪要案例：转写文本 → 元信息/议题/行动项/风险 4 板块
步骤2 创建目录（5 min）
  · mkdir ~/.claude/skills/<skill-name>/（形态B 再建 references|templates|examples）
步骤3 编写 SKILL.md（30-45 min，核心工序）
  · YAML frontmatter：name（全小写+中划线）+ description（做什么+何时用+触发词）
  · 正文 6 模块：触发词→一句话描述→输入约定→执行步骤→输出契约→约束
  · 每个模块对照"反例→正例"自查（见第二节表格）
  · 复杂任务升级：执行步骤拆 Subagent 编排（并行/隔离并行/串行标注）；模板示例外挂
步骤4 测试、调优、部署（30 min）
  · /skills 验证已加载（看不到=查 frontmatter 第一行是否 ---）
  · 拿 1 份真实输入试跑 → 检查输出是否符合输出契约
  · 迭代：约束漏了什么（AI 把"提议"写成"决定"？→ 补约束）
  · 部署位：USER 级 ~/.claude/skills/ 或项目级 ./.claude/skills/
```

### 路线 2：调用/学习一个现成 Skill（第 5/6/8/9 章工作流）

```
步骤1 解剖参考 skill（10 min）：读 SKILL.md 的 description/结构/输入输出
步骤2 调用生成初稿（5 min）：按输入约定给素材 → AI 走 skill 流程输出
步骤3 人工审稿定稿（核心！）：改业务内容；AI 生成物只当"初稿"
步骤4 按需调优 skill（5 min）：改出来的好模板可反哺 skill 本体（仅通用改进时）
```

### 路线 3：构建项目型 Skill（第 7/12 章：prototype-design 范式）

```
· skill 本体：SKILL.md 只写 init 逻辑（检测空目录→拷 scaffold→替换项目名→跑同步脚本）
· scaffold 内预置：查看器/示例页/模板集/维护脚本/项目 CLAUDE.md（含红线+工作流+陷阱）
· 项目长出来后：PM 全程对话驱动（提需求→AI 跑 add_page.py→PM 写四段描述→sync→刷新验收）
· 维护准则：项目内沉淀模板(项目templates/)；通用改进才反哺 skill；validate.py 兜底一致性
```

---

## 五、Skill 质量验收检查单（写完自检）

- [ ] 目录名/name/触发词三者一致（全小写英文+中划线）
- [ ] SKILL.md 第一行是 `---`，含 name + description（description 含 5+ 触发关键词更佳）
- [ ] 正文 6 模块齐全（触发词/描述/输入约定/执行步骤/输出契约/约束）
- [ ] 执行步骤 ≥3 步且顺序明确（不是"读取并整理"一句话）
- [ ] 输出契约固定（文件数/板块顺序/字段/路径写死）
- [ ] 约束覆盖该场景的"AI 翻车点"（编造/粒度/格式漂移/越权判断）
- [ ] 用 /skills 验证已加载 + 真实输入试跑 1 次
- [ ] 简单功能用单体；模板多/任务重拆外挂；要长项目用二段式
