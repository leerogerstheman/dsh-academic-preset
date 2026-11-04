# dsh-academic-preset — 学术模式

给 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（DSH）加一个**面向大学学习与作业**的 Agent 预设：注册 id 为 `academic`，在模式选择里显示为「**学术模式**」。

内置的四个模式（标准 / PTC / 极简 / 创造）都以编程为中心。这个预设换掉的是**人格**和**工具集**：

- 它是学习搭档，不是编码助手：工作区里的文件对它是笔记、课件、数据、草稿和成品报告。
- 默认「讲解档」：给答案，也给推导、错因和一道变式题；说「只要答案」才收窄，说「先别给答案」就转苏格拉底式引导。
- 不代写、不编文献：查不到的写「待核实」，整篇代写的请求会退回到结构 + 逐段批改。

## 它包含什么

声明本身就是一份普通的 Cordis 补丁（`cordis.patch.yml`），只插入一条 `@deepseek-ai/dsh-agent-preset` 行：

| 插件 | 作用 |
|---|---|
| `dsh-persona` | 学术人格：四档答题、引用规范（GB/T 7714 / APA）、论文与综述结构、实验报告骨架、统计报告格式、中医药与药学专章、复习与 PPT 方法、AI 使用声明 |
| `dsh-agent-instructions` | 读取工作区 `AGENTS.md` 之类的课程约定 |
| `dsh-tool-fs` | `read` / `write` / `edit` — 笔记、草稿、数据 |
| `dsh-tool-fs-search` | `glob` / `grep` — 在课件与旧作业里找公式、定义、之前的解题套路 |
| `dsh-tool-web` | `web_search` / `web_fetch` — 查文献、查药典条目、核对数据与指南版本 |
| `dsh-tool-ask-user` | 把字数、格式、引用体系、截止时间这些要求问清楚 |
| `dsh-skill-filesystem` + `dsh-tool-skill` | 工作区技能；Desktop 端另有宿主的 Office 技能，可导 Word / Excel / PPT / PDF |
| `dsh-tool-todo` | 论文分章、复习分天的进度 |
| `dsh-compaction-basic` + `dsh-command-compact` + `dsh-compaction-tool-result-pruner` | 长课件与长文献的上下文压缩，`/compact` 手动压缩 |
| `dsh-tool-pwsh` / `dsh-tool-bash` | 跑 Python 数据脚本、格式转换、PDF 转文本 |
| `dsh-time-context` | 当前时间与耗时（排复习计划、算 deadline 用得上） |
| `dsh-tool-present` | 成品作为文件交付 |

**刻意没有包含**：plugin-manager、schedule、goal、plan-mode、子代理、workflow、jobs —— 这些是编程向或长任务向的重装备，学习/作业场景用不上，留着只会稀释模式菜单和工具目录。

## 答题档位

| 档位 | 触发 | 行为 |
|---|---|---|
| 讲解档（默认） | 直接问题目 | 结论 → 分步推导 → 常见错因 → 一道同型变式题 |
| 直给档 | 「只要答案」「赶时间」 | 结论 + 一步关键理由 |
| 引导档 | 「别直接给答案」「先给提示」 | 先问卡点，再按「方向 → 关键步骤 → 完整解」三级放 |
| 考试模拟 | 「模拟考」 | 限时整套题，收卷后按评分点给分 |

档位在单轮内保持一致，切换需要明说。

## 安装

### 方式一：让 DSH 自己装（推荐）

在**创造模式（Creator）**的会话里说：

> 安装 https://github.com/leerogerstheman/dsh-academic-preset 这个 bundle

### 方式二：手动

在 DSH 里用 `plugin_manager` 工具：

```json
{ "action": "install_bundle", "target": "git+https://github.com/leerogerstheman/dsh-academic-preset.git" }
```

本地开发时 `target` 直接给目录的绝对路径即可（会以 `link:` 方式挂进 profile，改完重载即生效）。

安装完成后，`plugin_manager` 的 `list_plugins` 里应当出现 `preset-academic`，且 `fiberPhase: active`。

## 使用

1. 新建任务 → 点模式按钮 → 选「**学术模式**」。
2. 想设为默认：**设置 → Agent 预设 → 学术模式** 卡片上点一下（设为新任务默认）。
3. 已存在的会话不会中途切换模式——模式在任务开始时确定。

## 自定义

- **改人格**：编辑 `cordis.patch.yml` 里 `persona.config.prefix` 的文本，重载 bundle 后生效。
- **加工具**：往 `plugins` 列表里加一行，`name` 用 `@deepseek-ai/dsh-*` 包名；可安装插件的完整清单见 `cordis-composition-reference` 技能的 `references/packages.md`。
- **换 id / 显示名**：改 `config.id`、`config.name`、`config.description`、`config.order`。
- **想加长流程**（比如固定的综述模板、题库生成脚本）：在 `$DSH_HOME/skills` 或工作区 `.dsh/skills` 里建一个 `SKILL.md` 目录即可被这个预设的 `skill-filesystem` 发现。

## 已知限制

- **PDF 没有内置解析器**。人格里给的路线是：pwsh 调 Word COM 另存 txt，或用 `pdftotext`；都没有时请用户贴段落。要彻底解决，需要另装一个带 PDF 解析的插件或技能。
- **Office 技能来自宿主**。Desktop 端由 `dsh-desktop-host` 在宿主层挂载 `dsh-skill-office` 与 `load_workspace_dependencies`，本预设不重复挂载；纯 CLI / 无宿主的部署里这七个 office 技能不会出现，需要自己补一行 `@deepseek-ai/dsh-skill-office`（配置 `assetRoot` / `node` / `cli`）。
- **不做整篇代写**是设计选择，不是能力限制。急用可以直接说「直给档」，或临时切回标准模式。

## 踩坑记录

`@deepseek-ai/dsh-tool-fs-search` 的 `sampleOverCapGlobResults` 是**必填字段**（没有默认值）。漏掉它不会报错到终端，而是让整个预设被判为 broken：设置页卡片带红色「加载失败」，新建任务页的模式菜单里直接看不到这个预设（菜单只列健康的预设）。诊断文本在卡片悬停可见。

预设是**即时激活**的：装完不用重启，但**已有会话**保持它启动时的插件版本，新建一个会话才能看到新工具集。

## 许可

MIT
