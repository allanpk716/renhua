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

**遗留**：全局 CLAUDE.md 的 RENHUA 常驻块（第一档，8 条精简版）未同步加限定词条目——该块管对话输出、不改写用户文本，影响面小；下次动第一档时从 rules.md 重新派生即可（架构决策 1 的派生关系）。
