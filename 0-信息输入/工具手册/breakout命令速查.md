# breakout CLI · 核心命令速查（AI 破局俱乐部）

> 版本基准 0.1.7（实测沉淀）。全局：所有命令支持 `--json`；分页 `--page/--size`（默认 20）；搜索 `-k/--keyword`。

## 0️⃣ 账号准备（一次性）

| 命令 | 作用 |
|------|------|
| `breakout auth login` | 登录（浏览器设备授权，凭据有效期数月） |
| `breakout auth status` | 看登录身份/有效期 |
| `breakout doctor` | 环境自检（登录/网络/能力/配置） |
| `breakout capabilities` | 当前账号可用能力清单 |

## 1️⃣ 刷大咖/搜干货（最高频）

| 目的 | 命令 |
|------|------|
| 关键词全局搜索主题 | `breakout topic list -k "智能体" --size 20` |
| 看某大咖的主题 | `breakout topic author --nickname 洋哥`（昵称自动解析） |
| 大咖+关键词过滤 | `breakout topic author --user-number 1 -k "skills"` |
| 主题详情（正文+互动数据） | `breakout topic detail <topicId>` |
| 我的主题 | `breakout topic mine` |
| 人物身份解析（免登录） | `breakout person resolve 洋哥` |

## 2️⃣ 互动（点赞/收藏/评论，全幂等）

| 操作 | 命令 |
|------|------|
| 点赞/取消 | `breakout topic like <topicId>` / `unlike` |
| 收藏/取消（可加文件夹） | `breakout topic collect <topicId> --folder 干货` / `uncollect` |
| 看收藏夹 | `breakout topic collections` |
| 发评论/回复 | `breakout topic comment <topicId> -c "内容"`（`--parent <评论ID>` 为回复） |
| 看评论/回复 | `breakout topic comments <topicId>` / `replies <评论ID>` |
| 删自己的评论 | `breakout topic comment-delete <评论ID>` |
| 评论点赞 | `breakout topic comment-like <评论ID>` |

## 3️⃣ 行动营 / 组局

| 目的 | 命令 |
|------|------|
| 近期行动营 | `breakout action list` |
| 行动营详情/手册 | `breakout action camp <ID>` / `manual <ID>` |
| 组局列表（最强筛选） | `breakout group list --province 四川 --city 成都 --status 报名中` |
| 组局详情 | `breakout group detail <ID>` |
| 我参加/发起的 | `breakout group joined` / `mine` |

## 4️⃣ AI 之心 / 勋章 / Skill 商城

| 目的 | 命令 |
|------|------|
| 余额/明细 | `breakout heart balance` / `ledger` |
| 勋章 | `breakout medal list` / `catalog` |
| 浏览 Skill 商城（207 个） | `breakout skill catalog -k 关键词` |
| 安装 Skill | `breakout skill install <Skill ID>` |

## 5️⃣ Agent 集成（AI 助手直连）

| 方式 | 命令 |
|------|------|
| MCP 服务 | `breakout mcp serve`（客户端配置 command=breakout args=[mcp serve]） |
| Codex 集成 | `breakout integrate codex` |
| 动态调用任意能力 | `breakout call <能力ID> --set 字段=值`（先 `capabilities describe <id>` 查 schema） |

## ⚠️ 已知陷阱速记

1. **时区偏移**：点赞/收藏的 `-status` 返回时间按 **UTC-8** 显示，别误判
2. **返回结构不一致**：`topic list/mine` 结果在顶层 `records`；`topic author` 在 `topics.records`——解析前先看结构
3. **`group history` 无分页参数**（位置参数传星球编号）
4. **ID 类型**：topicId = 数字字符串（从列表返回值取）；评论/组局/勋章 ID = UUID
5. **heart ranking/statistics、medal catalog 返回裸数组**（非对象包裹）
6. **`person resolve` 用位置参数**（无 `--nickname` 标志）
7. 写操作（评论/收藏/安装）需用户授权；组局**不得推算费用**

## 🎯 新手 3 条高频链路

```
① 刷干货：topic list -k 关键词 → topic detail 看正文 → topic collect 收藏
② 盯大咖：person resolve 洋哥 → topic author --nickname 洋哥 -k 主题词 → topic like
③ 找组局：group list --province X --city Y → group detail → （线下参加）
```
