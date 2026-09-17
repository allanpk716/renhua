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
