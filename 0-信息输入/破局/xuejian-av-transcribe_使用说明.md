# xuejian-av-transcribe 安装使用说明（Mac）


> ⚠️ **平台适用性提示（2026-09 补）**
> 本文写于 **Claude Code** 语境：文中路径 `~/.claude/skills/` 与「**触发词**」均为 **Claude Code CLI 专属**。
> **跨平台通用做法**：把 Skill 放在 **`.agents/skills/<名称>/SKILL.md`**（各平台均识别）；触发**靠 `description` 自动匹配**，不存在「触发词」机制。
> 若在 **WorkBuddy** 使用：走**技能页 → 导入本地文件/技能包**，无需关心目录路径。

把视频/音频 → 文案，一步到位（SenseVoice 转写 + DeepSeek 自动纠错）。

## 安装（3 步，2 分钟）

1. **下载本包** 并解压（`xuejian-av-transcribe-install/` 文件夹）
2. **打开终端**，进入文件夹执行一键安装：
   ```bash
   cd ~/Downloads/xuejian-av-transcribe-install   # 换成你的实际路径
   bash install.sh
   ```
3. **配置 API Key**（脚本会提示）：到 https://cloud.siliconflow.cn 注册拿 Key
   ```bash
   echo "export SILI_FLOW_API_KEY=你的Key" >> ~/.zshrc && source ~/.zshrc
   ```

## 使用（两种方式）

### 方式 A：Claude Code 对话触发（推荐）
```bash
claude
# 然后输入：
#   转写并修正 ~/Downloads/课程.mp4 的文案
#   或：/xuejian-av-transcribe ~/Downloads/访谈.m4a
```

### 方式 B：直接跑脚本
```bash
python ~/.claude/skills/xuejian-av-transcribe/scripts/transcribe_fix.py "你的文件.mp4"
# 可选参数：
#   --output-dir 目录     指定输出位置
#   --no-fix             只转写不修正
#   --fix-model 模型名    换修正模型（默认 deepseek-ai/DeepSeek-V4-Flash）
#   --segment-minutes 20  长音频切片长度
#   --keep-audio         保留中间音频
#   --json               输出机器可读结果
```

## 输出（与输入同目录）
| 文件 | 内容 |
|------|------|
| `xxx.raw.txt` | ASR 原始转写（修正前，便于人工比对） |
| `xxx.fixed.txt` | 修正后的文案（纯文本） |
| `xxx.report.json` | 模型/段数/字数等元信息 |

## 排错
| 问题 | 解法 |
|------|------|
| /skills 看不到 | 重启 claude 会话；检查 SKILL.md 第一行是 `---` |
| 401/402/403 | API Key 无效或余额不足 → 检查 SILI_FLOW_API_KEY |
| 转写为空 | 音频无语音音轨（纯音乐）或编码异常 |
| 找不到 ffmpeg | `pip3 install imageio-ffmpeg` |
| 修正块丢内容 | 用 .raw.txt 人工处理；或换更稳模型重跑（幂等可重跑） |

## 费用提示
- SenseVoice 转写价格极低；成本主要在 LLM 修正段（长音频 × 每 2500 字一次调用）
- 转写/修正共用硅基流动同一个 Key，账单集中

## ⚠️ 提醒（针对触发）
- SKILL.md 的 description 目前以**英文**写触发词（extract transcript/转写/语音转文字 等）。如果你习惯用中文长句触发且发现不自动命中，可在 `~/.claude/skills/xuejian-av-transcribe/SKILL.md` 的 description 里补上："当用户说：把视频/音频转成文字、提取文案、整理录音稿 时触发"——描述改得越准，自动触发概率越高。
