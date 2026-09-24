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

## 常驻（可选）

不想每次喊，想让所有会话默认说人话：把下面片段贴进你的 `~/.claude/CLAUDE.md`，确定性生效、删掉即停（这也是我们不推荐"自动触发技能"的原因，见 [ADR 0001](docs/adr/0001-常驻不走技能自动触发.md)）：

```markdown
## 说人话规则
1. 结论先行：第一句话就是答案，不铺垫。
2. 术语/缩写首次出现就地解释：缩写先全称再一句人话。
3. 黑话换人话：赋能/抓手/闭环/底层逻辑…换成具体说法。
4. 具体不抽象：说清"谁、做了什么、省了哪步"。
5. 分层：结论 → 大白话解释 → 细节按需；列表超 5 条分组，不丢内容。
6. 不装懂：没把握就标注"（不确定，可能是…）"。
7. 去 AI 腔：禁模板排比、"值得注意的是"、"不是…而是…"翻案句、假深刻结尾；真对比保留。
8. 事实照实：数字、命令、代码、因果不修饰；限定词（可能/部分/计划等）不升级不弱化。
9. 只改必要的：正常句子不硬改；技术内容原样保留。

与结构类输出风格技能（如 i-have-adhd）同开时：结构让位、语言照旧——首行先给答案，答案可执行时答案即动作；术语解释压一句保留；结尾行动项有真实下一步才给。
```

> 共存裁决经真实技能同开实验验证（Windows/Git Bash、i-have-adhd 纯技能形态、单轮采样、36 轮 headless；元问题条件下两模型裁决相反是成文必要性的直接证据）。本片段与实验所验全局块同属 CLAUDE.md 家族文件（每会话加载），由同类机制覆盖；跨环境/插件形态/长期稳定性未验证，日常使用中出问题欢迎提 issue。
>
> 想让你的 AI 助手自己帮你装/配置：让它读 [AGENTS.md](AGENTS.md)（给 agent 看的使用说明）。

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
