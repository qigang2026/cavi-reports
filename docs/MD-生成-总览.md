# MD 生成总览

**文档版本**：v2.0
**创建日期**：2026-08-27
**修订日期**：2026-08-27（v2.0：合并"字段表/数据源/质量保证/SKILL 契约"4 份技术文档为一站式宣讲；受众=技术宣讲/PM/研发/QA/集成方）
**目的**：一站式讲清"AI 怎么把车型数据变成 MD 报告"——业务背景 + 业务流程 + 字段 + 数据源 + 质量 + 接口
**范围**：只覆盖 **MD 文档的生成**；HTML 渲染、埋点、前端样式**不在本文档范围**

> **本文档是"技术宣讲版"，业务背景看 [`docs/AUTOCAVA 购车报告需求.md`](./AUTOCAVA%20购车报告需求.md)**。

---

## 第一部分：业务与原则

### 1.1 业务目标

PM 已确认 4 个目标（v1.2 强化第 4 条）：

| 目标 | 验收标准 |
|---|---|
| 文档标准化 | AI 生成 10+ 车型 MD 文档，格式一致率 > 95% |
| model 体系建立 | 每个车型有唯一 model，支持埋点追溯 |
| Skill 规范化 | cavi-guide-gen Skill 生成文档成功率 > 99% |
| **数据可靠**（v1.2 强化）| **AI 字段合规率 100%**（AI 不许篡改系统数字）|

### 1.2 标杆车系

> **当前业务范围：1 个标杆车型 Versa 2026（series_id=356）**。

| 维度 | 当前状态 | 未来扩展 |
|------|---------|---------|
| 车型 | Versa 2026 | 暂不考虑（标杆跑通后再议）|
| 市场 | MX | 暂不考虑 |
| 语言 | 西语 | 暂不考虑 |
| 能源 | 燃油 | 暂不考虑 |
| 展现样式 | 唯一：`cavi-report-v1` | 暂不考虑 |

### 1.3 AI 工作模式（核心原则）

> **PM 已确认：系统给原始数据，AI 负责润色 + 翻译。AI 不得编造数字或事实。**

| 类型 | 谁负责 | 例子 |
|------|--------|------|
| **系统字段**（结构化）| 后端系统 | 价格、版本、CAVI 分、销量、评论原文 |
| **AI 润色**（自然语言）| AI | 副标题、洞察、推荐语（≤ 30 字）|
| **AI 翻译**（多语言时）| AI | 把系统给的英文版翻译成西语 |

**AI 不可触碰的硬约束**（违反 = E203 = P0）：

```
❌ 不许编造价格 / 销量 / 评分 / 评论
❌ 不许把 "3.9" 改成 "4.0" 让数据好看
❌ 不许"润色"时篡改原始数据含义
✅ 可以润色：把 "118 HP 5.3L/100km" 变成 "动力够用且省油"
✅ 可以翻译：把英文 raw 数据译成西语
✅ 可以拼装：把系统分 + 评论拼成"一句洞察"
```

### 1.4 业务流程（端到端）

```
┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ 触发场景 │ →  │ 后端接入  │ →  │ 4 数据源  │ →  │ AI Agent │ →  │ MD 文件   │
│ S1-S4   │    │ 接收请求  │    │ 拉取数据  │    │ 生成+校验 │    │ 落地      │
└─────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
  用户/          series_id       缓存+重试     标准模板+       reports/es/
  调度/         参数校验       兜底默认值     字段表+LLM      nissan-versa-
  人工                          (E101-E103)     5步校验         2026.md
                                                              +数据快照
```

**4 种业务场景**：
- **S1** 用户主动（车系页点击"获取报告"）
- **S2** 数据变更触发（4 数据源任一字段更新）
- **S3** 定时刷新（每日 cron）
- **S4** 人工强制（PM/运营手动）

**5 步详解**：

| 步骤 | 谁 | 输入 | 输出 | 错误 |
|------|---|------|------|------|
| **① 触发** | 用户/调度/人工 | — | `series_id` | E001/E002 |
| **② 接入** | 后端 | 参数 | 数据快照请求 | E001 |
| **③ 数据** | 4 数据源 | series_id | 完整字段表 | E101/E102/E103 |
| **④ 生成** | AI Agent | 字段表 | MD 文档 | E201/E202/E203/E301/E302 |
| **⑤ 落地** | 后端 | MD 文本 | 报告上线 | E401/E500 |

> 详细业务流程、时序图、关键节点说明见 [`docs/AUTOCAVA 购车报告需求.md`](./AUTOCAVA%20购车报告需求.md) §二。

### 1.5 4 个真实数据源

PM 已确认 pc-v1.html 实际数据都来自以下 **4 个已存在**的系统：

| # | 数据源 | 提供字段 |
|---|--------|---------|
| 1 | **CAVI 内部 API** | 车系 / 版本（6 个）/ 规格 / MSRP / official / 月供估算 |
| 2 | **CAVI 评分系统** | 综合评分 4.6 / 推荐率 89% / 4 维度评分 |
| 3 | **销售系统** | 月销量 / 排名 / 评论人数 |
| 4 | **金融 API** | BBVA 月供 / Nissan 厂家金融 / 交换补贴 |

> AI Agent 不关心 API endpoint / TTL / 鉴权——后端负责接入。

---

## 第二部分：字段表（权威）

> **本节是字段的权威定义。AI / 研发 / QA 都要遵守。**

### 2.1 AI 不可触碰的字段清单（硬约束）

**规则**：以下字段值由系统提供，AI **只能原样填入**，**不得修改、不得润色**：

| 字段组 | 原因 |
|--------|------|
| 价格 / MSRP / official | 错一位数 → 客户被骗 / 决策错误 |
| 月供 / 利率 | 同上 |
| 销量 / 排名 | 客观数据 |
| CAVI 评分（综合 + 4 维度）| 系统算法输出 |
| 评论原文 | 真实用户声音 |
| 质保年限 / 里程 | 厂家承诺 |
| 任何数字、日期、版本号 | — |

### 2.2 Frontmatter 字段

文件顶部 YAML 段，**所有字段必填**（除非标注"可选"）。

| Key | 类型 | 来源 | 说明 | 兜底默认值 |
|-----|------|------|------|----------|
| `title` | string | AI | 车名 + 年款 | `Nissan {model} {year} 购车指南` |
| `series_id` | int | **系统** | CAVI 车系 ID = 356 | **必填，无兜底** |
| `model` | string | **系统** | 车型名 = `nissan-versa` | **必填，无兜底** |
| `year` | int | **系统** | 年款 = 2026 | **必填，无兜底** |
| `market` | enum | **系统** | `MX` | **必填，无兜底** |
| `lang` | enum | **系统** | `es` | **必填，无兜底** |
| `energy_type` | enum | **系统** | `燃油` | **必填，无兜底** |
| `body_type` | string | **系统** | `Sedán` | **必填，无兜底** |
| `device` | enum | **系统** | `pc` / `h5` | `pc` |
| `generated_at` | date | **系统** | ISO 8601 | 自动填 |
| `cavi_score` | decimal | **系统** | 0-5 | **必填**（AI 不可改）|
| `cavi_recommend` | int | **系统** | 0-100 | **必填**（AI 不可改）|
| `cavi_verdict` | string | **混合** | 系统 raw + AI 润色 ≤ 50 字 | `"Sin datos suficientes"` |
| `spec_strip` | list<3> | **系统** | 首屏 3 核心数据 | `[]` |
| `key_specs` | list<3-4> | **系统** | 卖点章 3-4 核心数据 | `[]` |
| `annual_cost` | object | **系统** | 年度用车成本 | `null` |
| `finance_cards` | object | **系统** | 4 张金融卡 | `null` |
| `competitors` | list<4> | **系统** | 4 张并排卡 | `[]` |
| `next_cards` | list<3> | **系统** | 3 张行动卡 | `[]` |

### 2.3 8 段结构 + 字段定义

> **章节结构与前端 pc-v1.html 完全对应**。西语 `block-eyebrow` 字面锁定。

#### 段 0 · 首屏（Hero + Spec strip + Trim selector + Compare bar）

| Key | 类型 | 来源 | 说明 |
|-----|------|------|------|
| `hero.eyebrow` | string | **系统** | `EXPERTO · CAVI` |
| `hero.title` | string | **系统** | `<model_name> {year}` |
| `hero.subtitle` | string | **AI** | ≤ 50 字 |
| `hero.desc` | text | **混合** | 系统给关键数 + AI 拼装 50-80 字 |
| `hero.image_url` | url | **系统** | 车图 CDN |
| `hero.cavi_score` | decimal | **系统** | 与 frontmatter 一致 |
| `hero.monthly_sales` | int | **系统** | 月销量 |
| `hero.rank_text` | string | **混合** | 系统数据 + AI 拼装 |

**Spec strip**（3 核心数据，固定 3 条）：

| Key | 类型 | 来源 | 例子 |
|-----|------|------|------|
| `label` | string | **系统** | `Mejor consumo` |
| `value` | string | **系统** | `18.81` |
| `unit` | string | **系统** | `km/L` |
| `sub` | string | **系统** | `SENSE MT` |
| `icon` | enum | **系统** | `fuel` / `trunk` / `trending` |

**Versions**（6 个版本，恰好 1 个 `is_recommended=true`）：

| Key | 类型 | 来源 |
|-----|------|------|
| `code` | string | **系统** |
| `name` | string | **系统** |
| `msrp` | int | **系统** |
| `official` | int | **系统** |
| `monthly` | int | **系统** |
| `is_recommended` | bool | **系统** |

**Hero CTA**：

| Key | 类型 | 来源 |
|-----|------|------|
| `text` | string | **系统**（`Consultar planes`）|
| `url` | url | **系统**（finance apply 链接）|
| `style` | enum | **系统**（`yellow-block`）|

#### 段 02 · PRECIO Y FINANCIAMIENTO

> block-eyebrow 字面: `02 · PRECIO Y FINANCIAMIENTO`（**字面锁定**）

**`finance_cards.main`（主价卡·深色）**：

| Key | 类型 | 来源 |
|-----|------|------|
| `version_name` | string | **系统**（如 `ADVANCE CVT`）|
| `price` | int | **系统**（整数本地货币）|
| `currency` | enum | **系统**（`MXN`）|
| `source` | string | **系统** |
| `price_base` | int | **系统** |
| `bonus_trade_in` | int | **系统**（负数）|
| `price_effective` | int | 计算 = base + bonus |

**`finance_cards.bank`（BBVA 36 个月）**：`bank_name` / `term` / `monthly` / `rate` / `loan_ratio` / `down_payment` / `loan_amount` / `total_interest`（全部**系统**）

**`finance_cards.factory`（Nissan 厂家 48 个月）**：`provider` / `term` / `monthly` / `rate` / `tag` / `plan` / `cat` / `down_payment`（全部**系统**）

**`finance_cards.trade_in`（交换补贴）**：`amount` / `condition` / `vigencia` / `applies_to` / `extra_bono`（全部**系统**）

**`finance_cta_bar`（章节内 cta）**：

| Key | 类型 | 来源 |
|-----|------|------|
| `title` | string | **AI** |
| `subtitle` | string | **AI** |
| `button_text` | string | **系统** |
| `url` | url | **系统** |

#### 段 03 · VENTAJAS CLAVE

> block-eyebrow: `03 · VENTAJAS CLAVE`

**`key_specs`**（3-4 条核心数据，**系统**给）+ **`insight` 段落**（**AI**润色 30-100 字）

#### 段 04 · CAVI · RESEÑAS

> block-eyebrow: `04 · CAVI · RESEÑAS`

**`cavi_dimensions`（4 维度锁定）**：

```yaml
cavi_dimensions:
  - { slug: cajuela,   label: "Cajuela",   value: 4.8, stars: 5 }
  - { slug: consumo,   label: "Consumo",   value: 4.6, stars: 5 }
  - { slug: seguridad, label: "Seguridad", value: 4.5, stars: 4 }
  - { slug: ruido,     label: "Ruido",     value: 3.9, stars: 3, is_weakness: true }
```

| Key | 类型 | 来源 |
|-----|------|------|
| `slug` | enum | **系统** |
| `label` | string | **系统** |
| `value` | decimal | **系统** 0-5 |
| `stars` | int | 计算 = round(value) |
| `is_weakness` | bool | **系统** |

**`featured_reviews`（2-3 条精选）**：

| Key | 类型 | 来源 |
|-----|------|------|
| `stars` | int | **系统** |
| `content` | text | **系统** |
| `version` | string | **系统** |
| `mileage_km` | int | **系统** |
| `location` | string | **系统** |

**`cavi_meta`**：`review_count` / `monthly_sales` / `rank_text`（**系统**）

#### 段 05 · COSTO DE USO

> block-eyebrow: `05 · COSTO DE USO`

```yaml
annual_cost:
  total: 17300
  currency: "MXN"
  items:
    - { slug: fuel,        amount: 8000 }
    - { slug: insurance,   amount: 4000 }
    - { slug: maintenance, amount: 2500 }
    - { slug: depreciation,amount: 2000 }
    - { slug: tax,         amount: 800  }
  assumptions: { annual_km: 15000 }
  saving_tips:
    - "Mantén presión de neumáticos en 32 PSI — ahorra 3% combustible"
```

| Key | 类型 | 来源 |
|-----|------|------|
| `total` | int | **系统** |
| `items[].amount` | int | **系统**（5 项必含 fuel/insurance/maintenance/depreciation/tax）|
| `assumptions.annual_km` | int | **系统** |
| `saving_tips[]` | list | **AI**（不涉数字）|

#### 段 06 · GARANTÍA Y SERVICIO

> block-eyebrow: `06 · GARANTÍA Y SERVICIO`

| Key | 类型 | 来源 |
|-----|------|------|
| `warranty.years` | int | **系统** |
| `warranty.mileage_km` | int | **系统** |
| `warranty.details` | text | **系统** |
| `maintenance.included_count` | int | **系统** |
| `maintenance.mileage_km` | int | **系统** |
| `service_network.count` | int | **系统** |
| `service_network.description` | text | **系统** |
| `support_features[]` | list | **系统** |

#### 段 07 · COMPETENCIA

> block-eyebrow: `07 · COMPETENCIA`

**结构固定 4 条**：1 条 `tag=SELF`（本车）+ 3 条竞品（`ALTERNATIVA` / `PREMIUM`）

| Key | 类型 | 来源 |
|-----|------|------|
| `tag` | enum | **系统**（`SELF`/`ALTERNATIVA`/`PREMIUM`）|
| `name` | string | **系统** |
| `price` | int | **系统** |
| `image_url` | url | **系统** |
| `pros` | text | **AI** ≤ 60 字（**不许编数字**）|
| `cons` | text | **AI** ≤ 60 字（**不许编数字**）|

#### 段 08 · SIGUIENTE PASO

> block-eyebrow: `08 · SIGUIENTE PASO`

| Key | 类型 | 来源 |
|-----|------|------|
| `type` | enum | **系统**（`whatsapp`/`test_drive`/`cavi_ai`）|
| `label` | string | **系统** |
| `sub` | text | **系统** |
| `url` | url | **系统** |

**3 张行动卡 type 必须各覆盖 1 个，不重复**。

### 2.4 关键约束

| 约束 | 说明 |
|------|------|
| **段 02..08 必须有** | 8 段是结构骨架，缺一段视为生成失败 |
| **西语 eyebrow 字面锁定** | `02 · PRECIO Y FINANCIAMIENTO` 等 7 个字符串**字面不能改** |
| **AI 不得编数字** | 价格/销量/评分/里程等系统字段 AI 不可触碰 |
| **跨字段一致** | cavi_score 在 frontmatter / Hero / 段 04 必须一致 |
| **价格是整数** | 不要带 `$`/`MXN`/`，` 千分位（模板渲染时加）|
| **slug 命名** | `snake_case` |

---

## 第三部分：数据获取与可靠性

### 3.1 字段来源映射

| 字段组 | 来源 | AI 能否改 |
|--------|------|----------|
| `title` | AI | 可 |
| `series_id` / `model` / `year` | 系统 | 否 |
| `cavi_score` | 系统（CAVI 评分系统）| 否 |
| `cavi_dimensions[].value` | 系统 | 否 |
| `featured_reviews[].content` | 系统 | 否 |
| `spec_strip[]` | 系统 | 否 |
| `versions[]`（6 个）| 系统 | 否 |
| `finance_cards.*.price` / `monthly` | 系统（金融 API）| 否 |
| `annual_cost.total` | 系统 | 否 |
| `warranty.*` | 系统 | 否 |
| `competitors[].name` / `price` | 系统 | 否 |
| `competitors[].pros` / `cons` | **AI** | **是**（但不许编数字）|
| `hero.subtitle` | **AI** | 是 |
| `hero.desc` | **混合** | 数字不变 |
| `hero.rank_text` | **混合** | 数字不变 |
| `finance_cta_bar.title` / `subtitle` | **AI** | 是 |
| `annual_cost.saving_tips[]` | **AI** | 是（不涉数字）|
| `selling_points.insight` | **AI** | 是 |

### 3.2 字段兜底默认值

> **兜底 ≠ AI 编。** 兜底是"系统缺失时显示'暂无'"，AI 不得补。

| 字段缺失 | 兜底行为 |
|---------|---------|
| `cavi_score` | `null` + 段 04 显示"暂无评分" |
| `cavi_recommend` | `null` + 不显示推荐率 |
| `cavi_verdict` | `"Sin datos suficientes"` |
| `featured_reviews` | `[]` + 段 04 隐藏评论卡片 |
| `competitors` | `[]` + 段 07 显示"暂无竞品数据" |
| `versions[].monthly` | `0` + UI 显示"咨询" |
| `finance_cards.bank` | `null` + 显示"联系经销商" |
| `finance_cards.factory` | `null` + 显示"联系经销商" |
| `bonus_trade_in` | `0` |
| `monthly_sales` | `null` + Hero 不引用 |
| `warranty.years` | `3`（行业默认，**系统兜底不是 AI 编**）|
| `annual_cost.total` | `null` + 段 05 显示"暂无数据" |
| `hero.image_url` | `null` + 隐藏车图 |

**兜底原则**：
- **结构数据**（价格/版本/CAVI 总分）缺失 → **不生成报告**，触发告警
- **非结构数据**（描述/洞察/推荐语）缺失 → 显示"暂无"占位
- **影响购买决策**（价格/版本/CAVI）缺失 → **必须告警**

### 3.3 数据新鲜度

| 等级 | 字段 | 缓存 TTL |
|------|------|---------|
| **实时** | 价格、库存、利率 | ≤ 1h |
| **准实时** | 月销量、CAVI 评分 | ≤ 24h |
| **周级** | 质保、评论精选、服务网络 | ≤ 7d |
| **月级** | 销量排名 | ≤ 30d |
| **静态** | 模板常量、章节文案、Logo | 永不过期 |

### 3.4 数据快照（必须保存）

每次生成 MD 时，保存"数据快照"用于回放：

```yaml
# data_snapshot.yaml
generated_at: 2026-08-27T14:30:00Z
series_id: 356
data_version: v20260827-1430
sources:
  cavi_score: { value: 4.6, source: "cavi-rating-system", fetched_at: "...", cache_ttl: 86400 }
  versions: { source: "cavi-internal-api", fetched_at: "...", count: 6 }
  # ...
```

**用途**：
- 调试："为什么这份报告价格不对？"
- 回放：同一快照 → 同一 MD 哈希
- 审计：3 个月后查历史数据

### 3.5 失败处理

**单字段级重试**：

```
读取字段 X → try 1 (立即) → 失败
          → try 2 (200ms 后) → 失败
          → try 3 (1000ms 后) → 失败
          → 兜底默认值
```

**整体失败**：3+ 个核心字段同时失败 → 30s 后重试 1 次，仍失败则**不生成报告 + 告警**。

**部分失败处理**：

| 失败字段数 | 处理 |
|-----------|------|
| 0 | 正常生成 |
| 1-2 个非核心 | 正常生成 + 字段标"暂无" |
| 1-2 个核心 | 正常生成 + 告警（PM 关注）|
| 3+ 个核心 | 阻塞生成 + 立即告警 |
| 价格/版本缺失 | 阻塞 + 告警（**不允许无价格报告上线**）|

---

## 第四部分：质量保证（5 步校验）

### 4.1 验收流程

```
生成 MD
   ↓
[自动校验 ① 字段完整性]
   ↓ pass
[自动校验 ② 跨字段一致性]  ← 15 条规则
   ↓ pass
[自动校验 ③ 结构完整性]    ← 8 段 + 西语 eyebrow 字面
   ↓ pass
[自动校验 ④ AI 字段合规]   ← AI 不得篡改数字
   ↓ pass
[人工抽样审核 ⑤ 5%]
   ↓ pass
发布 / 失败回滚
```

**任何一步失败 → 阻塞发布**。

### 4.2 跨字段一致性 15 条

| ID | 规则 | 失败处理 |
|----|------|---------|
| C1 | **CAVI 总分唯一**（frontmatter = Hero = 段 04）| **阻塞** |
| C2 | **CAVI 推荐率唯一** | **阻塞** |
| C3 | **推荐版本唯一**（1 个 is_recommended=true）| **阻塞** |
| C4 | **主价卡 = 推荐版本 MSRP** | **阻塞** |
| C5 | **主价卡版本名 = 推荐版本名** | **阻塞** |
| C6 | **4 维度数量 = 4** | **阻塞** |
| C7 | **spec_strip 数量 = 3** | **阻塞** |
| C8 | **key_specs 数量 3-4** | **阻塞** |
| C9 | **competitors 数量 = 4** | **阻塞** |
| C10 | **next_cards 数量 = 3** | **阻塞** |
| C11 | **竞品首条是本车**（`competitors[0].tag = "SELF"`）| **阻塞** |
| C12 | **next_cards type 不重** | **阻塞** |
| C13 | **1 个 is_weakness** | 警告 |
| C14 | **annual_cost 占比之和 ≈ 100** | 警告 |
| C15 | **价格非负** | **阻塞** |

### 4.3 AI 字段合规校验（v1.2 关键）

> **AI 不得篡改系统原始数字**。

**三类 AI 字段**：

| 类型 | AI 权限 | 校验 |
|------|--------|------|
| **纯 AI 生成** | 自由写 | 仅长度 + 风格校验 |
| **混合字段**（系统值 + AI 拼装）| 可改字面，**不许改数字** | 数字溯源校验 |
| **AI 不可动** | 完全不动 | 字面一致性校验 |

**混合字段数字校验规则**：

| 规则 | 说明 |
|------|------|
| 数字必须出现 | 系统给的数字必须原样出现在 AI 输出中 |
| 数字不许变 | 数字字面值必须跟系统一致（"374,990"不许改成"374000"或"375k"）|
| 可增加语义 | AI 可加修饰词（"Top #1"）但**不许改数字**|
| 可改格式 | "MXN 374,990"和"374,990 MXN"都接受（顺序可调）|

**示例**：

| 系统给 | AI 输出 | 结果 |
|--------|--------|------|
| `monthly_sales=7486, cavi_score=4.6` | `"7,486 ventas/mes, CAVI 4.6/5"` | ✅ 通过 |
| `monthly_sales=7486, cavi_score=4.6` | `"7,500 ventas/mes, CAVI 4.6"` | ❌ 阻塞（7486→7500）|
| `monthly_sales=7486, cavi_score=4.6` | `"Top ventas, CAVI 4.6"` | ✅ 通过（省略数字）|

### 4.4 8 段结构 + 西语 eyebrow

```python
SPANISH_EYEBROWS = {
    '02': '02 · PRECIO Y FINANCIAMIENTO',
    '03': '03 · VENTAJAS CLAVE',
    '04': '04 · CAVI · RESEÑAS',
    '05': '05 · COSTO DE USO',
    '06': '06 · GARANTÍA Y SERVICIO',
    '07': '07 · COMPETENCIA',
    '08': '08 · SIGUIENTE PASO',
}
```

**校验**：模板渲染时 eyebrow 必须从 `SPANISH_EYEBROWS` 取。**任何变体 = 失败**。

**变量残留检测**：MD 中任何 `{{xxx}}` 残留 = 失败（AI 没替换）。

### 4.5 风格合规

| 字段 | 字数限制 |
|------|---------|
| `title` | ≤ 100 |
| `hero.subtitle` | ≤ 50 |
| `hero.desc` | 30-100 |
| `cavi_verdict` | ≤ 50 |
| `insight` | 30-100 |
| `saving_tips[]` | 每条 ≤ 50 |
| `competitors[].pros` | ≤ 60 |
| `competitors[].cons` | ≤ 60 |
| `next_cards[].sub` | ≤ 80 |

**敏感词**（西语/中文/英文）：basura / 智商税 / 辣鸡 / trash / scam 等——检测到 = 阻塞 + 告警。

### 4.6 关键指标

| 指标 | 目标 |
|------|------|
| 自动校验通过率 | > 95% |
| 跨字段一致率 | > 99.5% |
| 字段完整率 | > 99% |
| 西语 eyebrow 命中率 | 100% |
| 变量残留数 | 0 |
| **AI 字段合规率** | **100%** |
| P0 失败率 | < 0.5% |

---

## 第五部分：SKILL 调用契约

### 5.1 输入参数

**必填**：

```yaml
series_id: int           # 当前仅支持 356（Versa）
```

**可选**：

```yaml
force_update: bool       # 强制重生成（忽略缓存）
priority: enum           # high / normal / low
callback_url: url        # 异步回调
metadata: object         # 附加元数据
```

**输入示例**：

```json
{
  "series_id": 356,
  "force_update": false,
  "priority": "normal",
  "callback_url": "https://autocava.com.mx/api/v1/callbacks/md-gen",
  "metadata": { "user_id": "u_12345", "trigger": "user_request" }
}
```

### 5.2 输出

**同步模式**：

```json
{
  "status": "success",
  "md_file_path": "reports/es/nissan-versa-2026.md",
  "data_version": "v20260827-1430",
  "generated_at": "2026-08-27T14:30:00Z",
  "duration_ms": 4200,
  "validation": {
    "auto_pass": true,
    "fields_complete": "99.5%",
    "consistency_check": "pass",
    "ai_compliance_check": "pass"
  }
}
```

**partial 状态**：非关键字段缺失，MD 已生成但带兜底。

### 5.3 错误码

| Code | 含义 | 处理 |
|------|------|------|
| `E001_INVALID_INPUT` | 输入参数缺失或非法 | 修正参数重试 |
| `E002_INVALID_SERIES` | `series_id` 不在白名单 | 暂不支持其他车型 |
| `E101_DATA_FETCH_FAILED` | 数据源获取失败（3 次重试后）| 重试 1 次（60s）|
| `E102_DATA_INCOMPLETE_CRITICAL` | 关键字段缺失（价格/版本/CAVI）| 阻塞 + 报警 |
| `E103_DATA_STALE` | 数据陈旧（关键字段 2× TTL）| 提示 PM |
| `E201_VALIDATION_FAILED` | 自动校验不通过 | 不重试 |
| `E202_CROSS_FIELD_INCONSISTENT` | 跨字段不一致 | 不重试 |
| **`E203_AI_TAMPERED_DIGITS`** | **AI 润色时篡改系统数字（P0）**| **不重试 + 报警** |
| `E301_AI_GENERATION_FAILED` | AI 调用失败 | 重试 1 次（30s）|
| `E302_AI_RESPONSE_INVALID` | AI 返回非法格式 | 重试 1 次（30s）|
| `E401_TIMEOUT` | 单次生成超 60s | 重试 1 次（60s）|
| `E500_INTERNAL_ERROR` | 系统错误 | 报警 + 重试 |

### 5.4 并发与幂等

- **幂等**：同一输入 + 同一数据快照 → 同一 MD 哈希
- **并发**：同一 `series_id` 同一时刻只允许 1 个生成任务（key = `lock:md-gen:{series_id}`，TTL 300s）
- **排队**：`high` 跳过队列；`normal` FIFO 60s；`low` FIFO 无超时

### 5.5 可观测性

**每次生成产出**：
- `reports/{lang}/{model}-{year}.md`（MD 文件）
- `reports/_snapshots/{model}-{year}.{timestamp}.yaml`（数据快照）
- `logs/md-gen/{date}/{model}-{year}.log`（生成日志）

**关键指标**：P50 < 30s / P99 < 60s / 成功率 > 99% / AI 字段合规率 100% / 缓存命中率 > 70%

### 5.6 安全

- 鉴权：`Authorization: Bearer {token}`，校验失败 401
- 速率：单 token 60 次/分钟，单 IP 300 次/分钟，超限 429
- 数据隔离：跨租户严格隔离

---

## 第六部分：边界与依赖

### 6.1 我们做的 vs 不做的

**我们做**：
- MD 文档的"内容"完整且可靠
- MD 文档能 1:1 渲染出 pc-v1.html
- 字段值有源（系统给）、AI 润色（不编数）
- 跨字段一致、西语 eyebrow 字面、8 段结构

**我们不做**：
- HTML 模板的样式（CSS / 响应式）
- 前端交互（Tab / Calculator / 分享）
- 埋点 / 转化追踪
- 数据获取 API 的具体实现（后端做）
- 多车型 / 多市场 / 多语言扩展

### 6.2 文档依赖关系

```
🎤 业务宣讲（1 份）
└── docs/AUTOCAVA 购车报告需求.md  ← 业务背景 + 业务流程

📘 技术宣讲（1 份 · 本文档）
└── docs/MD-生成-总览.md  ← 一站式

🤖 AI 规范（3 份）
├── skills/cavi-guide-gen/SKILL.md  ← AI 怎么调
├── skills/cavi-guide-gen/assets/standard-template-v3.md  ← AI 填的骨架
└── skills/cavi-guide-gen/examples.md  ← 完整参考
```

### 6.3 成功标准

| 维度 | 目标 |
|------|------|
| 字段完整率 | > 99% |
| 跨字段一致率 | > 99.5% |
| 西语 eyebrow 命中率 | 100% |
| **AI 字段合规率** | **100%** |
| 效率 | 单次生成 < 60s |
| 可回放 | 同一输入 + 同一数据快照 → 同一 MD 哈希 |

### 6.4 风险

| 风险 | 缓解 |
|------|------|
| **AI 篡改系统数字**（P0）| 硬约束 + 数字溯源校验 + E203 |
| AI 生成格式不一致 | 模板约束 + 5 步校验 + 5% 抽样 |
| 数据源不稳定 | 字段级缓存 + 重试 + 兜底 |
| 数据陈旧 | 数据快照记录 fetched_at，2× TTL 触发警告 |
| LLM 润色越界 | 数字溯源 + 敏感词检测 |

---

## 第七部分：变更日志

- **v1.0**（2026-08-27）：初版
- **v1.1**（2026-08-27）：明确"单一展现样式"前提
- **v1.2**（2026-08-27）：聚焦 1 个标杆 + AI 润色不编数 + 4 个真实数据源
- **v2.0**（2026-08-27）：
  - **合并 4 份独立技术文档为一站式**：字段表（原 `MD-数据-需求清单.md`）+ 数据源（原 `MD-生成-数据源与可靠性.md`）+ 质量保证（原 `MD-生成-质量保证.md`）+ SKILL 契约（原 `MD-生成-SKILL-调用契约.md`）
  - 整体重构成"业务与原则 / 字段 / 数据 / 质量 / 契约 / 边界"6 大部分
  - 受众明确为"技术宣讲 / PM / 研发 / QA / 集成方"
  - 关联 AI 文档（3 份）保持独立，不混进宣讲

---

*相关文档：*
- *业务背景：[`docs/AUTOCAVA 购车报告需求.md`](./AUTOCAVA%20购车报告需求.md)*
- *AI 实现：[`skills/cavi-guide-gen/SKILL.md`](../skills/cavi-guide-gen/SKILL.md)*
- *AI 骨架：[`skills/cavi-guide-gen/assets/standard-template-v3.md`](../skills/cavi-guide-gen/assets/standard-template-v3.md)*
- *AI 参考：[`skills/cavi-guide-gen/examples.md`](../skills/cavi-guide-gen/examples.md)*
