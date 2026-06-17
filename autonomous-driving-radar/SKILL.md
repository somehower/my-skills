---
name: autonomous-driving-radar
description: 一键生成自动驾驶领域近期技术 + 产业动态雷达报告。覆盖 arXiv 论文、GitHub 热门项目、产业新闻、Reddit/知乎/HN 社区讨论、X (Twitter) 热帖。支持历史去重、按主题归类、生成结构化中文 Markdown 报告。Use when the user wants to get latest autonomous driving updates, asks for "AD radar", "自动驾驶雷达", "本周/本月自动驾驶进展", or similar intent about scanning recent autonomous driving developments.
license: MIT
---

# Autonomous Driving Radar (自动驾驶技术雷达)

## When to Use

激活本 Skill 当用户表达以下任一意图：

- "跑一下自动驾驶雷达" / "AD 雷达" / "自动驾驶雷达"
- "看看最近自动驾驶有什么新东西" / "最近自动驾驶有什么进展"
- "本周/本月自动驾驶进展"
- "autonomous driving radar" / "self-driving updates this week"
- 任何"扫一遍自动驾驶领域最新论文/项目/动态"的意图

## Default Configuration

| 项 | 默认值 | 说明 |
|---|---|---|
| 覆盖范围 | 自动驾驶全栈 | 感知 / 端到端 / VLA / 世界模型 / 扩散策略 / Occupancy / 规划 / 仿真 / 量产动态 |
| 时间窗 | 近 30 天 | 用户可覆盖（如"近 7 天"、"近 1 季度"） |
| 报告语言 | 中文 | 专有名词保留英文（diffusion、AR、VLA、BEV、Occupancy、world model 等） |
| 频率 | 每周 1 次 | 适合定时调度 |
| 去重档案 | 「AD雷达-已收录索引」文档 | 跑过的条目不再重复 |
| 报告归档 | 「AD雷达 YYYY-WWW」文档 | 每期独立文档 |
| 单期规模 | 15~25 条核心条目 | 防止过载 |

如用户未指定，按默认值执行；如指定则覆盖。

## Information Sources

### 1. 论文（arXiv + 顶会）
**关键词组合（可组合查询）**：
- `autonomous driving end-to-end`
- `vision language action driving` (VLA)
- `world model autonomous driving`
- `diffusion policy driving` / `diffusion planning`
- `BEV perception` / `occupancy prediction`
- `motion planning autonomous` / `trajectory prediction`

**顶会渠道**：CVPR / ICCV / NeurIPS / CoRL / ICRA / RSS 最近接收或公开预印。

### 2. 开源项目（GitHub）
- 搜索 `autonomous driving` + 近 30 天有显著动态
- 重点关注组织：`hustvl` / `OpenDriveLab` / `wayve-ai` / `NVlabs` / `nv-tlabs` / `Tsinghua-MARS-Lab` / `autonomousvision` / `valeoai` / `carla-simulator` / `autowarefoundation` / `QwenLM`
- 同步关注工具类：仿真器、benchmark、awesome 清单

### 3. 产业新闻
- **中文**：36氪、智驾网、车东西、新智驾、量子位、虎嗅
- **英文**：The Verge / Reuters / TechCrunch / Electrek / Bloomberg / CNBC 的 autonomous 板块
- **关注主体**：
  - 国内车企智驾：华为 ADS、小鹏 XNGP / 图灵、理想 AD Max / MindVLA、蔚来 NOP+、比亚迪天神之眼、问界
  - 国内 Robotaxi：百度 Apollo、文远知行、小马智行、Momenta、元戎启行、轻舟智航
  - 海外：Tesla FSD、Waymo、Cruise、Zoox、Wayve、Mobileye
  - 法规 / 量产 / 融资 / 重大事故 / 高管变动

### 4. 社区讨论
- Reddit: `r/SelfDrivingCars`、`r/MachineLearning`（含 autonomous tag）
- 知乎：自动驾驶 / 端到端 / VLA / 世界模型话题热榜
- HackerNews: 含 "autonomous driving" / "self-driving" / "FSD" 关键词

### 5. X (Twitter) — 路径 1：免授权搜索
通过搜索引擎抓 `site:x.com` / `site:twitter.com` / `site:nitter.net`：
- **KOL 账号**：@karpathy、@elonmusk、@DrJimFan、@Wayve_ai、@DriveLabs、@OpenDriveLab、@woven_toyota、@AIDRIVR、@WholeMarsBlog、@greentheonly
- **话题标签**：#FSD、#autonomousdriving、#VLA、#endtoend、#selfdriving、#Tesla
- 设 `searchTimeRange: month` 限定近期
- 如原帖不可访问，使用提到该帖的二手报道作为可验证锚点，并标注"转引自 XX"

## Execution Flow

### Step 1: 准备
1. **读取去重档案**：调用 `readDocument` 读「AD雷达-已收录索引」，提取已收录条目（标题 + URL）集合
2. **确定本期编号**：用 ISO 周数命名，如 `2026-W25`
3. **计算时间窗**：默认 `今天 - 30 天 ~ 今天`

### Step 2: 5 路并行检索（sub-agent 并发）
**关键**：在**同一个工具调用块**里同时发出 5 个 `callSubAgent`，并行而非串行。

| Agent | 职责 | 数量目标 |
|---|---|---|
| A | arXiv + 顶会论文 | 8-12 篇 |
| B | GitHub 热门项目 | 8-12 个 |
| C | 产业新闻（国内+海外） | 8-12 条 |
| D | 社区讨论（Reddit + 知乎 + HN） | 6-10 个话题 |
| E | X (Twitter) 热帖（路径 1） | 6-10 条 |

每个 sub-agent 的产出格式严格统一：
```
- 标题 / 项目名 / 主题
- 团队 / 作者 / 来源
- 一句话亮点（中文，专有名词保留英文）
- URL
- 发布日期
- 主题归类标签
```

### Step 3: 去重 + 合并
- 与去重档案逐条比对，**同标题（忽略大小写空白）或同 URL 视为重复**，丢弃
- 按主题归类：端到端 / VLA / 世界模型 / Diffusion / BEV-Occupancy / 规划 / 仿真 / 量产 / 法规

### Step 4: 生成正式报告
调用 `createDocument` 创建新文档「AD雷达 YYYY-WWW (MM-DD)」，按下方模板填充。

### Step 5: 更新去重索引
调用 `replaceDocumentContent` 把本期所有条目追加到「AD雷达-已收录索引」对应分区，并在末尾追加历史期次链接。

### Step 6: 交付总结
向用户输出：
1. 三个文档链接（报告 / 索引 / 本 Skill）
2. 本期 Top 3 看点速览
3. 收录条目统计
4. 询问反馈（粒度 / 分类 / 是否要深读某篇）

## Report Template

```markdown
# 🚗 自动驾驶技术雷达 · YYYY 第 WW 周
> **采集时间窗**：YYYY-MM-DD ~ YYYY-MM-DD（近 30 天）
> **信息源**：arXiv + GitHub + 产业媒体 + Reddit/知乎/HN + X (Twitter)
> **生成时间**：YYYY-MM-DD
> **已剔除历史报告 N 条重复条目**

---

## 🔥 本期 Top 3 看点
1. ...（一句话，点出最具影响力的事件）
2. ...
3. ...

---

## 📄 重点论文

### 主题：端到端 / E2E
- **English Title**（团队） — 中文一句话亮点。[arXiv](url)（YYYY-MM-DD）

### 主题：VLA / 视觉语言动作
- ...

### 主题：World Model / 世界模型
- ...

### 主题：Diffusion / 规划
- ...

### 主题：BEV / Occupancy / 感知
- ...

### 主题：仿真 / 数据集
- ...

---

## 💻 热门开源项目

### 端到端 / VLA / Diffusion
- **[owner/repo](url)** — 中文一句话定位 + 近期动态

### World Model / 数据生成
- ...

### VLA / 工具链
- ...

---

## 📰 产业动态

### 🇨🇳 国内
| 日期 | 事件 | 标签 |
|---|---|---|
| MM-DD | 标题 + 一句话核心 | 量产/法规/融资/技术发布 |

### 🌍 海外
| 日期 | 事件 | 标签 |
|---|---|---|
| MM-DD | 标题 + 一句话核心 | 量产/法规/融资/技术发布 |

---

## 🐦 X / 社区热议

- **@账号**：一句话观点（中文转述） — 日期 [link]

### 社区主线话题
1. 主题标题 — 核心争议 2-3 句
2. ...

---

## 🧭 趋势观察

**本期关键词**：`Tag1` · `Tag2` · `Tag3`

一句话总结（150 字内）：...

**值得持续追踪的三条线**：
- ...
- ...
- ...

---

## 📚 延伸阅读
- ...
```

## Deduplication Index Schema

「AD雷达-已收录索引」文档结构：

```markdown
# 自动驾驶雷达 - 已收录索引

> 去重规则：同标题（忽略大小写空白）或同 URL 视为重复

## 📄 已收录论文（arXiv / 顶会）
### YYYY-WWW
- [WWW] 标题 | 团队 | arXiv ID 或 URL

## 💻 已收录开源项目（GitHub）
### YYYY-WWW
- [WWW] owner/repo | 一句话定位

## 📰 已收录产业新闻
### YYYY-WWW 国内
- [WWW] 标题 (日期)
### YYYY-WWW 海外
- [WWW] 标题 (日期)

## 🐦 已收录 X 热帖 / 讨论
### YYYY-WWW
- [WWW] @作者: 主题摘要

## 🧭 历史期次
- **YYYY-WWW (YYYY-MM-DD)**: [报告链接](url) — 简短描述
```

## Quality Rules

- ❌ 单条结果如**来源不可靠（无明确作者/链接）则丢弃**，宁缺毋滥
- ✅ 中文报告，但**论文标题保留英文原文**，便于检索
- ✅ 每期控制在 **15~25 条核心条目**，避免过载
- ✅ X 帖子优先选**转发 / 点赞高**的（搜索结果摘要里能体现）
- ✅ 如某类信息源本期无新增，明确写"本期无新增"，**不要硬凑**
- ✅ 时间窗严格遵守，超出的丢弃；如某类信息确实稀缺可放宽到 45 天，并在条目后**标注实际日期**
- ✅ 不重复推送历史期次已收录的条目（标题 + URL 双重比对）

## Notes for Implementation

- **必须用 sub-agent 并行**：5 路检索同时发起，节省时间
- **去重档案是 source of truth**：每次执行前必读，执行后必写
- **跨期对比**：从第 2 期开始，可以在"趋势观察"里点出与上期对比的变化
- **定时调度**：建议每周一早上 9:00 触发，结合 cron 工具
- **路径升级**：如果用户后续提供 X API key，可从路径 1（搜索引擎）升级到路径 2（X API），获得更精细的时间线和互动数据
