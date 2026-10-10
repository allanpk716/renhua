# AGENTS.md — 给 AI agent 的使用说明

你是被用户叫来安装、配置或使用 renhua（人话）技能的 agent。本文件是给你的操作说明；用法入口在 `SKILL.md`，规则在 `rules.md`。

## 这是什么

renhua 是一个 Claude Code 技能：把看不懂的 AI 输出改成大白话（黑话、自造词换成日常说法，术语就地解释，结论先行，分层），**事实零改动**——数字、命令、代码、因果、限定词（可能/部分/计划等）不升级不弱化。词表在 `jargon.md`，常驻规则在 `resident.md`。

## 安装（用户没装时）

三选一，装完让用户重启 Claude Code（或新开会话）才会出现：

```bash
# 方式一：git clone
git clone https://github.com/allanpk716/renhua.git ~/.claude/skills/renhua

# 方式二：npx skills
npx skills add allanpk716/renhua
```

方式三（手动）：拷 `SKILL.md`、`rules.md`、`jargon.md`、`resident.md` 四个文件到 `~/.claude/skills/renhua/`。目录结构必须是 `skills/renhua/SKILL.md`（多包一层目录会找不到）。

Windows 进阶：想"仓库一更新技能自动跟着新"可用 junction 代替复制：

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\renhua" -Target "<本仓库的绝对路径>"
```

删 junction 只能用 `rmdir`，**绝不能用 `rm -rf`**——Git Bash 的 rm 会顺着链接把真实仓库里的文件一起删掉。

## 触发语义（重要）

- **仅显式触发**：用户输入 `/renhua`，或明确说"说人话 / 改写 / 改文案 / 用 STE 解释 / renhua"才调用。其他任务不要自动套用——这是设计决定（见 [ADR 0001](docs/adr/0001-常驻不走技能自动触发.md)），不是遗漏
- 三种用法：**说人话**（改写给定文本，先列不能动的事实，改完对一遍）／**改界面文案**（出"原文案 → 新文案"对照表）／**讲清楚做了什么**（按 ASD-STE100 八成程度）
- 另有 **`/renhua on`**（本次会话后续输出都说人话）和 **`/renhua 常驻`**（每个会话自动说人话）
- 用户要求正式文体（公文/论文/合同）时，用户要求优先，不套大白话

## 常驻（可选）

用户不想每次喊：让用户说 `/renhua 常驻`，或你直接照 SKILL.md「开启 / 关闭常驻」节做——把 `resident.md` 的内容给用户看，确认后写进用户 `~/.claude/CLAUDE.md` 的 `<!-- RENHUA_START -->` / `<!-- RENHUA_END -->` 标记之间；已有标记就整段替换。关闭：删掉标记块。

用户的 CLAUDE.md 里如果已经有一段没有标记的"## 说人话规则"（以前手动贴的旧版），先问用户要不要删掉它，避免两份规则同时生效。

**不要**用改宽技能 description 的"模型自动触发"来实现常驻——概率性行为，长会话会忘、弱模型更差（ADR 0001 记录了取舍）。

## 与其他输出风格技能同开（如 i-have-adhd）

按共存裁决执行，总原则"结构让 adhd，语言让 renhua"：

1. **首行先给答案**；答案本身可执行（如一条命令）时答案即动作；纯问答没有真实动作就不硬造。以本条具体条款为准——不要按总原则反推出"动作优先"
2. 术语解释保留，压成一句括号；列表超 5 条分组或分层，主干在前，不丢内容
3. 结尾行动项：有真实下一步才给；纯问答、解释型回答豁免

这条裁决写在 `resident.md` 末尾，随常驻生效。

## 维护

- 用户遇到新黑话：往 `jargon.md` 对应表加一行即可（动表不动规则）
- 不要擅自改 `rules.md` 的规则语义；用户要求修改时，先复述要改哪条、确认后再动，并同步 `resident.md`。已经写进用户 CLAUDE.md 的常驻块，让用户再说一次 `/renhua 常驻` 即可刷新
- SKILL.md 的用法二、用法三是用户实测好用的提示词，保持原样，不要拆成细则
