# renhua（人话）

把 AI 说的看不懂的话，变成人能看懂的大白话。

一个 Claude Code 技能：黑话、自造词换成日常说法，术语跟一句解释，结论先行；数字、命令、代码、因果一个字不改。还能改界面文案、讲清楚做了什么，或者设成每个会话自动说人话。

## 为什么需要它

AI 写东西常犯的毛病，你多半见过：

- 满嘴黑话："赋能""抓手""闭环""底层逻辑"，不结合上下文根本看不懂
- 缩写和自造词满天飞，从不解释
- 先铺垫三段才说结论，或者根本没有结论
- 一股 AI 腔：排比、"值得注意的是"、"不是…而是…"、假深刻的结尾

renhua 把这些换成大白话，只改表达，不改事实。

## 安装

方式一（推荐，可 `git pull` 更新）：

```bash
git clone https://github.com/allanpk716/renhua.git ~/.claude/skills/renhua
```

方式二（npx skills）：

```bash
npx skills add allanpk716/renhua
```

方式三：手动拷贝本仓库的 `SKILL.md`、`rules.md`、`jargon.md`、`resident.md` 到 `~/.claude/skills/renhua/`。

装完重启 Claude Code（或新开会话）。想让 AI 助手帮你装：让它读 [AGENTS.md](AGENTS.md)。

## 用法

### 1. 说人话

```
/renhua 把下面这段改成人话：本项目通过底层数字化能力沉淀，赋能研发全链路，实现降本增效……
```

支持粘贴文本、文件路径，或者"你刚才那条回复"。输出改好的全文，后面列出改了哪几处。默认只改看不懂的部分；说"狠狠改"就连风格一起重写。

### 2. 改界面文案

```
/renhua 改界面文案：<粘贴菜单和功能说明，或给出文件路径>
```

输出"原文案 → 新文案"对照表：名称 2–4 字，描述不超过 20 字，功能不改，拿不准的先问你。

### 3. 讲清楚做了什么

```
/renhua 用 STE 解释刚才做了什么
```

按 ASD-STE100（航空业的简化技术英语写作标准）八成的程度来写：短句、一句只说一件事。适合实验报告、任务总结。

### 4. 本次对话都说人话

```
/renhua on
```

之后每条回复都第一句给结论，不用黑话和自造词，术语第一次出现时解释一句。

### 5. 每个会话都自动说人话（常驻）

```
/renhua 常驻
```

AI 先给你看要加的内容，你确认后，它把 [resident.md](resident.md) 里的规则写进 `~/.claude/CLAUDE.md`。新开的会话就会自动照着写。

- 关掉：`/renhua 常驻 关`
- 规则更新后：再喊一次 `/renhua 常驻`，换成新版
- 不想让 AI 改文件：自己把 `resident.md` 的内容贴进 `~/.claude/CLAUDE.md`

常驻是让模型照着规则写，不是硬性保证；偶尔没做到，喊一声 `/renhua` 改一下就行。技能本身不会自己跳出来，要么手动喊，要么开常驻。

## 规则和文件

核心规则（人话十条）：结论先行 / 术语即出现即解释 / 黑话换人话 / 具体不抽象 / 分层输出 / 锁事实 / 不装懂 / 去 AI 腔 / 只改必要的 / 技术内容不动。

| 文件 | 是什么 |
|---|---|
| `SKILL.md` | 技能入口：几种用法怎么做 |
| `rules.md` | 人话十条完整版，带正反例 |
| `jargon.md` | 黑话对照表。遇到新黑话，加一行就行（欢迎提 PR） |
| `resident.md` | 常驻规则，`/renhua 常驻` 写进 CLAUDE.md 的就是它 |
| `AGENTS.md` | 给 AI 助手看的安装、配置说明 |

## 致谢

本项目的思路与词表方向受益于这些项目（词表为自编，非复制）：

- [MrGeDiao/shuorenhua](https://github.com/MrGeDiao/shuorenhua)（说人话）
- [KKKKhazix/human-writing](https://github.com/KKKKhazix/human-writing)（活人感写作）
- [blader/humanizer](https://github.com/blader/humanizer)
- [op7418/Humanizer-zh](https://github.com/op7418/Humanizer-zh)
- [justjavac/ali-words](https://github.com/justjavac/ali-words)（互联网黑话词表）
- [mcsrainbow/chinese-internet-jargon](https://github.com/mcsrainbow/chinese-internet-jargon)
- [itorr/nbnhhsh](https://github.com/itorr/nbnhhsh)（能不能好好说话）

## License

MIT
