# MD 生成 · 数据源与可靠性

**文档版本**：v1.2
**创建日期**：2026-08-27
**修订日期**：2026-08-27（v1.2：聚焦 4 个真实数据源，删除假设的 API 细节）
**目的**：明确 4 个真实数据源的接入原则、缓存策略、失败处理，**确保生成结果可靠**
**关联**：`docs/MD-数据-需求清单.md`（字段表）· `docs/MD-生成-质量保证.md`（验收）

---

## 一、4 个真实数据源（已确认）

PM 已确认 pc-v1.html 实际数据都来自以下 4 个**已存在**的系统：

| # | 数据源 | 提供字段 | 系统类型 |
|---|--------|---------|---------|
| 1 | **CAVI 内部 API** | 车系 / 版本（6 个）/ 规格 / MSRP / official / 月供 | 内部系统 |
| 2 | **CAVI 评分系统** | 综合分 / 推荐率 / 4 维度评分 | 内部系统 |
| 3 | **销售系统** | 月销量 / 排名 / 评论人数 | 内部系统 |
| 4 | **金融 API** | BBVA 月供 / Nissan 厂家金融 / 交换补贴 | 外部 API |

> **AI Agent 不关心 API endpoint / TTL / 鉴权**——后端负责接入，AI 端只关心"用这些数据生成 MD"。

---

## 二、4 类字段（按"AI 能否动"分类）

```
┌────────────────────────────────────────────────────┐
│  ① 系统原始数据（AI 不可动）                          │
│     价格/销量/CAVI 分/评论原文/规格/质保              │
│     AI 只能原样填入                                  │
└────────────────────────────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────┐
│  ② 系统 + AI 润色（混合字段）                         │
│     系统给原始值 + AI 润色为自然语言                │
│     AI 可以改字面意思，不许改数字                    │
└────────────────────────────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────┐
│  ③ 纯 AI 生成                                         │
│     副标题、洞察、推荐语、saving_tips               │
│     不涉数字，AI 自由生成                            │
└────────────────────────────────────────────────────┘
```

**原则**：层级越靠下，AI 自由度越大；层级越靠上，AI 不能动。

---

## 三、字段级来源映射（核心字段）

| 字段组 | 来源 | AI 能否改 |
|--------|------|----------|
| `title` | AI | 可 |
| `series_id` / `model` / `year` | 系统 | 否 |
| `market` / `lang` / `energy_type` / `body_type` | 系统 | 否 |
| `cavi_score` | 系统（CAVI 评分系统）| 否 |
| `cavi_recommend` | 系统 | 否 |
| `cavi_verdict` | 混合 | 数字不变 |
| `cavi_dimensions[].value` | 系统 | 否 |
| `cavi_dimensions[].label` | 系统 | 否 |
| `featured_reviews[].content` | 系统 | 否 |
| `spec_strip[]` | 系统 | 否 |
| `versions[]`（6 个）| 系统 | 否 |
| `versions[].is_recommended` | 系统 | 否 |
| `finance_cards.*.price` / `monthly` / `rate` | 系统（金融 API）| 否 |
| `finance_cards.trade_in.*` | 系统 | 否 |
| `annual_cost.total` / `items[].amount` | 系统 | 否 |
| `warranty.*` / `service_network.*` | 系统 | 否 |
| `competitors[].name` / `price` | 系统 | 否 |
| `competitors[].pros` / `cons` | **AI** | **是**（但不许编数字）|
| `next_cards[].label` / `sub` | 系统 | 否 |
| `hero.subtitle` | **AI** | 是 |
| `hero.desc` | **混合** | 数字不变 |
| `hero.rank_text` | **混合** | 数字不变 |
| `finance_cta_bar.title` / `subtitle` | **AI** | 是 |
| `annual_cost.saving_tips[]` | **AI** | 是（不涉数字）|
| `selling_points.insight` | **AI** | 是 |
| `key_specs` 的 4 维度洞察 | **AI** | 是 |

---

## 四、AI 不可触碰的硬约束（再强调一次）

**这是质量保证的第一道关卡**：

| 字段 | 不可改的原因 |
|------|------------|
| 价格 / MSRP / official | 错一位数 → 客户被骗 / 决策错误 |
| 月供 / 利率 | 同上 |
| 销量 / 排名 | 客观数据 |
| CAVI 评分 | 系统算法输出，不可人为"美化" |
| 评论原文 | 真实用户声音，AI 改字面 = 篡改 |
| 质保年限 / 里程 | 厂家承诺 |

**校验机制**：见 `docs/MD-生成-质量保证.md` §5.4（AI 字段合规校验）。

---

## 五、字段兜底默认值表

> **重要**：兜底 ≠ AI 编。兜底是"系统缺失时显示'暂无'"，AI 不得补"。

| 字段 | 缺失时的兜底 |
|------|------------|
| `cavi_score` | `null` + 段 04 标题改为"用户口碑（暂无评分）" |
| `cavi_recommend` | `null` + 不显示推荐率 |
| `cavi_verdict` | `"Sin datos suficientes"`（这是兜底文字，不是 AI 编的）|
| `featured_reviews` | `[]` + 段 04 隐藏评论卡片 |
| `competitors` | `[]` + 段 07 显示"暂无竞品数据" |
| `versions[].monthly` | `0` + UI 显示"咨询" |
| `finance_cards.bank` | `null` + 银行卡位置显示"联系经销商" |
| `finance_cards.factory` | `null` + 厂家卡位置显示"联系经销商" |
| `bonus_trade_in` | `0` |
| `monthly_sales` | `null` + Hero 不引用销量 |
| `warranty.years` | `3`（行业默认，**这是系统兜底不是 AI 编**）|
| `annual_cost.total` | `null` + 段 05 显示"暂无数据" |
| `hero.image_url` | `null` + 隐藏车图 |

**兜底原则**：
- **结构数据**（价格/版本/CAVI 总分）缺失 → **不生成报告**，触发告警
- **非结构数据**（描述/洞察/推荐语）缺失 → 显示"暂无"占位
- **影响购买决策**（价格/版本/CAVI）缺失 → **必须告警**

---

## 六、缓存策略（原则层面）

> **AI Agent 不直接管缓存**，但需要知道缓存原则，因为：
> - 数据快照要记录 fetched_at
> - 缓存超期要触发"陈旧"警告

### 6.1 数据新鲜度

| 等级 | 字段 | 更新频率 | 缓存 TTL |
|------|------|---------|---------|
| **实时** | 价格、库存、利率 | 每次生成时拉取 | ≤ 1h |
| **准实时** | 月销量、CAVI 评分 | 每日刷新 | ≤ 24h |
| **周级** | 质保、评论精选、服务网络 | 每周刷新 | ≤ 7d |
| **月级** | 销量排名 | 每月刷新 | ≤ 30d |
| **静态** | 模板常量、章节文案、Logo | 永久 | 永不过期 |

### 6.2 数据快照（必须保存）

每次生成 MD 时，保存"数据快照"用于回放：

```yaml
# data_snapshot.yaml
generated_at: 2026-08-27T14:30:00Z
series_id: 356
data_version: v20260827-1430
sources:
  cavi_score: { value: 4.6, source: "cavi-rating-system", fetched_at: "...", cache_ttl: 86400 }
  versions: { source: "cavi-internal-api", fetched_at: "...", count: 6 }
  finance_bank: { value: {...}, source: "finance-api", fetched_at: "..." }
# ...
```

**用途**：
- 调试："为什么这份报告价格不对？" → 看快照
- 回放：同一快照 + 同一模板 → 同一 MD 哈希
- 审计：3 个月前用户看的报告当时数据是什么？

---

## 七、失败处理

### 7.1 单字段级重试

```
读取字段 X →
  try 1 (立即)         → 失败
  try 2 (200ms 后)     → 失败
  try 3 (1000ms 后)    → 失败
  → 兜底默认值（按 §五）
```

### 7.2 整体失败

**触发**：3+ 个核心字段同时失败（车系/版本/CAVI 任一失败）

**行为**：
- 等待 30s
- 重试 1 次
- 仍失败 → **不生成报告**，触发告警

### 7.3 部分失败处理

| 失败字段数 | 处理 |
|-----------|------|
| 0 | 正常生成 |
| 1-2 个非核心 | 正常生成 + 字段标"暂无" |
| 1-2 个核心 | 正常生成 + 告警（PM 关注）|
| 3+ 个核心 | 阻塞生成 + 立即告警 |
| 价格/版本缺失 | 阻塞 + 告警（**不允许无价格报告上线**）|

---

## 八、跨字段一致性

### 8.1 一致性规则

| 规则 | 涉及字段 |
|------|---------|
| **CAVI 总分唯一** | frontmatter.cavi_score = hero.cavi_score = cavi_summary.s |
| **CAVI 推荐率唯一** | frontmatter.cavi_recommend = cavi_meta.recommend |
| **推荐版本唯一** | `versions[].is_recommended=true` 恰好 1 个 |
| **主价卡 = 推荐版本 MSRP** | `finance_cards.main.price` = `versions[recommended].msrp` |
| **4 维度数量 = 4** | `cavi_dimensions` 长度 = 4 |
| **spec_strip 数量 = 3** | 长度 = 3 |
| **competitors 数量 = 4** | 长度 = 4 |
| **next_cards 数量 = 3** | 长度 = 3 |
| **竞品首条是本车** | `competitors[0].tag = "SELF"` |
| **价格非负** | 所有 price/amount ≥ 0 |
| **AI 字段不篡改数字** | `competitors[].pros/cons` 中出现的数字必须能在系统源数据中找到 |

### 8.2 校验时机

| 时机 | 校验内容 |
|------|---------|
| **数据获取后**（生成前）| 必填字段、字段类型、字段范围 |
| **MD 渲染后**（生成后）| 西语 eyebrow 字面、跨字段一致性、变量替换完整 |
| **人工审核时** | 抽样校验（5% 报告）|

---

## 九、可观测性

### 9.1 每次生成产出

- **MD 文件**：`reports/es/nissan-versa-2026.md`
- **数据快照**：`reports/_snapshots/nissan-versa-2026.{timestamp}.yaml`
- **生成日志**：`logs/md-gen/{date}/nissan-versa-2026.log`

### 9.2 关键指标

| 指标 | 目标 |
|------|------|
| 生成成功率 | > 99% |
| 字段完整率 | > 99% |
| 兜底率 | < 5% |
| 跨字段一致率 | > 99.5% |
| 单次生成耗时 | < 60s |
| 数据新鲜度 | > 95% |
| **AI 字段合规率** | **100%**（AI 不许篡改数字）|

### 9.3 告警

| 告警 | 触发 |
|------|------|
| 价格缺失 | `versions[].msrp` 任一缺失 |
| CAVI 评分缺失 | `cavi_score` 缺失 |
| AI 字段篡改 | 校验发现 AI 润色改了数字 |
| 数据陈旧 | 关键字段超 2× TTL |
| 生成失败 | 3+ 核心字段失败 |

---

## 十、相关文档

- *总览：`docs/MD-生成-总览.md`*
- *字段权威表：`docs/MD-数据-需求清单.md`*
- *质量验收：`docs/MD-生成-质量保证.md`*
- *AI 骨架：`skills/cavi-guide-gen/assets/standard-template-v3.md`*
