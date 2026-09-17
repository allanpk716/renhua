# renhua（人话）

把 AI 说的看不懂的话，变成人能看懂的大白话。

一个 Claude Code 技能：黑话换成人话、缩写就地展开、术语跟一句解释、结论先行。改写已有文本，或让本次对话接下来都说人话。

## 为什么需要它

AI 写东西常犯的毛病，你多半见过：

- 满嘴黑话："赋能""抓手""闭环""底层逻辑"，不结合上下文根本看不懂
- 缩写和自造词满天飞，从不解释
- 先铺垫三段才说结论，或者根本没有结论
- 一股 AI 腔：排比、"值得注意的是"、"不是…而是…"、假深刻的结尾

renhua 把这些统统换成大白话，而且**数字、命令、代码、结论的因果关系一律不动**——只改表达，不改事实。

## 安装

方式一（推荐，可 git pull 更新）：

```bash
git clone https://github.com/allanpk716/renhua.git ~/.claude/skills/renhua
```

方式二（npx skills）：

```bash
npx skills add allanpk716/renhua
```

方式三：手动拷贝本仓库的 SKILL.md、rules.md、jargon.md 到 `~/.claude/skills/renhua/`。

## 用法

**改写一段文本**（支持粘贴文本 / 文件路径 / "你刚才那条回复"）：

```
/renhua 把下面这段改成人话：本项目通过底层数字化能力沉淀，赋能研发全链路，实现降本增效……
```

**让本次对话接下来都说人话**：

```
/renhua on
```

默认只改看不懂的部分（保守档）；说"狠狠改"就连风格一起重写（激进档）。

## 核心规则（人话十条）

结论先行 / 术语即出现即解释 / 黑话换人话 / 具体不抽象 / 分层输出 / 锁事实 / 不装懂 / 去 AI 腔 / 只改必要的 / 技术内容不动。完整版见 [rules.md](rules.md)。

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
