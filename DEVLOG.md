# 开发记录（DEVLOG）

**项目**：renhua（人话）技能
**时间**：2026-09-17，一天内完成 调研 → 设计 → 开发 → 发布 → 验收 → 第一档自动化
**仓库**：https://github.com/allanpk716/renhua

---

## 1. 项目时间线

| 阶段 | 产物 |
|---|---|
| 双线调研（并行 agent） | ①AI 文风人性化/去黑话开源工具调研 ②AI 对话通俗化+结论先行调研（存 research_things，私有） |
| 头脑风暴 | 分类 Architectural；形态=改写器+提醒器一体；组织=主文件+对照表；命名=renhua |
| 设计定稿 | spec（research_things/docs/superpowers/specs/2026-09-17-renhua-skill-design.md） |
| 实施计划 | plan（同目录 plans/2026-09-17-renhua-skill.md，6 文件全文内联） |
| 开发+发布 | 6 文件 → git → GitHub 公开仓库 → 验收 5/5 |
| 自动化第一档 | 人话规则精简版进全局 `~/.claude/CLAUDE.md`（RENHUA_START/END 标记块） |
| 安装方式修正 | clone 副本 → JUNCTION 链接（本记录 §4 问题 5） |

## 2. 架构决策（为什么长这样）

1. **rules.md 单独成文件**：人话十条是"单一事实源"。将来三档自动化（CLAUDE.md 常驻 / output style / Stop 钩子扫描）都从它派生，避免维护两套规则。
2. **jargon.md 单独成文件**：词表是要长期养的（碰到新黑话加一行），和规则分开，动表不动规则。它同时是第三档扫描脚本的原料。
3. **SKILL.md 的 description 写窄**（"仅在用户明确要求时使用"）：显式调用是设计决定——不叫它就不插手，防自动触发误伤普通任务。
4. **保守档默认 / 激进档 opt-in**（"狠狠改"）：过度改写和黑话一样糟，验收测试 3 专门防误伤。
5. **词表自编 + README 致谢**：开源发布避协议纠纷（不整段复制 shuorenhua / human-writing / ali-words 等 MIT/Unlicense 项目的文本，思路借鉴+出处致谢）。
6. **锁事实方法论**（改写前列事实清单、改完回读）：来自 shuorenhua 的验证过的做法，保证"只改表达不改事实"。

## 3. 验收记录（2026-09-17，claude -p headless 新会话实测）

| # | 测试 | 结果 |
|---|---|---|
| 1 | 黑话浓度高文本 | ✅ 结论挪第一句、黑话全消、"数字化中台"就地解释、nuance 标注 |
| 2 | 术语堆砌文本 | ✅ PRNU/TCMS/K8s/HPA 全展开+人话，技术事实零改动 |
| 3 | 已通顺文本 | ✅ 直说"这段不用改"+理由，不硬找活干（防误伤） |
| 4 | 提醒模式 `/renhua on` | ✅ 一句确认不啰嗦，后续回答结论先行/术语解释/分层 |
| 5 | 未调用时 | ✅ 普通问答完全不被 renhua 干扰 |

## 4. 开发遇到的问题（现象 → 根因 → 解法）

### 问题 1：目标路径确认费了周折
- 现象：用户说"workspaces 下的 agents"，`C:\Users\allan716\workspaces`、D/E 盘都找不到
- 根因：实际是 `C:\WorkSpace\agent\`（WorkSpace 和 agent 都**不带 s**）
- 解法：`ls -d /c/*ork*` 逐盘定位；顺带发现旧空壳 `C:\WorkSpace\agent\去除AI味\`（2026-06-25 的空文件夹，未动，待处置）

### 问题 2：jargon.md 验证行数与计划不符
- 现象：计划预期 `grep -c "^|"` ≥70，实际 62
- 根因：**计划里的估算算错了**，不是实现缺陷——实际构成：表1 黑话 42 条+表头 2 行=44，表2 AI 腔 10+2=12，表3 缩写 4+2=6，合计 62，符合 spec 的"初始约 40 条"
- 教训：计划里的验证阈值要先数一遍再写；验收时对不上的数字先核对构成再定性

### 问题 3：claude -p 默认权限读不到技能附属文件
- 现象：headless 测试中模型说"没有 ~/.claude/skills/renhua/ 的读取权限，rules.md 和 jargon.md 没能打开"，但输出仍然符合规则
- 根因：`claude -p` 默认权限模式比交互会话严格，Skill 工具加载 SKILL.md（内嵌了要点）没问题，但模型主动 Read 附属文件被拒
- 解法：测试时加 `--allowedTools Read`；技能侧 SKILL.md 内嵌关键要点做兜底（即使读不到表也能干活）。交互会话无此问题
- 影响：headless/CI 场景下不是严格对照词表，属已知限制

### 问题 4：claude -p 的参数传递坑
- 现象：`claude -p --allowedTools "Read" '<中文prompt>'` 报错 "Input must be provided either through stdin or as a prompt argument"
- 根因：选项与位置参数的解析问题（Git Bash 引号+中文组合）
- 解法：**prompt 走 stdin 管道**：`echo '<prompt>' | claude -p --allowedTools Read`

### 问题 5：安装方式选错，后照 xcheck 惯例改为 JUNCTION（2026-09-17 傍晚修正）
- 现象：初版用 `git clone` 装到 `~/.claude/skills/renhua/`——和开发目录成了两份副本，改开发目录要 push+pull 才同步，易漂移
- 根因：没先看本机已有的技能安装惯例。实际上本机一大批技能（xcheck、mattpocock 系、orca-cli、research…）都是 **JUNCTION**：`~/.claude/skills/xcheck → C:\WorkSpace\agent\xcheck\xcheck`，开发目录是唯一真身
- 解法：删 clone 副本 → `~/.claude/skills/renhua` 用 junction 指向 `C:\WorkSpace\agent\renhua`。**注意 Git Bash 里 `cmd //c 'mklink /J ...'` 会因 MSYS 路径转换报"文件名、目录名或卷标语法不正确"——用 PowerShell 最稳**：
  ```powershell
  New-Item -ItemType Junction -Path 'C:\Users\allan716\.claude\skills\renhua' -Target 'C:\WorkSpace\agent\renhua'
  ```
- 教训：**装技能前先 `cmd //c dir C:\Users\allan716\.claude\skills` 看本机惯例**（JUNCTION=开发目录真身；普通 DIR=独立安装），别默认 clone

### 问题 6：CC Switch 潜在覆盖风险（观察中，未证实）
- 已知实锤：CC Switch 切供应商会全量覆盖 `settings.json`（抹 hooks，Orca/Ferryman 两案例）
- 未证实：是否会动 `~/.claude/CLAUDE.md`（第一档 RENHUA 块写在这里）
- 观察点：切完供应商 `grep RENHUA ~/.claude/CLAUDE.md`；若被抹，解法=把块塞进 CC Switch 的提示词模板
- 第三档（Stop 钩子）将来实施时**必须同步写进 CC Switch 模板**

## 5. 后续路线（三档自动化）

| 档 | 状态 | 说明 |
|---|---|---|
| 一：全局 CLAUDE.md 常驻规则 | ✅ 已实施（2026-09-17 17:40） | RENHUA_START/END 块，8 条精简规则，新会话自动生效 |
| 二：output style | ⬜ 备选 | 与 concise 等风格互斥，实施步骤见评估文档 |
| 三：Stop 钩子+词表扫描 | ⬜ 备选 | 最强硬；CC Switch 模板坑+误伤白名单+重试上限都要处理 |

三档详情、测试方法、回滚方法：`research_things/20260917_1742_renhua自动化三档方案与评估要点.md`（私有）。

## 6. 相关文档指针（均在 research_things，私有）

- 上游调研：`20260917_1554_AI文风人性化_去黑话开源工具调研.md`、`20260917_1619_AI对话通俗化_结论先行与表达顺序调研.md`
- 设计：`docs/superpowers/specs/2026-09-17-renhua-skill-design.md`
- 实施计划：`docs/superpowers/plans/2026-09-17-renhua-skill.md`

## 7. 2026-09-24 更新：防"改顺了但意思变了"（对齐 Humanizer-zh 上游修正）

**起因**：参考项目 [Humanizer-zh](https://github.com/op7418/Humanizer-zh) 2026-09-23 大修（基于 blader/humanizer v3.0.0 + PR #39），核心治"文章改顺了，意思却变了"：确定程度漂移（可能→确定）、机械删排比误杀真信息、机械拆真对比。对照检查发现本技能同源风险：jargon.md 表 2"三段排比→保留信息量最大的一条，删其余"字面上会误杀真三连；规则 6 锁事实不锁限定词。

**基线测试（改前，5 轮子代理实测）**：构造带限定词（部分/可能/偶尔/基本/还没统计）、真三连（构建、测试、上线）、真对比（写入路径而不是读取路径）的文本，跑激进档改写——Opus 无规则对照、Opus 带技能（易/难各一）、Sonnet、Haiku 共 5 轮，**5 测 5 过，本地不复现失败**。但 Haiku/Sonnet 在改动清单里明确写出"没按'三连删两条'处理——那条修法是给凑数排比的"：**字面指令是错的，正确结果全靠模型用规则 6/9 自觉覆盖它**。字面≠行为即缺陷：本机 CC Switch 会切非 Anthropic 模型（§4 问题 6），仓库公开、他人可能装到任何模型上，不能指望模型自觉兜底。

**改动（把字面改成模型实际在做的事）**：

1. rules.md 规则 6：锁事实加"限定词（可能/据称/部分/偶尔/基本/仅/超过/计划/正在）不许升级降级"+ 2 个反例（部分变全部、怀疑变结论）
2. rules.md 规则 8：加"判据是'有没有信息'，不是长得像不像 AI；真对比、三项真功能保留"
3. rules.md 规则 9：加作者有意排比的保留示例
4. jargon.md 表 2"三段排比"行：先分真假——各有信息全保留，凑数排比才删
5. jargon.md 表 2"不是 X 而是 Y"行：先分真假——都是事实就拆成两个并列陈述，纯抬语气才删 X
6. SKILL.md：锁事实清单加限定词、回读加"程度漂移也算变（可能→确定、部分→全部、计划→已完成）"

**验证（改后）**：双向关卡文本（真三连必须全留 + 假三连"创新、突破和全新的可能"必须删 + 限定词锁定 + 真对比两边都在）在 Haiku、Sonnet 各一轮，**全过且逐条引用新规则作为依据**——行为从"模型自觉覆盖错误指令"变成"按指令执行"，不再赌模型聪明程度。矫枉过正风险（新措辞导致什么都保留）同轮证伪：假三连照删。

**遗留**：全局 CLAUDE.md 的 RENHUA 常驻块（第一档，8 条精简版）未同步加限定词条目——该块管对话输出、不改写用户文本，影响面小；下次动第一档时从 rules.md 重新派生即可（架构决策 1 的派生关系）。（2026-09-24 晚已随 §8 重派生解决）

## 8. 2026-09-24 晚：i-have-adhd 共存实验 → 异构评审 → 裁决定案 → 落地

**起因**：用户计划 renhua 常驻与 [i-have-adhd](https://github.com/ayghri/i-have-adhd)（50.8k stars，纯输出风格规则技能，手动开启、会话内持续）长期同开。四处置碰撞点：首行（答案 vs 动作）、长度（解释+分层 vs 列表≤5）、结尾行动项、限定词（adhd 预检删含糊副词 vs 锁事实）。

**共存实验**（预注册 SPEC，36 轮 headless，A 基线/B 同开真技能/C 加裁决三条件，Sonnet/Haiku 全跑+Opus 抽查；账本 `adhd-coexist-test/`，已 git 排除）：
- B 下行为场景全部未爆发（首行给答案、术语保留、步骤零丢、正式文体保住）；真问题在**裁决权漂移**——S7 元问题下 sonnet/haiku 对"首句答案 vs 首行动作"给出相反裁决且各自自洽（一个按"漏动作代价大"、一个按"CLAUDE.md 优先于技能"）。C 注入裁决后三问统一且模型能指出依据。
- 边缘：2/4 轮（B/C 合计）"部分用户"半格弱化为"有/一些"（无极性翻转）；36 份输出与矩阵一一对应。
- **结论口径（经评审修正）**：单轮探索性通过；裁决三条有必要成文（S7 证据）；同开行为无伤害（B 证据）；"有效"仅验证到文本层（C 证明可读可引用，注入后问依据存在循环性，行为纠偏未测）；最终载体效力待验收。

**异构评审**（/xcheck --night，codex+kimi 两轮，账本 `.xcheck/20260924-215429/` 与 `20260924-220802/`）：首轮共识"骨架对、结论措辞超证据"4 条阻断（有效宣称/全部保住/单轮/机制等价断言）→ 自动修订一次（rev1：结论降格+撤回等价断言+新增落地验收两路径）→ 复审双 AGREE。插曲：codex 首轮称矩阵只有 34 份——查盘证伪（S4 行 6 份被数成 4）。

**定案（三处同源，rules.md 为源）**：① 全局常驻块重派生（9 条：限定词加"偶尔"+不弱化、分层加列表分组、新增"只改必要的+技术内容不动"，末尾加裁决行——顺带解决 §7 遗留漂移）② README"常驻（可选）"节（片段+裁决行+适用范围句）③ SKILL.md 边界情况节。另立 [ADR 0001](docs/adr/0001-常驻不走技能自动触发.md)（常驻走 CLAUDE.md 家族确定性配置，不走技能自动触发，不加 disable-model-invocation 硬开关）、CONTEXT.md 术语定稿。**裁决本体一字未改**：首行先给答案（可执行即动作，纯问答不硬造）；术语解释压一句保留、列表超 5 分组不丢内容；结尾行动项纯问答豁免。

**路径 a 验收**（落地时，7 轮 headless，`adhd-coexist-test/out-accept/`）：
- **全局块载体 4/4 通过**：S7 两模型均明确引用块内裁决行（"CLAUDE.md 里 2026-09-24 定的"）且三问一致；S1 首行答案、无硬凑行动项、术语在场。
- **README 片段**：与全局块同属 CLAUDE.md 家族（每会话加载），由同类机制覆盖（评审 F12 建议采纳，README 已注明）。
- **SKILL.md 载体未完全通过（1/2）**：`/renhua on`+adhd 会话中 haiku 三问与裁决完全一致；sonnet 问(1)被总原则"结构让 adhd"带偏成"动作优先"压过具体条款"首行先给答案"（引用原文无误、S1 行为符合，属元认知层偏差）。按预注册规则该载体回落路径 b：不删条款、降宣称——SKILL.md 边界行已注明"效力以全局常驻块为准"并把具体条款提到总原则之前（修呈现根因，语义未动）。

**适用范围**（README/ADR 固定文案）：共存验证在 Windows 10/Git Bash、i-have-adhd 纯技能形态（插件 hooks 未测）、单轮采样、含本人全局配置与本机记忆的会话环境下完成；跨环境/跨用户/插件形态/长期稳定性未验证。

**观察项**：S5×裁决 1 交互（正式文体首行）；限定词弱化若日常出现真删除（升级为删"可能"），补"限定词保护"裁决 4；可选低成本补强——C 变体（注入裁决但提问不提示其存在）、独立盲评抽查 out/S3_*.txt。
